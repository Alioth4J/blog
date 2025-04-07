---
title: Upgrading from Spring MVC to Netty for Performance Improvements
description: The road Corneast walked
date: 2025-04-07
slug: springmvc2netty
image: 
categories:
    - Java
---

## Background
Recently I'm developing [Corneast](https://github.com/alioth4j/corneast), a distributed token middleware aiming to provide excellent performance for high concurrency scenario. I'm thinking ways to improve its performance, as a result, I upgraded Spring MVC to Netty for higher throughput.  

Spring MVC is built on top of Servlet containers which may face limitations in high-concurrency situations. Netty is an asynchronous event-driven network application framework offering lightwight and high erformance, which is what Corneast need. Thus, a upgrade is conducted.  

This blog records the road Corneast has walked from Spring MVC to Netty.  

## Steps
### Import Dependency
It's easy when using Spring Boot.  

```xml
<dependency>
	<groupId>io.netty</groupId>
	<artifactId>netty-all</artifactId>
</dependency>
```

### Setup Netty Server
Determine the port:  
```yml
netty:
  server:
    port: 8088
```

I use a single thread as the boss group to accept new connections, and set the worker group thread count as 4 times of CPU cores to violently handle requests. Here I don't care the thread context switching; instead, aiming to utilize the CPU more fully.  

```java
bossGroup = new NioEventLoopGroup(1);
workerGroup = new NioEventLoopGroup(80); // 4 * (CPU cores)
```

Some tuning of Netty:  
```java
bootstrap.group(bossGroup, workerGroup)
.channel(NioServerSocketChannel.class)
.option(ChannelOption.SO_BACKLOG, 1024)
.option(ChannelOption.TCP_NODELAY, true)
.option(ChannelOption.SO_REUSEADDR, true)
.handler(new LoggingHandler(LogLevel.INFO))
.childOption(ChannelOption.SO_KEEPALIVE, false)
.childOption(ChannelOption.ALLOCATOR, new PooledByteBufAllocator(true, 40, 40, 8192, 11))
.childOption(ChannelOption.RCVBUF_ALLOCATOR, new AdaptiveRecvByteBufAllocator(64, 8192, 65535))
```

Here comes the pipeline.  

I use protobuf instead of json for more performance, so the ProtobufDecoder and ProtobufEncoder are necessary.  

How to route the request? When using Spring MVC, the framework did all the things for us. Now Netty has to route the requests through a type property of the request to the corresponding handler itself.  

```java
.childHandler(new ChannelInitializer<SocketChannel>() {
@Override
protected void initChannel(SocketChannel ch) {
    // protobuf handler
    ch.pipeline().addLast(new ProtobufDecoder(RequestProto.RequestDTO.getDefaultInstance()));
    ch.pipeline().addLast(new ProtobufEncoder());

    // route handler
    ch.pipeline().addLast(requestRouteHandler);
}
});
```

### Strategy Pattern
There are three primitives in Corneast: register, reduce and query. Correspondingly, I need three handlers.  

Here I choose Strategy Pattern to implement the logic. The `RequestRouteHandler` holds all the strategies and choose corresponding one to handle the request.  

A trick to get all the strategies is using injection of the Spring container.  
The key of the injected `Map` is the bean name and the value is the concrete strategy.  

We should make sure the bean name and the request type are the same.  

```java
@Component
@ChannelHandler.Sharable
public class RequestRouteHandler extends SimpleChannelInboundHandler<RequestProto.RequestDTO> {

    @Autowired
    private Map<String/* beanName */, RequestHandlingStrategy/* object of strategy */> requestHandlingStrategyMap;

    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, RequestProto.RequestDTO requestDTO) throws Exception {
        // choose the strategy
        String requestType = requestDTO.getType();
        RequestHandlingStrategy requestHandlingStrategy = requestHandlingStrategyMap.get(requestType);
        // handle
        CompletableFuture<ResponseProto.ResponseDTO> responseCompletableFuture = requestHandlingStrategy.handle(requestDTO);
        // write and flush the response
        responseCompletableFuture.whenComplete((responseDTO, t) -> {
            channelHandlerContext.writeAndFlush(responseDTO);
        });
    }

}
```

Strategy interface:  
```java
public interface RequestHandlingStrategy {

    CompletableFuture<ResponseProto.ResponseDTO> handle(RequestProto.RequestDTO requestDTO);

}
```

One concrete implementation:  
```java
@Component("reduce")
public class ReduceRequestHandlingStrategy implements RequestHandlingStrategy {

    @Autowired
    private List<RedissonClient> redissonClients;

    private int nodeSize;

    @PostConstruct
    public void init() {
        this.nodeSize = redissonClients.size();
    }

    private static final Map<String, ResponseProto.ResponseDTO> cachedSuccessResponses = new ConcurrentHashMap<>();
    private static final Map<String, ResponseProto.ResponseDTO> cachedFailResponses = new ConcurrentHashMap<>();

    private static final Random random = new Random();

    // reused objects
    private static final ResponseProto.ResponseDTO.Builder responseBuilder = ResponseProto.ResponseDTO.newBuilder().setType("reduce");
    private static final ResponseProto.ReduceRespDTO.Builder successReduceRespBuilder = ResponseProto.ReduceRespDTO.newBuilder().setSuccess(true);
    private static final ResponseProto.ReduceRespDTO.Builder failReduceRespBuilder = ResponseProto.ReduceRespDTO.newBuilder().setSuccess(false);

    private static final String luaScript = """
                                            local n = tonumber(redis.call('GET', KEYS[1]) or "0")
                                            if n > 0 then
                                                redis.call('DECR', KEYS[1])
                                                return 1
                                            else 
                                                return 0
                                            end
                                            """;

    @Override
    @Async("reduceExecutor")
    public CompletableFuture<ResponseProto.ResponseDTO> handle(RequestProto.RequestDTO requestDTO) {
        return CompletableFuture.supplyAsync(() -> {
            String key = requestDTO.getReduceReqDTO().getKey();
            // pick a redissonClient randomly
            RedissonClient redissonClient = redissonClients.get(random.nextInt(nodeSize));
            long result = redissonClient.getScript().eval(RScript.Mode.READ_WRITE, luaScript, RScript.ReturnType.INTEGER, List.of(key));
            if (result == 1) {
                if (!cachedSuccessResponses.containsKey(key)) {
                    cachedSuccessResponses.put(key, responseBuilder
                                                    .setReduceRespDTO(successReduceRespBuilder
                                                                      .setKey(requestDTO.getReduceReqDTO().getKey())
                                                                      .build())
                                                    .build());
                }
                return cachedSuccessResponses.get(key);
            } else {
                if (!cachedFailResponses.containsKey(key)) {
                    cachedFailResponses.put(key, responseBuilder
                                                 .setReduceRespDTO(failReduceRespBuilder
                                                                   .setKey(requestDTO.getReduceReqDTO().getKey())
                                                                   .build())
                                                 .build());
                }
                return cachedFailResponses.get(key);
            }
        });
    }

}
```

As you can see, some objects are reused to save the spending of creating new objects.  

Maybe more optimizations will be conducted here in the future.  

### Tests
For the convenience of tests, I wrote a util class to generate request binary files.  

It will run when the application starts.  

```java
@Component
public class ProtobufRequestGenerator implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        // params
        String key = DateTimeFormatter.ofPattern("yyyyMMdd").format(LocalDate.now());
        Long tokenCount = 1000L;

        // create dir
        File dir = new File("corneast-test", "request");
        if (!dir.exists()) {
            boolean success = dir.mkdirs();
            if (!success) {
                System.err.println("Unable to mkdirs: " + dir.getCanonicalPath());
                return;
            }
        }

        // register request
        RequestProto.RequestDTO registerRequestDTO = RequestProto.RequestDTO.newBuilder()
                .setType("register")
                .setRegisterReqDTO(RequestProto.RegisterReqDTO.newBuilder().setKey(key).setTokenCount(1000).build())
                .build();
        byte[] registerReqByteArray = registerRequestDTO.toByteArray();
        File registerReqFile = new File(dir, "register.bin");
        try (FileOutputStream fos = new FileOutputStream(registerReqFile)) {
            fos.write(registerReqByteArray);
        } catch (IOException e) {
            handleIOException(e);
        }

        // reduce request
        RequestProto.RequestDTO reduceRequestDTO = RequestProto.RequestDTO.newBuilder()
                .setType("reduce")
                .setReduceReqDTO(RequestProto.ReduceReqDTO.newBuilder().setKey(key).build())
                .build();
        byte[] reduceReqByteArray = reduceRequestDTO.toByteArray();
        File reduceReqFile = new File(dir, "reduce.bin");
        try (FileOutputStream fos = new FileOutputStream(reduceReqFile)) {
            fos.write(reduceReqByteArray);
        } catch (IOException e) {
            handleIOException(e);
        }

        // query request
        RequestProto.RequestDTO queryRequestDTO = RequestProto.RequestDTO.newBuilder()
                .setType("query")
                .setQueryReqDTO(RequestProto.QueryReqDTO.newBuilder().setKey(key))
                .build();
        byte[] queryReqByteArray = queryRequestDTO.toByteArray();
        File queryReqFile = new File(dir, "query.bin");
        try (FileOutputStream fos = new FileOutputStream(queryReqFile)) {
            fos.write(queryReqByteArray);
        } catch (IOException e) {
            handleIOException(e);
        }
    }

    private static void handleIOException(IOException e) {
        System.err.println("Something went wrong when writing request to the file: " + e.getMessage());
        e.printStackTrace();
    }

}
```

I use JSR223 Sampler in JMeter to test the performance.  
```groovy
import java.net.Socket
import org.apache.commons.io.FileUtils

String host = "127.0.0.1"
int port = 8088

// Change this!
// Place the binary file into a jmeter-readable place.
String filePath = "/var/home/alioth4j/Documents/reduce.bin"
File requestFile = new File(filePath)

if (!requestFile.exists()) {
    throw new IllegalStateException("File does not exist: " + requestFile)
}

byte[] requestData = FileUtils.readFileToByteArray(requestFile)

Socket socket = null
try {
    socket = new Socket(host, port)

    socket.withCloseable { sock ->
        def out = sock.getOutputStream()
        out.write(requestData)
        out.flush()

        def in = sock.getInputStream()
        byte[] buffer = new byte[1024]
        int bytesRead = in.read(buffer)
        if (bytesRead > 0) {
            byte[] responseData = buffer[0..<bytesRead]
            String responseHex = responseData.encodeHex().toString()

            vars.put("responseHex", responseHex)
        }
    }
} catch (Exception e) {
    throw e
}
```

## Conclusion
After the upgrade, Spring Framework serves as an IoC container meanwhile Netty routes and handles the requests.  

Done.  


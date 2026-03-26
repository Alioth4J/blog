---
title: Referece Counting in Netty 
description: 
date: 2026-03-26
slug: netty-refcnt
image: 
categories:
    - Java
---

## `ReferenceCounted`
In Netty, `ReferenceCounted` is implemented under reference counting mechanism.  

When a `ReferenceCounted` object is created, the reference count starts with `1`. Then each time this object is passed to another component, `retain()` should be called and after finishing using it, `release()` should be called. When the reference count gets to `0`, it is dead and the memory will be reclaimed.  

```java
public interface ReferenceCounted {

    int refCnt();

    ReferenceCounted retain();
    ReferenceCounted retain(int increment);

    ReferenceCounted touch();
    ReferenceCounted touch(Object hint);

    boolean release();
    boolean release(int decrement);
}
```

## Implementation Class
`ByteBuf` is the most common implementation of `ReferenceCounted`.  

`ByteBuf` is a high-performance byte container (like `java.nio.ByteBuffer`). It can be assigned in direct memory. For example:  

```java
.childOption(ChannelOption.ALLOCATOR,
             new PooledByteBufAllocator(boolean preferDirect, int nHeapArena, int nDirectArena, int pageSize, int maxOrder))
```

Direct buffers can't be controlled by Java GCs, so reference counting is used to manage their lifecycles. Java programmers have something more to do here.  

## Channel Handler
Extending `ChannelInboundHandlerAdapter` is a common way to develop a handler. This class just does the basic things, such as `ctx.fireXxx()`. We invoke `ReferenceCounted#retain` and `ReferenceCounted#release` ourselves when the `msg` is a `ReferenceCounted` object.  

```java
public class ChannelInboundHandlerAdapter extends ChannelHandlerAdapter implements ChannelInboundHandler {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ctx.fireChannelRead(msg);
    }
    
    // ...
    
}
```

`SimpleChannelInboundHandler` is a convenient class and it does more things for users. There is a template method pattern. Users usually override `channelRead0()`. But pay attention that it will call `release()` method for you.  

```java
public abstract class SimpleChannelInboundHandler<I> extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        boolean release = true;
        try {
            if (acceptInboundMessage(msg)) {
                @SuppressWarnings("unchecked")
                I imsg = (I) msg;
                channelRead0(ctx, imsg);
            } else {
                release = false;
                ctx.fireChannelRead(msg);
            }
        } finally {
            if (autoRelease && release) {
                ReferenceCountUtil.release(msg); // release here
            }
        }
    }
    
    protected abstract void channelRead0(ChannelHandlerContext ctx, I msg) throws Exception;

    // ...

}
```

This method releases for `ReferenceCounted` objects. If `msg` is not a `ReferenceCounted` object, it does nothing.

```java
public final class ReferenceCountUtil {

    public static boolean release(Object msg) {
        if (msg instanceof ReferenceCounted) {
            return ((ReferenceCounted) msg).release();
        }
        return false;
    }
    
    // ...

}
```

## Summary
At first I have two misconceptions of Netty's reference counting mechanism:  
1. No need to call `release()` in `SimpleChannelInboundHandler`? -- Someone else does it for you.  
2. `release()` is called for every inbound message? -- No, there is type check.  

After grasping these points I make my mind clearer when using all kinds of components of Netty together.  

This is a less-seen scenario where Java users need to apply reference counting themselves.


---
title: Diving into Tomcat's Startup
description: From Bootstrap to Catalina
date: 2025-03-15
slug: tomcat-startup
image: 
categories:
    - Java
    - Tomcat 
---

## Bootstrap
In Tomcat, `org.apache.catalina.startup.Bootstrap` mainly handles the startup process.  

### Static Block
Before `main` method being invoked, the static block is invoked first.  

The responsibility of the static block is setting `File catalinaHomeFile` and `File catalinaBaseFile`.  

#### Set `catalinaHomeFile`.  
1. Try getting from system property.  
```java
String home = System.getProperty(Constants.CATALINA_HOME_PROP);
```

2. First fallback: checking whether the current dir is `bin` directory.
```java
File bootstrapJar = new File(userDir, "bootstrap.jar");
if (bootstrapJar.exists()) {
    File f = new File(userDir, "..");
    // ...
```

3. Second fallback: use current directory.  
```java
File f = new File(userDir);
```

4. Set to system property.  
```java
System.setProperty(Constants.CATALINA_HOME_PROP, catalinaHomeFile.getPath());
```

#### Set `catalinaBaseFile`.  
1. Try getting from system property.  
```java
String base = System.getProperty(Constants.CATALINA_BASE_PROP);
```

2. Fallback: use `catalinaHomeFile`.  
```java
catalinaBaseFile = catalinaHomeFile;
```

3. Set to system property.  
```java
System.setProperty(Constants.CATALINA_BASE_PROP, catalinaBaseFile.getPath());
```

Note that retrieving the canonical path is necessary.  
```java
try {
    baseFile = baseFile.getCanonicalFile();
} catch (IOException ioe) {
    baseFile = baseFile.getAbsoluteFile();
}
```

### `main` Method
#### Initialization
##### Lock
```java
synchronized (daemonLock) {
```

##### Create `Bootstrap` Instance
```java
Bootstrap bootstrap = new Bootstrap();
```

##### Invoke `bootstrap.init()`

`init()`  
\  
 -> `initClassLoaders()`  
 |\  
 | -> create `commonLoader`, `catalinaLoader` and `sharedLoader` by `createClassLoader()` method  
 |  \  
 |   -> `ClassLoaderFactory.createClassLoader(repositories, parent);`  
 |  
 -> use `catalinaLoader` to load `Catalina` class  
 |  
 -> use reflection to set `sharedLoader` as `catalina`'s parent class loader  
 |  
 -> set `catalinaDaemon`  

Highlights of reflection:  
```java
String methodName = "setParentClassLoader";
Class<?> paramTypes[] = new Class[1];
paramTypes[0] = Class.forName("java.lang.ClassLoader");
Object paramValues[] = new Object[1];
paramValues[0] = sharedLoader;
Method method = startupInstance.getClass().getMethod(methodName, paramTypes);
method.invoke(startupInstance, paramValues);
```

##### Set the `bootstrap` Instance as the `daemon`
Currently the initialization of `bootstrap` has completed.  
```java
daemon = bootstrap;
```

#### Command Handling
Use the last args as the command, default `start`.  

Do the corresponding thing of the command.  
```java
String command = "start";
if (args.length > 0) {
    command = args[args.length - 1];
}

if (command.equals("startd")) {
    args[args.length - 1] = "start";
    daemon.load(args);
    daemon.start();
} else if (command.equals("stopd")) {
    args[args.length - 1] = "stop";
    daemon.stop();
} else if (command.equals("start")) {
    daemon.setAwait(true);
    daemon.load(args);
    daemon.start();
    if (null == daemon.getServer()) {
        System.exit(1);
    }
} else if (command.equals("stop")) {
    daemon.stopServer(args);
} else if (command.equals("configtest")) {
    daemon.load(args);
    if (null == daemon.getServer()) {
        System.exit(1);
    }
    System.exit(0);
} else {
    log.warn("Bootstrap: command \"" + command + "\" does not exist.");
}
```

We can see we actually do:  
```java
    daemon.setAwait(true);
    daemon.load(args);
    daemon.start();
    if (null == daemon.getServer()) {
        System.exit(1);
    }
```

Note that `daemon` is an object of `Bootstrap`.  

Note that bootrap invokes Catalina's same-name methods through reflection.  

#### `setAwait(boolean await)` Method
Use reflection to invoke `catalinaDaemon`'s `setAwait` method.  
```java
Class<?> paramTypes[] = new Class[1];
paramTypes[0] = Boolean.TYPE;
Object paramValues[] = new Object[1];
paramValues[0] = Boolean.valueOf(await);
Method method = catalinaDaemon.getClass().getMethod("setAwait", paramTypes);
method.invoke(catalinaDaemon, paramValues);
```

#### `load(args)` Method
Use reflection to invoke `catalinaDaemon`'s `invoke` method.  
```java
String methodName = "load";
Object param[];
Class<?> paramTypes[];
if (arguments == null || arguments.length == 0) {
    paramTypes = null;
    param = null;
} else {
    paramTypes = new Class[1];
    paramTypes[0] = arguments.getClass();
    param = new Object[1];
    param[0] = arguments;
}
Method method = catalinaDaemon.getClass().getMethod(methodName, paramTypes);
if (log.isTraceEnabled()) {
    log.trace("Calling startup class " + method);
}
method.invoke(catalinaDaemon, param);
```

#### `start()` Method
Use reflecton to invoke `catalinaDaemon`'s `start` method
```java
Method method = catalinaDaemon.getClass().getMethod("start", (Class[]) null);
method.invoke(catalinaDaemon, (Object[]) null);
```

## Catalina
I will explain src in the sequence of invocation by `Bootstrap`.  

### `setAwait(boolean b)` Method
Just set property `await`.  
```java
public void setAwait(boolean b) {
    await = b;
}
```

### `load(String[] args)` Method
1. process arguments  
2. invoke `load()` method  

#### Process Arguments

Traverse all the arguments, use several `boolean` variables as flags of command options. When a argument is a "-option", the corresponding variable will be set. In the next loop, it can receive the argument of this option.  

The following code snippet is part of the source.  
```java
boolean isConfig = false;
for (String arg : args) {
    if (isConfig) {
        configFile = arg;
        isConfig = false;
    } else if (arg.equals("-config")) {
       isConfig = true;
    }
}
```

#### Invoke `load()` Method

load()  
 \  
  -> if (loaded) return;  
  |  
  -> initNaming();
  |  
  -> parseServerXml(true);  
  |  
  -> getServer() and set  
  |  
  -> initStreams();  
  |  
  -> getServer().init();  

##### `initNaming()`
If JDNI is enabled, it sets the relevant system properties to specify the JNDI packages and the context factory class, ensuring that the correct naming service implementation is loaded during subsequent JNDI operations.  
```java
protected void initNaming() {
    // Setting additional variables
    if (!useNaming) {
        log.info(sm.getString("catalina.noNaming"));
        System.setProperty("catalina.useNaming", "false");
    } else {
        System.setProperty("catalina.useNaming", "true");
        String value = "org.apache.naming";
        String oldValue = System.getProperty(javax.naming.Context.URL_PKG_PREFIXES);
        if (oldValue != null) {
            value = value + ":" + oldValue;
        }
        System.setProperty(javax.naming.Context.URL_PKG_PREFIXES, value);
        if (log.isDebugEnabled()) {
            log.debug(sm.getString("catalina.namingPrefix", value));
        }
        value = System.getProperty(javax.naming.Context.INITIAL_CONTEXT_FACTORY);
        if (value == null) {
            System.setProperty(javax.naming.Context.INITIAL_CONTEXT_FACTORY,
                    "org.apache.naming.java.javaURLContextFactory");
        } else {
            log.debug(sm.getString("catalina.initialContextFactory", value));
        }
    }
}
```

##### `parseServerXml(true)`
The parameter of this method is used to identify start and stop. So this time is `true`.  
  
First of all, set configuration source as `conf/server.xml`.    
```java
ConfigFileLoader.setSource(new CatalinaBaseConfigurationSource(Bootstrap.getCatalinaBaseFile(), getConfigFile()));
File file = configFile();
```

There are two ways to get configuration from `server.xml`:  
1. use `Digester` to parse xml file  
2. use generated code  

At first we don't have any generated code.  

##### Use `Digester` to Parse Xml File
1. open file source
2. create digester
3. get input stream
4. parse xml file
5. generate code if needed
```java
try (ConfigurationSource.Resource resource = ConfigFileLoader.getSource().getServerXml()) {
    Digester digester = start ? createStartDigester() : createStopDigester();
    InputStream inputStream = resource.getInputStream();
    InputSource inputSource = new InputSource(resource.getURI().toURL().toString());
    inputSource.setByteStream(inputStream);
    digester.push(this);
    if (generateCode) {
        digester.startGeneratingCode();
        generateClassHeader(digester, start);
    }
    digester.parse(inputSource);
    if (generateCode) {
        generateClassFooter(digester);
        try (FileWriter writer = new FileWriter(
                new File(serverXmlLocation, start ? "ServerXml.java" : "ServerXmlStop.java"))) {
            writer.write(digester.getGeneratedCode().toString());
        }
        digester.endGeneratingCode();
        Digester.addGeneratedClass(xmlClassName);
    }
} catch (Exception e) {
    log.warn(sm.getString("catalina.configFail", file.getAbsolutePath()), e);
    if (file.exists() && !file.canRead()) {
        log.warn(sm.getString("catalina.incorrectPermissions"));
    }
}

```

`createStartDigester()` method:  

Instantiate and initialize.  

Many rules are added to `Digester` to configure how an XML node is converted to a Java object and how relationships between objects are established.  

For instance:  
- `digester.addObjectCreate()`: create the object.
- `digester.addSetProperties()`: inject xml attributes to java object.
- `digester.addSetNext()`: invoke parent's method to set this object.
```java
Digester digester = new Digester();
digester.setValidating(false);
// ...
digester.addObjectCreate("Server", "org.apache.catalina.core.StandardServer", "className");
digester.addSetProperties("Server");
digester.addSetNext("Server", "setServer", "org.apache.catalina.Server");
// ...
```

Core parse:  
```java
digester.parse(inputSource);
```

Generate code:  
Generate before and after `digester.parse()`.  

No tricks, just use `StringBuilder` to append strings. For example:   
```java
protected void generateClassHeader(Digester digester, boolean start) {
    StringBuilder code = digester.getGeneratedCode();
    code.append("package ").append(generatedCodePackage).append(';').append(System.lineSeparator());
    code.append("public class ServerXml");
    if (!start) {
        code.append("Stop");
    }
    code.append(" implements ");
    code.append(ServerXml.class.getName().replace('$', '.')).append(" {").append(System.lineSeparator());
    code.append("public void load(").append(Catalina.class.getName());
    code.append(' ').append(digester.toVariableName(this)).append(") throws Exception {")
        .append(System.lineSeparator());
}
```

###### Use Generated Code
If we already have generate code, we do not need to parse `server.xml` again. We can make use of generated code.  

Use `Catalina`'s class loader to load `GeneratedCodeLoader`'s class and use refletion to create an instance. Then set it to Digester.  
```java
if (useGeneratedCode && !Digester.isGeneratedCodeLoaderSet()) {
    String loaderClassName = generatedCodePackage + ".DigesterGeneratedCodeLoader";
    try {
        Digester.GeneratedCodeLoader loader = (Digester.GeneratedCodeLoader) Catalina.class.getClassLoader()
                .loadClass(loaderClassName)
                .getDeclaredConstructor()
                .newInstance();
        Digester.setGeneratedCodeLoader(loader);
```

Then get the generated file location.  
```java
File serverXmlLocation = null;
String xmlClassName = null;
if (generateCode || useGeneratedCode) {
    xmlClassName = start ? generatedCodePackage + ".ServerXml" : generatedCodePackage + ".ServerXmlStop";
}
if (generateCode) {
    if (generatedCodeLocationParameter != null) {
        generatedCodeLocation = new File(generatedCodeLocationParameter);
        if (!generatedCodeLocation.isAbsolute()) {
            generatedCodeLocation = new File(Bootstrap.getCatalinaHomeFile(), generatedCodeLocationParameter);
        }
    } else if (generatedCodeLocation == null) {
        generatedCodeLocation = new File(Bootstrap.getCatalinaHomeFile(), "work");
    }
    serverXmlLocation = new File(generatedCodeLocation, generatedCodePackage);
    if (!serverXmlLocation.isDirectory() && !serverXmlLocation.mkdirs()) {
        log.warn(sm.getString("catalina.generatedCodeLocationError", generatedCodeLocation.getAbsolutePath()));
        // Disable code generation
        generateCode = false;
    }
}
```

Use `ServerXml` to load.  
```java
ServerXml serverXml = null;
if (useGeneratedCode) {
    serverXml = (ServerXml) Digester.loadGeneratedClass(xmlClassName);
}

if (serverXml != null) {
    try {
        serverXml.load(this);
    } catch (Exception e) {
        log.warn(sm.getString("catalina.configFail", "GeneratedCode"), e);
    }
}
```

##### `getServer()` and Set
Currently the `server.xml` has been parsed and a `Server` object has been created and set as `catalina`'s property.  
```java
Server s = getServer();
if (s == null) {
    return;
}

getServer().setCatalina(this);
getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());
```

##### `initStreams()`
Use custom streams.  
```java
protected void initStreams() {
    System.setOut(new SystemLogHandler(System.out));
    System.setErr(new SystemLogHandler(System.err));
}
```

##### `getServer().init()`
Here we invoke `Server#init()` method! The embeded components will be initiated sequentially.  


---
title: Use Reflection Like the Builder Pattern
description: com.alioth4j.reflect
date: 2025-01-29
slug: reflect
image: 
categories:
    - Java
    - Design Patterns
---

## Conceptions
Traditional reflection can often lead to verbose code. For instance, `setAccessible(true)` is a kind of regular code snippet. In `com.alioth4j.reflect` library, I **encapsulate the regular usage of reflection API** to simplify the processes of object creation and method invocation.  
Meanwhile, drawing inspiration from the Builder pattern, I reimagine the way to use reflection: **method chaining**. This approach enchances code readability and mainainability.  

## Step-by-Step Usage
### Step 1: Instantiate
```java
new Reflect<MyClass>()
    .clazz(MyClass.class)
    .construct(Class<?>[] parameterTypes, Object[] parameterValues)
```
or
```java
new Reflect<MyClass>(MyClass.class, Class<?>[] constructorParameterTypes, Object[] constructorParameterValues)
```
or
```java
new Reflect<MyClass>(existingObject)
```
or
```java
new Reflect<MyClass>()
    .exist(existingObject)
```

### Step 2: Invoke methods
```java
.set(String fieldName, Object fieldValue) | .set(Map<String/* fieldName */, Object/* fieldValue */> setterMap)
.invoke(String methodName, Class<?>[] parameterTypes, Object[] parameterValues)
```

### Step 3: Get the instance
```java
.reflect()
```

## An Usage Example
```java
MyClass myInstance = new Reflect<MyClass>()
                .clazz(MyClass.class)
                .construct(Class<?>[] parameterTypes, Object[] parameterValues)
                .set(String fieldName, Object fieldValue)
                .invoke(String methodName, Class<?>[] parameterTypes, Object[] parameterValues)
                .reflect();
```

## Implementation
### `public class Reflect<I>`
#### Generics
Use `<I>` to improve type safety.  

#### Fields
Two fields:
```java
private Class<? extends I> clazz;
private I instance;
```

#### Constructors
Constructor is (part of) the instantiation step. Just fill in the fields.  

#### Methods
- Encapsulate the reflection API inside.
- Return `this` object to support method chaining.

### `public class ReflectException`
- Thrown when an exception occurs during the usage of `com.alioth4j.reflect.Reflect`
- It is an encapsulation of the reflection-related exceptions, providing an unified way to handle exceptions
- As reflection is an unsafe mechanism itself, the safety should be guaranteed by the user

## Conclusion
With the elegance of the Builder pattern, this library offers developers a simpler and cleaner way to leverage reflection in Java applications.  

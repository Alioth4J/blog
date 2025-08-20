---
title: Tessy or Messy?
description: Practical troubleshooting and operations
date: 2025-08-20
slug: tessy
image: 
categories:
    - Test
---

Nothing to say, just go straight into it.  

## Environment
OS: Windows 10  
Tessy version: 4.3  

## Fault Tolerence
> **The golden rule of atomicity: Never modify the only copy!**

It is significant to make a copy of the whole Tessy project and only modify one of them to keep everything in control, else the project may break and be unable to fix.  

The paths of the `includes` will be shown as absolute paths when you copy and paste them, so make your fault tolerance directory a sibling of the original. In this way it's easy to write a script to change the paths.  

The same, when editing a test module, I suggest copy and paste it first.  

## Overall Process
0. Create Test Modules
1. Overview: Add `.c` files; Add `.h` files; Add micros; Add test cases
2. TID: Create stubs; Set `In/Out/Irrelevant`
3. TDE: Implement stubs; Fill in test cases
4. Execute the tests

## Troubles
### Lack of Something?
Make sure you have the access to all the git submodules.  

### Header files
1. Do not add something you don't need as it may cause some errors.  
2. The order of header files makes sense.  
3. Good luck.

### `printf`
Do not create stubs for `printf`.  

### error: unknown type name '<type_name>'; did you mean '<irrelevant_type_name>'
Lack of something?  

This may also because Tessy does not recognize `typedef`.  

You need to define this type as micro.  

### error: undefined reference to Enum
Add enum type as `int` and enum objects as numbers.  

### Compilation Error in Tessy-generated Files
Look into that file, add the lacking one.  

### error: typedef redefinition with different types '<type1>' vs '<type2>'
Pay attention to the order of included headers, which make sense.  

My trick is placing the cross-compilation tools in the first place.  

### Different Stub Code for Different Test Case
Create specific stub code for certain "test step".  

### Multi Occurrence of Single Function
```c
static int i = 0;

if (i == 0) {
    // do something...
    i++;
    return <something>;
} else if (i == 1) {
    // do something...
    i++;
    return <something>;
} else {
    // do something...
    return <something>;
}
```

### Functional Pointer
In "Declaration/Definition"  

In "Declaration":  
```c
extern <return_type> function_name(arg_type args);
```

In "Definitions":  
```c
<return_type> function_name(arg_type args) {
    // do something...
    return <something>;
}
```

Fill the unit of the test case with function name.  

### Need a Struct as Return Value
The return type is the pointer to a struct. Later some fields of this struct will be used.  

In "Definitions":  
```c
extern struct <struct_type> <struct_name> = { .<field_name> = <value> };
```

Then use it in the stub code.  

Other approaches, such as `malloc`, will fail.  

### Need a Fake Address
```c
static int fake;
return (Type*)&fake;
```

### Unable to Run Test Cases Suddenly
You did nothing, but the test cases became grey. It may because Tessy 'forgets' something. That sucks.  

Check whether the stubs or stub code or global variables still exist.  

### INTERFACE CHANGED: This test object's interface has changed. You need to perform a reuse in the IDA perspective.
In "IDA", commit.  

Test cases may be lost.  

### REMOVED: This test object has been removed (or renamed). You may assign the test datato another existing object and perform a reuse in the IDA perspective.
Remove non-existing functions before changing source files.  

### Stubbing for Side effects
For some cases, you need to stub for side effects.  

## About Unit Test
You may fall into the trap of pursuing high coverage rate and pretty reports when using Tessy.  

Do not write tests for writing tests.  

You are supposed to write test cases according to software business instead of existing code.  

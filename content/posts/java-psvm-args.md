---
author:
  name: "Alioth4J"
date: 2024-04-10
linktitle: Java 的 public static void main(String[] args) 是什么
title: Java 的 public static void main(String[] args) 是什么
type:
- post
- posts
weight: 10
series:
- Java
aliases:
- /blog/java-psvm-args/
---

### 简介
public static void main(String[] args) 缩写为 psvm，是 Java 程序执行的起始位置
### public - 公有
访问修饰符，表明该方法为公有，可以被其他任何类访问
### static - 静态
静态方法属于类，而不属于某一个对象
### void - 返回值类型为void
这是方法的返回值类型（无需返回值）
### main - 方法名
方法名规定为 main，注意小写开头
### (String[] args) - 参数
String[] 是参数类型，为字符串数组
args 是参数名
### 该方法的参数来自哪里？
来自**命令行**
#### 使用命令行运行 Java 程序的基本步骤：
1. 在文本编辑器中编写代码，保存为 .java  文件
2. javac 命令编译代码，例如 `javac ClassName.java`
3. java 命令运行 Java 程序并传递命令行参数，例如 `java ClassName arg0 arg1 arg2`

在第3点中，arg0、arg1、arg2 就是命令行参数，传入 args 这个字符串数组


---
author:
  name: "Alioth4J"
date: 2024-04-15
linktitle: AOP 简要介绍
title: AOP 简要介绍
type:
- post
- posts
weight: 10
series:
- Java
aliases:
- /blog/aop-introduction/
---

## 概念
Aspect Oriented Programming 面向切面编程
## 目的
将 **横切关注点** 从 **核心业务逻辑** 中分离出来
## 术语
#### 横切关注点 Cross-cutting concerns
多个类或对象中的公共行为
#### 切面 Aspect
封装横切关注点的类
#### 连接点 JoinPoint
方法调用的某个特定时刻
#### 切点 Pointcut
用于匹配连接点的表达式
#### 通知 Advice
在连接点要执行的操作
#### 织入 Weaving
连接切面和目标对象的过程，也就是将通知应用到连接点的过程
## 应用场景
- 日志记录
- 事务管理
- 权限控制
- ...
## 实现方式
- 动态代理
- 字节码操作
---
title: 面向对象设计（业务篇）
description: public interface ObjectOrientedDesign4Biz
date: 2024-10-17
slug: oo-design-biz
image: 
categories:
    - Java
    - OO
---

## 需求分析
- 业务无非 CRUD
- 设计一个业务时，思考与其他业务的关联

## 架构设计
### 模块的拆分
- 将不同的功能划分到不同的模块，在模块层面达到高内聚、低耦合
### 模块之间的调用
- 同层之间的模块可以通过同步接口调用
- 上下层系统之间可以使用消息中间件异步调用
### 模块内部
- IoC/DI
- MVC
- DDD

## 类的设计
### 需要哪些类
- MVC 的接口和实现类
- POJO
- 工具类
- ...

### 设计原则和设计模式
- 时刻考虑设计原则（SOLID、基于接口而非实现编程等）
- 思考哪些地方能用上设计模式，常用策略模式、模板方法模式、责任链模式

### 接口的设计
- 要符合接口隔离原则，粒度要小
- 在细粒度的接口外使用粗粒度的接口进行一次封装，给外部调用

### 功能设计
- 需要干啥就干啥
- 使用已有的专门的类，如线程池等

## 扩展、重构
- 添加新的功能以满足发展的业务需求
- 通过重构提高代码质量
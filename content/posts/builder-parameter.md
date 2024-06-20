---
author:
  name: "Alioth4J"
date: 2024-05-25
linktitle: 使用建造者模式构建方法参数，将方法与方法参数解耦
title: 使用建造者模式构建方法参数，将方法与方法参数解耦
type:
- post
- posts
weight: 10
series:
- Design Patterns
aliases:
- /blog/builder-parameter/
---

## MinIO中使用建造者模式构建方法的参数
例如：  
```
PutObjectArgs putObjectArgs = PutObjectArgs.builder()
                    .bucket(BUCKET_NAME)
                    .object(objectName)
                    .contentType(file.getContentType())
                    .stream(file.getInputStream(), file.getSize(), ObjectWriteArgs.MIN_MULTIPART_SIZE).build();
minioClient.putObject(putObjectArgs);
```
#### 以往将多个参数传入`()`中的问题
- 可读性较差  
一行代码过长，分多行也不够明确  
参数对应的是什么，需要记住或者ide提示
- 错误重载
- 违反开闭原则
当需要新增或修改参数时，需要修改方法签名
## 使用建造者模式构建方法参数的优点
#### 提供了一种规范
方法入参唯一，是`methodNameArgs`
#### 便于参数构建
- 链式调用清晰明了（编写、阅读、维护）
- 可选参数处理提供了灵活性
#### 符合开闭原则
需要新增或修改参数时，只需要修改建造者类
## 这些好处，本质上来源于**方法与方法参数的解耦**

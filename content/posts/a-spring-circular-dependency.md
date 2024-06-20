---
author:
  name: "Alioth4J"
date: 2024-05-21
linktitle: 记一次Spring循环依赖问题及解决方案
title: 记一次Spring循环依赖问题及解决方案
type:
- post
- posts
weight: 10
series:
- Spring
aliases:
- /blog/a-spring-circular-dependency/
---

## 背景
配置 Spring Security 配置类
## 具体情况
- `SecurityConfig`类中使用`@Bean`将`PasswordEncoder`对象配置为Bean  
- `SecurityConfig`类中使用`@Autowired`注入`AdminService`对象  
- `AdminServiceImpl`中使用`@Autowired`注入`PasswordEncoder`对象  
如此产生了循环依赖问题
## 解决方案
新增`PasswordEncoderConfig`类，在其中将`PasswordEncoder`对象配置为Bean  
```
@Configuration
public class PasswordEncoderConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
也就是说，改变了`PasswordEncoder`配置为Bean的位置，以此解决了循环依赖问题
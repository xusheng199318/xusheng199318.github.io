---
title: SpringBoot自动配置原理
date: 2019-04-20 13:54:32
tags:
---

### 开启自动配置

> spring-boot-autoconfiguration
>
> @EnableAutoConfiguration
>
> ​	exclude = Class<?>[]
>
> ​	AutoConfigutationImportSelector
>
> ​	META-INF/spring.factories
>
> ​		org.springframework.boot.autoconfigure.EnableAutoConfiguration
>
> @SpringBootApplication

### 条件注解

> @Conditional
>
> @ConditionalOnClass
>
> @ConditionalOnBean
>
> @ConditionalOnMissingBean
>
> @ConditionalOnProperty

### 手动实现自动配置

> 1. 声明自动配置类，@Configuration
---
title: SpringBoot自动配置原理
date: 2019-04-20 13:54:32
tags: Spring
---

### 开启自动配置

> SpringBoot的自动配置主要是由spring-boot-autoconfiguration为其实现的
>
> 其提供的自动配置项在META-INF下的spring.factories的`org.springframework.boot.autoconfigure.EnableAutoConfiguration=\`
>
> 配置项下

### 条件注解

> @Conditional
>
> @ConditionalOnClass：classpath下存在类
>
> @ConditionalOnBean：Spring上下文中存在Bean
>
> @ConditionalOnMissingBean：Spring上下文中不存在Bean
>
> @ConditionalOnProperty：配置文件中存在属性

### 手动实现自动配置

> 1. 自动配置类中引入相关依赖
>
> ~~~xml
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-autoconfigure</artifactId>
> </dependency>
> ~~~
>
> 2. 声明配置类
>
> ~~~java
> //声明该类为配置类
> @Configuration
> //如果在classpath下存在GreetingApplicationRunner类就进行配置
> @ConditionalOnClass(GreetingApplicationRunner.class)
> public class GreetingAutoconfig {
> 
>     @Bean
>     //如果Spring上下文中不存在GreetingApplicationRunner这个Bean就处理
>     @ConditionalOnMissingBean(GreetingApplicationRunner.class)
>     //如果配置中greeting.enabled的值为true就处理，如果不存在该属性，默认为true
>     @ConditionalOnProperty(name = "greeting.enabled", havingValue = "true", matchIfMissing = true)
>     public GreetingApplicationRunner greetingApplicationRunner() {
>         return new GreetingApplicationRunner();
>     }
> }
> ~~~
>
> 3. 在配置类中的resources下创建META-INF目录，再创建spring.factories文件
>
> ~~~yaml
> org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
> com.arthur.spring.autoconfig.GreetingAutoconfig
> ~~~
>
>
>
> 4. 在工程中引入含有`GreetingApplicationRunner`类的jar包，启动该工程时会进行自动配置
>
> ~~~xml
> <dependency>
>     <groupId>com.arthur</groupId>
>     <artifactId>spring-boot-autoconfig</artifactId>
>     <version>0.0.1-SNAPSHOT</version>
> </dependency>
> ~~~
>
>
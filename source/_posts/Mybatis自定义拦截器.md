---
title: Mybatis自定义拦截器
date: 2020-03-17 21:47:46
categories:
- 数据库
tags:
- java
---

# Mybatis自定义拦截器

## 一、Interceptor 接口

 Mybatis 为我们提供了一个 Interceptor 接口，通过实现该接口就可以定义我们自己的拦截器。

```java
package org.apache.ibatis.plugin;
import java.util.Properties;

public interface Interceptor {
  Object intercept(Invocation invocation) throws Throwable;
  Object plugin(Object target);
  void setProperties(Properties properties);
}
```

<!-- more -->

### 1、plugin 方法

plugin 方法是拦截器用于封装目标对象的，通过该方法我们可以返回目标对象本身，也可以返回一个它的代理。当返回的是代理的时候我们可以对其中的方法进行拦截来调用 intercept 方法，当然也可以调用其他方法，

### 2、 intercept 方法

intercept 方法就是要进行拦截的时候要执行的方法。

### 3、setProperties 方法

setProperties 方法是用于在 Mybatis 配置文件中指定一些属性的。

## 二、定义自己的拦截器

### 1、plugin 方法和 intercept 方法：

定义自己的Interceptor最重要的是要实现 plugin 方法和 intercept 方法：

 ① plugin 方法中决定是否要进行拦截进而决定要返回一个什么样的目标对象。

> 对于 plugin 方法而言，其实 Mybatis 已经为我们提供了一个实现。Mybatis 中有一个叫做Plugin 的类，里面有一个静态方法 wrap(Object target,Interceptor interceptor)，通过该方法可以决定要返回的对象是目标对象还是对应的代理。

 ② intercept 方法就是要进行拦截的时候要执行的方法。

### 2、@Intercepts和@Signature 注解

对于实现自己的 Interceptor 而言有两个很重要的注解，一个是 @Intercepts，其值是一个@Signature 数组。

@Intercepts 用于表明当前的对象是一个 Interceptor，而 @Signature则表明要拦截的接口、方法以及对应的参数类型。@Signature 的参数：

- type：要拦截的接口

- method：需要拦截的方法，存在于要拦截的接口之中

- args：被拦截的方法所需要的参数

> MyBatis 自定义拦截器，可以拦截的接口只有四种 :
>
> - Executor.class，拦截执行器的方法
> - StatementHandler.class，拦截Sql语法构建的处理
> - ParameterHandler.class,  拦截参数的处理
> - ResultSetHandler.class, 拦截结果集的处理

### 3、intercept 方法的参数 invocation

```java
public class Invocation {
    private final Object target;
    private final Method method;
    private final Object[] args;
	......
}
```

- target：*代理对象*

- method：*被监控方法对象*

- args：*当前被监控方法运行时需要的实参*


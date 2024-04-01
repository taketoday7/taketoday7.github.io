---
layout: post
title:  使用AspectJ为带注解的类提供通知方法
category: spring
copyright: spring
excerpt: Spring AOP
---

## 1. 概述

在本教程中，我们将在调用配置类的方法时使用AspectJ编写跟踪日志输出。
通过使用AOP通知来编写跟踪日志输出，我们将逻辑封装到单个编译单元中。

我们的示例扩展了AspectJ简介中提供的信息。

## 2. 跟踪日志记录注解


与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-modules/spring-aop-1)上获得。
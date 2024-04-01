---
layout: post
title:  Spring自注入
category: spring
copyright: spring
excerpt: Spring
---

## 1. 概述

自注入意味着Spring bean将自身作为依赖注入。它使用Spring容器获取对自身的引用，然后使用该引用执行某些操作。

在这个简短的教程中，我们将了解如何在Spring中使用自注入。

## 2. 自注入用例

自注入最常见的用例是在需要将切面应用于自引用的方法或类时绕过[Spring AOP](https://www.baeldung.com/spring-aop)限制。

假设我们有一个执行某些业务逻辑的服务类，并且需要在其自身上调用一个方法作为该逻辑的一部分：

```java
@Service
public class MyService {
    public void doSomething() {
        // ...
        doSomethingElse();
    }

    @Transactional
    public void doSomethingElse() {
        // ...
    }
}
```

**但是，当我们运行我们的应用程序时，我们可能会注意到[@Transactional](https://www.baeldung.com/spring-transactional-propagation-isolation)没有被应用**。这是因为doSomething()方法直接调用doSomethingElse()，它绕过了Spring代理。

为了解决这个问题，我们可以使用自注入来获取对Spring代理的引用并通过该代理调用该方法。

## 3. 使用@Autowired自注入

我们可以在Spring中通过在字段、构造函数参数或bean的setter方法上使用@Autowired注解来执行自注入。

下面是将@Autowired与字段一起使用的示例：

```java
@Component
public class MyBean {
    @Autowired
    private MyBean self;

    public void doSomething() {
        // use self reference here
    }
}
```

使用构造函数参数：

```java
@Component
public class MyBean {
    private MyBean self;

    @Autowired
    public MyBean(MyBean self) {
        this.self = self;
    }

    // ...
}
```

## 4. 使用ApplicationContextAware进行自注入

执行自注入的另一种方法是实现ApplicationContextAware接口。此接口允许bean了解Spring应用程序上下文并获得对自身的引用：

```java
@Component
public class MyBean implements ApplicationContextAware {
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {
        this.context = context;
    }

    public void doSomething() {
        MyBean self = context.getBean(MyBean.class);
        // ...
    }
}
```

## 5. 缺点

当一个bean注入自身时，**它会造成对该bean职责的混淆**，并使跟踪应用程序中的数据流变得更加困难。

**自注入也会创建[循环依赖](https://www.baeldung.com/circular-dependencies-in-spring)**。从2.6版本开始，如果我们的项目有循环依赖，Spring Boot将引发异常。当然，有一些解决方法。

## 6. 总结

在这篇简短的文章中，我们学习了在Spring中使用自注入的几种方法以及何时使用它。我们还了解了Spring中自注入的一些缺点。
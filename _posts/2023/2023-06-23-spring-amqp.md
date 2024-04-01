---
layout: post
title:  使用Spring AMQP进行消息传递
category: messaging
copyright: messaging
excerpt: RabbitMQ
---

## 1. 概述

在本教程中，我们将使用Spring AMQP框架探索基于消息的AMQP通信。

## 2. 基于消息的通信

消息传递是一种在应用程序之间进行通信的技术，它依赖于异步消息传递，而不是基于同步-请求响应的体系架构。**消息的生产者和消费者通过称为消息代理的中间消息传递层解耦**。消息代理提供消息的持久存储、消息过滤和消息转换等功能。

在用Java编写的应用程序之间进行消息传递的情况下，通常使用JMS(Java Message Service) API。**对于不同供应商和平台之间的互操作性，我们将无法使用JMS客户端和代理。这就是AMQP派上用场的地方**。

## 3. AMQP–高级消息队列协议

AMQP是用于异步消息通信的开放标准规范，它提供了如何构建消息的描述。

### 3.1 AMQP与JMS有何不同

由于AMQP是平台中立的二进制协议标准，因此可以用不同的编程语言编写依赖库，并在不同的环境中运行。

不存在基于供应商的协议锁定，从一个JMS代理迁移到另一个时就是这种情况。有关更多详细信息，请参阅[JMS与AMQP](https://www.linkedin.com/pulse/jms-vs-amqp-eran-shaham)和[了解AMQP](https://docs.spring.io/spring-amqp/reference/html/)。一些广泛使用的AMQP代理包括RabbitMQ、OpenAMQ和StormMQ。

### 3.2 AMQP实体

简而言之，AMQP由交换机、队列和绑定组成：

+ 交换机就像邮局或邮箱，客户端将消息发布到AMQP交换机。有四种内置的交换机类型：
    + Direct交换机：通过匹配完整的路由键将消息路由到队列
    + Fanout交换机：将消息路由到与其绑定的所有队列
    + Topic交换机：通过将路由键与模式匹配，将消息路由到多个队列
    + Header交换机：基于消息标头路由消息
    
+ 队列使用路由键绑定到交换器

+ 消息通过路由键发送到交换机，然后交换机将消息副本分发到队列

有关更多详细信息，请查看[AMQP概念](https://www.rabbitmq.com/tutorials/amqp-concepts.html)和[路由拓扑](https://spring.io/blog/2011/04/01/routing-topologies-for-performance-and-scalability-with-rabbitmq/)。

### 3.3 Spring AMQP

Spring AMQP包含两个模块：spring-amqp和spring-rabbit。这些模块共同提供了以下内容的抽象：

+ AMQP实体：我们使用Message、Queue、Binding和Exchange类创建实体
+ 连接管理：我们使用CachingConnectionFactory连接到RabbitMQ代理
+ 消息发布：我们使用RabbitTemplate发送消息
+ 消息消费：我们使用@RabbitListener从队列中读取消息

## 4. 设置RabbitMQ代理

我们需要一个可供我们连接的RabbitMQ代理，最简单的方法是使用Docker获取并运行RabbitMQ镜像：

```shell
docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
```

我们公开5672端口，以便我们的应用程序可以连接到RabbitMQ。

并且我们还公开端口15672，以便我们可以通过管理页面：http://localhost:15672或HTTP API：http://localhost:15672/api/index.html来访问我们的RabbitMQ控制台。

## 5. 创建Spring AMQP应用程序

现在让我们创建一个简单的应用程序，使用Spring AMPQ来发送和接收一个简单的“Hello, world!“消息。

### 5.1 Maven依赖项

为了将spring-amqp和spring-rabbit模块添加到我们的项目中，我们将spring-boot-starter-amqp依赖项添加pom.xml：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

最新版本的依赖项可以在[Maven Central](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-amqp)上找到。

### 5.2 连接到RabbitMQ服务

**我们将使用Spring Boot的自动配置来创建ConnectionFactory、RabbitTemplate和RabbitAdmin bean**。并且，Spring Boot默认使用用户名和密码“guest”连接到端口5672上的RabbitMQ服务。因此，我们只需使用@SpringBootApplication标注我们的应用程序主启动类：

```java
@SpringBootApplication
public class HelloWorldMessageApplication {
  // ...
}
```

### 5.3 创建队列

**为了创建我们的队列，我们只需定义一个Queue类型的bean**。RabbitAdmin会找到它并将其绑定到默认交换机，路由键为“myQueue”：

```java
@Bean
public Queue myQueue() {
    return new Queue("myQueue", false);
}
```

我们将队列设置为非持久队列，这样当RabbitMQ服务停止时，队列及其上的任何消息都将被删除。但是请注意，重新启动应用程序不会对队列产生任何影响。

### 5.4 发送消息

让我们**使用RabbitTemplate发送一条“Hello, world!”消息**：

```java
@Bean
public ApplicationRunner runner(RabbitTemplate template) {
    return args -> template.convertAndSend("myQueue", "Hello, world!");
}
```

### 5.5 消费消息

我们将通过使用@RabbitListener标注方法来实现消息的消费：

```java
@RabbitListener(queues = MY_QUEUE_NAME)
public void listen(String in) {
    System.out.println("Message read from myQueue : " + in);
}
```

## 6. 运行应用程序

首先，我们启动RabbitMQ服务：

```shell
docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
```

然后，我们通过运行HelloWorldMessageApplication.java来运行Spring Boot应用程序，执行main()方法：

```shell
mvn spring-boot:run -Dstart-class=cn.tuyucheng.taketoday.springamqp.simple.HelloWorldMessageApplication
```

当应用程序运行时，我们可以看到：

+ 应用程序向默认交换机发送一条消息，其中“myQueue”作为路由键
+ 然后，队列“myQueue”接收到消息
+ 最后，listen方法消费来自“myQueue”的消息并将其打印在控制台上

我们还可以通过[http://localhost:15672](http://localhost:15672)访问RabbitMQ管理台来确保我们的消息已经发送并消费。

## 7. 总结

在本教程中，我们介绍了基于AMQP协议的基于消息的架构，并使用Spring AMQP构建了一个简单的应用程序。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/messaging-modules/spring-amqp)上获得。
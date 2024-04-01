---
layout: post
title:  在RabbitMQ中创建动态队列
category: messaging
copyright: messaging
excerpt: RabbitMQ
---

## 1. 简介

[RabbitMQ](https://www.baeldung.com/rabbitmq)是一个消息代理，提供了各种组件之间的异步通信。它提供了最流行的消息传递协议[AMPQ(高级消息队列协议)](https://www.baeldung.com/rabbitmq-spring-amqp)的实现。

在本教程中，我们将介绍如何使用Java客户端库在RabbitMQ中动态创建队列。

## 2. RabbitMQ消息传递模型

在开始之前，让我们快速概览一下RabbitMQ消息传递的工作原理。

我们首先需要了解AMQP的构建块，也称为AMQP实体。[交换机、队列和绑定](https://www.baeldung.com/java-rabbitmq-exchanges-queues-bindings)统称为AMQP实体。

在RabbitMQ中，消息生产者从不直接将消息发送到队列。相反，它使用交换机作为路由中介。消息生产者将消息发布到交换机，然后交换机根据称为绑定的路由规则将这些消息路由到各种队列。然后代理将消息传递给订阅队列的消费者，或者消费者按需从队列中拉取/获取消息。将消息传递给消费者是基于FIFO模型进行的。

## 3. 连接初始化

RabbitMQ为所有主要编程语言提供客户端库，对于Java，客户端与RabbitMQ Broker通信的标准方式是使用RabbitMQ的amqp-client Java库。让我们将这个库的[Maven依赖项](https://search.maven.org/artifact/com.rabbitmq/amqp-client/5.16.0/jar)添加到我们项目的pom.xml文件中：

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.16.0</version>
</dependency>
```

**为了让客户端能够与RabbitMQ Broker交互，我们首先需要建立一个连接**。一旦建立了连接，我们就可以从现有的Connection创建一个Channel。**AMQP通道基本上是共享单个TCP连接的轻量级连接**。通道帮助我们在单个TCP连接之上多路复用多个逻辑连接。

Java RabbitMQ客户端库使用[工厂模式](https://www.baeldung.com/creational-design-patterns)来创建新连接。

首先，让我们创建一个新的ConnectionFactory实例。接下来，我们将设置创建连接所需的所有参数。至少，这需要告知RabbitMQ主机的地址：

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("amqp.tuyucheng.com");
```

其次，我们将使用我们创建的ConnectionFactory实例中的newConnection()工厂方法来获取新的Connection实例：

```java
Connection connection = factory.newConnection();
```

最后，让我们使用createChannel()方法从现有的Connection创建一个新的Channel：

```java
Channel channel = connection.createChannel();
```

我们成功连接到RabbitMQ Broker并创建了一个Channel，我们现在准备使用创建的Channel将命令发送到RabbitMQ服务器。

此外，我们可以[为单连接或多连接的通道设置不同的策略](https://www.baeldung.com/java-rabbitmq-channels-connections)。

## 4. 创建动态队列

Java RabbitMQ客户端库提供了多种易于使用的方法来创建和管理队列，让我们来看看一些重要的方法。

### 4.1 创建队列

**要动态创建队列，我们使用之前创建的Channel实例中的queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object\> arguments)方法**。如果队列不存在，则此方法会创建一个队列。该方法接收以下参数：

-   queue：要创建的队列的名称
-   durable：布尔标志，表示要创建的队列是否应该是持久的(即队列将在服务器重启后继续存在)
-   exclusive：布尔标志，表示要创建的队列是否应该是独占的(即仅限于此连接)
-   autoDelete：布尔标志，表示要创建的队列是否应自动删除(即服务器将在不再使用时将其删除)
-   arguments：队列的其他属性

让我们看一下创建队列的Java代码：

```java
AMQP.Queue.DeclareOk declareOk = channel.queueDeclare("tuyucheng-queue", true, false, false, null);
```

上面的代码片段创建了一个名为'tuyucheng-queue'的队列。成功创建队列后，该方法将返回AMQP.Queue.DeclareOk的实例。如果在创建队列时出现任何错误，该方法将抛出IOException。

此外，我们将使用返回的AMQP.Queue.DeclareOk实例来获取有关队列的信息-例如队列名称、队列的消费者数量以及队列中包含的消息数量。让我们看一下从DeclareOk实例获取队列名称的代码片段：

```java
String queueName = declareOk.getQueue();
```

上面的代码片段将返回创建的队列的名称。同样，我们可以获取队列的消费者计数和消息计数：

```java
int messageCount = declareOk.getMessageCount();
int consumerCount = declareOk.getConsumerCount();
```

我们已经了解了如何使用RabbitMQ Java客户端库动态创建队列。接下来，让我们看看如何检查队列是否存在。

### 4.2 检查队列是否存在

RabbitMQ Java客户端库还提供了一种检查队列是否存在的方法，**我们将使用方法queueDeclarePassive(String queue)来检查队列是否存在**。如果队列存在，此方法返回AMQP.Queue.DeclareOk的实例。如果队列不存在，队列是独占的，或者如果有任何其他错误，该方法将抛出IOException。

让我们看一下检查队列是否已经存在的Java代码：

```java
AMQP.Queue.DeclareOk declareOk = channel.queueDeclarePassive("tuyucheng-queue");
```

代码片段检查队列“tuyucheng-queue”是否存在。

最后，我们关闭通道和连接：

```java
channel.close();
connection.close();
```

我们还可以使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)来初始化通道和连接对象，以便它们自动关闭。

## 5. 总结

在本文中，我们首先了解了如何与RabbitMQ服务器建立连接并打开通信通道。

然后，我们使用RabbitMQ Java客户端库来演示如何动态创建队列并检查其是否存在。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/messaging-modules/rabbitmq)上获得。
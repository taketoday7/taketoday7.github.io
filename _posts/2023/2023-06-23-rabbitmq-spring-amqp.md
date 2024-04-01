---
layout: post
title:  使用Spring AMQP进行RabbitMQ消息调度
category: messaging
copyright: messaging
excerpt: RabbitMQ
---

## 1. 概述

在本教程中，我们将探讨[Spring AMQP](https://spring.io/projects/spring-amqp)和[RabbitMQ](http://www.rabbitmq.com/)中Fanout和Topic交换机的概念。

在高层次上，**Fanout交换机会将相同的消息广播到所有绑定队列，而Topic交换机使用路由键将消息传递到特定的绑定队列**。

对于本教程，建议你先阅读[使用Spring AMQP进行消息传递](https://www.baeldung.com/spring-amqp)。

## 2. 配置Fanout交换机

让我们设置一个Fanout交换器并绑定两个队列，当我们向这个交换机发送消息时，两个队列都会收到该消息。Fanout交换机会忽略消息中包含的任何路由键。

**Spring AMQP允许我们在Declarables对象中聚合队列、交换机和绑定的所有声明**：

```java
@Bean
public Declarables fanoutBindings() {
    Queue fanoutQueue1 = new Queue("fanout.queue1", false);
    Queue fanoutQueue2 = new Queue("fanout.queue2", false);
    FanoutExchange fanoutExchange = new FanoutExchange("fanout.exchange");

    return new Declarables(
        fanoutQueue1,
        fanoutQueue2,
        fanoutExchange,
        BindingBuilder.bind(fanoutQueue1).to(fanoutExchange),
        BindingBuilder.bind(fanoutQueue2).to(fanoutExchange));
}
```

## 3. 配置Topic交换机

现在，我们将设置一个包含两个队列的Topic交换机，每个队列都有不同的绑定模式：

```java
@Bean
public Declarables topicBindings() {
    Queue topicQueue1 = new Queue(topicQueue1Name, false);
    Queue topicQueue2 = new Queue(topicQueue2Name, false);

    TopicExchange topicExchange = new TopicExchange(topicExchangeName);

    return new Declarables(
        topicQueue1,
        topicQueue2,
        topicExchange,
        BindingBuilder
            .bind(topicQueue1)
            .to(topicExchange).with("*.important.*"),
        BindingBuilder
            .bind(topicQueue2)
            .to(topicExchange).with("#.error"));
}
```

**Topic交换机允许我们使用不同的路由键模式将队列绑定到它**，这非常灵活，允许我们将具有相同模式甚至多个模式的多个队列绑定到同一个队列。

当消息的路由键与模式匹配时，它将被放入队列中。**如果一个队列有多个与消息的路由键匹配的绑定，则只有一个消息副本放置在队列中**。

我们的绑定模式可以使用星号(“*”)来匹配特定位置的单词，或者使用井号(“#”)来匹配0个或多个单词。

因此，我们的topicQueue1将接收包含3个单词模式的路由键的消息，中间的单词是“important”-例如：“user.important.error”或“blog.important.notification”。

而我们的topicQueue2将接收路由键以单词error结尾的消息；例如“error”、“user.important.error”或“blog.post.save.error”。

## 4. 配置生产者

我们将使用RabbitTemplate的convertAndSend方法来发送示例消息：

```java
String message = " payload is broadcast";
return args -> {
    rabbitTemplate.convertAndSend(FANOUT_EXCHANGE_NAME, "", "fanout" + message);
    rabbitTemplate.convertAndSend(TOPIC_EXCHANGE_NAME, ROUTING_KEY_USER_IMPORTANT_WARN, "topic important warn" + message);
    rabbitTemplate.convertAndSend(TOPIC_EXCHANGE_NAME, ROUTING_KEY_USER_IMPORTANT_ERROR, "topic important error" + message);
};
```

**RabbitTemplate为不同的交换机类型提供了许多重载的convertAndSend()方法**。

当我们向Fanout交换机发送消息时，路由键将被忽略，消息被传递到所有绑定队列。

当我们向Topic交换机发送消息时，我们需要传递一个路由键。根据此路由键，消息将被传递到特定队列。

## 5. 配置消费者

最后，让我们设置四个消费者(每个队列一个)来消费生成的消息：

```java
@RabbitListener(queues = {BroadcastConfig.FANOUT_QUEUE_1_NAME})
public void receiveMessageFromFanout1(String message) {
    System.out.println("Received fanout 1 message: " + message);
}

@RabbitListener(queues = {BroadcastConfig.FANOUT_QUEUE_2_NAME})
public void receiveMessageFromFanout2(String message) {
    System.out.println("Received fanout 2 message: " + message);
}

@RabbitListener(queues = {BroadcastConfig.TOPIC_QUEUE_1_NAME})
public void receiveMessageFromTopic1(String message) {
    System.out.println("Received topic 1 (" + BroadcastConfig.BINDING_PATTERN_IMPORTANT + ") message: " + message);
}

@RabbitListener(queues = {BroadcastConfig.TOPIC_QUEUE_2_NAME})
public void receiveMessageFromTopic2(String message) {
    System.out.println("Received topic 2 (" + BroadcastConfig.BINDING_PATTERN_ERROR + ") message: " + message);
}
```

**我们使用@RabbitListener注解来配置消费者**，注解中传递的唯一参数是队列的名称。消费者在这里不知道交换机或路由键。

## 6. 运行应用程序

我们的示例项目是一个Spring Boot应用程序，因此它将初始化应用程序以及与RabbitMQ的连接，并配置所有队列、交换机和绑定。

默认情况下，我们的应用程序需要一个在本地主机端口5672上运行的RabbitMQ实例，可以在application.yaml中修改这些默认值。

我们的项目在URI(/broadcast)上公开HTTP端点，该端点接收请求正文中的消息。

当我们向此URI发送包含“Test”正文的请求时，我们应该在输出中看到类似的内容：

```shell
Received topic 1 (*.important.*) message: topic important warn payload is broadcast
Received fanout 2 message: fanout payload is broadcast
Received topic 2 (#.error) message: topic important error payload is broadcast
Received fanout 1 message: fanout payload is broadcast
Received topic 1 (*.important.*) message: topic important error payload is broadcast
```

当然，我们无法保证这些消息的输出顺序。

## 7. 总结

在这个简短的教程中，我们介绍了Spring AMQP和RabbitMQ的Fanout和Exchange交换机。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/messaging-modules/spring-amqp)上获得。
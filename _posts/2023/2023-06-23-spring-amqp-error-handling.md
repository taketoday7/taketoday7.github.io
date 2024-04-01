---
layout: post
title:  使用Spring AMQP进行错误处理
category: messaging
copyright: messaging
excerpt: RabbitMQ
---

## 1. 概述

异步消息传递是一种松耦合的分布式通信，在实现[事件驱动架构](https://www.baeldung.com/cqrs-event-sourced-architecture-resources)方面变得越来越流行。幸运的是，[Spring框架](https://www.baeldung.com/spring-intro)提供了Spring AMQP模块，允许我们构建基于AMQP的消息传递解决方案。

另一方面，**在这种环境中处理错误可能是一项艰巨的任务**。因此，在本教程中，我们将介绍处理错误的不同策略。

## 2. 环境搭建

对于本教程，**我们将使用实现AMQP标准的[RabbitMQ](https://www.baeldung.com/rabbitmq)**。此外，Spring AMQP提供了spring-rabbit模块，这使得集成变得非常容易。

让我们将RabbitMQ作为独立服务器运行，通过执行以下命令在[Docker容器](https://www.baeldung.com/dockerizing-spring-boot-application)中运行它：

```shell
docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
```

有关详细配置和项目依赖项设置，请参阅我们的[Spring AMQP文章](https://www.baeldung.com/spring-amqp)。

## 3. 失败场景

通常，由于其分布式特性，与单体应用程序或单个打包应用程序相比，基于消息传递的系统中可能会出现更多类型的错误。

以下是可能的一些意外情况：

+ 网络或I/O相关：网络连接和I/O操作的一般故障
+ 协议或基础设施相关：通常代表消息传递基础设施的错误配置
+ 代理相关：指示客户端和AMQP代理之间配置不正确的故障。例如，达到定义的限制或阈值、身份验证或无效的策略配置
+ 应用程序和消息相关：通常表明违反某些业务或应用程序规则的异常

当然，以上错误类型并不完整，但包含最常见的错误类型。

**我们应该注意到，Spring AMQP开箱即用地处理与连接相关的低级问题，例如通过应用重试或重新排队策略。此外，大多数失败和错误都会转换为AmqpException或其子类之一**。

在接下来的部分中，我们将主要关注特定于应用程序和高级别的错误，然后介绍全局错误处理策略。

## 4. 项目设置

现在，让我们定义一个简单的队列和交换机配置：

```java
public static final String QUEUE_MESSAGES = "tuyucheng-messages-queue";
public static final String EXCHANGE_MESSAGES = "tuyucheng-messages-exchange";

@Bean
Queue messagesQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES).build();
}
 
@Bean
DirectExchange messagesExchange() {
    return new DirectExchange(EXCHANGE_MESSAGES);
}
 
@Bean
Binding bindingMessages() {
    return BindingBuilder.bind(messagesQueue()).to(messagesExchange()).with(QUEUE_MESSAGES);
}
```

接下来，让我们创建一个简单的生产者：

```java
public void sendMessage() {
    rabbitTemplate.convertAndSend(SimpleDLQAmqpConfiguration.EXCHANGE_MESSAGES,
        SimpleDLQAmqpConfiguration.QUEUE_MESSAGES, "Some message id:" + messageNumber++);
}
```

最后，一个抛出异常的消费者：

```java
@RabbitListener(queues = SimpleDLQAmqpConfiguration.QUEUE_MESSAGES)
public void receiveMessage(Message message) throws BusinessException {
    throw new BusinessException();
}
```

**默认情况下，所有失败的消息都将在目标队列的头部一次又一次地重新排队**。

让我们通过执行以下Maven命令来运行示例应用程序：

```shell
mvn spring-boot:run -Dstart-class=cn.tuyucheng.taketoday.springamqp.errorhandling.ErrorHandlingApplication
```

现在我们应该可以看到类似的结果输出：

```text
WARN 22260 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler :
  Execution of Rabbit message listener failed.
Caused by: cn.tuyucheng.taketoday.springamqp.errorhandling.errorhandler.BusinessException: null
```

因此，默认情况下，我们将在输出中看到无限数量的此类消息。

要改变这种行为，我们有两个选择：

-   在监听器端将default-requeue-rejected选项设置为false：spring.rabbitmq.listener.simple.default-requeue-rejected=false
-   抛出AmqpRejectAndDontRequeueException：这对于将来没有意义的消息可能有用，因此可以将其丢弃

现在，让我们了解如何以更智能的方式处理失败的消息。

## 5. 死信队列

**死信队列(DLQ)是保存未传递或失败消息的队列**。DLQ使我们能够处理错误或不良消息、监视故障模式并从系统异常中恢复。

更重要的是，这有助于防止队列中不断处理不良消息并降低系统性能的无限循环。

总而言之，有两个主要概念：死信交换机(DLX)和死信队列(DLQ)本身。事实上，**DLX是一种正常的交换机，我们可以将其定义为常见类型之一：direct、topic或fanout**。

**理解生产者对队列一无所知非常重要。它只知道交换机，并且所有生成的消息都根据交换机配置和消息路由键进行路由**。

现在让我们看看如何应用死信队列方法来处理异常。

### 5.1 基本配置

为了配置DLQ，我们需要在定义队列时指定其他参数：

```java
@Bean
Queue messagesQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES)
        .withArgument("x-dead-letter-exchange", "")
        .withArgument("x-dead-letter-routing-key", QUEUE_MESSAGES_DLQ)
        .build();
}
 
@Bean
Queue deadLetterQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES_DLQ).build();
}
```

在上面的示例中，我们使用了两个附加参数：x-dead-letter-exchange和x-dead-letter-routing-key。**x-dead-letter-exchange选项的空字符串值告诉代理使用默认的交换机**。

第二个参数与为简单消息设置路由键同样重要。此选项更改消息的初始路由键，以供DLX进一步路由。

### 5.2 失败消息路由

因此，当消息无法传递时，它会被路由到死信交换机。但正如我们已经指出的，**DLX是一个正常的交换机。因此，如果失败的消息路由键与交换机不匹配，则不会将其传递到DLQ**。

```text
Exchange: (AMQP default)
Routing Key: tuyucheng-messages-queue.dlq
```

因此，如果我们在示例中省略x-dead-letter-routing-key参数，则失败的消息将陷入无限重试循环中。

此外，消息的原始元信息可在x-death标头中找到：

```text
x-death:
  count: 1
  exchange: tuyucheng-messages-exchange
  queue: tuyucheng-messages-queue 
  reason: rejected
  routing-keys: tuyucheng-messages-queue 
  time: 1571232954
```

上述信息可在通常在端口15672上本地运行的RabbitMQ管理控制台中获得。

除了此配置之外，如果我们使用[Spring Cloud Stream](https://www.baeldung.com/spring-cloud-stream)，我们甚至可以通过利用配置属性republishToDlq和autoBindDlq来简化配置过程。

### 5.3 死信交换机

在上一节中，我们已经看到，当消息路由到死信交换机时，路由键会发生更改。但这种行为并不总是可取的，我们可以通过自己配置DLX并使用fanout类型定义它来更改它：

```java
public static final String DLX_EXCHANGE_MESSAGES = QUEUE_MESSAGES + ".dlx";
 
@Bean
Queue messagesQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES)
        .withArgument("x-dead-letter-exchange", DLX_EXCHANGE_MESSAGES)
        .build();
}
 
@Bean
FanoutExchange deadLetterExchange() {
    return new FanoutExchange(DLX_EXCHANGE_MESSAGES);
}
 
@Bean
Queue deadLetterQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES_DLQ).build();
}
 
@Bean
Binding deadLetterBinding() {
    return BindingBuilder.bind(deadLetterQueue()).to(deadLetterExchange());
}
```

**这次我们定义了fanout类型的自定义交换机，因此消息将被发送到所有有界队列**。此外，我们已将x-dead-letter-exchange参数的值设置为DLX的名称。同时，我们删除了x-dead-letter-routing-key参数。

现在，如果我们运行我们的示例，失败的消息应该传递到DLQ，但不更改初始路由键：

```text
Exchange: tuyucheng-messages-queue.dlx
Routing Key: tuyucheng-messages-queue
```

### 5.4 处理死信队列消息

当然，我们将它们移至死信队列的原因是为了可以在其他时间重新处理它们。

让我们为死信队列定义一个监听器：

```java
@RabbitListener(queues = QUEUE_MESSAGES_DLQ)
public void processFailedMessages(Message message) {
    log.info("Received failed message: {}", message.toString());
}
```

如果我们现在运行代码示例，我们应该看到日志输出：

```text
WARN 11752 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 11752 --- [ntContainer#1-1] c.t.t.s.e.consumer.SimpleDLQAmqpContainer  : Received failed message:
```

**我们收到一条失败的消息，但接下来我们应该做什么**？答案取决于特定的系统要求、异常类型或消息类型。

例如，我们可以将消息重新排队到原始目的地：

```java
@RabbitListener(queues = QUEUE_MESSAGES_DLQ)
public void processFailedMessagesRequeue(Message failedMessage) {
    log.info("Received failed message, requeueing: {}", failedMessage.toString());
    rabbitTemplate.send(EXCHANGE_MESSAGES, failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
}
```

但这样的异常逻辑和默认的重试策略并没有什么不同：

```text
INFO 23476 --- [ntContainer#0-1] c.t.t.s.e.c.RoutingDLQAmqpContainer : Received message: 
WARN 23476 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 23476 --- [ntContainer#1-1] c.t.t.s.e.c.RoutingDLQAmqpContainer : Received failed message, requeueing:
```

常见的策略可能需要重试处理一条消息n次，然后拒绝它。让我们通过利用消息标头来实现此策略：

```java
public void processFailedMessagesRetryHeaders(Message failedMessage) {
    Integer retriesCnt = (Integer) failedMessage.getMessageProperties()
        .getHeaders().get(HEADER_X_RETRIES_COUNT);
    if (retriesCnt == null) retriesCnt = 1;
    if (retriesCnt > MAX_RETRIES_COUNT) {
        log.info("Discarding message");
        return;
    }
    log.info("Retrying message for the {} time", retriesCnt);
    failedMessage.getMessageProperties()
        .getHeaders().put(HEADER_X_RETRIES_COUNT, ++retriesCnt);
    rabbitTemplate.send(EXCHANGE_MESSAGES, failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
}
```

首先，我们获取x-retries-count标头的值，然后将该值与最大允许值进行比较。随后，如果计数器达到尝试限制次数，则消息将被丢弃：

```text
WARN 1224 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 1224 --- [ntContainer#1-1] c.t.t.s.e.consumer.DLQCustomAmqpContainer  : Retrying message for the 1 time
WARN 1224 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 1224 --- [ntContainer#1-1] c.t.t.s.e.consumer.DLQCustomAmqpContainer  : Retrying message for the 2 time
WARN 1224 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 1224 --- [ntContainer#1-1] c.t.t.s.e.consumer.DLQCustomAmqpContainer  : Discarding message
```

我们应该补充一点，我们还可以使用x-message-ttl标头来设置消息应被丢弃的时间，这可能有助于防止队列无限增长。

### 5.5 停车场队列

另一方面，考虑一种情况，我们不能直接丢弃消息，例如，它可能是银行域中的交易。或者，有时一条消息可能需要手动处理，或者我们只需要记录失败次数超过n次的消息。

对于这种情况，就有了“停车场队列”的概念。我们可以**将DLQ中失败次数超过允许次数的所有消息转发到停车场队列进行进一步处理**。

现在让我们实现这个想法：

```java
public static final String QUEUE_PARKING_LOT = QUEUE_MESSAGES + ".parking-lot";
public static final String EXCHANGE_PARKING_LOT = QUEUE_MESSAGES + "exchange.parking-lot";
 
@Bean
FanoutExchange parkingLotExchange() {
    return new FanoutExchange(EXCHANGE_PARKING_LOT);
}
 
@Bean
Queue parkingLotQueue() {
    return QueueBuilder.durable(QUEUE_PARKING_LOT).build();
}
 
@Bean
Binding parkingLotBinding() {
    return BindingBuilder.bind(parkingLotQueue()).to(parkingLotExchange());
}
```

其次，我们重新构建监听器逻辑，向停车场队列发送消息：

```java
@RabbitListener(queues = QUEUE_MESSAGES_DLQ)
public void processFailedMessagesRetryWithParkingLot(Message failedMessage) {
    Integer retriesCnt = (Integer) failedMessage.getMessageProperties()
        .getHeaders().get(HEADER_X_RETRIES_COUNT);
    if (retriesCnt == null) retriesCnt = 1;
    if (retriesCnt > MAX_RETRIES_COUNT) {
        log.info("Sending message to the parking lot queue");
        rabbitTemplate.send(EXCHANGE_PARKING_LOT, failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
        return;
    }
    log.info("Retrying message for the {} time", retriesCnt);
    failedMessage.getMessageProperties()
        .getHeaders().put(HEADER_X_RETRIES_COUNT, ++retriesCnt);
    rabbitTemplate.send(EXCHANGE_MESSAGES, failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
}
```

最终，我们还需要处理到达停车场队列的消息：

```java
@RabbitListener(queues = QUEUE_PARKING_LOT)
public void processParkingLotQueue(Message failedMessage) {
    log.info("Received message in parking lot queue");
    // Save to DB or send a notification.
}
```

现在我们可以将失败的消息保存到数据库或者发送电子邮件通知。

让我们通过运行我们的应用程序来测试这个逻辑：

```text
WARN 14768 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 14768 --- [ntContainer#1-1] c.t.t.s.e.c.ParkingLotDLQAmqpContainer     : Retrying message for the 1 time
WARN 14768 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 14768 --- [ntContainer#1-1] c.t.t.s.e.c.ParkingLotDLQAmqpContainer     : Retrying message for the 2 time
WARN 14768 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : Execution of Rabbit message listener failed.
INFO 14768 --- [ntContainer#1-1] c.t.t.s.e.c.ParkingLotDLQAmqpContainer     : Sending message to the parking lot queue
INFO 14768 --- [ntContainer#2-1] c.t.t.s.e.c.ParkingLotDLQAmqpContainer     : Received message in parking lot queue
```

从输出中我们可以看到，经过几次失败的尝试，消息被发送到停车场队列。

## 6. 自定义错误处理

在上一节中，我们了解了如何使用专用队列和交换机来处理故障。**但是，有时我们可能需要捕获所有错误，例如将其记录或保存到数据库中**。

### 6.1 全局ErrorHandler

到目前为止，我们已经使用了默认的SimpleRabbitListenerContainerFactory，并且该工厂默认使用ConditionalRejectingErrorHandler。该处理程序捕获不同的异常并将它们转换为AmqpException层次结构中的异常之一。

值得一提的是，如果我们需要处理连接错误，那么我们需要实现ApplicationListener接口。

简单来说，**ConditionalRejectingErrorHandler决定是否拒绝特定消息**。当导致异常的消息被拒绝时，它不会被重新排队。

让我们定义一个自定义ErrorHandler，它将仅重新排队BusinessException：

```java
public class CustomErrorHandler implements ErrorHandler {
    @Override
    public void handleError(Throwable t) {
        if (!(t.getCause() instanceof BusinessException)) {
            throw new AmqpRejectAndDontRequeueException("Error Handler converted exception to fatal", t);
        }
    }
}
```

此外，当我们在监听器方法中抛出异常时，它被包装在ListenerExecutionFailedException中。因此，我们需要调用getCause方法来获取异常源。

### 6.2 FatalExceptionStrategy

在幕后，该处理程序使用FatalExceptionStrategy来检查是否应将异常视为致命异常。如果是这样，失败的消息将被拒绝。

默认情况下，以下异常是致命的：

-   MessageConversionException
-   MessageConversionException
-   MethodArgumentNotValidException
-   MethodArgumentTypeMismatchException
-   NoSuchMethodException
-   ClassCastException

**我们可以只提供FatalExceptionStrategy，而不是实现ErrorHandler接口**：

```java
public class CustomFatalExceptionStrategy extends ConditionalRejectingErrorHandler.DefaultExceptionStrategy {
    @Override
    public boolean isFatal(Throwable t) {
        return !(t.getCause() instanceof BusinessException);
    }
}
```

最后，我们需要将自定义策略传递给ConditionalRejectingErrorHandler构造函数：

```java
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory, SimpleRabbitListenerContainerFactoryConfigurer configurer) {
      SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
      configurer.configure(factory, connectionFactory);
      factory.setErrorHandler(errorHandler());
      return factory;
}
 
@Bean
public ErrorHandler errorHandler() {
    return new ConditionalRejectingErrorHandler(customExceptionStrategy());
}
 
@Bean
FatalExceptionStrategy customExceptionStrategy() {
    return new CustomFatalExceptionStrategy();
}
```

## 7. 总结

在本教程中，我们讨论了使用Spring AMQP(特别是RabbitMQ)时处理错误的不同方法。

每个系统都需要特定的错误处理策略，我们已经介绍了事件驱动架构中最常见的错误处理方法。此外，我们还可以结合多种策略来构建更全面、更强大的解决方案。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/messaging-modules/spring-amqp)上获得。
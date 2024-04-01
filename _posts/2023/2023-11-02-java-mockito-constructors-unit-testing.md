---
layout: post
title:  如何使用Mockito Mock构造函数进行单元测试
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在这个简短的教程中，我们将探讨使用Mockito和PowerMock在Java中有效mock构造函数的各种选项。

## 2. 使用PowerMock Mock构造函数

使用Mockito 3.3或更低版本无法mock构造函数或静态方法，在这种情况下，像[PowerMock](https://www.baeldung.com/intro-to-powermock)这样的库提供了额外的功能，使我们能够mock构造函数的行为并协调它们的交互。

## 3. 模型

让我们使用两个Java类来模拟支付处理系统。我们将创建一个PaymentService类，其中包含处理付款的逻辑，并提供用于指定付款模式的参数化构造函数和具有回退模式的默认构造函数的灵活性：

```java
public class PaymentService {
    private final String paymentMode;

    public PaymentService(String paymentMode) {
        this.paymentMode = paymentMode;
    }

    public PaymentService() {
        this.paymentMode = "Cash";
    }

    public String processPayment() {
        return this.paymentMode;
    }
}
```

PaymentProcessor类依赖PaymentService来执行支付处理任务，并提供两个构造函数，一个用于默认设置，另一个用于自定义支付模式：

```java
public class PaymentProcessor {
    private final PaymentService paymentService;

    public PaymentProcessor() {
        this.paymentService = new PaymentService();
    }

    public PaymentProcessor(String paymentMode) {
        this.paymentService = new PaymentService(paymentMode);
    }

    public String processPayment() {
        return paymentService.processPayment();
    }
}
```

## 4. 使用Mockito Mock默认构造函数

**在编写单元测试时，隔离我们想要测试的代码至关重要**。构造函数通常会创建我们不想在测试中涉及的依赖项，mock构造函数允许我们用mock对象替换真实对象，确保我们正在测试的行为特定于所检查的单元。

从Mockito 3.4版及更高版本开始，我们可以访问mockConstruction()方法，它允许我们mock对象构造。**我们指定要mock其构造函数的类作为第一个参数；此外，我们以MockInitializer回调函数的形式提供第二个参数**，这个回调函数允许我们在构造过程中定义和操作mock的行为：

```java
@Test
void whenConstructorInvokedWithInitializer_ThenMockObjectShouldBeCreated(){
    try(MockedConstruction<PaymentService> mockPaymentService = Mockito.mockConstruction(PaymentService.class,(mock,context)-> {
        when(mock.processPayment()).thenReturn("Credit");
    })){
        PaymentProcessor paymentProcessor = new PaymentProcessor();
        Assertions.assertEquals(1,mockPaymentService.constructed().size());
        Assertions.assertEquals("Credit", paymentProcessor.processPayment());
    }
}
```

mockConstruction()方法有多个重载版本，每个版本都适用于不同的用例。在下面的场景中，我们不使用MockInitializer来初始化mock对象。我们正在验证构造函数是否被调用过一次，并且缺少初始化程序可确保构造的PaymentService对象中paymentMode字段的null状态：

```java
@Test
void whenConstructorInvokedWithoutInitializer_ThenMockObjectShouldBeCreatedWithNullFields(){
    try(MockedConstruction<PaymentService> mockPaymentService = Mockito.mockConstruction(PaymentService.class)){
        PaymentProcessor paymentProcessor = new PaymentProcessor();
        Assertions.assertEquals(1,mockPaymentService.constructed().size());
        Assertions.assertNull(paymentProcessor.processPayment());
    }
}
```

## 5. 使用Mockito Mock参数化构造函数

在此示例中，我们设置了MockInitializer并调用了参数化构造函数。我们正在验证是否创建了一个Mock，并且它具有在初始化期间定义的所需值：

```java
@Test
void whenConstructorInvokedWithParameters_ThenMockObjectShouldBeCreated(){
    try(MockedConstruction<PaymentService> mockPaymentService = Mockito.mockConstruction(PaymentService.class,(mock, context) -> {
        when(mock.processPayment()).thenReturn("Credit");
    })){
        PaymentProcessor paymentProcessor = new PaymentProcessor("Debit");
        Assertions.assertEquals(1,mockPaymentService.constructed().size());
        Assertions.assertEquals("Credit", paymentProcessor.processPayment());
    }
}
```

## 6. Mock构造函数的作用域

Java中的[try-with-resources](https://www.baeldung.com/java-try-with-resources)构造允许我们限制所创建的Mock的作用域，在此块中，对指定类的公共构造函数的任何调用都会创建mock对象。当在块之外的任何地方调用真正的构造函数时，将会调用它。

在下面的示例中，我们没有定义任何初始值设定项，而是多次调用默认构造函数和参数化构造函数。然后，mock的行为是在构建后定义的。

我们验证3个mock对象确实已创建并且遵守我们预定义的mock行为：

```java
@Test
void whenMultipleConstructorsInvoked_ThenMultipleMockObjectsShouldBeCreated(){
    try(MockedConstruction<PaymentService> mockPaymentService = Mockito.mockConstruction(PaymentService.class)){
        PaymentProcessor paymentProcessor = new PaymentProcessor();
        PaymentProcessor secondPaymentProcessor = new PaymentProcessor();
        PaymentProcessor thirdPaymentProcessor = new PaymentProcessor("Debit");

        when(mockPaymentService.constructed().get(0).processPayment()).thenReturn("Credit");
        when(mockPaymentService.constructed().get(1).processPayment()).thenReturn("Online Banking");

        Assertions.assertEquals(3,mockPaymentService.constructed().size());
        Assertions.assertEquals("Credit", paymentProcessor.processPayment());
        Assertions.assertEquals("Online Banking", secondPaymentProcessor.processPayment());
        Assertions.assertNull(thirdPaymentProcessor.processPayment());
    }
}
```

## 7. 依赖注入和构造函数Mock

当我们使用[依赖注入](https://www.baeldung.com/spring-dependency-injection)时，我们可以直接传递mock对象，避免需要mock构造函数。通过这种方法，我们可以在实例化被测试的类之前mock依赖关系，从而无需mock任何构造函数。

让我们在PaymentProcessor类中引入第三个构造函数，其中PaymentService作为依赖项注入：

```java
public PaymentProcessor(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

我们已经将依赖项与PaymentProcessor类解耦，这使我们能够单独测试我们的单元，并通过mock控制依赖项的行为，如下所示：

```java
@Test
void whenDependencyInjectionIsUsed_ThenMockObjectShouldBeCreated(){
    PaymentService mockPaymentService = Mockito.mock(PaymentService.class);
    when(mockPaymentService.processPayment()).thenReturn("Online Banking");
    PaymentProcessor paymentProcessor = new PaymentProcessor(mockPaymentService);
    Assertions.assertEquals("Online Banking", paymentProcessor.processPayment());
}
```

但是，在我们无法控制源代码中如何管理依赖项的情况下，特别是当依赖项注入不可行时，mockConstruction()成为有效mock构造函数的有用工具。

## 8. 总结

这篇简短的文章展示了通过Mockito和PowerMock mock构造函数的不同方法，我们还讨论了在可行的情况下优先考虑依赖注入的优点。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/mockito-2)上获得。
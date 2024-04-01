---
layout: post
title:  使用不同的参数Mock相同的方法
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在Java中[mock](https://www.baeldung.com/mockito-series)方法时，根据传入的参数接收不同的响应可能很有用。在本文中，我们将根据需求的复杂性研究实现该目标的不同方法。

## 2. 设置

首先，**让我们创建一个要mock的示例服务**：

```java
class ExampleService {
    int getValue(int arg) {
        return 1;
    }
}
```

我们有一个非常简单的服务，只有一个方法，该方法有一个int作为参数并返回一个int。请注意，参数和返回值没有关系，因此默认情况下，它将始终返回1。

## 3. 连续存根的局限性

让我们看看连续存根以及我们能用它做什么和不能做什么。**无论我们提供什么输入，我们都可以使用连续存根从mock中按顺序获取不同的参数**，这显然缺乏对将特定输入与所需输出匹配的控制，但在许多情况下很有用。为此，我们将要存根的方法传递给[when()](https://www.baeldung.com/mockito-behavior)；然后，我们链接调用thenReturn()，按照我们想要的顺序提供响应：

```java
@Test
void givenAMethod_whenUsingConsecutiveStubbing_thenExpectResultsInOrder(){
    when(exampleService.getValue(anyInt())).thenReturn(9, 18, 27);
    assertEquals(9, exampleService.getValue(1));
    assertEquals(18, exampleService.getValue(1));
    assertEquals(27, exampleService.getValue(1));
    assertEquals(27, exampleService.getValue(1));
}
```

**从断言中我们可以看到，尽管总是传入1作为参数，但我们还是按顺序收到了期望值**。一旦返回所有值，所有未来的调用都将返回最终值，如我们测试中的第四个调用所示。

## 4. 不同参数的存根调用

**我们可以扩展when()和thenReturn()的使用，为不同的参数返回不同的值**： 

```java
@Test
void givenAMethod_whenStubbingForMultipleArguments_thenExpectDifferentResults() {
    when(exampleService.getValue(10)).thenReturn(100);
    when(exampleService.getValue(20)).thenReturn(200);
    when(exampleService.getValue(30)).thenReturn(300);

    assertEquals(100, exampleService.getValue(10));
    assertEquals(200, exampleService.getValue(20));
    assertEquals(300, exampleService.getValue(30));
}
```

when()的参数是我们想要存根的方法，以及我们想要指定响应的值。通过将对when()的调用与thenReturn()链接起来，我们指示mock在收到正确的参数时返回请求的值。我们可以自由地将任意数量的这些应用到我们的mock中以处理一系列输入，每次提供预期的输入值时，我们都会收到请求的返回值。

## 5. 使用thenAnswer()

**提供最大控制的更复杂的选项是使用[thenAnswer()](https://www.baeldung.com/mockito-additionalanswers)**，这允许我们获取参数，对它们执行我们想要的任何计算，然后返回一个在与mock交互时输出的值：

```java
@Test
void givenAMethod_whenUsingThenAnswer_thenExpectDifferentResults() {
    when(exampleService.getValue(anyInt())).thenAnswer(invocation -> {
        int argument = (int) invocation.getArguments()[0];
        int result;
        switch (argument) {
        case 25:
            result = 125;
            break;
        case 50:
            result = 150;
            break;
        case 75:
            result = 175;
            break;
        default:
            result = 0;
        }
        return result;
    });
    assertEquals(125, exampleService.getValue(25));
    assertEquals(150, exampleService.getValue(50));
    assertEquals(175, exampleService.getValue(75));
}
```

上面，我们在提供的调用对象上使用getArguments()获取了参数。我们在这里假设了一个int参数，但我们可以满足几种不同的类型，我们还可以检查是否至少有一个参数并且强制转换为int是否成功。为了演示这些功能，我们使用[switch](https://www.baeldung.com/java-switch)语句根据输入返回不同的值。在底部，我们可以从断言中看到我们的mock服务返回switch语句的结果。

这个选项允许我们通过一次调用when()来处理无限数量的输入，牺牲的是测试的可读性和可维护性。

## 6. 总结

在本教程中，我们了解了配置mock方法以返回不同值的三种方法。我们研究了连续存根，发现它对于按顺序返回任何输入的已知值很有用，但除此之外就非常有限。对每个潜在输入使用when()与thenReturn()相结合提供了一个具有改进控制的简单解决方案。或者，我们可以使用thenAnswer()来最大程度地控制给定输入和预期输出之间的关系。根据测试的要求，这三种方法都有用。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/mockito-2)上找到。
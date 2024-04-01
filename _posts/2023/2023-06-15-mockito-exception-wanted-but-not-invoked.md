---
layout: post
title:  解决Mockito异常：需要但未调用
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将讨论使用[Mockito](https://www.baeldung.com/mockito-annotations)时可能遇到的常见错误。[异常](https://www.baeldung.com/java-exceptions)消息是：

```text
Wanted but not invoked:
// class name and location
Actually, there were zero interactions with this mock.
```

让我们了解此错误的潜在来源以及如何修复它。

## 2. 示例设置

首先，让我们创建我们稍后要mock的类。它包含一个单独的方法，始终返回“Tuyucheng”：

```java
class Helper {
    String getTuyuchengString() {
        return "Tuyucheng";
    }
}
```

现在让我们创建我们的主类。它在类级别声明一个Helper实例，**我们希望在单元测试期间mock这个实例**：

```java
class Main {
    Helper helper = new Helper();

    String methodUnderTest(int i) {
        if (i > 5) {
            return helper.getTuyuchengString();
        }
        return "Hello";
    }
}
```

最重要的是，我们定义了一个接收Integer作为参数并返回的方法：

-   如果Integer大于5，调用getTuyuchengString()的结果
-   如果Integer小于或等于5，则为常量

## 3. 调用真正的方法而不是Mock

让我们尝试为我们的方法编写[单元测试](https://www.baeldung.com/java-unit-testing-best-practices#whatIsUnitTesting)，我们将使用[@Mock](https://www.baeldung.com/mockito-annotations#mock-annotation)注解来创建mock的Helper。我们还将调用[MockitoAnnotations.openMocks()](https://www.baeldung.com/mockito-annotations#2-mockitoannotationsopenmocks)来启用Mockito注解。在测试方法中，我们将使用参数7调用methodUnderTest()并检查它是否委托给getTuyuchengString()：

```java
class MainUnitTest {

    @Mock
    Helper helper;

    Main main = new Main();

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void givenValueUpperThan5_WhenMethodUnderTest_ThenDelegatesToHelperClass() {
        main.methodUnderTest(7);
        Mockito.verify(helper)
                .getTuyuchengString();
    }
}
```

现在让我们运行我们的测试：

```text
Wanted but not invoked:
helper.getTuyuchengString();
-> at cn.tuyucheng.taketoday.wantedbutnotinvocked.Helper.getTuyuchengString(Helper.java:6)
Actually, there were zero interactions with this mock.
```

问题是**我们调用了[构造函数](https://www.baeldung.com/java-constructors)来实例化一个Main对象**。因此，Helper实例是通过调用new()创建的。**因此，我们使用的是真正的Helper对象而不是我们的mock对象**。要解决这个问题，我们需要在创建Main对象的基础上添加[@InjectMocks](https://www.baeldung.com/mockito-annotations#injectmocks-annotation)：

```java
@InjectMocks
Main main = new Main();
```

作为旁注，如果我们在methodUnderTest()的任何点用真实对象替换mock实例，我们将再次陷入同样的问题：

```java
String methodUnderTest(int i) {
    helper = new Helper();
    if (i > 5) {
        return helper.getTuyuchengString();
    }
    return "Hello";
}
```

简而言之，我们这里有两个注意点：

-   应该正确创建和注入mock
-   在任何时候都不应将mock对象替换为其他对象

## 4. 方法未被调用

我们现在将编写一个新的单元测试，它将检查将3作为参数传递给methodUnderTest()是否会调用getTuyuchengString()：

```java
@Test
void givenValueLowerThan5_WhenMethodUnderTest_ThenDelegatesToGetTuyuchengString() {
    main.methodUnderTest(3);
    Mockito.verify(helper)
       .getTuyuchengString();
}
```

再一次，我们可以运行测试：

```text
Wanted but not invoked:
helper.getTuyuchengString();
-> at cn.tuyucheng.taketoday.wantedbutnotinvocked.Helper.getTuyuchengString(Helper.java:6)
Actually, there were zero interactions with this mock.
```

这次，让我们仔细阅读错误信息。它说我们没有与mock交互。现在让我们回顾一下我们的方法规范：3小于5，因此methodUnderTest()返回一个常量而不是委托给getTuyuchengString()。**因此，我们的测试与规范相矛盾**。

在这种情况下，我们只有两种可能的结论：

-   规范是正确的：我们需要修复我们的测试，因为验证是无用的。
-   测试是正确的：我们的代码中有一个错误需要解决。

## 5. 总结

在本文中，我们在没有与mock交互的情况下调用[Mockito.verify()](https://www.baeldung.com/mockito-verify)并出现错误。我们指出我们需要正确地注入和使用mock。我们还看到这个错误是由不连贯的测试引起的。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/mockito-2)上获得。
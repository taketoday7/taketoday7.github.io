---
layout: post
title:  使用Mockito匹配Null
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在这个简短的教程中，我们将使用[Mockito](https://www.baeldung.com/mockito-annotations)检查null是否作为参数传递给方法。我们将看到如何直接匹配null并使用[ArgumentMatchers](https://www.baeldung.com/mockito-argument-matchers)。

## 2. 示例设置

首先，让我们创建一个简单的Helper类，其中包含一个单独的concat()方法，返回两个[String的拼接](https://www.baeldung.com/java-strings-concatenation)：

```java
class Helper {

    String concat(String a, String b) {
        return a + b;
    }
}
```

现在我们将添加一个Main类，它的方法methodUnderTest()调用concat()以将字符串Tuyucheng与null拼接起来：

```java
class Main {

    Helper helper = new Helper();

    String methodUnderTest() {
        return helper.concat("Tuyucheng", null);
    }
}
```

## 3. 只使用精确值

让我们设置测试类：

```java
class MainUnitTest {

    @Mock
    Helper helper;

    @InjectMocks
    Main main;

    @BeforeEach
    void openMocks() {
        MockitoAnnotations.openMocks(this);
    }

    // Add test method
}
```

得益于[@Mock](https://www.baeldung.com/mockito-annotations#mock-annotation)，我们创建了一个mock的Helper。然后我们通过[@InjectMocks](https://www.baeldung.com/mockito-annotations#injectmocks-annotation)将它注入到我们的Main实例中。最后，我们调用[MockitoAnnotations.openMocks()](https://www.baeldung.com/mockito-annotations#2-mockitoannotationsopenmocks)来启用Mockito注解。

我们的目标是编写一个[单元测试](https://www.baeldung.com/java-unit-testing-best-practices#whatIsUnitTesting)来验证methodUnderTest()委托给concat()。此外，我们要确保第二个参数为null。让我们保持简单并检查调用的第一个参数是Tuyucheng而第二个参数是null：

```java
@Test
void whenMethodUnderTest_thenSecondParameterNull() {
    main.methodUnderTest();
    Mockito.verify(helper)
        .concat("Tuyucheng", null);
}
```

我们调用[Mockito.verify()](https://www.baeldung.com/mockito-verify)来检查参数值是否符合预期。

## 4. 使用匹配器

现在，我们将使用Mockito的ArgumentMatchers来检查传递的值。由于第一个值与我们的示例无关，我们将使用any()匹配器：因此，任何输入都将通过。要检查第二个参数是否为null，我们可以简单地使用isNull()：

```java
@Test
void whenMethodUnderTest_thenSecondParameterNullWithMatchers() {
    main.methodUnderTest();
    Mockito.verify(helper)
        .concat(any(), isNull());
}
```

## 5. 总结

在本文中，我们学习了如何使用Mockito验证传递给方法的参数是否为null，我们通过检查精确值和使用ArgumentMatchers来做到这一点。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/mockito-2)上获得。
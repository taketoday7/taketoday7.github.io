---
layout: post
title:  在JUnit 4和5中断言抛出异常
category: unittest
copyright: unittest
excerpt: JUnit异常断言
---

## 1. 概述

在这个快速教程中，我们将介绍如何使用JUnit库测试是否引发了异常。

当然，我们将确保涵盖JUnit 4和JUnit 5版本。

## 2. JUnit 5

**JUnit 5 Jupiter Assertions API引入了用于断言异常的assertThrows方法**。

该方法需要一个预期异常的类型和一个Executable函数式接口作为参数，我们可以在其中通过lambda表达式传递被测代码：

```java
@Test
void whenExceptionThrown_thenAssertionSucceeds() {
    Exception exception = assertThrows(NumberFormatException.class, () -> Integer.parseInt("1a"));

    String expectedMessage = "For input string";
    String actualMessage = exception.getMessage();

    assertTrue(actualMessage.contains(expectedMessage));
}
```

**如果抛出预期的异常，assertThrows()方法会返回该异常，这使我们也可以对异常消息进行断言**。

此外，请务必注意，**当包含的代码抛出NumberFormatException类型或其任何子类型的异常时，此断言就会满足**。

这意味着如果我们将Exception作为预期的异常类型传递，则抛出的任何异常都将使断言成功，因为Exception是所有异常的父类。

如果我们将上面测试的预期异常更改为RuntimeException，这也将通过：

```java
@Test
void whenDerivedExceptionThrown_thenAssertionSucceeds() {
    Exception exception = assertThrows(RuntimeException.class, () -> Integer.parseInt("1a"));

    String expectedMessage = "For input string";
    String actualMessage = exception.getMessage();

    assertTrue(actualMessage.contains(expectedMessage));
}
```

assertThrows()方法可以对异常断言逻辑进行更细粒度的控制，因为我们可以在代码的特定部分使用它。

## 3. JUnit 4

在使用JUnit 4时，我们可以使用地**使用@Test注解的expected属性**来声明我们期望在带注解的测试方法中的任何位置抛出异常。

因此，当测试运行时，如果没有抛出指定的异常，它将失败，如果抛出则通过：

```java
@Test(expected = NullPointerException.class)
public void whenExceptionThrown_thenExpectationSatisfied() {
    String test = null;
    test.length();
}
```

在此示例中，我们期望测试代码抛出NullPointerException。

如果我们只对断言抛出异常感兴趣，这就足够了。**当我们需要验证异常的一些其他属性时，可以使用ExpectedException Rule**。

让我们看一个验证异常的message属性的示例：

```java
@Rule
public ExpectedException exceptionRule = ExpectedException.none();

@Test
public void whenExceptionThrown_thenRuleIsApplied() {
    exceptionRule.expect(NumberFormatException.class);
    exceptionRule.expectMessage("For input string");
    Integer.parseInt("1a");
}
```

在上面的示例中，我们首先声明ExpectedException Rule。然后在我们的测试中，我们断言尝试解析Integer值的代码将导致NumberFormatException，并且异常消息为“For input string”。

## 4. 总结

在本文中，我们介绍了JUnit 4和JUnit 5中的异常断言。

这些示例的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)上找到。
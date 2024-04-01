---
layout: post
title:  Java中的Catch块会捕获错误吗？
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在这篇简短的文章中，我们将展示如何正确捕获Java错误，并且我们将解释何时这样做没有意义。

有关Java中Throwable的详细信息，请查看我们关于[Java中异常处理](https://www.baeldung.com/java-exceptions)的文章。

## 2. 捕捉错误

由于Java中的java.lang.Error类不继承自java.lang.Exception，因此我们必须在catch语句中声明Error基类-或者我们想要捕获的特定Error子类，才能捕获它。

因此，如果我们运行以下测试用例，它将通过：

```java
@Test(expected = AssertionError.class)
public void whenError_thenIsNotCaughtByCatchException() {
    try {
        throw new AssertionError();
    } catch (Exception e) {
        Assert.fail(); // errors are not caught by catch exception
    }
}
```

但是，以下单元测试期望catch语句捕获错误：

```java
@Test
public void whenError_thenIsCaughtByCatchError() {
    try {
        throw new AssertionError();
    } catch (Error e) {
        // caught! -> test pass
    }
}
```

请注意，**Java虚拟机抛出错误以指示它无法从中恢复的严重问题**，例如内存不足和堆栈溢出等。

因此，**我们必须有一个非常非常好的理由来捕获错误！**

## 3. 总结

在本文中，我们了解了何时以及如何在Java中捕获Error。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-1)上获得。

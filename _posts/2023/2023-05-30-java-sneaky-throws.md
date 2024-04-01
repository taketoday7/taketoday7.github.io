---
layout: post
title:  Java中的Sneaky Throws
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在Java中，sneaky throw概念允许我们抛出任何受检异常，而无需在方法签名中明确定义它。这允许省略throws声明，有效地模仿了运行时异常的特征。

在本文中，我们将通过查看一些代码示例来了解如何在实践中完成此操作。

## 2. 关于Sneaky Throws

**受检异常是Java的一部分，而不是JVM**。在字节码中，我们可以不受限制地从任何地方抛出任何异常。

Java 8带来了一个新的类型推断规则，该规则规定只要允许，抛出的T就会被推断为RuntimeException。这提供了在没有辅助方法的情况下实现sneaky throws的能力。

sneaky throws的一个问题是你可能希望最终捕获异常，但Java编译器不允许你使用针对特定异常类型的异常处理程序捕获sneaky throws的受检异常。

## 3. Sneaky Throws实践

正如我们已经提到的，编译器和Java运行时可以看到不同的东西：

```java
public static <E extends Throwable> void sneakyThrow(Throwable e) throws E {
    throw (E) e;
}

private static void throwSneakyIOException() {
    sneakyThrow(new IOException("sneaky"));
}
```

**编译器看到带有推断为RuntimeException类型的throws T的签名**，因此它允许传播非受检的异常。Java运行时在throws中看不到任何类型，因为所有抛出都是相同的简单throw e。

这个快速测试演示了这个场景：

```java
@Test
public void throwSneakyIOException_IOExceptionShouldBeThrown() {
    assertThatThrownBy(() -> throwSneakyIOException())
        .isInstanceOf(IOException.class)
        .hasMessage("sneaky")
        .hasStackTraceContaining("SneakyThrowsExamples.throwSneakyIOException");
}
```

此外，可以使用字节码操作或Thread.stop(Throwable)抛出受检异常，但它很麻烦，不推荐使用。

## 4. 使用Lombok注解

来自[Lombok](https://projectlombok.org/)的@SneakyThrows注解允许你在不使用throws声明的情况下抛出受检的异常。当你需要从非常严格的接口(如Runnable)中的方法引发异常时，这会派上用场。

假设我们从Runnable中抛出一个异常；它只会传递给线程的未处理异常处理程序。

此代码将抛出Exception实例，因此你无需将其包装在RuntimeException中：

```java
@SneakyThrows
public static void throwSneakyIOExceptionUsingLombok() {
    throw new IOException("lombok sneaky");
}
```

此代码的缺点是你无法捕获未声明的受检异常。例如，**如果我们试图捕获上述方法偷偷抛出的IOException，我们会得到一个编译错误**。

现在，让我们调用throwSneakyIOExceptionUsingLombok并期望Lombok抛出IOException：

```java
@Test
public void throwSneakyIOExceptionUsingLombok_IOExceptionShouldBeThrown() {
    assertThatThrownBy(() -> throwSneakyIOExceptionUsingLombok())
        .isInstanceOf(IOException.class)
        .hasMessage("lombok sneaky")
        .hasStackTraceContaining("SneakyThrowsExamples.throwSneakyIOExceptionUsingLombok");
}
```

## 5. 总结

正如我们在本文中看到的，我们可以欺骗Java编译器将受检异常视为未受检异常。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-4)上获得。

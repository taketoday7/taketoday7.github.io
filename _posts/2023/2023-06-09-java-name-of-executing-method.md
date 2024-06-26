---
layout: post
title:  如何获取正在执行的方法的名称？
category: java-new
copyright: java-new
excerpt: Java 9
---

## 1. 概述

有时我们需要知道当前正在执行的Java方法的名称，这篇文章介绍了几种在当前执行堆栈中获取方法名称的简单方法。

## 2.Java9：StackWalking API

Java 9引入了[Stack-Walking](Java9中StackWalking简介.md) API，以一种惰性且高效的方式遍历JVM堆栈帧。为了使用此API找到当前正在执行的方法，我们可以编写一个简单的测试：

```java
class CurrentExecutingMethodUnitTest {

	@Test
	void givenJava9_whenWalkingTheStack_thenFindMethod() {
		StackWalker walker = StackWalker.getInstance();
		Optional<String> methodName = walker.walk(frames -> frames
				.findFirst()
				.map(StackWalker.StackFrame::getMethodName)
		);

		assertTrue(methodName.isPresent());
		assertEquals("givenJava9_whenWalkingTheStack_thenFindMethod", methodName.get());
	}
}
```

首先，我们使用[getInstance()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html#getInstance())工厂方法得到一个[StackWalker](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html)实例，然后我们使用[walk()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html#walk(java.util.function.Function))方法从上到下遍历栈帧： 

-   walk()方法可以将堆栈帧流Stream<StackFrame>转换为任何东西
-   给定流中的第一个元素是堆栈上的顶部帧
-   堆栈上的顶部帧始终表示当前正在执行的方法

因此，如果我们从流中获取第一个元素，我们就可以知道当前正在执行的方法的详细信息。更具体地说，我们可以使用[StackFrame.getMethodName()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.StackFrame.html#getMethodName())来查找方法名称。

### 2.1 优点

与其他方法相比(稍后会详细介绍)，Stack-Walking API有以下几个优点：

-   无需创建虚拟匿名内部类实例 — new Object().getClass() {}
-   无需创建虚拟异常 — new Throwable()
-   无需急切地捕获整个堆栈跟踪，此操作可能会很昂贵

相反，StackWalker只是以一种惰性的方式逐个遍历堆栈。在这种情况下，它只获取顶部帧，而不是像stacktrace方法那样快速捕获所有帧。

最重要的是，如果你使用的是Java 9+，请使用Stack-Walking API。

## 3. 使用getEnclosureMethod

我们可以使用getEnclosureMethod() API得到正在执行的方法的名称：

```java
void givenObject_whenGetEnclosingMethod_thenFindMethod() {
    String methodName = new Object() {}
      .getClass()
      .getEnclosingMethod()
      .getName();
       
    assertEquals("givenObject_whenGetEnclosingMethod_thenFindMethod", methodName);
}
```

## 4. 使用Throwable Stack Trace

使用Throwable堆栈跟踪为我们提供了当前正在执行的方法的堆栈跟踪：

```java
void givenThrowable_whenGetStacktrace_thenFindMethod() {
    StackTraceElement[] stackTrace = new Throwable().getStackTrace();
 
    assertEquals("givenThrowable_whenGetStacktrace_thenFindMethod",
      stackTrace[0].getMethodName());
}
```

## 5. 使用线程堆栈跟踪

此外，当前线程(从JDK 1.5开始)的堆栈跟踪通常包括正在执行的方法的名称：

```java
void givenCurrentThread_whenGetStackTrace_thenFindMethod() {
    StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
 
    assertEquals(
      "givenCurrentThread_whenGetStackTrace_thenFindMethod",
      stackTrace[1].getMethodName()); 
}
```

但是，我们需要记住，这种解决方案有一个明显的缺点。某些虚拟机可能会跳过一个或多个堆栈帧。虽然这并不常见，但我们应该意识到这可能会发生。

## 6. 总结

在本教程中，我们通过几个例子说明了如何获取当前执行的方法的名称。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-9-new-features)上获得。
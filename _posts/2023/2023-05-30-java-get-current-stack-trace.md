---
layout: post
title:  在Java中获取当前堆栈跟踪
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

作为Java开发人员，在处理[异常](https://www.baeldung.com/java-checked-unchecked-exceptions)时经常会遇到堆栈跟踪的概念。

在本教程中，我**们将了解堆栈跟踪是什么以及如何在编程/调试时使用它**。此外，我们还将介绍StackTraceElement类。最后，我们将学习如何使用Thread和Throwable类来获取它。

## 2. 什么是堆栈跟踪？

**堆栈跟踪(也称为回溯)是堆栈帧的列表**。简单来说，这些帧代表程序执行过程中的一个时刻。

**堆栈帧包含有关代码调用的方法的信息**。它是一个帧列表，从当前方法开始一直延伸到程序开始的时间。

为了更好地理解这一点，让我们看一个简单的示例，其中我们在异常后转储(dump)了当前堆栈跟踪：

```java
public class DumpStackTraceDemo {
    public static void main(String[] args) {
        methodA();
    }

    public static void methodA() {
        try {
            int num1 = 5 / 0; // java.lang.ArithmeticException: divide by zero
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

在上面的示例中，methodA()抛出ArithmeticException，这反过来又在catch块中转储当前的堆栈跟踪：

```text
java.lang.ArithmeticException: / by zero
at main.java.cn.tuyucheng.taketoday.tutorials.DumpStackTraceDemo.methodA(DumpStackTraceDemo.java:11)
at main.java.cn.tuyucheng.taketoday.tutorials.DumpStackTraceDemo.main(DumpStackTraceDemo.java:6)
```

## 3. StackTraceElement类

**堆栈跟踪由StackTraceElement类的元素组成**。我们可以使用以下方法分别获取类名和方法名：

-   getClassName：返回包含当前执行点的类的完全限定名
-   getMethodName：返回包含此堆栈跟踪元素表示的执行点的方法的名称

我们可以在[Java API文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/StackTraceElement.html)的StackTraceElement类中看到完整的方法列表及其详细信息。

## 4. 使用Thread类获取堆栈跟踪

我们可以通过调用Thread实例上的getStackTrace()方法从线程获取堆栈跟踪。它返回一个StackTraceElement数组，从中可以找到有关线程堆栈帧的详细信息。

让我们看一个例子：

```java
public class StackTraceUsingThreadDemo {

    public static void main(String[] args) {
        methodA();
    }

    public static StackTraceElement[] methodA() {
        return methodB();
    }

    public static StackTraceElement[] methodB() {
        Thread thread = Thread.currentThread();
        return thread.getStackTrace();
    }
}
```

在上面的类中，方法调用以下列方式发生-main() -> methodA() -> methodB() -> getStackTrace()。

让我们用下面的测试用例来验证它，其中测试用例方法调用methodA()：

```java
@Test
public void whenElementIsFetchedUsingThread_thenCorrectMethodAndClassIsReturned() {
    StackTraceElement[] stackTrace = new StackTraceUsingThreadDemo().methodA();
    
    StackTraceElement elementZero = stackTrace[0];
    assertEquals("java.lang.Thread", elementZero.getClassName());
    assertEquals("getStackTrace", elementZero.getMethodName());
    
    StackTraceElement elementOne = stackTrace[1];
    assertEquals("cn.tuyucheng.taketoday.tutorials.StackTraceUsingThreadDemo", elementOne.getClassName());
    assertEquals("methodB", elementOne.getMethodName());
    
    StackTraceElement elementTwo = stackTrace[2];
    assertEquals("cn.tuyucheng.taketoday.tutorials.StackTraceUsingThreadDemo", elementTwo.getClassName());
    assertEquals("methodA", elementTwo.getMethodName());
    
    StackTraceElement elementThree = stackTrace[3];
    assertEquals("test.java.cn.tuyucheng.taketoday.tutorials.CurrentStacktraceDemoUnitTest", elementThree.getClassName());
    assertEquals("whenElementIsFetchedUsingThread_thenCorrectMethodAndClassIsReturned", elementThree.getMethodName());
}
```

在上面的测试用例中，我们使用StackTraceUsingThreadDemo类的methodB()获取了一个StackTraceElement数组。然后，使用StackTraceElement类的getClassName()和getMethodName()方法验证堆栈跟踪中的方法和类名。

## 5. 使用Throwable类获取堆栈跟踪

当任何Java程序抛出一个Throwable对象时，我们可以通过调用getStackTrace()方法获取一个StackTraceElement对象数组，而不是简单地将其打印在控制台上或记录它。

让我们看一个例子：

```java
public class StackTraceUsingThrowableDemo {

    public static void main(String[] args) {
        methodA();
    }

    public static StackTraceElement[] methodA() {
        try {
            methodB();
        } catch (Throwable t) {
            return t.getStackTrace();
        }
        return null;
    }

    public static void methodB() throws Throwable {
        throw new Throwable("A test exception");
    }
}
```

在这里，方法调用以下列方式发生-main() -> methodA() -> methodB() -> getStackTrace()。

让我们使用测试来验证它：

```java
@Test
public void whenElementIsFecthedUsingThrowable_thenCorrectMethodAndClassIsReturned() {
    StackTraceElement[] stackTrace = new StackTraceUsingThrowableDemo().methodA();

    StackTraceElement elementZero = stackTrace[0];
    assertEquals("cn.tuyucheng.taketoday.tutorials.StackTraceUsingThrowableDemo", elementZero.getClassName());
    assertEquals("methodB", elementZero.getMethodName());

    StackTraceElement elementOne = stackTrace[1];
    assertEquals("cn.tuyucheng.taketoday.tutorials.StackTraceUsingThrowableDemo", elementOne.getClassName());
    assertEquals("methodA", elementOne.getMethodName());

    StackTraceElement elementThree = stackTrace[2];
    assertEquals("test.java.cn.tuyucheng.taketoday.tutorials.CurrentStacktraceDemoUnitTest", elementThree.getClassName());
    assertEquals("whenElementIsFecthedUsingThrowable_thenCorrectMethodAndClassIsReturned", elementThree.getMethodName());
}
```

在上面的测试用例中，我们使用StackTraceUsingThrowableDemo类的methodB()获取了一个StackTraceElement数组。然后，验证方法和类名以了解StackTraceElement类数组中元素的顺序。

## 6. 总结

在本文中，我们了解了Java堆栈跟踪以及如何在出现异常时使用printStackTrace()方法打印它。我们还研究了如何使用Thread和Throwable类获取当前堆栈跟踪。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-4)上获得。

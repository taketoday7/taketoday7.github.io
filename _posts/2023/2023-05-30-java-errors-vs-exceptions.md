---
layout: post
title:  Java中的错误和异常
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将了解Java错误和异常及其区别。

## 2. Throwable类

**[Error](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Error.html)和[Exception](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Exception.html)都是[Throwable](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Throwable.html)类的子类，用于表示异常情况的发生**。此外，只有Throwable及其子类的实例可以被Java虚拟机抛出或在catch子句中捕获。

创建Error和Exception的实例以包含有关情况的信息(例如，堆栈跟踪)：

![](/assets/images/2023/javaex/javaerrorsvsexceptions01.png)

## 3. Error

错误表示不应该发生的异常情况。发生严重问题时会抛出错误。**此外，错误被视为非受检的异常，应用程序不应尝试捕获和处理它们**。此外，错误发生在运行时并且无法恢复。

现在让我们看一个例子：

```java
public class ErrorExample {

    public static void main(String[] args) {
        throw new AssertionError();
    }
}
```

如果我们运行上面的代码，我们会得到以下信息：

```text
Exception in thread "main" java.lang.AssertionError:
at cn.tuyucheng.taketoday.exception.exceptions_vs_errors.ErrorExample.main(ErrorExample.java:6)
```

该代码导致了一个称为[AssertionError](https://docs.oracle.com/javase/7/docs/api/java/lang/AssertionError.html)的错误，抛出该错误以指示断言何时失败。

Java错误的其他示例包括[StackOverflowError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackOverflowError.html)、[LinkageError](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/LinkageError.html)、[IOError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/IOError.html)和[VirtualMachineError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/VirtualMachineError.html)。

## 4. Exception

**异常是应用程序可能想要捕获和处理的异常情况**。可以使用try-catch块恢复异常，并且可以在运行时和编译时发生。

一些用于[异常处理](https://www.baeldung.com/java-exceptions)的技术是try-catch块、throws关键字和try-with-resources块。

异常分为两类：运行时异常和受检异常。

### 4.1 运行时异常

[RuntimeException](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/RuntimeException.html)及其子类是Java虚拟机运行时可能抛出的异常。此外，它们是非受检的异常。非受检的异常不需要使用throws关键字在方法签名中声明，如果它们可以在方法执行后抛出并传播到方法范围之外。

让我们看一个例子：

```java
public class RuntimeExceptionExample {
    public static void main(String[] args) {
        int[] arr = new int[20];

        arr[20] = 20;

        System.out.println(arr[20]);
    }
}
```

运行上面的代码后，我们得到以下信息：

```text
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 20
  at cn.tuyucheng.taketoday.exception.exceptions_vs_errors.RuntimeExceptionExample.main(RuntimeExceptionExample.java:7)
```

正如我们所看到的，我们得到了一个[ArrayIndexOutOfBoundsException](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/ArrayIndexOutOfBoundsException.html)，它是[IndexOutOfBoundsException](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/IndexOutOfBoundsException.html)的子类，它本身是 RuntimeException的子类。

RuntimeException的其他子类包括[IllegalArgumentException](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IllegalArgumentException.html)、[NullPointerException](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/NullPointerException.html)和[ArithmeticException](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/ArithmeticException.html)。

### 4.2 受检异常

其他不是RuntimeException子类的异常是受检的异常。如果它们可以在方法执行后抛出并传播到方法范围之外，则需要使用throws关键字在方法签名中声明它们：

```java
public class CheckedExceptionExample {
    public static void main(String[] args) {
        try (FileInputStream fis = new FileInputStream(new File("test.txt"))) {
            fis.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

如果我们运行上面的代码，我们会得到以下信息：

```text
java.io.FileNotFoundException: test.txt (No such file or directory)
  at java.io.FileInputStream.open0(Native Method)
  at java.io.FileInputStream.open(FileInputStream.java:195)
  at java.io.FileInputStream.<init>(FileInputStream.java:138)
  at cn.tuyucheng.taketoday.exception.exceptions_vs_errors.CheckedExceptionExcample.main(CheckedExceptionExcample.java:9)
```

我们得到了一个[FileNotFoundException](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileNotFoundException.html)，它是[IOException](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/io/IOException.html)的子类，而IOException是Exception的子类。

[TimeoutException](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/TimeoutException.html)和[SQLException](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/SQLException.html)是受检异常的其他示例。

## 5. 总结

在本文中，我们了解了Java生态系统中错误和异常之间的区别。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-4)上获得。

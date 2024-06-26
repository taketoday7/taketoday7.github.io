---
layout: post
title:  在Java中包装与重新抛出异常
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

Java中的throw关键字用于显式抛出[自定义异常或内置异常](https://www.baeldung.com/java-exceptions)。但有时在catch块中，我们需要再次抛出相同的异常。这会导致重新抛出异常。

在本教程中，我们将讨论两种最常见的重新抛出异常的方法。

## 2. 重新抛出异常

有时在将异常传播到更高级别之前，我们可能需要执行一些操作。例如，我们可能想要回滚数据库事务、记录异常或发送电子邮件。

我们可以在catch块中执行此类操作并再次重新抛出异常。通过这种方式，更高级别会收到系统中发生异常的通知。

让我们通过一个例子来理解我们的案例。

下面，我们重新抛出相同的异常。而且，我们在抛出错误消息之前记录了一条错误消息：

```java
String name = null;

try {
    return name.equals("Joe"); // causes NullPointerException
} catch (Exception e) {
    // log
    throw e;
}
```

控制台将显示以下消息：

```text
Exception in thread "main" java.lang.NullPointerException
  at cn.tuyucheng.taketoday.exceptions.RethrowSameExceptionDemo.main(RethrowSameExceptionDemo.java:16)
```

正如我们所看到的，我们的代码只是重新抛出它捕获的任何异常。正因为如此，**我们得到了没有任何变化的原始堆栈跟踪**。

## 3. 包装异常

现在，让我们看一下不同的方法。

在这种情况下，我们将传递相同的异常作为不同异常的构造函数中的引用：

```java
String name = null;

try {
    return name.equals("Joe"); // causes NullPointerException
} catch (Exception e) {
    // log
    throw new IllegalArgumentException(e);
}
```

控制台将显示：

```text
Exception in thread "main" java.lang.IllegalArgumentException: java.lang.NullPointerException
  at cn.tuyucheng.taketoday.exceptions.RethrowDifferentExceptionDemo.main(RethrowDifferentExceptionDemo.java:24)
Caused by: java.lang.NullPointerException
  at cn.tuyucheng.taketoday.exceptions.RethrowDifferentExceptionDemo.main(RethrowDifferentExceptionDemo.java:18)
```

这一次，我们看到了原始异常和包装异常。这样，**我们的IllegalArgumentException实例将原始的NullPointerException包装为一个cause**。因此，我们可以显示更具体的异常而不是显示通用异常。

## 4. 总结

在这篇简短的文章中，我们介绍了重新抛出原始异常与首先包装它之间的主要区别。这两种方式在显示异常消息的方式上各不相同。

根据我们的要求，我们可以重新抛出相同的异常，也可以使用第二种方法将其包装为某些特定异常。第二种方法看起来更清晰并且易于回溯异常。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-2)上获得。

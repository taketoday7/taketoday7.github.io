---
layout: post
title:  捕捉Throwable是一种不好的做法吗？
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将了解捕获Throwable的含义。

## 2. Throwable类

在Java文档中，Throwable类被定义为“**Java语言中所有错误和异常的超类**”。

让我们看一下Throwable类的层次结构：

![](/assets/images/2023/javaexception/javacatchthrowablebadpractice01.png)

Throwable类有两个直接的子类-即Error和Exception类。

Error及其子类是非受检异常，而Exception的子类既可以是[受检异常也可以是非受检异常](https://www.baeldung.com/java-exceptions)。

让我们看看程序失败时可能遇到的情况类型。

## 3. 可恢复的情况

在某些情况下，恢复通常是可能的，并且可以使用Exception类的受检或非受检子类来处理。

例如，程序可能想要使用恰好不存在于指定位置的文件，从而导致抛出受检的FileNotFoundException。

另一个例子是程序试图在没有权限的情况下访问系统资源，导致抛出未受检的AccessControl异常。

根据Java文档，**Exception类“指示合理的应用程序可能想要捕获的条件”**。

## 4. 不可恢复的情况

在某些情况下，如果发生故障，程序可能会进入无法恢复的状态。常见的例子是发生堆栈溢出或JVM内存不足。

在这些情况下，JVM分别抛出StackOverflowError和OutOfMemoryError。顾名思义，它们是Error类的子类。

根据Java文档，**Error类“表示合理的应用程序不应尝试捕获的严重问题”**。

## 5. 可恢复和不可恢复情况示例

假设我们有一个API，允许调用者使用addIDsToStorage方法将唯一ID添加到某些存储设施：

```java
class StorageAPI {

    public void addIDsToStorage(int capacity, Set<String> storage) throws CapacityException {
        if (capacity < 1) {
            throw new CapacityException("Capacity of less than 1 is not allowed");
        }
        int count = 0;
        while (count < capacity) {
            storage.add(UUID.randomUUID().toString());
            count++;
        }
    }

    // other methods go here ...
}
```

调用addIDsToStorage时可能会出现几个潜在的故障点：

-   CapacityException：传递小于1的容量值时异常的受检子类
-   NullPointerException：如果提供空storage值而不是Set<String\>的实例，则为Exception的未受检子类
-   OutOfMemoryError：如果JVM在退出while循环之前内存不足，则为Error的非受检子类

CapacityException和NullPointerException情况是程序可以从中恢复的故障，但OutOfMemoryError是不可恢复的。

## 6. 捕捉Throwable

假设API的用户在调用addIDsToStorage时仅在try-catch中捕获Throwable：

```java
public void add(StorageAPI api, int capacity, Set<String> storage) {
    try {
        api.addIDsToStorage(capacity, storage);
    } catch (Throwable throwable) {
        // do something here
    }
}
```

这意味着调用代码以相同的方式对可恢复和不可恢复的情况做出反应。

处理异常的一般规则是try-catch块在捕获异常时必须尽可能具体。也就是说，必须避免包罗万象的情况。

**在我们的案例中捕获Throwable违反了这个一般规则**。为了分别对可恢复和不可恢复的情况做出反应，调用代码必须检查catch块内的Throwable对象的实例。

**更好的方法是使用特定的方法来处理异常并避免尝试处理不可恢复的情况**。

## 7. 总结

在本文中，我们研究了在try-catch块中捕获Throwable的含义。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-2)上获得。

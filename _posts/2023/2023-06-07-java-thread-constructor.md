---
layout: post
title:  为什么不在构造函数中启动一个线程？
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在这个快速教程中，我们将了解为什么不应该在构造函数中启动线程。

首先，我们将简要介绍Java和JVM中的发布概念。然后，我们将看到这个概念如何影响我们启动线程的方式。

## 2. 发布与逃逸

**每次我们使一个对象可用于其当前范围之外的任何其他代码时，我们基本上就是发布了该对象**。例如，当我们返回一个对象，将其存储到公共引用中，甚至将其传递给另一个方法时，就会发生发布。

**当我们发布一个不该拥有的对象时，我们就说这个对象逃逸了**。

有很多方法可以让对象引用逃逸，例如在对象完全构造之前发布对象。事实上，这是一种常见的逃逸形式：**当this引用在对象构造期间逃逸时**。

当this引用在构造过程中逃逸时，其他线程可能会看到该对象处于不正确且未完全构造的状态。反过来，这可能会导致奇怪的线程安全并发症。

## 3. 线程逃逸

**让this引用逃逸的最常见方法之一是在构造函数中启动线程**。为了更好地理解这一点，让我们考虑一个例子：

```java
public class LoggerRunnable implements Runnable {

    public LoggerRunnable() {
        Thread thread = new Thread(this); // this escapes
        thread.start();
    }

    @Override
    public void run() {
        System.out.println("Started...");
    }
}
```

在这里，我们显式地将this引用传递给Thread构造函数。因此，**新启动的线程可能能够在其完整构造完成之前看到封闭对象**。在并发上下文中，这可能会导致细微的错误。

**也可以隐式传递this引用**：

```java
public class ImplicitEscape {

    public ImplicitEscape() {
        Thread t = new Thread() {

            @Override
            public void run() {
                System.out.println("Started...");
            }
        };

        t.start();
    }
}
```

如上所示，我们正在创建一个从Thread派生的匿名内部类。**由于内部类维护对其封闭类的引用，因此this引用再次从构造函数中逃逸**。

在构造函数中创建线程本身并没有什么错。但是，**强烈建议不要立即启动它**，因为大多数情况下，我们最终会显式或隐式地逃逸this引用。

### 3.1 替代方案

我们可以为这种情况声明一个专用方法，而不是在构造函数中启动线程：

```java
public class SafePublication implements Runnable {

    private final Thread thread;

    public SafePublication() {
        thread = new Thread(this);
    }

    @Override
    public void run() {
        System.out.println("Started...");
    }

    public void start() {
        thread.start();
    }
}
```

如上所示，我们仍然发布Thread的this引用。但是这一次，我们是在构造函数返回后启动线程：

```java
SafePublication publication = new SafePublication();
publication.start();
```

因此，对象引用在其完全构造之前不会逃逸到另一个线程。

## 4. 总结

在这个快速教程中，在简要介绍了安全发布之后，我们了解了为什么不应该在构造函数中启动线程。

有关Java中的发布和逃逸的更多详细信息，请参阅《[Java Concurrency in Practice](https://learning.oreilly.com/library/view/java-concurrency-in/0321349601/)》一书。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-3)上获得。
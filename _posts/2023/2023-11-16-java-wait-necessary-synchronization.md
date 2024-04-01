---
layout: post
title:  为什么wait()需要同步？
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在Java中，我们有[wait()/notify() API](https://www.baeldung.com/java-wait-notify)，该API是线程间同步的方法之一。为了使用此API的方法，当前线程必须拥有被调用者的监视器。

在本教程中，我们将探讨此要求有意义的原因。

## 2. wait()的工作原理

首先，我们需要简单谈谈Java中wait()的工作原理。在Java中，根据JLS，[每个对象都有一个监视器](https://docs.oracle.com/javase/specs/jvms/se6/html/Instructions2.doc9.html)。本质上，这意味着我们可以同步任何我们程序的对象。这可能不是一个好的决定，但这就是我们现在所拥有的。

有了这个，当我们调用wait()时，我们隐式地做了两件事。首先，我们将当前线程放入该对象监视器的JVM内部等待集中。第二个是，一旦线程处于等待状态，我们(或JVM，就此而言)释放该对象上的同步锁。在这里，我们需要澄清-this一词表示我们调用wait()方法的对象。

然后，当前线程只是在集合中等待，直到另一个线程对该对象调用notify()/notifyAll()。

## 3. 为什么需要获取监视器？

在上一节中，我们看到JVM所做的第二件事是释放该对象上的同步锁。为了释放它，我们显然需要首先拥有它。其原因相对简单：**wait()上的同步是为了避免丢失唤醒问题而提出的要求**，这个问题本质上代表了一种情况，即我们有一个等待线程错过了通知信号。这主要是由于线程之间的竞争条件而发生的。让我们用一个例子来模拟这个问题。

假设我们有以下Java代码：

```java
private volatile Boolean jobIsDone;

private Object lock = new Object();

public void ensureCondition() {
    while (!jobIsDone) {
        try {
            lock.wait();
        } 
        catch (InterruptedException e) {
            // ...
        }
    }
}

public void complete() {
    jobIsDone = true;
    lock.notify();
}
```

快速说明-此代码将在运行时失败并出现IllegalMonitorStateException。这是因为，在这两种方法中，我们在wait()/notification()调用之前都不会请求锁对象监视器。因此，此代码纯粹用于演示和学习目的。

另外，假设我们有两个线程。因此，线程B正在做有用的工作。一旦完成，线程B需要调用complete()方法来发出完成信号。我们还有另一个线程A，它正在等待B执行的作业完成，线程A通过调用EnsureCondition()方法来检查条件。由于Linux内核级别上发生的[虚假唤醒问题](https://en.wikipedia.org/wiki/Spurious_wakeup#:~:text=A%20spurious%20wakeup%20happens%20when,been%20awakened%20for%20no%20reason.)，对条件的检查是在循环中进行的，但这是另一个主题。

## 4. 丢失唤醒的问题

让我们逐步分解我们的示例。假设线程A调用ensureCondition()并进入while循环，它检查了一个条件，该条件似乎是false，因此它进入了try块。因为我们是在多线程环境下操作，所以另一个线程B可以同时进入complete()方法。因此，B可以在线程A调用wait()之前将volatile变量jobIsDone设置为true并调用notification()。

在这种情况下，如果线程B永远不会再次进入complete()，线程A将永远等待，因此，与其关联的所有资源也将永远存在。如果线程A碰巧持有另一个锁，这不仅会导致死锁，还会导致内存泄漏，因为从线程A栈帧可到达的对象将保持活动状态。这是因为线程A被认为是活动的，并且它可以恢复执行。因此，GC不允许对A堆栈的方法中分配的对象进行垃圾回收。

## 5. 解决方案

因此，为了避免这种情况，我们需要同步。**因此，调用者在执行之前必须拥有被调用者的监视器**。因此，让我们重写代码，考虑同步问题：

```java
private volatile Boolean jobIsDone;
private final Object lock = new Object();

public void ensureCondition() {
    synchronized (lock) {
        while (!jobIsDone) {
            try {
                lock.wait();
            } 
            catch (InterruptedException e) { 
                // ...
            }
        }
    }
}

public void complete() {
    synchronized (lock) {
        jobIsDone = true;
        lock.notify();
    }
}
```

在这里，我们只是添加了一个同步块，在调用wait()/notify() API之前，我们尝试在其中获取锁对象监视器。现在，如果B在A调用wait()之前执行complete()方法，我们可以避免丢失唤醒。这是因为只有当A还没有获取锁对象监视器时，B才能执行complete()方法。因此，A无法在执行complete()方法时检查条件。

## 6. 总结

在本文中，我们讨论了为什么Java wait()方法需要同步。我们需要被调用者监视器的所有权，以避免丢失唤醒异常。如果我们不这样做，JVM将采取快速失败方法并抛出IllegalMonitorStateException。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-5)上获得。
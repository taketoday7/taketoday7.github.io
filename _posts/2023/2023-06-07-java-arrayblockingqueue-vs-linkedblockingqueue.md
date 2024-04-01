---
layout: post
title:  ArrayBlockingQueue vs LinkedBlockingQueue
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

**Java [BlockingQueue](https://www.baeldung.com/java-blocking-queue)接口代表一个线程安全的队列**。如果队列已满，它会阻止试图将元素放入队列的线程。如果队列为空，它会阻止试图从队列中获取元素的线程。

BlockingQueue有多种[实现](https://www.baeldung.com/java-concurrent-queues#1-arrayblockingqueue)，如ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue。

在本教程中，我们将了解ArrayBlockingQueue和LinkedBlockingQueue之间的区别。

## 2. ArrayBlockingQueue

ArrayBlockingqueue是一个有界队列，它在内部使用一个数组，我们可以在创建实例时指定数组的大小。

下面的代码片段显示了我们如何创建ArrayBlockingQueue的对象，我们指定内部数组的大小为10：

```java
int INIT_CAPACITY = 10;

BlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(INIT_CAPACITY, true);
```

如果我们在队列中插入超出定义容量的元素并且队列已满，则添加操作会抛出IllegalStateException。此外，如果我们将初始大小设置为小于1，我们将得到IllegalArgumentException。

**这里第二个参数表示公平策略**，我们可以选择设置公平策略来保持被阻塞的生产者和消费者线程的顺序。它允许以FIFO顺序对阻塞线程进行队列访问。因此，先进入等待状态的线程将首先获得访问队列的机会。这有助于避免线程饥饿。

## 3. LinkedBlockingQueue

LinkedBlockingQueue是BlockingQueue的可选有界实现。它由链表支持。

**我们也可以在创建其实例时指定容量。如果未指定，则将Integer.MAX_VALUE设置为容量**。

插入元素时动态创建链接节点。

让我们看看如何创建LinkedBlockingQueue：

```java
BlockingQueue<String> linkedBlockingQueue = new LinkedBlockingQueue<>();
```

## 4. ArrayBlockingQueue与LinkedBlockingQueue

尽管ArrayBlockingQueue和LinkedBlockingQueue都是BlockingQueue的实现，并且以FIFO顺序存储元素，但它们之间还是存在一定的差异。现在我们来看看这些差异：

| 特征         | ArrayBlockingQueue                                           | LinkedBlockingQueue                                          |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **实现**     | 由数组支持                                                   | 使用链接节点                                                 |
| **队列大小** | 这是一个有界队列。因此，在创建时必须指定初始容量             | 不必指定大小                                                 |
| **公平策略** | 可以设置公平策略                                             | 没有选项可以设置公平策略                                     |
| **锁数**     | 它使用一个ReentrantLock。put和take操作使用同一个锁           | 它使用单独的ReentrantLock进行读写操作。这可以防止生产者和消费者线程之间的争用 |
| **内存空间** | 由于必须在其中指定初始容量，因此我们最终可以分配比所需更多的空间 | 它通常不预先分配节点。因此，它的内存占用与其大小相匹配       |

## 5. 总结

在本文中，我们了解了ArrayBlockingQueue和LinkedBlockingQueue之间的区别。ArrayBlockingQueue由数组支持，LinkedBlockingQueue由链表支持。我们还谈到了ArrayBlockingQueue中存在的公平策略的附加功能以及两个队列的锁定机制和内存占用。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-collections-2)上获得。
---
layout: post
title:  锁条带简介
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将学习如何实现细粒度同步，也称为“Lock Striping(锁条带化)”，这是一种在保持良好性能的同时处理对数据结构的并发访问的模式。

## 2. 问题

由于[HashMap](https://www.baeldung.com/java-hashmap)的非同步特性，它不是线程安全的数据结构。这意味着来自多线程环境的操作可能会导致数据不一致。

为了克服这个问题，我们可以使用Collections#synchronizedMap方法转换原始Map，也可以使用HashTable数据结构。两者都将返回Map接口的线程安全实现，但它们是以性能为代价的。

**使用单个锁对象定义对数据结构的独占访问的方法称为粗粒度同步**。

在粗粒度同步实现中，对对象的每次访问都必须由一个线程一次进行，最终导致的是所有线程顺序访问。

我们的目标是允许并发线程在数据结构上操作，同时确保线程安全。

## 3. Lock Striping

为了达到我们的目标，我们将使用Lock Striping模式。锁条带化是一种在多个存储桶或条带上发生锁定的技术，这意味着访问一个存储桶只锁定该桶，而不是锁定整个数据结构。

有几种方法可以做到这一点：

+ 首先，我们可以为每个任务使用一个锁，从而最大限度地提高任务之间的并发性-不过，这具有更高的内存占用
+ 或者，我们可以为每个任务使用单个锁，这样可以使用更少的内存，但也会影响并发性能

**为了帮助我们管理这种性能-内存权衡，Guava附带了一个名为Striped的类**。它类似于[ConcurrentHashMap](https://www.baeldung.com/java-concurrent-map)中的逻辑，但Striped类做了自己的增强，它通过使用信号量或可重入锁减少了不同任务的同步。

## 4. 快速示例

让我们通过一个简单的例子来帮助我们理解这种模式的好处。

我们将比较HashMap与ConcurrentHashMap，以及单个锁与条带锁，从而进行四个实验。

对于每个实验，我们将在底层Map上执行并发读取和写入。不同的是我们访问每个存储桶的方式。

为此，我们将创建两个类：SingleLock和StripedLock。这些是完成工作的抽象类ConcurrentAccessExperiment的具体实现。

### 4.1 依赖

由于我们要使用Guava的Striped类，因此我们需要添加guava依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

### 4.2 主要流程

ConcurrentAccessExperiment类实现了前面描述的行为：

```java
public abstract class ConcurrentAccessExperiment {

    public final Map<String, String> doWork(Map<String, String> map, int tasks, int slots) {
        CompletableFuture<?>[] requests = new CompletableFuture<?>[tasks * slots];

        for (int i = 0; i < tasks; i++) {
            requests[slots * i + 0] = CompletableFuture.supplyAsync(putSupplier(map, i));
            requests[slots * i + 1] = CompletableFuture.supplyAsync(getSupplier(map, i));
            requests[slots * i + 2] = CompletableFuture.supplyAsync(getSupplier(map, i));
            requests[slots * i + 3] = CompletableFuture.supplyAsync(getSupplier(map, i));
        }
        CompletableFuture.allOf(requests).join();

        return map;
    }

    protected abstract Supplier<?> putSupplier(Map<String, String> map, int key);

    protected abstract Supplier<?> getSupplier(Map<String, String> map, int key);
}
```

请务必注意，由于我们的测试受CPU限制，所以我们将存储桶的数量限制为可用处理器的倍数。

### 4.3 使用ReentrantLock进行并发访问

现在我们将为我们的异步任务实现方法。

我们的SingleLock类使用ReentrantLock为整个数据结构定义了单个锁：

```java
public class SingleLock extends ConcurrentAccessExperiment {
    ReentrantLock lock;

    public SingleLock() {
        lock = new ReentrantLock();
    }

    protected Supplier<?> putSupplier(Map<String, String> map, int key) {
        return (() -> {
            lock.lock();
            try {
                return map.put("key" + key, "value" + key);
            } finally {
                lock.unlock();
            }
        });
    }

    protected Supplier<?> getSupplier(Map<String, String> map, int key) {
        return (() -> {
            lock.lock();
            try {
                return map.get("key" + key);
            } finally {
                lock.unlock();
            }
        });
    }
}
```

### 4.4 使用Striped进行并发访问

然后，StripedLock类为每个存储桶定义一个条带锁：

```java
public class StripedLock extends ConcurrentAccessExperiment {
    Striped<Lock> stripedLock;

    public StripedLock(int buckets) {
        stripedLock = Striped.lock(buckets);
    }

    protected Supplier<?> putSupplier(Map<String, String> map, int key) {
        return (() -> {
            int bucket = key % stripedLock.size();
            Lock lock = stripedLock.get(bucket);
            lock.lock();
            try {
                return map.put("key" + key, "value" + key);
            } finally {
                lock.unlock();
            }
        });
    }

    protected Supplier<?> getSupplier(Map<String, String> map, int key) {
        return (() -> {
            int bucket = key % stripedLock.size();
            Lock lock = stripedLock.get(bucket);
            lock.lock();
            try {
                return map.get("key" + key);
            } finally {
                lock.unlock();
            }
        });
    }
}
```

那么，哪种策略表现更好？

## 5. 测试结果

让我们使用[JMH](https://www.baeldung.com/java-microbenchmark-harness)(Java Microbenchmark Harness)来找出答案，可以通过教程末尾的源代码链接找到基准测试。

运行我们的基准测试，我们能够看到类似于以下内容的内容(请注意，吞吐量越高越好)：

```shell
Benchmark                                                Mode  Cnt  Score   Error   Units
ConcurrentAccessBenchmark.singleLockConcurrentHashMap   thrpt   10  0,059 ± 0,006  ops/ms
ConcurrentAccessBenchmark.singleLockHashMap             thrpt   10  0,061 ± 0,005  ops/ms
ConcurrentAccessBenchmark.stripedLockConcurrentHashMap  thrpt   10  0,065 ± 0,009  ops/ms
ConcurrentAccessBenchmark.stripedLockHashMap            thrpt   10  0,068 ± 0,008  ops/ms
```

## 6. 总结

在本教程中，我们探讨了如何在类似Map的结构中使用Lock Striping实现更好性能的不同方法。我们创建了一个基准测试来比较几个实现的结果。

从我们的基准测试结果中，我们可以了解不同的并发策略如何显著影响整个过程。Striped Lock模式带来了相当大的改进，因为它在HashMap和ConcurrentHashMap上都获得了约10%的额外分数。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-collections-2)上获得。
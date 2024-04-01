---
layout: post
title:  Java中的不可变Set
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，**我们将介绍在Java中构建不可变Set的不同方法**。

但首先，让我们了解不可变Set并看看我们为什么需要它。

## 2. 什么是不可变集合？

通常，**[不可变对象](https://www.baeldung.com/java-immutable-object)一旦创建就不能改变其内部状态**，这使得它默认是线程安全的。同样的逻辑适用于不可变集合。

假设我们有一个带有一些值的[HashSet](https://www.baeldung.com/java-hashset)实例，使其不可变将创建我们集合的“只读”版本。因此，**任何修改其状态的尝试都将抛出UnsupportedOperationException**。

那么，我们为什么需要它？

当然，不可变集合最常见的用例是多线程环境。因此，我们可以跨线程共享不可变数据而不用担心同步。

同时，有一点需要牢记：**不变性只与集合有关，而与它的元素无关**。此外，我们可以毫无问题地修改集合元素的实例引用。

## 3. 在核心Java中创建不可变集合

**只要掌握了核心Java类，我们就可以使用Collections.unmodifiableSet()方法来包装原始Set**。

首先，让我们创建一个简单的[HashSet](https://www.baeldung.com/java-hashset)实例并使用字符串值对其进行初始化：

```java
Set<String> set = new HashSet<>();
set.add("Canada");
set.add("USA");
```

接下来，让我们使用Collections.unmodifiableSet()：

```java
Set<String> unmodifiableSet = Collections.unmodifiableSet(set);
```

最后，为了确保我们的unmodifiableSet实例是不可变的，让我们创建一个简单的测试用例：

```java
@Test(expected = UnsupportedOperationException.class)
public void testUnmodifiableSet() {
    // create and initialize the set instance

    Set<String> unmodifiableSet = Collections.unmodifiableSet(set);
    unmodifiableSet.add("Costa Rica");
}
```

正如我们预期的那样，测试将成功运行。此外，add()操作在unmodifiableSet实例上被禁止，并且会抛出UnsupportedOperationException。

现在，让我们通过向初始set实例添加相同的值来更改它：

```java
set.add("Costa Rica");
```

这样，我们间接修改了不可修改的集合。因此，当我们打印unmodifiableSet实例时：

```text
[Canada, USA, Costa Rica]
```

正如我们所看到的，“Costa Rica”元素也存在于不可修改的集合中。

## 4. 在Java 9中创建不可变集合

从Java 9开始，Set.of(elements)静态工厂方法可用于创建不可变集合：

```java
Set<String> immutable = Set.of("Canada", "USA");
```

## 5. 在Guava中创建不可变集合

**另一种构造不可变集合的方法是使用Guava的ImmutableSet类**。它将现有数据复制到一个新的不可变实例中。因此，当我们改变原始Set时，ImmutableSet中的数据不会改变。

与核心Java实现一样，任何修改创建的不可变实例的尝试都将抛出UnsupportedOperationException。

现在，让我们探索创建不可变实例的不同方法。

### 5.1 ImmutableSet.copyOf

简单地说，ImmutableSet.copyOf()方法返回集合中所有元素的副本：

```java
Set<String> immutable = ImmutableSet.copyOf(set);
```

因此，在更改初始集合后，不可变实例将保持不变：

```text
[Canada, USA]
```

### 5.2 使用ImmutableSet.of()

类似地，使用ImmutableSet.of()方法我们可以立即创建一个具有给定值的不可变集合：

```java
Set<String> immutable = ImmutableSet.of("Canada", "USA");
```

当我们不指定任何元素时，ImmutableSet.of()将返回一个空的不可变集合。

这可以与Java 9的Set.of()进行比较。

## 6.  总结

在这篇简短的文章中，我们讨论了Java语言中的不可变集合。此外，我们展示了如何使用核心Java、Java 9和Guava库的Collections API创建不可变集合。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-set-1)上获得。
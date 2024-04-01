---
layout: post
title:  从Java集合中删除元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本快速教程中，**我们将讨论四种不同的方法来从Java集合中删除与特定谓词匹配的元素**。

## 2. 定义我们的集合

首先，我们将说明两种改变原始数据结构的方法。然后我们讨论另外两个选项，它们不是删除元素，而是创建一个不包含它们的原始集合的副本。

让我们在整个示例中使用以下集合来演示我们如何使用不同的方法获得相同的结果：

```java
Collection<String> names = new ArrayList<>();
names.add("John");
names.add("Ana");
names.add("Mary");
names.add("Anthony");
names.add("Mark");
```

## 3. 使用Iterator移除元素

**Java的[Iterator](https://www.baeldung.com/java-iterator)使我们能够遍历和删除[Collection](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)中的每个单独元素**。

为此，我们首先需要使用[iterator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#iterator())方法检索其元素的迭代器。之后，我们可以在[next](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html#next())的帮助下访问每个元素，并使用[remove](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html#remove())删除它们：

```java
Iterator<String> i = names.iterator();

while(i.hasNext()) {
    String e = i.next();
    if (e.startsWith("A")) {
        i.remove();
    }
}
```

尽管它很简单，但我们应该考虑一些注意事项：

-   **根据集合的不同，我们可能会遇到[ConcurrentModificationException](https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator)异常**
-   我们需要先遍历元素，然后才能删除它们
-   **根据集合的不同，remove的行为可能与预期不同**。例如：ArrayList.Iterator从集合中移除元素并将后续数据向左移动，而LinkedList.Iterator只是调整指向下一个元素的指针。因此，在删除元素时，LinkedList.Iterator的性能比ArrayList.Iterator好得多

## 4. Java 8和Collection.removeIf()

Java 8在Collection接口中引入了一个新方法，**它提供了一种使用[Predicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html)删除元素的更简洁的方法**：

```java
names.removeIf(e -> e.startsWith("A"));
```

重要的是要注意，与Iterator方法相反，[removeIf](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#removeIf(java.util.function.Predicate))在LinkedList和ArrayList中的表现相似。

在Java 8中，ArrayList覆盖了默认实现(依赖于Iterator)并实现了一种不同的策略：首先，它遍历元素并标记与我们的Predicate匹配的元素；之后，它会进行第二次迭代以移除(并移动)在第一次迭代中标记的元素。

## 5. Java 8和Stream

Java 8的主要新特性之一是添加了[Stream](https://www.baeldung.com/java-8-streams-introduction)(和[Collectors](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collector.html))。有多种方法可以从源创建Stream。但是，影响Stream实例的大多数操作不会改变其源，相反，API侧重于创建源的副本并执行我们可能需要的任何操作。

让我们来看看我们如何使用Stream和Collectors来查找/过滤匹配和不匹配我们的Predicate的元素。

### 5.1 使用流删除元素

**使用Stream删除或过滤元素非常简单**，我们只需要使用我们的Collection创建一个Stream实例，使用我们的Predicate调用[filter](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#filter(java.util.function.Predicate))，然后在Collectors的帮助下[收集](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#collect(java.util.stream.Collector))结果：

```java
Collection<String> filteredCollection = names
    .stream()
    .filter(e -> !e.startsWith("A"))
    .collect(Collectors.toList());
```

与以前的方法相比，流式处理的侵入性更小，它促进隔离并允许从同一源创建多个副本。但是，我们应该记住，它也会增加应用程序使用的内存。

### 5.2 Collectors.partitioningBy

将Stream.filter和Collectors结合起来非常方便，尽管**我们可能会遇到同时需要匹配和不匹配元素的情况**。在这种情况下，我们可以利用[Collectors.partitioningBy](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#partitioningBy(java.util.function.Predicate,java.util.stream.Collector))：

```java
Map<Boolean, List<String>> classifiedElements = names
    .stream()
    .collect(Collectors.partitioningBy((String e) -> !e.startsWith("A")));

String matching = String.join(",", classifiedElements.get(true));
String nonMatching = String.join(",", classifiedElements.get(false));
```

此方法返回一个仅包含两个键true和false的Map，每个键分别指向包含匹配元素和非匹配元素的列表。

## 6. 总结

在本文中，我们介绍了一些从集合中删除元素的方法及其一些注意事项。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-2)上获得。
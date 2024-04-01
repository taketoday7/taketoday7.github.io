---
layout: post
title:  在Java中迭代一个Set
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

遍历元素是我们可以对集合执行的最基本的操作之一。

在本教程中，我们将了解如何迭代Set的元素，以及它与List或数组上的类似任务有何不同。

## 2. 访问Set中的元素

与List和许多其他集合不同，Set不是有序的。它们的元素没有索引，并且根据实现的不同，它们可能无法保持顺序。

这意味着我们不能通过编号访问集合中的特定元素。因此，我们不能使用典型的for循环或任何其他基于索引的方法。

### 2.1 Iterator

迭代集合的最基本和接近原始的方法是调用每个Set公开的iterator方法：

```java
Set<String> names = Sets.newHashSet("Tom", "Jane", "Karen");
Iterator<String> namesIterator = names.iterator();
```

**然后，我们可以使用获得的迭代器逐个获取该Set的元素**。最具标志性的方法是检查迭代器在while循环中是否有下一个元素：

```java
while(namesIterator.hasNext()) {
   System.out.println(namesIterator.next());
}
```

我们还可以使用Java 8中新增的forEachRemaining方法：

```java
namesIterator.forEachRemaining(System.out::println);
```

我们还可以混合使用这些解决方案：

```java
String firstName = namesIterator.next(); // save first name to variable
namesIterator.forEachRemaining(System.out::println); // print rest of the names
```

所有其他方法都将以某种方式在底层使用Iterator。

## 3. Stream

每个Set都公开了spliterator()方法。因此，**一个Set可以很容易地转换为Stream**：

```java
names.stream().forEach(System.out::println);
```

我们还可以利用丰富的[Stream API](https://www.baeldung.com/java-8-streams)来创建更复杂的管道。例如，让我们映射、记录然后将集合的元素归约为单个字符串：

```java
String namesJoined = names.stream()
    .map(String::toUpperCase)
    .peek(System.out::println)
    .collect(Collectors.joining());
```

## 4. 增强For循环

虽然我们不能使用简单的索引for循环来迭代Set，但我们可以使用Java 5中引入的增强for循环功能：

```java
for (String name : names) {
    System.out.println(name);
}
```

## 5. 使用索引进行迭代

### 5.1 转换为数组

Set没有索引，但我们可以人为地添加索引。一种可能的解决方案是简单地**将Set转换为一些更易于理解的数据结构，如数组**：

```java
Object[] namesArray = names.toArray();
for (int i = 0; i < namesArray.length; i++) {
    System.out.println(i + ": " + namesArray[i]);
}
```

请注意，单独转换为数组将迭代一次Set。因此，就复杂性而言，我们将迭代Set两次。如果性能至关重要，这可能是个问题。

### 5.2 使用索引压缩

另一种方法是创建一个索引并用我们的Set压缩它。虽然我们可以在纯Java中做到这一点，但有一些库提供了专门用于此的工具。

例如，我们可以使用Vavr的流：

```java
Stream.ofAll(names)
    .zipWithIndex()
    .forEach(t -> System.out.println(t._2() + ": " + t._1()));
```

## 6. 总结

在本教程中，我们研究了迭代Set实例元素的各种方法。我们探讨了迭代器、流和循环的用法，以及它们之间的区别。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-2)上获得。
---
layout: post
title:  将多个元素添加到Java ArrayList
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. ArrayList概述

在这个快速教程中，我们将展示如何向已初始化的ArrayList添加多个元素。

关于ArrayList的使用介绍，请参考[这里的这篇文章](https://www.baeldung.com/java-arraylist)。

## 2. addAll

首先，我们将介绍一种将多个元素添加到ArrayList中的简单方法。

首先，我们将使用addAll()，它以一个集合作为参数：

```java
List<Integer> anotherList = Arrays.asList(5, 12, 9, 3, 15, 88);
list.addAll(anotherList);
```

**请务必记住，第一个列表中添加的元素将引用与anotherList中的元素相同的对象**。

因此，对其中一个列表所做的每项修改都会影响两个列表。

## 3. Collections.addAll

Collections类专门包含对集合进行操作或返回集合的静态方法。

其中之一是addAll，它需要一个目标列表，要添加的元素可以单独指定或指定为数组。

这是一个如何将它与单个元素一起使用的示例：

```java
List<Integer> list = new ArrayList<>();
Collections.addAll(list, 1, 2, 3, 4, 5);
```

另一个用两个数组来举例说明操作：

```java
List<Integer> list = new ArrayList<>();
Integer[] otherList = new Integer[] {1, 2, 3, 4, 5};
Collections.addAll(list, otherList);
```

**与上一节中解释的方式类似，此处两个列表的内容将引用相同的对象**。

## 4. 使用Java 8

这个版本的Java通过添加新工具打开了我们的可能性。我们将在下一个示例中探讨的是Stream：

```java
List<Integer> source = ...;
List<Integer> target = ...;

source.stream()
    .forEachOrdered(target::add);
```

这种方式的主要优点是有机会使用skip和filter。在下一个示例中，我们将跳过第一个元素：

```java
source.stream()
    .skip(1)
    .forEachOrdered(target::add);
```

可以根据我们的需要过滤元素。例如，整数值：

```java
source.stream()
    .filter(i -> i > 10)
    .forEachOrdered(target::add);
```

最后，在某些情况下我们希望以空安全的方式工作。对于这些，我们可以使用Optional：

```java
Optional.ofNullable(source).ifPresent(target::addAll)
```

在上面的示例中，我们通过方法addAll将元素从source添加到source。

## 5. 总结

在本文中，我们探讨了将多个元素添加到已初始化的ArrayList的不同方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-array-list)上获得。
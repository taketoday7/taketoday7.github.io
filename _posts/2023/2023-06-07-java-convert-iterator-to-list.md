---
layout: post
title:  将Iterator转换为List
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将学习如何在Java中将[Iterator](https://www.baeldung.com/java-iterator)转换为[List](https://www.baeldung.com/java-init-list-one-line)。我们将介绍一些使用while循环、Java 8和一些常用库的示例。

我们将在所有示例中使用包含Integer的Iterator：

```java
Iterator<Integer> iterator = Arrays.asList(1, 2, 3).iterator();
```

## 2. 使用While循环

让我们从Java 8之前传统上使用的方法开始。我们将**使用while循环将Iterator转换为List**：

```java
List<Integer> actualList = new ArrayList<Integer>();
while (iterator.hasNext()) {
    actualList.add(iterator.next());
}

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 3. 使用Java 8 Iterator.forEachRemaining

在Java 8及更高版本中，我们可以使用Iterator的forEachRemaining()方法来构建我们的List。我们将List接口的add()方法作为[方法引用](https://www.baeldung.com/java-method-references)传递：

```java
List<Integer> actualList = new ArrayList<Integer>();
iterator.forEachRemaining(actualList::add);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 4. 使用Java 8 Stream API

接下来，我们将使用[Java 8 Stream API](https://www.baeldung.com/java-8-streams)将Iterator转换为List。为了使用Stream API，我们**需要先将Iterator转换为[Iterable](https://www.baeldung.com/java-iterable-to-stream)**。我们可以使用Java 8 Lambda表达式来做到这一点：

```java
Iterable<Integer> iterable = () -> iterator;
```

现在，我们可以**使用StreamSupport类的stream()和collect()方法来构建List**：

```java
List<Integer> actualList = StreamSupport
    .stream(iterable.spliterator(), false)
    .collect(Collectors.toList());

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 5. 使用Guava

**Google的[Guava库](https://www.baeldung.com/guava-lists)提供了创建可变和[不可变](https://www.baeldung.com/java-immutable-object)List的选项**，因此我们将看到这两种方法。

让我们首先使用ImmutableList.copyOf()方法创建一个不可变列表：

```java
List<Integer> actualList = ImmutableList.copyOf(iterator);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

现在，让我们使用Lists.newArrayList()方法创建一个可变列表：

```java
List<Integer> actualList = Lists.newArrayList(iterator);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 6. 使用Apache Commons

**[Apache Commons Collections](https://www.baeldung.com/apache-commons-collection-utils)库提供了处理列表的选项**。我们将使用IteratorUtils进行转换：

```java
List<Integer> actualList = IteratorUtils.toList(iterator);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 7. 总结

在本文中，我们介绍了一些将Iterator转换为List的选项。虽然还有一些其他方法可以实现这一点，但我们已经介绍了几个常用的选项。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上获得。
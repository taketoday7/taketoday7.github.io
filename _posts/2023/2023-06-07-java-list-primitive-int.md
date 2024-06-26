---
layout: post
title:  Java中的原始整数List
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，**我们将学习如何构建包含原始整数值的列表**。

我们将探索使用核心Java和外部库的解决方案。

## 2. 自动装箱

在Java中，[泛型类型](https://www.baeldung.com/java-generics)参数必须是引用类型。**这意味着我们不能做类似List<int\>的事情**。

相反，我们可以使用List<Integer\>并利用自动装箱。[自动装箱](https://www.baeldung.com/java-wrapper-classes)帮助我们使用List<Integer\>接口，就好像它包含原始int值一样。在底层，它仍然是对象的集合，而不是基本类型。

核心Java解决方案只是为了能够将基本类型与泛型[集合](https://www.baeldung.com/java-collections)一起使用而进行的调整。此外，**它还附带装箱和拆箱转换的成本**。

但是，我们可以使用Java中的其他选项和其他第三方库。让我们看看下面如何使用它们。

## 3. 使用Stream API

**很多时候，我们实际上并不需要创建一个列表**，而只是需要对其进行操作。

在这些情况下，使用Java 8的[Stream API](https://www.baeldung.com/java-8-streams-introduction)而不是完全创建一个列表可能是可行的。**IntStream类提供了一系列支持顺序聚合操作的原始int元素**。

让我们快速看一个例子：

```java
IntStream stream = IntStream.of(5, 10, 0, 2, -8);
```

IntStream.of()静态方法返回一个顺序的IntStream。

同样，我们可以从现有的整数数组创建一个IntStream：

```java
int[] primitives = {5, 10, 0, 2, -8};
IntStream stream = IntStream.of(primitives);
```

此外，我们可以应用标准的Stream API操作来迭代、过滤和聚合整数。例如，我们可以计算正整数值的平均值：

```java
OptionalDouble average = stream.filter(i -> i > 0).average();
```

最重要的是，在处理流时不使用自动装箱。

## 4. 使用Trove库

**Trove是一个高性能库，它为Java提供原始集合**。

要使用Maven设置Trove，我们需要在我们的pom.xml中包含[trov4j依赖项](https://search.maven.org/search?q=net.sf.trove4jtrove4j)：

```xml
<dependency>
    <groupId>net.sf.trove4j</groupId>
    <artifactId>trove4j</artifactId>
    <version>3.0.2</version>
</dependency>
```

使用Trove，**我们可以创建List、Map和Set**。

例如，有一个接口TIntList及其TIntArrayList实现来处理int值列表：

```java
TIntList tList = new TIntArrayList();
```

尽管TIntList不能直接实现List，但它的方法非常具有可比性。我们讨论的其他解决方案遵循类似的模式。

**使用TIntArrayList的最大好处是提高了性能和内存消耗**。不需要额外的装箱/拆箱，因为它将数据存储在int[]数组中。

## 5. 使用Fastutil

另一个使用原始类型的高性能库是[Fastutil](http://fastutil.di.unimi.it/)。让我们添加[fastutil依赖项](https://search.maven.org/search?q=it.unimi.dsifastutil)：

```xml
<dependency>
    <groupId>it.unimi.dsi</groupId>
    <artifactId>fastutil</artifactId>
    <version>8.1.0</version>
</dependency>
```

下面开始使用它：

```java
IntArrayList list = new IntArrayList();
```

**默认构造函数IntArrayList()在内部创建一个默认容量为16的基本类型数组**。同样，我们可以从现有数组初始化它：

```java
int[] primitives = new int[] {5, 10, 0, 2, -8};
IntArrayList list = new IntArrayList(primitives);
```

## 6. 使用Colt

**[Colt](https://dst.lbl.gov/ACSSoftware/colt/api/index.html)是一个开源的高性能科学和技术计算库**。cern.colt包包含可调整大小的列表，其中包含原始数据类型，例如int。

首先，让我们添加[colt依赖项](https://search.maven.org/search?q=coltcolt)：

```xml
<dependency>
    <groupId>colt</groupId>
    <artifactId>colt</artifactId>
    <version>1.2.0</version>
</dependency>
```

该库提供的原始列表是cern.colt.list.IntArrayList：

```java
cern.colt.list.IntArrayList coltList = new cern.colt.list.IntArrayList();
```

默认初始容量为10。

## 7. 使用Guava

**[Guava](https://www.baeldung.com/whats-new-in-guava-18)提供了多种拼接原始数组和集合API的方法**。com.google.common.primitives包包含所有适应原始类型的类。

例如，ImmutableIntArray类允许我们创建一个不可变的int元素列表。

假设我们有以下int值数组：

```java
int[] primitives = new int[] {5, 10, 0, 2};
```

我们可以简单地用数组创建一个列表：

```java
ImmutableIntArray list = ImmutableIntArray.builder().addAll(primitives).build();
```

此外，它还提供了一个列表API，其中包含我们期望的所有标准方法。

## 8. 总结

在这篇简短的文章中，我们展示了多种使用原始整数创建列表的方法。在我们的示例中，我们使用了Trove、Fastutil、Colt和Guava库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-3)上获得。
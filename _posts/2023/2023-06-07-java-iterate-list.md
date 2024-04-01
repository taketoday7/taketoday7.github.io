---
layout: post
title:  在Java中迭代List的方法
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

遍历列表的元素是程序中最常见的任务之一。

在本教程中，我们将回顾在Java中执行此操作的不同方法。我们将专注于按顺序遍历列表，不过[反向](https://www.baeldung.com/java-list-iterate-backwards)操作也很简单。

## 2. for循环

首先，让我们回顾一些[for循环](https://www.baeldung.com/java-loops)选项。

我们将首先为我们的示例定义国家/地区列表：

```java
List<String> countries = Arrays.asList("Germany", "Panama", "Australia");
```

### 2.1 循环基础

最常见的遍历流程控制语句是基本的for循环。

for循环定义了三种用分号分隔的语句。第一条语句是初始化语句。第二个定义终止条件。最后一条语句是更新子句。

这里我们简单地使用一个整数变量作为索引：

```java
for (int i = 0; i < countries.size(); i++) {
    System.out.println(countries.get(i));
}
```

在初始化时，我们必须声明一个整型变量来指定起点。此变量通常用作列表索引。

终止条件是一个在评估后返回布尔值的表达式。一旦这个表达式的计算结果为false，循环就结束了。

update子句用于修改索引变量的当前状态，递增或递减它直到终止点。

### 2.2 增强for循环

增强for循环是一个简单的结构，允许我们访问列表的每个元素。它类似于基本的for循环，但更具可读性和紧凑性。因此，它是遍历列表最常用的形式之一。

请注意，增强for循环比基本的for循环更简单：

```java
for (String country : countries) {
    System.out.println(country); 
}
```

## 3. 迭代器

[Iterator](https://www.baeldung.com/java-iterator)是一种设计模式，它为我们提供了一个标准接口来遍历数据结构，而不必担心内部表示。

这种遍历数据结构的方式提供了许多优点，其中我们可以强调我们的代码不依赖于实现。

因此，该结构可以是二叉树或双向链表，因为Iterator将我们从执行遍历的方式中抽象出来。这样，我们就可以轻松地替换代码中的数据结构，而不会出现令人不快的问题。

### 3.1 Iterator

在Java中，迭代器模式反映在java.util.Iterator类中，它广泛用于Java集合。Iterator中有两个关键方法hasNext()和next()。

在这里，我们将演示两者的使用：

```java
Iterator<String> countriesIterator = countries.iterator();

while(countriesIterator.hasNext()) {
    System.out.println(countriesIterator.next()); 
}
```

hasNext()方法**检查列表中是否还有剩余元素**。

next()方法**返回迭代中的下一个元素**。

### 3.2 ListIterator

ListIterator允许我们以正向或反向顺序遍历元素列表。

使用ListIterator向前遍历列表遵循类似于Iterator使用的机制。通过这种方式，我们可以使用next()方法将迭代器向前移动，并且可以使用hasNext()方法找到列表的末尾。

如我们所见，ListIterator看起来与我们之前使用的Iterator非常相似：

```java
ListIterator<String> listIterator = countries.listIterator();

while(listIterator.hasNext()) {
    System.out.println(listIterator.next());
}
```

## 4. forEach()

### 4.1 Iterable.forEach()

**从Java 8开始，我们可以使用[forEach()方法](https://www.baeldung.com/foreach-java)遍历列表的元素**。该方法定义在[Iterable](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Iterable.html)接口中，可以接收Lambda表达式作为参数。

语法非常简单：

```java
countries.forEach(System.out::println);
```

在forEach函数之前，Java中的所有迭代器都是活动的，这意味着它们涉及一个遍历数据集合的for或while循环，直到满足特定条件。

通过在Iterable接口中引入forEach作为函数，所有实现Iterable的类都添加了forEach函数。

### 4.2 Stream.forEach()

我们还可以将值集合转换为Stream，并可以访问forEach()、map()和filter()等操作。

在这里，我们将演示Stream的典型用途：

```java
countries.stream().forEach((c) -> System.out.println(c));
```

## 5. 总结

在本文中，我们演示了使用Java API迭代列表元素的不同方法。这些选项包括for循环、增强for循环、Iterator、ListIterator和forEach()方法(包含在Java 8中)。

然后我们学习了如何将forEach()方法与Stream一起使用。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-2)上获得。
---
layout: post
title:  Java中Stream到Iterable的转换
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

Java [Stream](https://www.baeldung.com/java-8-streams-introduction)API是在Java 8中引入的，它提供了处理元素序列的功能。Stream API支持对管道中的对象集合进行链接操作，以产生所需的结果。

在本教程中，我们将研究将Stream用作Iterable的方法。

## 2. Iterable和Iterator

[Iterable](https://www.baeldung.com/java-iterator-vs-iterable#iterable-interface)是自Java 1.5以来可用的接口。实现此接口的类允许类的对象成为[for-each](https://www.baeldung.com/java-for-loop#foreach)循环语句的目标。实现类不存储有关其迭代状态的任何信息，并且应生成自身的有效迭代器。

**Collection接口扩展了Iterable接口，并且Collection接口的所有具体实现(例如[ArrayList](https://www.baeldung.com/java-iterate-list)或[HashSet](https://www.baeldung.com/java-iterate-set))都通过实现Iterable的iterator()方法来生成迭代器**。

[Iterator<T\>](https://www.baeldung.com/java-iterator)接口也是Java Collection框架的一部分，从Java 1.2开始可用。实现Iterator<T\>的类必须提供遍历集合的实现，例如移动到下一个元素、检查是否还有更多元素或从集合中删除当前元素的能力：

```java
public interface Iterator<E> {
    boolean hasNext();

    E next();

    void remove();
}
```

## 3. 问题陈述

现在我们已经了解了Iterator和Iterable接口的基础知识以及它们所扮演的角色，让我们来理解问题陈述。

实现Collection接口的类本质上实现了Iterable<T\>接口。另一方面，流略有不同。**值得注意的是，Stream<T\>扩展的接口BaseStream<T\>有一个方法iterator()，但没有实现Iterable接口**。

这个限制带来了无法在Stream上使用增强的for-each循环的挑战。

我们将在接下来的部分中探讨一些解决此问题的方法，并最终探讨为什么Stream与Collection不同，它不扩展Iterable接口。

## 4. 在Stream上使用iterator()将Stream转换为Iterable

Stream接口的iterator()方法返回流元素的迭代器。这是一个终端流操作：

```java
Iterator<T> iterator();
```

但是，我们仍然无法在增强的for-each循环中使用生成的迭代器：

```java
private void streamIterator(List<String> listOfStrings) {
    Stream<String> stringStream = listOfStrings.stream();
    // this does not compile
    for (String eachString : stringStream.iterator()) {
        doSomethingOnString(eachString);
    }
}
```

正如我们之前看到的，“for-each循环”适用于Iterable而不是Iterator。为了解决这个问题，我们将迭代器转换为一个Iterable实例，然后应用所需的for-each循环。**Iterable<T\>是一个函数式接口这一事实允许我们使用lambda编写代码**：

```java
for (String eachString : (Iterable<String>) () -> stringStream.iterator()) {
    doSomethingOnString(eachString);
}
```

**我们可以使用[方法引用](https://www.baeldung.com/java-method-references)方法进行更多重构**：

```java
for (String eachString : (Iterable<String>) stringStream::iterator) {
    doSomethingOnString(eachString.toLowerCase());
}
```

在for-each循环中使用Iterable之前，也可以使用一个临时变量iterableStream来保存Iterable：

```java
Iterable<String> iterableStream = () -> stringStream.iterator();
for (String eachString : iterableStream) {
    doSomethingOnString(eachString, sentence);
}
```

## 5. 通过转换为集合在for-each循环中使用Stream

我们在上面讨论了Collection接口如何扩展Iterable接口。因此，我们可以将给定的Stream转换为集合并将结果用作Iterable：

```java
for(String eachString : stringStream.collect(Collectors.toList())) {
    doSomethingOnString(eachString);
}
```

## 6. 为什么Stream没有实现Iterable

我们看到了如何将Stream用作Iterable。List和Set等集合是在其中存储数据的数据结构，旨在在其生命周期内多次使用。这些对象被传递给不同的方法，经历多次更改，最重要的是，被多次迭代。

**另一方面，流是一次性数据结构，因此并非设计为使用for-each循环进行迭代**。流也根本不应该一遍又一遍地迭代，并在流已经关闭和操作时抛出IllegalStateException。因此，尽管Stream提供了一个iterator()方法，但它并没有扩展Iterable。

## 7. 总结

在本文中，我们研究了将Stream用作Iterable的不同方式。

我们简要讨论了Iterable和Iterator之间的差异，以及Stream<T\>不实现Iterable<T\>接口的原因。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。

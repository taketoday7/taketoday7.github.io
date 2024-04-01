---
layout: post
title:  如何将单个元素添加到流中
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在这篇快速文章中，我们将了解如何**将元素添加到Java 8 Stream**，这不像将元素添加到普通集合那样直观。

## 2. 前置

我们可以通过调用静态Stream.concat()方法轻松地将给定元素添加到Stream中：

```java
@Test
public void givenStream_whenPrependingObject_thenPrepended() {
    Stream<Integer> anStream = Stream.of(1, 2, 3, 4, 5);

    Stream<Integer> newStream = Stream.concat(Stream.of(99), anStream);

    assertEquals(newStream.findFirst().get(), (Integer) 99);
}
```

## 3. 追加

同样，要将元素附加到流的末尾，我们只需要反转参数即可。

请记住，**Stream可以表示无限序列**，因此在某些情况下你可能永远无法访问新元素：

```java
@Test
public void givenStream_whenAppendingObject_thenAppended() {
    Stream<String> anStream = Stream.of("a", "b", "c", "d", "e");

    Stream<String> newStream = Stream.concat(anStream, Stream.of("A"));

    List<String> resultList = newStream.collect(Collectors.toList());
 
    assertEquals(resultList.get(resultList.size() - 1), "A");
}
```

## 4. 在特定索引处

Java中的Stream(java.util.stream)是支持顺序和并行聚合操作的元素序列。没有向特定索引添加值的函数，因为它不是为此类事情而设计的。但是，有几种方法可以实现它。

**一种方法是使用终端操作将流的值收集到ArrayList，然后简单地使用add(int index, E element)方法**。请记住，这将为你提供所需的结果，但你**也会失去Stream的惰性，因为你需要在插入新元素之前使用它**。

另一种方法是使用Spliterator并将元素按顺序迭代到目标索引(使用Spliterator类的迭代器)。其概念是将流分成两个流(A和B)。流A从索引0开始到目标索引，流B是剩余的元素。然后将元素添加到流A，最后拼接流A和B。

在这种情况下，由于流没有被贪婪地消耗，因此它是完全惰性的；你可以在恒定时间内在给定索引处插入一个元素：

```java
private static  Stream insertInStream(Stream stream, T elem, int index) {
    Spliterator spliterator = stream.spliterator();
    Iterator iterator = Spliterators.iterator(spliterator);

    return Stream.concat(Stream.concat(Stream.generate(iterator::next)
        .limit(index), Stream.of(elem)), StreamSupport.stream(spliterator, false));
}
```

现在，让我们测试我们的代码以确保一切都按预期工作：

```java
@Test
public void givenStream_whenInsertingObject_thenInserted() {
    Stream<Double> anStream = Stream.of(1.1, 2.2, 3.3);
    Stream<Double> newStream = insertInStream(anStream, 9.9, 3);

    List<Double> resultList = newStream.collect(Collectors.toList());
 
    assertEquals(resultList.get(3), (Double) 9.9);
}
```

## 5. 总结

在这篇简短的文章中，我们了解了如何将单个元素添加到Stream，无论是在开头、结尾还是在给定位置。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-2)上获得。

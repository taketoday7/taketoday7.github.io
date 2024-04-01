---
layout: post
title:  在Java中使用空流
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在这个简短的教程中，我们将快速了解Java Stream中的中间操作和终端操作、创建空Stream的一些方法以及如何检查空Stream。

## 2. Stream和Stream操作

[Stream API](https://www.baeldung.com/java-8-streams-introduction)是Java 8的主要特性之一。Stream是我们可以对其进行迭代和执行操作的一系列元素。

**[流操作](https://www.baeldung.com/java-8-streams-introduction#operations)具体分为两种类型-中间和终端**。中间操作和终端操作可以链接在一起以形成流管道。

顾名思义，终端操作出现在流管道的末尾并返回诸如distinct()之类的结果或产生诸如forEach()之类的副作用。

另一方面，中间操作(例如sorted())将一个Stream转换为另一个Stream。因此，我们可以链接多个中间操作。

**任何终端或中间操作实际上都不会更改Stream的源，但会产生结果**。此外，中间操作以惰性方式执行；计算仅在启动终端操作后执行。

让我们看一个例子：

```java
Stream<Integer> numbers = Stream.of(1, 2, 3, 4, 5, 6);

int sumOfEvenNumbers = numbers
    .filter(number -> number%2 == 0)
    .reduce(0, Integer::sum);

Assert.assertEquals(sumOfEvenNumbers, 12);
```

在这里，我们创建了一个整数流。我们使用了一个中间操作filter()来创建另一个偶数流。最后，我们使用了终端操作reduce()来获取所有偶数的总和。

## 3. 在Java中创建空流

有时，我们可能需要将Stream作为参数传递给方法或从方法中返回。**空流对于处理NullPointerExceptions很有用**。此外，一些Stream操作，例如findFirst()、findAny()、min()和max()，会检查空Stream并相应地返回结果。

有多种[创建Stream](https://www.baeldung.com/java-8-streams#stream)的方法。因此，也有多种方法可以创建空流。

首先，**我们可以简单地使用Stream类的[empty()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#empty--)方法**：

```java
Stream<String> emptyStream = Stream.empty();
```

empty()方法返回一个空的String类型的顺序流。

**我们还可以使用of()方法创建任何类型的空Stream**。of()方法返回一个顺序有序的Stream，其中包含作为参数传递给它的元素。如果我们不传递任何参数，它会返回一个空的Stream：

```java
Stream<String> emptyStr = Stream.of();
```

同样，我们可以使用IntStream创建原始类型的Stream：

```java
IntStream intStream = IntStream.of(new int[]{});
```

Arrays类有一个方法stream()，它接收一个数组作为参数，并返回一个与参数类型相同的Stream。我们可以通过传递一个空数组作为参数使用它来创建一个空的Stream：

```java
IntStream emptyIntStream = Arrays.stream(new int[]{});
```

最后，我们可以使用List或Set等Collection的stream()方法来创建一个空的Stream，只需传递一个空集合：

```java
Stream<Integer> emptyIntStream = new ArrayList<Integer>().stream();
```

## 4. 检查空流

**我们可以通过简单地使用一种[短路终端操作](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps)(例如findFirst()或findAny())来检查空Stream**：

```java
Stream<String> emptyStream = Stream.empty();
assertTrue(emptyStream.findAny().isEmpty());
```

在这里，如果流为空，则findFirst()返回一个空的Optional。然后我们检查Optional中是否存在值。由于Stream为空，因此Optional中不存在任何值，并且它返回false。

但是，**我们必须记住，我们只能对Stream进行一次操作。如果我们重用Stream，我们可能会遇到一个IllegalStateException，说**：

```text
IllegalStateException: stream has already been operated upon or closed.
```

因此，**我们只能执行一个消耗流的操作**。如果我们想重用Stream，我们必须处理这个[IllegalStateException](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception)。

为了解决这个问题，我们可以在需要检查其是否为空时使用Supplier函数接口创建一个新的Stream：

```java
Supplier<Stream<Integer>> streamSupplier = Stream::of;

Optional<Integer> result1 = streamSupplier.get().findAny();
assertTrue(result1.isEmpty());
Optional<Integer> result2 = streamSupplier.get().findFirst();
assertTrue(result2.isEmpty());
```

在这里，我们首先定义了一个空的Stream。然后我们创建了一个Stream<Integer\>类型的streamSupplier对象。因此，每次调用get()方法都会返回一个新的空Stream对象，我们可以在该对象上安全地执行另一个Stream操作。

## 5. 总结

在这篇简短的文章中，我们看到了在Java中创建空Stream的一些方法。我们还探讨了如何检查Stream是否为空以及如何多次重用Stream，同时避免在Stream已经关闭或操作时抛出著名的IllegalStateException。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-5)上获得。

---
layout: post
title:  什么是[Ljava.lang.Object？
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将了解[Ljava.lang.Object的含义以及如何访问对象的正确值。

## 2. Java Object类

在Java中，如果我们想直接从对象打印一个值，我们可以尝试的第一件事就是调用它的toString方法：

```java
Object[] arrayOfObjects = { "John", 2, true };
assertTrue(arrayOfObjects.toString().startsWith("[Ljava.lang.Object;"));
```

**如果我们运行测试，它会成功，但通常这不是一个非常有用的结果**。

我们要做的是打印数组中的值。相反，我们得到的是[Ljava.lang.Object，类的名称，在Object.class中实现的那样：

```java
getClass().getName() + '@' + Integer.toHexString(hashCode())
```

当我们直接从对象获取类名时，我们从JVM获取内部名称及其类型，这就是为什么我们有额外的字符，如[和L，它们分别代表Array和ClassName类型。

## 3. 打印有意义的值

为了能够正确打印结果，我们可以使用[java.util](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/package-summary.html)包中的一些类。

### 3.1 Arrays

例如，我们可以使用[Arrays](https://www.baeldung.com/java-util-arrays)类中的两个方法来处理转换。

对于一维数组，我们可以使用toString方法：

```java
Object[] arrayOfObjects = { "John", 2, true };
assertEquals(Arrays.toString(arrayOfObjects), "[John, 2, true]");
```

对于更多维的数组，我们使用deepToString方法：

```java
Object[] innerArray = { "We", "Are", "Inside" };
Object[] arrayOfObjects = { "John", 2, innerArray };
assertEquals(Arrays.deepToString(arrayOfObjects), "[John, 2, [We, Are, Inside]]");
```

### 3.2 Stream

**JDK 8中的一项重要新功能是引入了[Java Stream](https://www.baeldung.com/java-8-streams-introduction)，其中包含用于处理元素序列的类**：

```java
Object[] arrayOfObjects = { "John", 2, true };
List<String> listOfString = Stream.of(arrayOfObjects)
    .map(Object::toString)
    .collect(Collectors.toList());
assertEquals(listOfString.toString(), "[John, 2, true]");
```

首先，我们使用辅助方法of创建了一个流。我们使用map将数组中的所有对象转换为字符串，然后使用collect将其收集到列表中以打印值。

## 4. 总结

在本教程中，我们了解了如何从数组中打印有意义的信息并避免使用默认的[Ljava.lang.Object。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-guides)上获得。
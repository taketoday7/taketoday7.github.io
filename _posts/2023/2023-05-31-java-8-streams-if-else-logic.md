---
layout: post
title:  如何在Java Stream中使用if/else逻辑
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在本教程中，我们将演示如何使用Java Stream实现if/else逻辑。作为本教程的一部分，我们将创建一个简单的算法来识别奇数和偶数。

## 2. forEach()中的常规if/else逻辑

首先，让我们创建一个Integer列表，然后在Integer流forEach()方法中使用传统的if/else逻辑：

```java
List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

ints.stream()
    .forEach(i -> {
        if (i.intValue() % 2 == 0) {
            Assert.assertTrue(i.intValue() % 2 == 0);
        } else {
            Assert.assertTrue(i.intValue() % 2 != 0);
        }
    });
```

**我们的forEach方法包含if-else逻辑，它使用Java模运算符验证Integer是奇数还是偶数**。

## 3. if/else逻辑与filter()

其次，让我们看一下使用Stream filter()方法的更优雅的实现：

```java
Stream<Integer> evenIntegers = ints.stream()
    .filter(i -> i.intValue() % 2 == 0);
Stream<Integer> oddIntegers = ints.stream()
    .filter(i -> i.intValue() % 2 != 0);

evenIntegers.forEach(i -> Assert.assertTrue(i.intValue() % 2 == 0));
oddIntegers.forEach(i -> Assert.assertTrue(i.intValue() % 2 != 0));
```

上面我们使用Stream filter()方法实现了if/else逻辑，**将Integer List分成两个Stream，一个用于偶数，另一个用于奇数**。

## 4. 总结

在这篇简短的文章中，我们探讨了如何创建Java Stream以及如何使用forEach()方法实现if/else逻辑。

此外，我们还学习了如何使用Stream filter方法以更优雅的方式获得类似的结果。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-3)上获得。

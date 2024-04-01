---
layout: post
title:  Java List UnsupportedOperationException
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将讨论在使用大多数List实现的某些API时可能发生的常见异常–UnsupportedOperationException。

java.util.List具有比普通数组更多的功能。例如，仅通过一个内置方法调用，就可以检查特定元素是否存在于数据结构内部。这通常就是为什么我们有时需要将数组转换为List或Collection的原因。

有关核心Java List实现(ArrayList)的介绍，请参阅[本文](https://www.baeldung.com/java-arraylist)。

## 2. UnsupportedOperationException

发生此错误的常见方式是当我们使用java.util.Arrays中的asList()方法时：

```java
public static List asList(T... a)
```

它返回：

-   **给定数组大小的固定大小列表**
-   与原始数组中的元素类型相同的元素，并且必须是Object
-    与原始数组中的元素顺序相同
-   可序列化并实现[RandomAccess](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/RandomAccess.html)的列表

由于T是一个[可变参数](https://www.baeldung.com/java-varargs)，我们可以直接传递一个数组或元素作为参数，该方法将创建一个固定大小的初始化列表：

```java
List<String> flowers = Arrays.asList("Ageratum", "Allium", "Poppy", "Catmint");
```

我们也可以传递一个实际的数组：

```java
String[] flowers = { "Ageratum", "Allium", "Poppy", "Catmint" };
List<String> flowerList = Arrays.asList(flowers);
```

**由于返回的List是一个固定大小的List，我们不能添加/删除元素**。

尝试添加更多元素会导致UnsupportedOperationException：

```java
String[] flowers = { "Ageratum", "Allium", "Poppy", "Catmint" }; 
List<String> flowerList = Arrays.asList(flowers); 
flowerList.add("Celosia");
```

此异常的根源是返回的对象未实现add()操作，因为它与java.util.ArrayList不同。

**这是一个来自java.util.Arrays的ArrayList**。

得到相同异常的另一种方法是尝试从获取的列表中删除一个元素。

另一方面，有一些方法可以在我们需要的时候获得一个可变的列表。

其中之一是直接根据asList()的结果创建ArrayList或任何类型的列表：

```java
String[] flowers = { "Ageratum", "Allium", "Poppy", "Catmint" }; 
List<String> flowerList = new ArrayList<>(Arrays.asList(flowers));
```

## 3. 总结

总之，重要的是要了解向列表添加更多元素可能不仅仅是不可变列表的问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-2)上获得。
---
layout: post
title:  Java中的数组：参考指南
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将深入探讨Java语言中的一个核心概念-数组。

我们将首先了解什么是数组，然后了解如何使用它们；总的来说，我们将介绍如何：

-   数组入门
-   读写数组元素
-   遍历数组
-   将数组转换为其他对象，如List或Stream
-   排序、搜索和组合数组

## 2. 什么是数组？

首先，我们需要定义什么是数组？根据[Java文档](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html)，**数组是包含固定数量的相同类型值的对象**。数组的元素是有索引的，这意味着我们可以使用数字(称为索引)访问它们。

我们可以将数组视为单元格的编号列表，每个单元格都是一个保存值的变量。在Java中，编号从0开始。

有原始类型数组和对象类型数组。这意味着我们可以使用int、float、boolean等数组，但也可以使用String、Object和自定义类型的数组。

## 3. 设置数组

现在数组已经明确定义，让我们深入了解它们的用法。

我们将涵盖很多主题，教我们如何使用数组。我们将学习一些基础知识，例如如何声明和初始化数组，但我们还将涵盖更高级的主题，例如排序和搜索数组。

让我们先从声明和初始化开始。

### 3.1 声明

在Java中声明数组有两种方式：

```java
int[] anArray;
```

或者：

```java
int anOtherArray[];
```

**前者比后者应用更广泛**。

### 3.2 初始化

同样，有多种方法可以初始化数组。我们将在这里看到主要的，但[本文](https://www.baeldung.com/java-initialize-array)详细介绍了数组初始化。

让我们从一个简单的方法开始：

```java
int[] anArray = new int[10];
```

通过使用此方法，**我们初始化了一个包含10个int元素的数组**。请注意，我们需要指定数组的大小。

使用此方法时，我们将每个元素初始化为其默认值，此处为0。初始化Object数组时，元素默认为null。

现在，我们将看到另一种方法，使我们可以在创建数组时直接为数组设置值：

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};
```

在这里，我们初始化了一个包含数字1到5的5元素数组。使用此方法时，我们不需要指定数组的长度，它是在大括号之间声明的元素数。

## 4. 访问元素

现在让我们看看如何访问数组的元素，我们可以通过访问数组单元位置来实现这一点。

例如，以下代码片段将向控制台打印10：

```java
anArray[0] = 10;
System.out.println(anArray[0]);
```

请注意我们如何使用索引来访问数组单元格，**括号之间的数字就是我们要访问的数组的具体位置**。

访问单元格时，如果传递的索引为负数或超出最后一个单元格，Java将抛出ArrayIndexOutOfBoundException。

**我们应该注意不要使用负索引，或者大于或等于数组大小的索引**。

## 5. 遍历数组

逐个访问元素可能很有用，但我们可能希望遍历数组。让我们看看如何实现这一目标。

第一种方法是使用for循环：

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};
for (int i = 0; i < anArray.length; i++) {
    System.out.println(anArray[i]);
}
```

这应该将数字1到5打印到控制台。**如我们所见，我们使用了length属性。这是一个公共属性，为我们提供了数组的大小**。

当然，也可以使用其他循环机制，例如while或do while。但是，对于Java集合，可以使用foreach循环遍历数组：

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};
for (int element : anArray) {
    System.out.println(element);
}
```

此示例等效于上一个示例，但我们去掉了索引样板代码。在以下情况下，**foreach循环是一个选项**：

-   我们不需要修改数组(在元素中放入另一个值不会修改数组中的元素)
-   我们不需要索引来做其他事情

## 6. 可变参数

我们已经介绍了创建和操作数组的基础知识。现在，我们将深入探讨更高级的主题，从可变参数开始。提醒一下，可变参数用于将任意数量的参数传递给方法：

```java
void varargsMethod(String... varargs) {}
```

此方法可以采用0到任意数量的字符串参数。可以在[此处](https://www.baeldung.com/java-varargs)找到有关可变参数的文章。

这里我们要知道的是，在方法体内，可变参数变成了一个数组。但是，**我们也可以直接传递一个数组作为参数**。让我们看看如何重用上面声明的示例方法：

```java
String[] anArray = new String[] {"Milk", "Tomato", "Chips"};
varargsMethod(anArray);
```

将与以下行为相同：

```java
varargsMethod("Milk", "Tomato", "Chips");
```

## 7. 将数组转换为List

数组很有用，但有时处理List会更方便。我们将在本节看到如何将数组转换为List。

我们将首先通过创建一个空列表并迭代数组以将其元素添加到列表中来实现它：

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};

List<Integer> aList = new ArrayList<>();
for (int element : anArray) {
    aList.add(element);
}
```

但还有另一种方法，更简洁一点：

```java
Integer[] anArray = new Integer[] {1, 2, 3, 4, 5};
List<Integer> aList = Arrays.asList(anArray);
```

**静态方法Arrays.asList接收一个可变参数并创建一个包含传递值的列表**。不幸的是，这种方法有一些缺点：

-   无法使用原始类型数组
-   我们不能在创建的列表中添加或删除元素，因为它会抛出UnsupportedOperationException

## 8. 将数组转换为Stream

我们现在可以将数组转换为列表，但是从Java 8开始，我们可以访问Stream API，并且我们可能希望将数组转换为Stream。Java为我们提供了[Arrays.stream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#stream(double[]))方法：

```java
String[] anArray = new String[] {"Milk", "Tomato", "Chips"};
Stream<String> aStream = Arrays.stream(anArray);
```

将Object数组传递给方法时，它将返回匹配类型的Stream(例如，对于Integer数组为Stream<Integer\>)。当传递一个原始类型流时，它将返回相应的原始流。

也可以只在数组的一个子集上创建流：

```java
Stream<String> anotherStream = Arrays.stream(anArray, 1, 3);
```

这将创建一个仅包含“Tomato”和“Chips”字符串的Stream<String\>(第一个索引包含在内，而第二个索引不包含)。

## 9. 排序数组

现在让我们看看如何对数组进行排序，即按特定顺序重新排列其元素。**Arrays类为我们提供了[sort](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(byte[]))方法**。有点像stream方法，sort有很多重载。

重载方法包含排序：

-   原始类型数组：按升序排序
-   对象数组(对象必须实现Comparable接口)：按照自然顺序排序(依赖于Comparable的compareTo方法)
-   泛型数组：根据给定的比较器排序

此外，可以仅对数组的特定部分进行排序(将开始和结束索引传递给该方法)。

排序方法背后的算法分别是快速排序和归并排序，对于原始数组和其他数组。

让我们看一些例子：

```java
int[] anArray = new int[] {5, 2, 1, 4, 8};
Arrays.sort(anArray); // anArray is now {1, 2, 4, 5, 8}

Integer[] anotherArray = new Integer[] {5, 2, 1, 4, 8};
Arrays.sort(anotherArray); // anotherArray is now {1, 2, 4, 5, 8}

String[] yetAnotherArray = new String[] {"A", "E", "Z", "B", "C"};
Arrays.sort(yetAnotherArray, 1, 3, Comparator.comparing(String::toString).reversed()); // yetAnotherArray is now {"A", "Z", "E", "B", "C"}
```

## 10. 在数组中搜索

搜索数组非常简单，我们可以遍历数组并在数组元素中搜索元素：

```java
int[] anArray = new int[] {5, 2, 1, 4, 8};
for (int i = 0; i < anArray.length; i++) {
    if (anArray[i] == 4) {
        System.out.println("Found at index " + i);
        break;
    }
}
```

在这里，我们搜索了数字4，并在索引3处找到了它。

**如果我们有一个排序数组，我们可以使用另一种解决方案**：二分查找。[这篇文章](https://www.baeldung.com/java-binary-search)解释了二分查找的原理。

**幸运的是，Java为我们提供了Arrays.binarySearch方法**，我们必须给它一个数组和一个要搜索的元素。

如果是泛型数组，我们还必须首先给它一个用于对数组进行排序的比较器。

让我们看一个二分搜索方法用法的例子：

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};
int index = Arrays.binarySearch(anArray, 4);
System.out.println("Found at index " + index);
```

当我们在第四个单元格中存储数字4时，这将返回索引3作为结果。请注意，我们使用了一个已经排序的数组。

## 11. 拼接数组

最后，让我们看看如何拼接两个数组。**这个想法是创建一个数组，其长度是要拼接的两个数组的总和**。之后我们必须**添加第一个数组的元素，然后是第二个数组的元素**：

```java
int[] anArray = new int[] {5, 2, 1, 4, 8};
int[] anotherArray = new int[] {10, 4, 9, 11, 2};

int[] resultArray = new int[anArray.length + anotherArray.length];
for (int i = 0; i < resultArray.length; i++) {
    resultArray[i] = (i < anArray.length ? anArray[i] : anotherArray[i - anArray.length]);
}
```

如我们所见，当索引仍然小于第一个数组长度时，我们从该数组中添加元素。然后我们从第二个数组添加元素。我们可以使用Arrays.setAll方法来避免编写循环：

```java
int[] anArray = new int[] {5, 2, 1, 4, 8};
int[] anotherArray = new int[] {10, 4, 9, 11, 2};

int[] resultArray = new int[anArray.length + anotherArray.length];
Arrays.setAll(resultArray, i -> (i < anArray.length ? anArray[i] : anotherArray[i - anArray.length]));
```

此方法将根据给定的函数设置所有数组元素，此函数将索引与结果相关联。

这是合并到数组的第三个选项：System.arraycopy。此方法接收源数组、源位置、目标数组、目标位置和定义要复制的元素数的int：

```java
System.arraycopy(anArray, 0, resultArray, 0, anArray.length);
System.arraycopy(anotherArray, 0, resultArray, anArray.length, anotherArray.length);
```

如我们所见，我们复制了第一个数组，然后复制第二个数组(在第一个数组的最后一个元素之后)。

## 12. 总结

在这篇详细的文章中，我们介绍了Java中数组的基本用法和一些高级用法。

我们看到Java通过[Arrays实用类](https://www.baeldung.com/java-util-arrays)提供了很多处理数组的方法。还有一些实用程序类可以在[Apache Commons](https://www.baeldung.com/apache-commons-collection-utils)或[Guava](https://www.baeldung.com/guava-collections)等库中操作数组。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-guides)上获得。
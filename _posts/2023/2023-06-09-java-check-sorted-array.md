---
layout: post
title:  在Java中检查数组是否排序
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将看到检查数组是否已排序的不同方法。

## 2. 使用循环

一种检查方法是使用for循环，我们可以一个一个地迭代数组的所有值。

### 2.1 原始数组

简而言之，我们将迭代除最后一个位置之外的所有位置。这是因为我们要将一个位置与下一个位置进行比较。

如果其中一些未排序，该方法将返回false。如果所有比较均未返回false，则表示数组已排序：

```java
boolean isSorted(int[] array) {
    for (int i = 0; i < array.length - 1; i++) {
        if (array[i] > array[i + 1])
            return false;
    }
    return true;
}
```

### 2.2 实现Comparable的对象

我们可以对实现Comparable的对象做类似的事情。我们不使用大于号，而是**使用compareTo**：

```java
boolean isSorted(Comparable[] array) {
    for (int i = 0; i < array.length - 1; ++i) {
        if (array[i].compareTo(array[i + 1]) > 0)
            return false;
    }
    return true;
}
```

### 2.3 未实现Comparable的对象

但是，如果我们的对象没有实现Comparable怎么办？在这种情况下，我们可以改为**创建一个Comparator**。

在这个例子中，我们将使用Employee对象。这是一个包含3个字段的简单POJO：

```java
public class Employee implements Serializable {
    private int id;
    private String name;
    private int age;

    // getters and setters
}
```

然后，我们需要选择想要排序的字段。在这里，让我们按age字段排序：

```java
Comparator<Employee> byAge = Comparator.comparingInt(Employee::getAge);
```

然后，我们可以更改我们的方法以也采用Comparator：

```java
boolean isSorted(Object[] array, Comparator comparator) {
    for (int i = 0; i < array.length - 1; ++i) {
        if (comparator.compare(array[i], (array[i + 1])) > 0)
            return false;
    }

    return true;
}
```

## 3. 递归

当然，我们可以改用[递归](https://www.baeldung.com/java-recursion)。这里的想法是我们将检查数组中的两个位置，然后递归直到我们检查完每个位置。

### 3.1 原始数组

在这个方法中，**我们检查最后两个位置。如果它们已排序，我们将再次调用该方法，但使用之前的位置**。如果这些位置之一未排序，该方法将返回false：

```java
boolean isSorted(int[] array, int length) {
    if (array == null || length < 2) 
        return true; 
    if (array[length - 2] > array[length - 1])
        return false;
    return isSorted(array, length - 1);
}
```

### 3.2 实现Comparable的对象

现在，让我们再看看实现Comparable的对象。我们将看到与compareTo相同的方法将起作用：

```java
boolean isSorted(Comparable[] array, int length) {
    if (array == null || length < 2) 
        return true; 
    if (array[length - 2].compareTo(array[length - 1]) > 0)
        return false;
    return isSorted(array, length - 1);
}
```

### 3.3 未实现Comparable的对象

最近，对于Employee对象，添加Comparator参数：

```java
boolean isSorted(Object[] array, Comparator comparator, int length) {
    if (array == null || length < 2)
        return true;
    if (comparator.compare(array[length - 2], array[length - 1]) > 0)
        return false;
    return isSorted(array, comparator, length - 1);
}
```

## 4. 总结

在本教程中，我们了解了如何检查数组是否已排序。我们看到了迭代和递归解决方案。

**我们的建议是使用循环解决方案。它更干净，更易于阅读**。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-sorting)上获得。
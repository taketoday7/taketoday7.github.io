---
layout: post
title:  在Java中对数组进行排序
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将讨论按升序和降序对[数组](https://www.baeldung.com/java-arrays-guide)进行排序的常用方法。

我们将研究使用Java的[Arrays](https://www.baeldung.com/java-util-arrays)类排序方法以及实现我们自己的Comparator来对数组的值进行排序。

## 2. 对象定义

在开始之前，让我们快速定义几个我们将在整个教程中排序的数组。首先，我们将创建一个整数数组和一个字符串数组：

```java
int[] numbers = new int[] { -8, 7, 5, 9, 10, -2, 3 };
String[] strings = new String[] { "learning", "java", "with", "tuyucheng" };
```

我们还创建一个Employee对象数组，其中每个Employee都有一个id和一个name属性：

```java
Employee john = new Employee(6, "John");
Employee mary = new Employee(3, "Mary");
Employee david = new Employee(4, "David");
Employee[] employees = new Employee[] { john, mary, david };
```

## 3. 升序排序

Java的[Arrays.sort](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(byte[]))方法为我们提供了一种快速简单的方法来对实现Comparable接口的原始类型数组或对象数组进行升序排序。

**对原始类型进行排序时，Arrays.sort方法使用[快速排序](https://www.baeldung.com/java-quicksort)的双轴实现。但是，在对对象进行排序时，会使用[归并排序](https://www.baeldung.com/java-merge-sort)的迭代实现**。

### 3.1 原始类型

要按升序对原始数组进行排序，我们将数组传递给sort方法：

```java
Arrays.sort(numbers);
assertArrayEquals(new int[] { -8, -2, 3, 5, 7, 9, 10 }, numbers);
```

### 3.2 实现Comparable的对象

对于实现Comparable接口的对象，与我们的原始数组一样，我们也可以简单地将数组传递给sort方法：

```java
Arrays.sort(strings);
assertArrayEquals(new String[] { "tuyucheng", "java", "learning", "with" }, strings);
```

### 3.3 未实现Comparable的对象

**对未实现Comparable接口的对象(例如我们的Employees数组)进行排序时，需要我们指定自己的比较器**。

我们可以在Java 8中很容易地做到这一点，方法是在我们的Comparator中指定我们想要比较我们的Employee对象的属性：

```java
Arrays.sort(employees, Comparator.comparing(Employee::getName));
assertArrayEquals(new Employee[] { david, john, mary }, employees);
```

在本例中，我们指定希望按Employee的name属性对其进行排序。

**我们还可以通过使用Comparator的[thenComparing](https://www.baeldung.com/java-8-comparator-comparing)方法链接比较来根据多个属性对对象进行排序**：

```java
Arrays.sort(employees, Comparator.comparing(Employee::getName).thenComparing(Employee::getId));
```

## 4. 降序排序

### 4.1 原始类型

按降序对基本数组进行排序并不像按升序排序那么简单，因为Java不支持对基本类型使用比较器。为了克服这个不足，我们有几个选择。

首先，**我们可以按升序对数组进行排序，然后对数组进行就地反转**。

其次，可以将我们的数组转换为列表，**使用Guava的[Lists.reverse()](https://www.baeldung.com/guava-lists)方法**，然后将我们的列表转换回数组。

最后，我们可以将数组转换为Stream，然后将其映射回int数组。它有一个很好的优势，那就是作为单行代码并且只使用核心Java：

```java
numbers = IntStream.of(numbers).boxed().sorted(Comparator.reverseOrder()).mapToInt(i -> i).toArray();
assertArrayEquals(new int[] { 10, 9, 7, 5, 3, -2, -8 }, numbers);
```

这样做的原因是boxed将每个int转换为Integer，它确实实现了Comparator。

### 4.2 实现Comparable的对象

按降序对实现Comparable接口的对象数组进行排序非常简单，我们需要做的就是传递一个比较器作为sort方法的第二个参数。

**在Java 8中，我们可以使用Comparator.reverseOrder()来指示我们希望数组按降序排序**：

```java
Arrays.sort(strings, Comparator.reverseOrder());
assertArrayEquals(new String[] { "with", "learning", "java", "tuyucheng" }, strings);
```

### 4.3 未实现Comparable的对象

类似于对实现Comparable的对象进行排序，我们可以通过在比较定义的末尾添加reversed()来反转自定义比较器的顺序：

```java
Arrays.sort(employees, Comparator.comparing(Employee::getName).reversed());
assertArrayEquals(new Employee[] { mary, john, david }, employees);
```

## 5. 总结

在本文中，我们讨论了如何使用Arrays.sort方法按升序和降序对原始类型数组和对象数组进行排序。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-sorting)上获得。
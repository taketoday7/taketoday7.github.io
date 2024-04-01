---
layout: post
title:  在Java中将一个List复制到另一个List
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将探讨将一个List复制到另一个List的不同方法，以及在此过程中产生的常见错误。

有关集合的使用介绍，请参阅[此处的这篇文章](https://www.baeldung.com/java-collections)。

## 2. 构造函数

复制List的一种简单方法是使用以集合作为参数的构造函数：

```java
List<Plant> copy = new ArrayList<>(list);
```

**由于我们在这里复制引用，而不是克隆对象，因此对一个元素所做的每一次修改都会影响两个列表**。

因此，最好使用构造函数来复制不可变对象：

```java
List<Integer> copy = new ArrayList<>(list);
```

Integer是一个不可变的类；它的值是在创建实例时设置的，并且永远不会改变。

因此，一个Integer引用可以被多个列表和线程共享，并且任何人都无法更改它的值。

## 3. List ConcurrentAccessException

**使用列表的一个常见问题是ConcurrentAccessException**。这通常意味着我们在尝试复制列表时正在修改列表，很可能是在另一个线程中。

要解决此问题，我们必须：

-   使用专为并发访问而设计的集合
-   适当地锁定集合以迭代它
-   找到一种避免需要复制原始集合的方法

考虑到我们最后的方法，它不是线程安全的。如果我们想用第一个选项解决我们的问题，我们可能需要使用CopyOnWriteArrayList，其中所有可变操作都是通过制作底层数组的新副本来实现的。

要了解更多信息，请参阅[这篇文章](https://www.baeldung.com/java-copy-on-write-arraylist)。

如果我们想锁定Collection，可以使用锁定原语来序列化读/写访问，例如ReentrantReadWriteLock。

## 4. AddAll

另一种复制元素的方法是使用addAll方法：

```java
List<Integer> copy = new ArrayList<>();
copy.addAll(list);
```

**无论何时使用此方法，请务必记住，与构造函数一样，两个列表的内容将引用相同的对象**。

## 5. Collections.copy

Collections类专门包含对集合进行操作或返回集合的静态方法。

其中之一是copy，它需要一个源列表和一个至少与源一样长的目标列表。

它将维护目标列表中每个复制元素的索引，例如原始元素：

```java
List<Integer> source = Arrays.asList(1,2,3);
List<Integer> dest = Arrays.asList(4,5,6);
Collections.copy(dest, source);
```

在上面的示例中，dest列表中的所有先前元素都被覆盖了，因为两个列表的大小相同。

如果目标列表大小大于源列表：

```java
List<Integer> source = Arrays.asList(1, 2, 3);
List<Integer> dest = Arrays.asList(5, 6, 7, 8, 9, 10);
Collections.copy(dest, source);
```

在这里，只有前三个元素被覆盖，而列表中的其余元素被保留。

## 6. 使用Java 8

这个版本的Java通过添加新工具扩展了我们的可能性。我们将在以下示例中探讨的是Stream：

```java
List<String> copy = list.stream()
    .collect(Collectors.toList());
```

此选项的主要优点是能够使用skip和filter。在下一个示例中，我们将跳过第一个元素：

```java
List<String> copy = list.stream()
    .skip(1)
    .collect(Collectors.toList());
```

也可以通过字符串的长度进行过滤，或者通过比较对象的属性来进行过滤：

```java
List<String> copy = list.stream()
    .filter(s -> s.length() > 10)
    .collect(Collectors.toList());
List<Flower> flowers = list.stream()
    .filter(f -> f.getPetals() > 6)
    .collect(Collectors.toList());
```

我们可能希望以空安全的方式工作：

```java
List<Flower> flowers = Optional.ofNullable(list)
    .map(List::stream)
    .orElseGet(Stream::empty)
    .collect(Collectors.toList());
```

我们也可能希望以这种方式跳过一个元素：

```java
List<Flower> flowers = Optional.ofNullable(list)
    .map(List::stream).orElseGet(Stream::empty)
    .skip(1)
    .collect(Collectors.toList());
```

## 7. 使用Java 10

最后，Java 10版本允许我们创建一个包含给定Collection元素的不可变列表：

```java
List<T> copy = List.copyOf(list);
```

唯一的条件是给定的Collection不能为null，或包含任何null元素。

## 8. 总结

在本文中，我们学习了在不同Java版本中将一个List复制到另一个List的各种方法。我们还检查了该过程中产生的一个常见错误。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-3)上获得。
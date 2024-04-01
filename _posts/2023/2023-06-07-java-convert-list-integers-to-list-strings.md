---
layout: post
title:  将整数列表转换为字符串列表
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

从JDK 5开始，Java就支持[泛型](https://www.baeldung.com/java-generics)。**Java泛型给我们带来的一个好处就是类型安全**。例如，当我们将[List](https://www.baeldung.com/tag/java-list)对象myList声明为List<Integer\>时，我们不能将类型不是Integer的元素放入myList。

但是，当我们使用泛型集合时，我们经常希望将Collection<TypeA\>转换为Collection<TypeB\>。

在本教程中，我们将以List<Integer\>为例，探讨如何将List<Integer\>转换为List<String\>。

## 2. 准备一个List<Integer\>对象作为示例

为简单起见，我们将使用单元测试断言来验证我们的转换是否按预期工作。因此，让我们首先[初始化一个整数列表](https://www.baeldung.com/java-init-list-one-line)：

```java
List<Integer> INTEGER_LIST = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
```

如上面的代码所示，我们在INTEGER_LIST对象中有7个整数。现在，我们的目标是**将INTEGER_LIST中的每个整数元素转换为String**，例如，1转化为“1”，2转化为“2”，等等。最后，结果应该等于：

```java
List<String> EXPECTED_LIST = Arrays.asList("1", "2", "3", "4", "5", "6", "7");
```

在本教程中，我们将介绍三种不同的方法：

-   使用Java 8的[Stream API](https://www.baeldung.com/java-8-streams)
-   使用Java for循环
-   使用[Guava](https://www.baeldung.com/guava-guide)库

接下来，让我们看看它们的实际效果。

## 3. 使用Java 8 Stream的map()方法

Java Stream API在Java 8及更高版本上可用。它提供了许多方便的接口，使我们可以轻松地将Collection作为流来处理。

例如，**将List<TypeA\>转换为List<TypeB\>的一种典型方法是Stream的map()方法**：

```java
theList.stream().map( .. the conversion logic.. ).collect(Collectors.toList());
```

那么接下来，让我们看看如何使用map()方法将List<Integer\>转换为List<String\>：

```java
List<String> result = INTEGER_LIST.stream().map(i -> i.toString()).collect(Collectors.toList());
assertEquals(EXPECTED_LIST, result);
```

如上面的代码示例所示，我们将[Lambda表达式](https://www.baeldung.com/java-8-lambda-expressions-tips)传递给map()，调用每个元素(Integer)的toString()方法将其转换为String。

如果我们运行它，测试就会通过。因此，Stream的map()方法完成了这个任务。

## 4. 使用for循环

我们已经看到Stream的map()方法可以解决这个问题。但是，正如我们所提到的，Stream API仅在Java 8及更高版本中可用。因此，如果我们使用的是较旧的Java版本，则需要以另一种方式解决问题。

例如，我们可以通过一个简单的for循环来进行转换：

```java
List<String> result = new ArrayList<>();
for (Integer i : INTEGER_LIST) {
    result.add(i.toString());
}

assertEquals(EXPECTED_LIST, result);
```

上面的代码表明**我们首先创建了一个新的List<String\>对象result。然后，我们在for循环中迭代List<Integer\>列表中的元素，将每个Integer元素转换为String，并将字符串添加到result列表**。

如果我们运行，测试也会通过。

## 5. 使用Guava库

当我们使用集合时，转换集合的类型是一个非常标准的操作，一些流行的外部库提供了实用方法来进行转换。

在本节中，我们将使用Guava来展示如何解决我们的问题。

首先，让我们在pom.xml中添加Guava库依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

当然，我们可以在[Maven Central](https://search.maven.org/classic/#search|gav|1|g%3A"com.google.guava"ANDa%3A"guava")上找到最新版本。

接下来，我们可以使用Guava的[Lists.transform()](https://www.baeldung.com/guava-filter-and-transform-a-collection#transform-a-collection)方法来解决我们的问题：

```java
List<String> result = Lists.transform(INTEGER_LIST, Functions.toStringFunction());
assertEquals(EXPECTED_LIST, result);
```

**transform()方法对INTEGER_LIST中的每个元素应用toStringFunction()并返回转换后的列表**。

如果我们运行它，测试也会通过。

## 6. 总结

在这篇简短的文章中，我们学习了三种将List<Integer\>转换为List<String\>的方法。如果我们的Java版本是8+，Stream API将是最直接的转换方式。否则，我们可以通过循环应用转换或使用外部库，例如Guava。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-2)上获得。
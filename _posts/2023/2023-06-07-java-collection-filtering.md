---
layout: post
title:  如何在Java中过滤集合
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将了解在Java中过滤集合的不同方法-即查找满足特定条件的所有元素。

这是几乎所有Java应用程序中都存在的一项基本任务。

因此，为此目的提供功能的库数量很多。

特别是，在本教程中，我们将介绍：

-   Java 8 Streams的filter()函数
-   Java 9 filtering收集器
-   相关的Eclipse Collections API
-   Apache的CollectionUtils filter()方法
-   Guava的Collections2 filter()方法

## 2. 使用流

自从引入Java 8以来，在我们必须处理数据集合的大多数情况下，[Stream](https://www.baeldung.com/java-8-streams-introduction)都发挥了关键作用。

因此，在大多数情况下，这是首选方法，因为它是用Java构建的，不需要额外的依赖项。

### 2.1 使用流过滤集合

为了简单起见，**在所有示例中，我们的目标是创建一个方法，该方法仅从整数值集合中检索偶数**。

因此，我们可以将用于评估每个元素的条件表达为“value % 2 == 0”。

在所有情况下，我们都必须将此条件定义为Predicate对象：

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> streamsPredicate = item -> item % 2 == 0;

    return baseCollection.stream()
        .filter(streamsPredicate)
        .collect(Collectors.toList());
}
```

重要的是要注意，**我们在本教程中分析的每个库都提供了自己的Predicate实现**，但它们仍然被定义为函数接口，因此允许我们使用Lambda函数来声明它们。

在本例中，我们使用了Java提供的预定义收集器，它将元素累积到一个列表中，但我们也可以使用其他收集器，如[上一篇文章](https://www.baeldung.com/java-8-collectors)中所讨论的那样。

### 2.2 在Java 9中对集合进行分组后过滤

Stream允许我们使用[groupingBy收集器](https://www.baeldung.com/java-groupingby-collector)聚合元素。

然而，如果我们像上一节那样进行过滤，一些元素可能会在这个收集器发挥作用之前的早期阶段被丢弃。

为此，**Java 9引入了filtering收集器，目的是在分组后处理子集合**。

按照我们的示例，假设我们要在过滤掉奇数之前根据每个Integer的位数对我们的集合进行分组：

```java
public Map<Integer, List<Integer>> findEvenNumbersAfterGrouping(Collection<Integer> baseCollection) {
    Function<Integer, Integer> getQuantityOfDigits = item -> (int) Math.log10(item) + 1;
    
    return baseCollection.stream()
        .collect(groupingBy(
            getQuantityOfDigits,
            filtering(item -> item % 2 == 0, toList())));
}
```

简而言之，如果我们使用这个收集器，我们可能会得到一个空值条目，而如果我们在分组之前进行过滤，则收集器根本不会创建这样的条目。

当然，我们会根据我们的要求选择方法。

## 3. 使用Eclipse Collections

**我们还可以利用其他一些第三方库来实现我们的目标，无论是因为我们的应用程序不支持Java 8，还是因为我们想利用Java未提供的一些强大功能**。

Eclipse Collections就是这种情况，它是一个努力跟上新范例、发展和接受所有最新Java版本引入的变化的库。

### 3.1 依赖

让我们首先将以下依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.eclipse.collections</groupId>
    <artifactId>eclipse-collections</artifactId>
    <version>9.2.0</version>
</dependency>
```

[eclipse-collections](https://mvnrepository.com/artifact/org.eclipse.collections/eclipse-collections)包括所有必要的数据结构接口和API本身。

### 3.2 使用Eclipse Collections集合过滤集合

现在让我们在它的一种数据结构上使用eclipse的过滤功能，比如它的MutableList：

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> eclipsePredicate = item -> item % 2 == 0;
 
    Collection<Integer> filteredList = Lists.mutable
        .ofAll(baseCollection)
        .select(eclipsePredicate);

    return filteredList;
}
```

作为替代方案，我们可以使用Iterate的select()静态方法来定义filteredList对象：

```java
Collection<Integer> filteredList = Iterate.select(baseCollection, eclipsePredicate);
```

## 4. 使用Apache的CollectionUtils

要开始使用Apache的CollectionUtils库，我们可以查看[这个](https://www.baeldung.com/apache-commons-collection-utils)简短的教程，其中介绍了它的用途。

但是，在本教程中，我们将重点关注其filter()实现。

### 4.1 依赖

首先，我们需要在pom.xml文件中添加以下依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
```

### 4.2 使用CollectionUtils过滤集合

下面使用CollectionUtils的方法：

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> apachePredicate = item -> item % 2 == 0;

    CollectionUtils.filter(baseCollection, apachePredicate);
    return baseCollection;
}
```

我们必须考虑到此方法通过删除与条件不匹配的每个元素来修改baseCollection。

这意味着**基础集合必须是可变的，否则会抛出异常**。

## 5. 使用Guava的Collections2

和以前一样，我们可以阅读我们之前的文章[在Guava中过滤和转换集合](https://www.baeldung.com/guava-filter-and-transform-a-collection)以获取有关此主题的更多信息。

### 5.1 依赖

让我们首先在pom.xml文件中添加[guava依赖项](https://mvnrepository.com/artifact/com.google.guava/guava)：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

### 5.2 使用Collections2过滤集合

正如我们所见，这种方法与上一节中遵循的方法非常相似：

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> guavaPredicate = item -> item % 2 == 0;
        
    return Collections2.filter(baseCollection, guavaPredicate);
}
```

同样，这里我们定义了一个特定于Guava的Predicate对象。

在这种情况下，Guava不会修改baseCollection，它会生成一个新的，因此我们可以使用不可变集合作为输入。

## 6. 总结

总之，我们已经看到在Java中有许多不同的过滤集合的方法。

尽管Stream通常是首选方法，但最好了解并牢记其他库提供的功能。

特别是如果我们需要支持旧的Java版本。但是，如果是这种情况，我们需要记住整个教程中使用的最新Java特性，例如lambda应该用匿名类代替。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-2)上获得。
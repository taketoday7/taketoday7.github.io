---
layout: post
title:  确定Java List中的所有元素是否相同
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将了解如何确定List中的所有元素是否相同。

我们还将使用大O表示法查看每个解决方案的时间复杂度，给出最坏的情况。

## 2. 示例

假设我们有以下3个列表：

```java
notAllEqualList = Arrays.asList("Jack", "James", "Sam", "James");
emptyList = Arrays.asList();
allEqualList = Arrays.asList("Jack", "Jack", "Jack", "Jack");
```

我们的任务是提出仅对emptyList和allEqualList返回true的不同解决方案。

## 3. 基本循环

首先，确实要使所有元素相等，它们都必须等于第一个元素。让我们在循环中利用它：

```java
public boolean verifyAllEqualUsingALoop(List<String> list) {
    for (String s : list) {
        if (!s.equals(list.get(0)))
            return false;
    }
    return true;
}
```

这很好，因为虽然时间复杂度为O(n)，但它通常可能提前退出。

## 4. HashSet

我们还可以使用[HashSet](https://www.baeldung.com/java-hashset)，因为它的所有元素都是不同的。**如果我们将List转换为HashSet并且生成的大小小于或等于1，那么我们知道列表中的所有元素都是相等的**：

```java
public boolean verifyAllEqualUsingHashSet(List<String> list) {
    return new HashSet<String>(list).size() <= 1;
}
```

将[List转换为HashSet需要O(n)](https://www.baeldung.com/java-collections-complexity)时间，而[调用size需要O(1)](https://www.baeldung.com/java-collections-complexity)。因此，我们的总时间复杂度仍然为O(n)。

## 5. Collection API

另一种解决方案是使用[Collections API](https://www.baeldung.com/java-collections)的frequency(Collection c, Object o)方法。**此方法返回Collection c中与Object o匹配的元素数**。

因此，如果frequency结果等于列表的大小，我们就知道所有元素都相等：

```java
public boolean verifyAllEqualUsingFrequency(List<String> list) {
    return list.isEmpty() || Collections.frequency(list, list.get(0)) == list.size();
}
```

与之前的解决方案类似，时间复杂度为O(n)，因为在内部，Collections.frequency()使用基本循环。

## 6. Stream

[Java 8中的Stream API](https://www.baeldung.com/java-8-streams)为我们提供了更多的替代方法来检测列表中的所有元素是否相等。

### 6.1 distinct()

让我们看一个使用distinct()方法的特定解决方案。

为了验证列表中的所有元素是否相等，**我们计算其流中的不同元素**：

```java
public boolean verifyAllEqualUsingStream(List<String> list) {
    return list.stream()
        .distinct()
        .count() <= 1;
}
```

如果此流的计数小于或等于1，则所有元素都相等，我们返回true。

操作的总成本为O(n)，这是遍历所有流元素所花费的时间。

### 6.2 allMatch()

Stream API的allMatch()方法提供了一个完美的解决方案来确定此流的所有元素是否与提供的谓词匹配：

```java
public boolean verifyAllEqualAnotherUsingStream(List<String> list) {
    return list.isEmpty() || list.stream()
        .allMatch(list.get(0)::equals);
}
```

与前面使用Stream的示例类似，此示例具有O(n)时间复杂度，即遍历整个流的时间。

## 7. 第三方库

如果我们停留在早期版本的Java上并且无法使用Stream API，**我们可以使用第三方库，例如Google Guava和Apache Commons**。

在这里，我们有两个非常相似的解决方案，遍历元素列表并将其与第一个元素匹配。因此，我们可以很容易地计算出时间复杂度为O(n)。

### 7.1 Maven依赖项

要使用其中任何一个，我们可以分别将[guava](https://search.maven.org/classic/#search|ga|1|g%3A"com.google.guava"a%3A"guava")或[commons-collections4](https://search.maven.org/classic/#search|ga|1|a%3A"commons-collections4"g%3A"org.apache.commons")添加到我们的项目中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

### 7.2 Google Guava

在[Google Guava](https://www.baeldung.com/guava-filter-and-transform-a-collection)中，如果列表中的所有元素都满足谓词，则静态方法[Iterables.all()](https://www.baeldung.com/guava-filter-and-transform-a-collection)返回true：

```java
public boolean verifyAllEqualUsingGuava(List<String> list) {
    return Iterables.all(list, new Predicate<String>() {
        public boolean apply(String s) {
            return s.equals(list.get(0));
        }
    });
}
```

### 7.3 Apache Commons

同样，Apache Commons库也提供了一个实用类[IterableUtils](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/IterableUtils.html)，其中包含一组静态实用方法来对Iterable实例进行操作。

特别是，如果列表中的所有元素都满足谓词，则静态方法IterableUtils.matchesAll()返回true：

```java
public boolean verifyAllEqualUsingApacheCommon(List<String> list) {
    return IterableUtils.matchesAll(list, new org.apache.commons.collections4.Predicate<String>() {
        public boolean evaluate(String s) {
            return s.equals(list.get(0));
        }
    });
}
```

## 8. 总结

在本文中，我们从简单的Java功能开始，学习了验证List中的所有元素是否相等的不同方法，然后展示了使用Stream API和第三方库Google Guava和Apache Commons的替代方法。

我们还了解到，每个解决方案都为我们提供了相同的O(n)时间复杂度。但是，我们需要根据使用方式和使用点来选择最好的。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-3)上获得。
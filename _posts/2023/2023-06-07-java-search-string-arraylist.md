---
layout: post
title:  在ArrayList中搜索字符串
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将研究**在[ArrayList](https://www.baeldung.com/java-arraylist)中搜索String的不同方法**。我们的目的是检查ArrayList中的任何元素中是否存在特定的非空字符序列，并返回包含所有匹配元素的列表。

## 2. 基本循环

首先，让我们使用一个基本循环，使用Java的String类的contains方法来搜索给定输入字符串中的字符序列：

```java
public List<String> findUsingLoop(String search, List<String> list) {
    List<String> matches = new ArrayList<String>();

    for(String str: list) {
        if (str.contains(search)) {
            matches.add(str);
        }
    }

    return matches;
}
```

## 3. Stream

**[Java 8 Stream API](https://www.baeldung.com/java-8-streams)通过使用函数式操作为我们提供了更紧凑的解决方案**。

首先，我们将使用filter()方法在输入列表中搜索输入字符串，然后，我们将使用collect方法创建并填充包含匹配元素的列表：

```java
public List<String> findUsingStream(String search, List<String> list) {
    List<String> matchingElements = list.stream()
        .filter(str -> str.trim().contains(search))
        .collect(Collectors.toList());

    return matchingElements;
}
```

## 4. 第三方库

**如果无法使用Java 8 Stream API，我们可以看看Commons Collections、Google Guava等第三方库**。

要使用它们，我们只需要在我们的pom.xml文件中添加[Guava](https://search.maven.org/classic/#search|ga|1|g%3A"com.google.guava"a%3A"guava")、[Commons Collections](https://search.maven.org/classic/#search|ga|1|a%3A"commons-collections4"g%3A"org.apache.commons")依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

### 4.1 Commons Collections

Commons Collections为我们提供了一个方法IterableUtils.filteredIterable()，它将给定的Iterable与Predicate相匹配。


让我们调用IterableUtils.filteredIterable()，定义谓词以仅选择那些包含输入字符串的元素。然后，我们将使用IteratorUtils.toList()将Iterable转换为List：

```java
public List<String> findUsingCommonsCollection(String search, List<String> list) {
    Iterable<String> result = IterableUtils.filteredIterable(list, new Predicate<String>() {
        public boolean evaluate(String listElement) {
            return listElement.contains(search);
        }
    });

    return IteratorUtils.toList(result.iterator());
}
```

### 4.2 Google Guava

Google Guava使用Iterables.filter()方法为Apache的filteredIterable()提供了类似的解决方案。让我们使用它来过滤列表并仅返回与我们的搜索字符串匹配的元素：

```java
public List<String> findUsingGuava(String search, List<String> list) {         
    Iterable<String> result = Iterables.filter(list, Predicates.containsPattern(search));

    return Lists.newArrayList(result.iterator());
}
```

## 5. 总结

在本教程中，我们学习了在ArrayList中搜索字符串的不同方法。我们首先从一个简单的for循环开始，然后继续使用Stream API的方法。最后，我们看到了一些使用两个第三方库的示例-Google Guava和Commons Collections。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-2)上获得。
---
layout: post
title:  如何将一个流拆分为多个流
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

Java的Stream API是一个功能强大且用途广泛的数据处理工具。根据定义，流操作是对一组数据的单次迭代。

但是，有时我们希望以不同方式处理部分流并获得多组结果。

在本教程中，我们将学习如何将流拆分为多个组并独立处理它们。

## 2. 使用收集器

**[Stream](https://www.baeldung.com/java-streams)应该被操作一次并且有一个终端操作**。它可以有多个中间操作，但数据在关闭之前只能收集一次。

这意味着Stream API规范明确禁止分叉流并禁止对每个分叉进行不同的中间操作。这将导致多个终端操作。但是，我们可以在终端操作里面拆分流。这会创建一个分为两组或更多组的结果。

### 2.1 使用partitioningBy进行二进制拆分

如果我们想将一个流一分为二，我们可以使用Collectors类中的partitioningBy。它接收Predicate并返回一个Map，该Map将满足谓词的元素分组在布尔true键下，其余元素分组在false下。

假设我们有一个文章列表，其中包含有关它们应该发布到的目标站点以及它们是否应该被推荐的信息。

```java
List<Article> articles = Lists.newArrayList(
    new Article("Tuyucheng", true),
    new Article("Tuyucheng", false),
    new Article("Programming Daily", false),
    new Article("The Code", false));
```

我们将把它分成两组，一组只包含Tuyucheng文章，第二组包含其余文章：

```java
Map<Boolean, List<Article>> groupedArticles = articles.stream()
    .collect(Collectors.partitioningBy(a -> a.target.equals("Tuyucheng")));
```

让我们看看Map中的true和false键下归档了哪些文章：

```java
assertThat(groupedArticles.get(true)).containsExactly(
    new Article("Tuyucheng", true),
    new Article("Tuyucheng", false));
assertThat(groupedArticles.get(false)).containsExactly(
    new Article("Programming Daily", false),
    new Article("The Code", false));
```

### 2.2 使用groupingBy拆分

如果我们想要有更多的类别，那么我们需要使用groupingBy方法。它需要一个将每个元素分类到一个组中的函数。然后它返回一个Map，该Map将每个组分类器链接到它的元素集合。

假设我们要按目标站点对文章进行分组。返回的Map将具有包含站点名称的键和包含与给定站点关联的文章集合的值：

```java
Map<String, List<Article>> groupedArticles = articles.stream()
    .collect(Collectors.groupingBy(a -> a.target));
assertThat(groupedArticles.get("Tuyucheng")).containsExactly(
    new Article("Tuyucheng", true),
    new Article("Tuyucheng", false));
assertThat(groupedArticles.get("Programming Daily")).containsExactly(new Article("Programming Daily", false));
assertThat(groupedArticles.get("The Code")).containsExactly(new Article("The Code", false));
```

## 3. 使用teeing

从Java 12开始，我们有另一种二进制拆分选项。我们可以使用teeing收集器，**teeing将两个收集器组合成一个**。每个元素都由它们处理，然后使用提供的merge函数合并为单个返回值。

### 3.1 teeing与Predicate

**teeing收集器与Collectors类中称为filtering的另一个收集器很好地配对**。它接收一个Predicate并使用它来过滤已处理的元素，然后将它们传递给另一个收集器。

让我们将文章分为Tuyucheng组和非Tuyucheng组并计算它们。我们还将使用List构造函数作为merge函数：

```java
List<Long> countedArticles = articles.stream().collect(Collectors.teeing(
    Collectors.filtering(article -> article.target.equals("Tuyucheng"), Collectors.counting()),
    Collectors.filtering(article -> !article.target.equals("Tuyucheng"), Collectors.counting()),
    List::of));
assertThat(countedArticles.get(0)).isEqualTo(2);
assertThat(countedArticles.get(1)).isEqualTo(2);
```

### 3.2 teeing与重叠结果

此解决方案与之前的解决方案之间有一个重要区别。我们之前创建的组没有重叠，源流中的每个元素最多属于一个组。使用teeing，我们不再受此限制的约束，因为每个收集器都可能处理整个流。让我们看看如何利用它。

我们可能希望将文章分为两组，一组仅包含精选文章，另一组仅包含Tuyucheng文章。生成的文章集可能会重叠，因为一篇文章可以同时为Tuyucheng文章和精选文章。

这次我们不再计数，而是将它们收集到列表中：

```java
List<List<Article>> groupedArticles = articles.stream().collect(Collectors.teeing(
    Collectors.filtering(article -> article.target.equals("Tuyucheng"), Collectors.toList()),
    Collectors.filtering(article -> article.featured, Collectors.toList()),
    List::of));

assertThat(groupedArticles.get(0)).hasSize(2);
assertThat(groupedArticles.get(1)).hasSize(1);

assertThat(groupedArticles.get(0)).containsExactly(
    new Article("Tuyucheng", true),
    new Article("Tuyucheng", false));
assertThat(groupedArticles.get(1)).containsExactly(new Article("Tuyucheng", true));
```

## 4. 使用RxJava

虽然Java的Stream API是一个有用的工具，但有时它还不够。其他解决方案，例如[RxJava](https://www.baeldung.com/rx-java)提供的响应流，也许能够帮助我们。让我们看一个简短的例子，看看我们如何使用一个Observable和多个Subscribers来获得与我们的Stream例子相同的结果。

### 4.1 创建Observable

**首先，我们需要从我们的文章列表中创建一个Observable实例**。我们可以使用Observable类的from工厂方法：

```java
Observable<Article> observableArticles = Observable.from(articles);
```

### 4.2 过滤Observable

接下来，我们需要创建Observables来过滤文章。为此，我们将使用Observable类中的filter方法：

```java
Observable<Article> tuyuchengObservable = observableArticles.filter(
    article -> article.target.equals("Tuyucheng"));
Observable<Article> featuredObservable = observableArticles.filter(
    article -> article.featured);
```

### 4.3 创建多个Subscribers

**最后，我们需要订阅Observables并提供一个Action来描述我们想要对文章做什么**。一个真实的例子是将它们保存在数据库中或将它们发送给客户端，但我们将满足于将它们添加到列表中：

```java
List<Article> tuyuchengArticles = new ArrayList<>();
List<Article> featuredArticles = new ArrayList<>();
tuyuchengObservable.subscribe(tuyuchengArticles::add);
featuredObservable.subscribe(featuredArticles::add);
```

## 5. 总结

在本教程中，我们学习了如何将流拆分成组并分别处理它们。首先，我们查看了旧的Stream API方法：groupingBy和partitionBy。接下来，我们使用了一种更新的方法，利用Java 12中引入的teeing方法。最后，我们研究了如何使用RxJava以更大的弹性实现类似的结果。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。

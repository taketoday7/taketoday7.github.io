---
layout: post
title:  Java 8收集器toMap
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将讨论Collectors类的toMap()方法。我们将使用它来将Stream收集到Map实例中。

**对于此处涵盖的所有示例，我们将使用Book列表作为起点并将其转换为不同的Map实现**。

## 2. List转化为Map

我们将从最简单的情况开始，将List转换为Map。

下面是我们如何定义Book类：

```java
class Book {
    private String name;
    private int releaseYear;
    private String isbn;

    // getters and setters
}
```

我们将创建一个Book列表来验证我们的代码：

```java
List<Book> bookList = new ArrayList<>();
bookList.add(new Book("The Fellowship of the Ring", 1954, "0395489318"));
bookList.add(new Book("The Two Towers", 1954, "0345339711"));
bookList.add(new Book("The Return of the King", 1955, "0618129111"));
```

对于这种情况，我们将使用toMap()方法的以下重载：

```java
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper)
```

**使用toMap，我们可以指明如何获取Map的键和值的策略**：

```java
public Map<String, String> listToMap(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getIsbn, Book::getName));
}
```

我们可以很容易地验证它是否有效：

```java
@Test
public void whenConvertFromListToMap() {
    assertTrue(convertToMap.listToMap(bookList).size() == 3);
}
```

## 3. 解决键冲突

**上面的示例运行良好，但是重复键会发生什么情况呢？**

让我们想象一下，我们将每个Book的发行年份作为Map的键：

```java
public Map<Integer, Book> listToMapWithDupKeyError(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getReleaseYear, Function.identity()));
}
```

鉴于我们之前的Book列表，我们会看到一个IllegalStateException：

```java
@Test(expected = IllegalStateException.class)
public void whenMapHasDuplicateKey_without_merge_function_then_runtime_exception() {
    convertToMap.listToMapWithDupKeyError(bookList);
}
```

要解决它，我们需要使用带有附加参数的不同方法，即mergeFunction：

```java
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
    Function<? super T, ? extends U> valueMapper,
    BinaryOperator<U> mergeFunction)
```

让我们引入一个合并函数，该函数指示在发生冲突的情况下，我们保留现有条目：

```java
public Map<Integer, Book> listToMapWithDupKey(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getReleaseYear, Function.identity(), (existing, replacement) -> existing));
}
```

或者换句话说，我们得到的是first-win行为：

```java
@Test
public void whenMapHasDuplicateKeyThenMergeFunctionHandlesCollision() {
    Map<Integer, Book> booksByYear = convertToMap.listToMapWithDupKey(bookList);
    assertEquals(2, booksByYear.size());
    assertEquals("0395489318", booksByYear.get(1954).getIsbn());
}
```

## 4. 其他Map类型

默认情况下，toMap()方法将返回一个HashMap。

**但是我们可以返回不同的Map实现**：

```java
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
    Function<? super T, ? extends U> valueMapper,
    BinaryOperator<U> mergeFunction,
    Supplier<M> mapSupplier)
```

其中mapSupplier是一个函数，它返回一个带有结果的新的空Map。

### 4.1 List转化为ConcurrentMap

让我们以同样的例子为例，添加一个mapSupplier函数来返回ConcurrentHashMap：

```java
public Map<Integer, Book> listToConcurrentMap(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getReleaseYear, Function.identity(), (o1, o2) -> o1, ConcurrentHashMap::new));
}
```

并继续测试我们的代码：

```java
@Test
public void whenCreateConcurrentHashMap() {
    assertTrue(convertToMap.listToConcurrentMap(bookList) instanceof ConcurrentHashMap);
}
```

### 4.2 排序Map

最后，让我们看看如何返回排序后的Map。为此，我们将使用[TreeMap](https://www.baeldung.com/java-treemap)作为mapSupplier参数。

由于默认情况下TreeMap是根据其键的自然顺序进行排序的，因此我们不必自己明确地对Book进行排序：

```java
public TreeMap<String, Book> listToSortedMap(List<Book> books) {
    return books.stream() 
        .collect(Collectors.toMap(Book::getName, Function.identity(), (o1, o2) -> o1, TreeMap::new));
}
```

因此在我们的例子中，返回的TreeMap将按书名的字母顺序排序：

```java
@Test
public void whenMapisSorted() {
    assertTrue(convertToMap.listToSortedMap(bookList).firstKey().equals("The Fellowship of the Ring"));
}
```

## 5. 总结

在本文中，我们研究了Collectors类的toMap()方法。它允许我们从Stream创建一个新的Map。

我们还学习了如何解决键冲突和创建不同的Map实现。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上获得。
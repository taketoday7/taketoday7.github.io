---
layout: post
title:  检查Java数组是否包含值
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本文中，我们将探讨在数组中搜索指定值的不同方法。

我们还将使用[JMH](https://www.baeldung.com/java-microbenchmark-harness)(Java Microbenchmark Harness)比较这些方法的执行情况，以确定哪种方法效果最好。

## 2. 设置

对于我们的示例，我们将使用一个数组，其中包含为每个测试随机生成的字符串：

```java
String[] seedArray(int length) {
    String[] strings = new String[length];
    Random value = new Random();
    for (int i = 0; i < length; i++) {
        strings[i] = String.valueOf(value.nextInt());
    }
    return strings;
}
```

为了在每个基准测试中重用数组，我们将声明一个内部类来保存数组和计数，这样我们就可以为JMH声明它的作用域：

```java
@State(Scope.Benchmark)
public static class SearchData {
    static int count = 1000;
    static String[] strings = seedArray(1000);
}
```

## 3. 基本搜索

**搜索数组的三种常用方法是作为List、Set或使用检查每个成员直到找到匹配项的循环**。

让我们从实现每个算法的三个方法开始：

```java
boolean searchList(String[] strings, String searchString) {
    return Arrays.asList(SearchData.strings)
        .contains(searchString);
}

boolean searchSet(String[] strings, String searchString) {
    Set<String> stringSet = new HashSet<>(Arrays.asList(SearchData.strings));
    
    return stringSet.contains(searchString);
}

boolean searchLoop(String[] strings, String searchString) {
    for (String string : SearchData.strings) {
        if (string.equals(searchString))
        return true;
    }
    
    return false;
}
```

我们将使用这些类注解来告诉JMH以微秒为单位输出平均时间，并运行5次预热迭代以确保我们的测试可靠：

```java
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 5)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
```

并循环运行每个测试：

```java
@Benchmark
public void searchArrayLoop() {
    for (int i = 0; i < SearchData.count; i++) {
        searchLoop(SearchData.strings, "T");
    }
}

@Benchmark
public void searchArrayAllocNewList() {
    for (int i = 0; i < SearchData.count; i++) {
        searchList(SearchData.strings, "T");
    }
}

@Benchmark
public void searchArrayAllocNewSet() {
    for (int i = 0; i < SearchData.count; i++) {
        searchSet(SearchData.strings, "S");
    }
}
```

当我们对每种方法运行1000次搜索时，我们的结果如下所示：

```text
SearchArrayTest.searchArrayAllocNewList  avgt   20    937.851 ±  14.226  us/op
SearchArrayTest.searchArrayAllocNewSet   avgt   20  14309.122 ± 193.844  us/op
SearchArrayTest.searchArrayLoop          avgt   20    758.060 ±   9.433  us/op
```

**循环搜索比其他搜索更有效**，但这至少部分是因为我们使用集合的方式。

我们在每次调用searchList()时创建一个新的List实例，在每次调用searchSet()时创建一个新的List和一个新的HashSet。创建这些对象会产生额外的成本，而循环遍历数组则不会。

## 4. 更高效的搜索

当我们创建List和Set的单个实例然后在每次搜索中重用它们时会发生什么？

让我们试一试：

```java
public void searchArrayReuseList() {
    List asList = Arrays.asList(SearchData.strings);
    for (int i = 0; i < SearchData.count; i++) {
        asList.contains("T");
    }
}

public void searchArrayReuseSet() {
    Set asSet = new HashSet<>(Arrays.asList(SearchData.strings));
    for (int i = 0; i < SearchData.count; i++) {
        asSet.contains("T");
    }
}
```

我们将使用与上面相同的JMH注解运行这些方法，并包含简单循环的结果以供比较。

我们看到非常不同的结果：

```text
SearchArrayTest.searchArrayLoop          avgt   20    758.060 ±   9.433  us/op
SearchArrayTest.searchArrayReuseList     avgt   20    837.265 ±  11.283  us/op
SearchArrayTest.searchArrayReuseSet      avgt   20     14.030 ±   0.197  us/op
```

**搜索List的速度比以前略快，而Set减少到循环所需时间的不到1%！**

现在我们已经消除了从每次搜索中创建新集合所需的时间，这些结果是有意义的。

搜索哈希表(HashSet的底层结构)的时间复杂度为0(1)，而ArrayList的底层结构为0(n)。

## 5. 二分查找

另一种搜索数组的方法是[二分查找](https://www.baeldung.com/java-binary-search)。虽然非常有效，但二分查找需要预先对数组进行排序。

让我们对数组进行排序并尝试二分查找：

```java
@Benchmark
public void searchArrayBinarySearch() {
    Arrays.sort(SearchData.strings);
    for (int i = 0; i < SearchData.count; i++) {
        Arrays.binarySearch(SearchData.strings, "T");
    }
}
```

```text
SearchArrayTest.searchArrayBinarySearch  avgt   20     26.527 ±   0.376  us/op
```

二分搜索速度非常快，尽管效率低于HashSet：二分搜索的最坏情况性能为0(logn)，这将其性能置于数组搜索和哈希表之间。

## 6. 总结

我们已经看到了几种搜索数组的方法。

根据我们的结果，HashSet最适合搜索值列表。但是，我们需要提前创建它们并将它们存储在Set中。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-operations-basic)上获得。
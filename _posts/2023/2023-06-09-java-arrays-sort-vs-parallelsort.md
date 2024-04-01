---
layout: post
title:  Arrays.sort与Arrays.parallelSort
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

我们都使用过Arrays.sort()来对对象或原始类型数组进行排序。在JDK 8中，创建者增强了API以提供一种新方法：Arrays.parallelSort()。

在本教程中，我们将比较sort()和parallelSort()方法。

## 2. Arrays.sort()

Arrays.sort()方法对对象或原始类型数组进行排序。**该方法中使用的排序算法是[双轴快速排序](https://www.baeldung.com/algorithm-quicksort)**。换句话说，它是快速排序算法的自定义实现，以实现更好的性能。

**此方法是单线程的**，有两种变体：

-   sort(array)：将整个数组按升序排序
-   sort(array, fromIndex, toIndex)：仅对从fromIndex到toIndex的元素进行排序

让我们看一下这两种变体的示例：

```java
@Test
public void givenArrayOfIntegers_whenUsingArraysSortMethod_thenSortFullArrayInAscendingOrder() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    Arrays.sort(array);

    assertArrayEquals(expected, array);

}

@Test
public void givenArrayOfIntegers_whenUsingArraysSortWithRange_thenSortRangeOfArrayAsc() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 10, 4, 1, 2, 6, 7, 8, 9, 3, 5 };

    Arrays.sort(array, 2, 8);

    assertArrayEquals(expected, array);
}
```

让我们总结一下这种方法的优缺点：

| 优点         | 缺点         |
|:-----------|:-----------|
| 在较小的数据集上更快 | 大型数据集的性能下降 |
|            | 未利用系统的多个核心 |

## 3. Arrays.parallelSort()方法

此方法也对对象或原始类型数组进行排序。与sort()类似，它也有两个变体来对完整数组和部分数组进行排序：

```java
@Test
public void givenArrayOfIntegers_whenUsingArraysParallelSortMethod_thenSortFullArrayInAscendingOrder() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    Arrays.parallelSort(array);

    assertArrayEquals(expected, array);
}

@Test
public void givenArrayOfIntegers_whenUsingArraysParallelSortWithRange_thenSortRangeOfArrayAsc() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 10, 4, 1, 2, 6, 7, 8, 9, 3, 5 };

    Arrays.parallelSort(array, 2, 8);

    assertArrayEquals(expected, array);
}
```

parallelSort()在功能上有所不同。与sort()使用单个线程按顺序对数据进行排序不同，**它使用并行归并排序算法**。它将数组分解为子数组，这些子数组本身经过排序然后合并。

**为了执行并行任务，它使用ForkJoin池**。

但是我们要知道它只有在满足一定条件的情况下才会使用并行。如果数组大小小于或等于8192或处理器只有一个核心，则它使用顺序双轴快速排序算法。否则，它使用并行排序。

让我们总结一下使用它的优缺点：

|优点                       |缺点               |
|:-------------------------|:-----------------|
|为大型数据集提供更好的性能|较小大小的数组较慢|
|利用系统的多个核心         |                    |

## 4. 比较

现在让我们看看这两种方法在不同大小的数据集上的表现。以下数字是使用[JMH基准测试](https://www.baeldung.com/java-microbenchmark-harness)得出的，测试环境使用AMD A10 PRO 2.1Ghz四核处理器和JDK 17.0.5：

|数组大小|Arrays.sort()	|Arrays.parallelSort()|
|:-------|:-----------|:--------------------|
|1000     |0.048        |0.054                 |
|10000    |0.847        |0.425                 |
|100000   |7.570        |4.395                 |
|1000000  |65.301       |37.998                |

## 5. 总结

在这篇简短的文章中，我们了解了sort()和parallelSort()的不同之处。

根据性能结果，我们可以得出总结，当我们要对大型数据集进行排序时，parallelSort()可能是更好的选择。但是，对于较小的数组，最好使用sort()，因为它提供更好的性能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-sorting)上获得。
---
layout: post
title:  在Java中查找List中的所有重复项
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将学习在Java列表中查找重复项的不同方法。

给定一个包含重复元素的整数列表，我们将在其中找到重复元素。例如，给定输入列表\[1, 2, 3, 3, 4, 4, 5]，输出列表将为\[3, 4]。

## 2. 使用Collection查找重复项

在本节中，我们将讨论使用Collections提取列表中存在的重复元素的两种方法。

### 2.1 使用Set的contains()方法

Java中的Set不包含重复项。Set中的contains()方法仅当元素已存在于其中时才返回true。

如果contains()返回false，我们将向Set添加元素。否则，我们会将元素添加到输出列表中。因此，输出列表包含重复元素：

```java
List<Integer> listDuplicateUsingSet(List<Integer> list) {
    List<Integer> duplicates = new ArrayList<>();
    Set<Integer> set = new HashSet<>();
    for (Integer i : list) {
        if (set.contains(i)) {
            duplicates.add(i);
        } else {
            set.add(i);
        }
    }
    return duplicates;
}
```

让我们编写一个测试来检查列表重复项是否仅包含重复元素：

```java
@Test
void givenList_whenUsingSet_thenReturnDuplicateElements() {
    List<Integer> list = Arrays.asList(1, 2, 3, 3, 4, 4, 5);
    List<Integer> duplicates = listDuplicate.listDuplicateUsingSet(list);
    Assert.assertEquals(duplicates.size(), 2);
    Assert.assertEquals(duplicates.contains(3), true);
    Assert.assertEquals(duplicates.contains(4), true);
    Assert.assertEquals(duplicates.contains(1), false);
}
```

这里我们看到输出列表只包含两个元素3和4。

**这种方法对列表中的n个元素花费[O(n)时间](https://www.baeldung.com/java-collections-complexity)，对集合占用大小为n的额外空间**。

### 2.2 使用Map并存储元素的频率

我们可以使用Map来存储每个元素的频率，然后仅当元素的频率不为1时才将它们添加到输出列表：

```java
List<Integer> listDuplicateUsingMap(List<Integer> list) {
    List<Integer> duplicates = new ArrayList<>();
    Map<Integer, Integer> frequencyMap = new HashMap<>();
    for (Integer number : list) {
        frequencyMap.put(number, frequencyMap.getOrDefault(number, 0) + 1);
    }
    for (int number : frequencyMap.keySet()) {
        if (frequencyMap.get(number) != 1) {
            duplicates.add(number);
        }
    }
    return duplicates;
}
```

让我们编写一个测试来检查列表重复项是否仅包含重复元素：

```java
@Test
void givenList_whenUsingFrequencyMap_thenReturnDuplicateElements() {
    List<Integer> list = Arrays.asList(1, 2, 3, 3, 4, 4, 5);
    List<Integer> duplicates = listDuplicate.listDuplicateUsingMap(list);
    Assert.assertEquals(duplicates.size(), 2);
    Assert.assertEquals(duplicates.contains(3), true);
    Assert.assertEquals(duplicates.contains(4), true);
    Assert.assertEquals(duplicates.contains(1), false);
}
```

这里我们看到输出列表只包含两个元素，3和4。

**这种方法对列表中的n个元素花费O(n)时间，对Map占用大小为n的额外空间**。

## 3. 在Java 8中使用Stream

在本节中，我们将讨论使用Stream提取列表中存在的重复元素的三种方法。

### 3.1 使用filter()和Set.add()方法

Set.add()将指定的元素添加到此集合中(如果它不存在)。如果此集合已包含该元素，则调用保持集合不变并返回false。

在这里，我们将使用Set并将列表转换为流。将流添加到Set中，将重复的元素[过滤](https://www.baeldung.com/java-stream-filter-lambda)收集到List中：

```java
List<Integer> listDuplicateUsingFilterAndSetAdd(List<Integer> list) {
    Set<Integer> elements = new HashSet<Integer>();
    return list.stream()
        .filter(n -> !elements.add(n))
        .collect(Collectors.toList());
}
```

让我们编写一个测试来检查列表重复项是否仅包含重复元素：

```java
@Test
void givenList_whenUsingFilterAndSetAdd_thenReturnDuplicateElements() {
    List<Integer> list = Arrays.asList(1, 2, 3, 3, 4, 4, 5);
    List<Integer> duplicates = listDuplicate.listDuplicateUsingFilterAndSetAdd(list);
    Assert.assertEquals(duplicates.size(), 2);
    Assert.assertEquals(duplicates.contains(3), true);
    Assert.assertEquals(duplicates.contains(4), true);
    Assert.assertEquals(duplicates.contains(1), false);
}
```

在这里，我们看到输出元素仅包含两个元素3和4，正如预期的那样。

**这种将filter()与Set.add()结合使用的方法是查找具有O(n)时间复杂度和集合大小为n的额外空间的重复元素的最快算法**。

### 3.2 使用Collections.frequency()

Collections.frequency()返回指定集合中元素的数量，该数量等于指定值。在这里，我们将List转换为Stream并仅从Collections.frequency()中过滤掉返回值大于1的元素。

我们会将这些元素收集到Set中以避免重复，最后将Set转换为List：

```java
List<Integer> listDuplicateUsingCollectionsFrequency(List<Integer> list) {
    List<Integer> duplicates = new ArrayList<>();
    Set<Integer> set = list.stream()
        .filter(i -> Collections.frequency(list, i) > 1)
        .collect(Collectors.toSet());
    duplicates.addAll(set);
    return duplicates;
}
```

让我们编写一个测试来检查重复项是否仅包含重复元素：

```java
@Test
void givenList_whenUsingCollectionsFrequency_thenReturnDuplicateElements() {
    List<Integer> list = Arrays.asList(1, 2, 3, 3, 4, 4, 5);
    List<Integer> duplicates = listDuplicate.listDuplicateUsingCollectionsFrequency(list);
    Assert.assertEquals(duplicates.size(), 2);
    Assert.assertEquals(duplicates.contains(3), true);
    Assert.assertEquals(duplicates.contains(4), true);
    Assert.assertEquals(duplicates.contains(1), false);
}
```

正如预期的那样，输出列表仅包含两个元素，3和4。

**这种使用Collections.frequency()的方法是最慢的，因为它将每个元素与列表进行比较-Collections.frequency(list, i)，其复杂度为O(n)。所以整体复杂度为O(n * n)。它还需要一个大小为n的额外空间用于集合**。

### 3.3 使用Map和Collectors.groupingBy()

[Collectors.groupingBy()](https://www.baeldung.com/java-groupingby-collector)返回一个收集器，该收集器对输入元素实施级联“分组依据”操作。

它根据分类函数对元素进行分组，然后使用指定的下游收集器对具有给定键的关联值执行归约操作。分类函数将元素映射到某个键类型K。下游收集器对输入元素进行操作并产生D类型的结果。生成的收集器生成Map<K, D\>。

在这里，我们将使用Function.identity()作为分类函数，使用Collectors.counting()作为下游收集器。

Function.identity()返回一个始终返回其输入参数的函数。Collectors.counting()返回一个收集器，它接收计算输入元素数的元素。如果不存在任何元素，则结果为0。因此，我们将使用Collectors.groupingBy()获得元素及其频率的Map。

然后我们将这个Map的EntrySet转成一个Stream，只过滤掉大于1的元素，收集到一个Set中，避免重复。然后将Set转换为List：

```java
List<Integer> listDuplicateUsingMapAndCollectorsGroupingBy(List<Integer> list) {
    List<Integer> duplicates = new ArrayList<>();
    Set<Integer> set = list.stream()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
        .entrySet()
        .stream()
        .filter(m -> m.getValue() > 1)
        .map(Map.Entry::getKey)
        .collect(Collectors.toSet());
    duplicates.addAll(set);
    return duplicates;
}
```

让我们编写一个测试来检查列表重复项是否仅包含重复元素：

```java
@Test
void givenList_whenUsingMapAndCollectorsGroupingBy_thenReturnDuplicateElements() {
    List<Integer> list = Arrays.asList(1, 2, 3, 3, 4, 4, 5);
    List<Integer> duplicates = listDuplicate.listDuplicateUsingCollectionsFrequency(list);
    Assert.assertEquals(duplicates.size(), 2);
    Assert.assertEquals(duplicates.contains(3), true);
    Assert.assertEquals(duplicates.contains(4), true);
    Assert.assertEquals(duplicates.contains(1), false);
}
```

这里我们看到输出元素只包含两个元素3和4。

**Collectors.groupingBy()需要O(n)时间。对生成的EntrySet执行filter()操作，但复杂度仍然为O(n)，因为Map查找时间为O(1)。它还需要一个额外的空间n用于集合**。

## 4. 总结

在本文中，我们了解了在Java中从List中提取重复元素的不同方法。

我们讨论了使用Set和Map的方法及其使用Stream的相应方法。使用Stream的代码更具声明性，并且无需外部迭代器即可清楚地传达代码的意图。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-5)上获得。
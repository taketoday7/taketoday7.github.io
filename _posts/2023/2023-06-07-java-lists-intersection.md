---
layout: post
title:  Java中两个List的交集
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将学习如何检索两个[List的交集](https://www.baeldung.com/cs/list-intersection)。

与许多其他事情一样，由于在[Java 8](https://www.baeldung.com/java-streams)中引入了Stream，这变得更加容易。

## 2. 两个字符串列表的交集

让我们创建两个具有一些交集的List<String\>-两者都有一些重复的元素：

```java
List<String> list = Arrays.asList("red", "blue", "blue", "green", "red");
List<String> otherList = Arrays.asList("red", "green", "green", "yellow");
```

现在我们将**借助Stream方法确定列表的交集**：

```java
Set<String> result = list.stream()
    .distinct()
    .filter(otherList::contains)
    .collect(Collectors.toSet());

Set<String> commonElements = new HashSet(Arrays.asList("red", "green"));

Assert.assertEquals(commonElements, result);
```

首先，我们使用distinct删除重复的元素。然后，我们使用filter来选择也包含在otherList中的元素。

最后，我们使用Collector转换输出。交集应该只包含每个公共元素一次。顺序无关紧要，因此toSet是最直接的选择，但我们也可以使用toList或其他收集器方法。

有关更多详细信息，请查看我们的[Java 8收集器指南](https://www.baeldung.com/java-8-collectors)。

## 3. 自定义类列表的交集

如果我们的List不是包含String而是包含我们创建的自定义类的实例怎么办？好吧，只要我们遵循Java的约定，使用Stream方法的解决方案就可以很好地用于我们的自定义类。

**contains方法如何决定特定对象是否出现在列表中？基于equals方法。因此，我们必须覆盖equals方法并确保它根据相关属性的值比较两个对象**。

例如，如果两个矩形的宽度和高度相等，则它们是相等的。

如果我们不覆盖equals方法，我们的类将使用父类的equals实现。在最后，或者更确切地说，在继承链结束时，将执行Object类的equals方法。然后，仅当两个实例引用堆上完全相同的对象时，它们才相等。

有关equals方法的更多信息，请参阅我们关于[Java equals()和hashCode()契约](https://www.baeldung.com/java-equals-hashcode-contracts)的文章。

## 4. 总结

在这篇简短的文章中，我们了解了如何使用Stream来计算两个列表的交集。还有许多其他操作非常乏味，但如果我们知道Java Stream API的方法，它们就非常简单了。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-2)上获得。
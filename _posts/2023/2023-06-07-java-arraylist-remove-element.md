---
layout: post
title:  从ArrayList中删除元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将了解如何使用不同的技术从Java中的ArrayList中删除元素。给定一个sports列表，让我们看看如何去掉以下列表中的一些元素：

```java
List<String> sports = new ArrayList<>();
sports.add("Football");
sports.add("Basketball");
sports.add("Baseball");
sports.add("Boxing");
sports.add("Cycling");
```

## 2. ArrayList#remove

ArrayList有两种可用的方法来删除元素，**传递要删除的元素的索引**，或者**传递要删除的元素本身**(如果存在)。我们将看到这两种用法。

### 2.1 按索引删除

使用remove传递一个索引作为参数，我们可以**删除列表中指定位置的元素**，并将任何后续元素向左移动，从它们的索引中减去1。remove方法执行后会返回被移除的元素：

```java
sports.remove(1); // since index starts at 0, this will remove "Basketball"
assertEquals(4, sports.size());
assertNotEquals(sports.get(1), "Basketball");
```

### 2.2 按元素删除

另一种方法是使用此方法**从列表中删除第一次出现的元素**。正式地说，我们将删除索引最低的元素(如果存在)，如果不存在，则列表保持不变：

```java
sports.remove("Baseball");
assertEquals(4, sports.size());
assertFalse(sports.contains("Baseball"));
```

## 3. 迭代时删除

有时我们想在循环时从ArrayList中删除一个元素。由于不会产生ConcurrentModificationException，我们需要使用Iterator类来正确执行此操作。

让我们看看如何在循环中删除元素：

```java
Iterator<String> iterator = sports.iterator();
while (iterator.hasNext()) {
    if (iterator.next().equals("Boxing")) {
        iterator.remove();
    }
}
```

## 4. ArrayList#removeIf(JDK 8+)

如果我们使用JDK 8或更高版本，我们可以**利用ArrayList#removeIf删除满足给定谓词的ArrayList的所有元素**。

```java
sports.removeIf(p -> p.equals("Cycling"));
assertEquals(4, sports.size());
assertFalse(sports.contains("Cycling"));
```

最后，我们可以[使用像Apache Commons这样的第三方库](https://www.baeldung.com/java-array-remove-element)来做到这一点，如果我们想更深入，我们可以看到如何以[有效的方式删除所有特定的出现](https://www.baeldung.com/java-remove-value-from-list)。

## 5. 总结

在本教程中，我们研究了在Java中从ArrayList中删除元素的各种方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-array-list)上获得。
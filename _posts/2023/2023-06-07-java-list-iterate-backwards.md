---
layout: post
title:  通过List向后迭代
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将了解在Java中向后迭代列表的各种方法。

## 2. Java中的Iterator

[Iterator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html)是[Java集合框架](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html)中的一个接口，它允许我们迭代集合中的元素。它是在Java 1.2中作为[Enumeration](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Enumeration.html)的替代品引入的。

## 3. 使用核心Java向后迭代

### 3.1 反向for循环

最简单的实现是**使用for循环从列表的最后一个元素开始，并在到达列表开头时递减索引**：

```java
for (int i = list.size(); i-- > 0; ) {
    System.out.println(list.get(i));
}
```

### 3.2 ListIterator

**我们可以使用[ListIterator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ListIterator.html)来迭代列表中的元素**。

将列表的大小作为索引提供给ListIterator将为我们提供一个指向列表末尾的迭代器：

```java
ListIterator listIterator = list.listIterator(list.size());
```

这个迭代器现在允许我们以相反的方向遍历列表：

```java
while (listIterator.hasPrevious()) {
    System.out.println(listIterator.previous());
}
```

### 3.3 Collections.reverse()

**Java中的[Collections](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#reverse(java.util.List))类提供了一个静态方法来反转指定列表中元素的顺序**：

```java
Collections.reverse(list);
```

然后可以使用反转列表向后迭代原始元素：

```java
for (String item : list) {
    System.out.println(item);
}
```

但是，**此方法通过就地更改元素的顺序来反转实际列表，在许多情况下可能并不理想**。

## 4. 使用Apache的ReverseListIterator向后迭代

**Apache Commons Collections库有一个很好的[ReverseListIterator](https://commons.apache.org/proper/commons-collections/javadocs/api-3.2.2/org/apache/commons/collections/iterators/ReverseListIterator.html)类，它允许我们循环遍历列表中的元素而无需实际反转它**。

在开始之前，我们需要从[Maven Central](https://search.maven.org/classic/#search|gav|1|g%3A"org.apache.commons"ANDa%3A"commons-collections4")导入最新的依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

我们可以通过将原始列表作为构造函数参数传递来创建一个新的ReverseListIterator：

```java
ReverseListIterator reverseListIterator = new ReverseListIterator(list);
```

然后我们可以使用这个迭代器向后遍历列表：

```java
while (reverseListIterator.hasNext()) {
    System.out.println(reverseListIterator.next());
}
```

## 5. 使用Guava的Lists.reverse()向后迭代

同样，**Google Guava库也在其[Lists](https://google.github.io/guava/releases/snapshot-jre/api/docs/com/google/common/collect/Lists.html#reverse-java.util.List-)类中提供了一个静态reverse()方法，该方法返回所提供列表的反向视图**。

可以在[Maven Central](https://search.maven.org/classic/#search|gav|1|g%3A"com.google.guava"ANDa%3A"guava")上找到最新的Guava版本：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

在Lists类上调用静态方法reverse()以相反的方式为我们提供列表：

```java
List<String> reversedList = Lists.reverse(list);
```

然后可以使用反转列表向后迭代原始列表：

```java
for (String item : reversedList) {
    System.out.println(item);
}
```

**此方法返回一个新列表，其中的元素顺序与原始列表相反**。

## 6.  总结

在本文中，我们研究了在Java中向后遍历列表的不同方法。我们浏览了一些使用核心Java以及流行的第三方库的示例。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-1)上获得。
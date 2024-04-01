---
layout: post
title:  扩展数组的长度
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将介绍扩展Java[数组](https://www.baeldung.com/java-arrays-guide)的不同方式。

## 2. 使用Arrays.copyOf

首先，让我们看看Arrays.copyOf。我们将复制数组并向副本添加一个新元素：

```java
public Integer[] addElementUsingArraysCopyOf(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = Arrays.copyOf(srcArray, srcArray.length + 1);
    destArray[destArray.length - 1] = elementToAdd;
    return destArray;
}
```

Arrays.copyOf的工作方式是它**获取srcArray并将length参数中指定的元素数量复制到它内部创建的新数组中**。新数组的大小是我们提供的参数。

需要注意的一点是，当length参数大于源数组的大小时，Arrays.copyOf将用null填充目标数组中的额外元素。

**根据数据类型，填充行为会有所不同**。例如，如果我们使用原始数据类型代替Integer，那么额外的元素将用0填充。在char的情况下，Arrays.copyOf将用null填充额外的元素，在boolean的情况下，用false填充。

## 3. 使用ArrayList

我们将要研究的下一个方法是使用ArrayList。

我们首先**将数组转换为ArrayList**，然后添加元素。然后我们**将ArrayList转换回数组**：

```java
public Integer[] addElementUsingArrayList(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = new Integer[srcArray.length + 1];
    ArrayList<Integer> arrayList = new ArrayList<>(Arrays.asList(srcArray));
    arrayList.add(elementToAdd);
    return arrayList.toArray(destArray);
}
```

请注意，我们通过将srcArray转换为Collection来传递它。srcArray将填充ArrayList中的底层数组。

另外，要注意的另一件事是我们已将目标数组作为参数传递给toArray。**此方法会将底层数组复制到destArray**。

## 4. 使用System.arraycopy

最后，我们来看看System.arraycopy，它与Arrays.copyOf非常相似：

```java
public Integer[] addElementUsingSystemArrayCopy(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = new Integer[srcArray.length + 1];
    System.arraycopy(srcArray, 0, destArray, 0, srcArray.length);
    destArray[destArray.length - 1] = elementToAdd;
    return destArray;
}
```

一个有趣的事实是**Arrays.copyOf内部使用了这个方法**。

在这里我们可以注意到，我们**将元素从srcArray复制到destArray，然后将新元素添加到destArray**。

## 5. 性能

所有解决方案中的一个共同点是我们必须以某种方式创建一个新数组，其原因在于数组在内存中的分配方式。数组包含一个连续的内存块用于超快速查找，这就是为什么我们不能简单地调整它的大小。

这当然会影响性能，尤其是对于大型数组。这就是[ArrayList](https://www.baeldung.com/java-arraylist)过度分配的原因，有效地减少了JVM需要重新分配内存的次数。

但是，如果我们进行大量插入，数组可能不是正确的数据结构，我们应该考虑[LinkedList](https://www.baeldung.com/java-linkedlist)。

## 6. 总结

在本文中，我们探讨了将元素添加到数组末尾的不同方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-operations-basic)上获得。
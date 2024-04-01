---
layout: post
title:  Java中的WeakHashMap指南
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将研究java.util包中的[WeakHashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/WeakHashMap.html)。

为了理解数据结构，我们将在这里使用它来推出一个简单的缓存实现。但是，请记住，这是为了了解Map的工作原理，创建自己的缓存实现几乎总是一个坏主意。

简单地说，WeakHashMap是Map接口的基于哈希表的实现，其键是[WeakReference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/WeakReference.html)类型。

当WeakHashMap的键不再正常使用时，WeakHashMap中的条目将自动删除，这意味着没有指向该键的单个[Reference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html)。当垃圾回收(GC)进程丢弃一个键时，它的条目实际上已从Map中删除，因此此类的行为与其他Map实现有些不同。

## 2. 强引用、软引用和弱引用

要了解WeakHashMap的工作原理，**我们需要查看WeakReference类**-这是WeakHashMap实现中键的基本结构。在Java中，我们有三种主要类型的引用，我们将在以下部分中对其进行解释。

### 2.1 强引用

强引用是我们在日常编程中使用的最常见的引用类型：

```java
Integer prime = 1;
```

变量prime对值为1的Integer对象具有强引用，任何具有指向它的强引用的对象都不符合GC的条件。

### 2.2 软引用

简单地说，在JVM绝对需要内存之前，不会对具有指向它的[SoftReference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/SoftReference.html)的对象进行垃圾回收。

让我们看看如何在Java中创建SoftReference：

```java
Integer prime = 1;  
SoftReference<Integer> soft = new SoftReference<Integer>(prime); 
prime = null;
```

prime对象有指向它的强引用。

接下来，我们将prime强引用包装成软引用。在使该强引用为null之后，prime对象符合GC的条件，但只有在JVM绝对需要内存时才会被收集。

### 2.3 弱引用

仅被弱引用引用的对象被急切地垃圾回收；在这种情况下，GC不会等到它需要内存。

我们可以通过以下方式在Java中创建一个WeakReference：

```java
Integer prime = 1;  
WeakReference<Integer> soft = new WeakReference<Integer>(prime); 
prime = null;
```

当我们创建一个prime引用null时，prime对象将在下一个GC周期中被垃圾回收，因为没有其他强引用指向它。

**WeakReference类型的引用用作WeakHashMap中的键**。

## 3. WeakHashMap作为高效的内存缓存

假设我们要构建一个缓存，将大图像对象保存为值，将图像名称保存为键。我们想选择一个合适的Map实现来解决这个问题。

使用简单的HashMap不是一个好的选择，因为值对象可能会占用大量内存。更重要的是，它们永远不会被GC进程从缓存中回收，即使它们不再在我们的应用程序中使用。

理想情况下，我们需要一个允许GC自动删除未使用对象的Map实现。当大图像对象的键未在我们的应用程序中的任何地方使用时，该条目将从内存中删除。

幸运的是，WeakHashMap正是具有这些特性。让我们测试我们的WeakHashMap并看看它的行为：

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImage = new BigImage("image_id");
UniqueImageName imageName = new UniqueImageName("name_of_big_image");

map.put(imageName, bigImage);
assertTrue(map.containsKey(imageName));

imageName = null;
System.gc();

await().atMost(10, TimeUnit.SECONDS).until(map::isEmpty);
```

我们创建一个WeakHashMap实例用于存储我们的BigImage对象。我们将BigImage对象作为值，将imageName对象引用作为键。imageName将作为WeakReference类型存储在Map中。

接下来，我们将imageName引用设置为null，因此不再有指向bigImage对象的引用。WeakHashMap的默认行为是在下一次GC时回收没有引用它的条目，因此这个条目将被下一个GC进程从内存中删除。

我们调用System.gc()来强制JVM触发GC进程。在GC循环之后，我们的WeakHashMap将是空的：

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImageFirst = new BigImage("foo");
UniqueImageName imageNameFirst = new UniqueImageName("name_of_big_image");

BigImage bigImageSecond = new BigImage("foo_2");
UniqueImageName imageNameSecond = new UniqueImageName("name_of_big_image_2");

map.put(imageNameFirst, bigImageFirst);
map.put(imageNameSecond, bigImageSecond);
 
assertTrue(map.containsKey(imageNameFirst));
assertTrue(map.containsKey(imageNameSecond));

imageNameFirst = null;
System.gc();

await().atMost(10, TimeUnit.SECONDS)
    .until(() -> map.size() == 1);
await().atMost(10, TimeUnit.SECONDS)
    .until(() -> map.containsKey(imageNameSecond));
```

请注意，只有imageNameFirst引用设置为null。imageNameSecond引用保持不变。触发GC后，Map将只包含一个条目-imageNameSecond。

## 4. 总结

在本文中，我们研究了Java中的引用类型以充分理解java.util.WeakHashMap的工作原理。我们创建了一个简单的缓存，它利用WeakHashMap的行为并测试它是否按我们预期的那样工作。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。
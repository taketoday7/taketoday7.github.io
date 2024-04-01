---
layout: post
title:  Java List接口
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将介绍Java List接口。我们将讨论List提供的方法、其实现和使用场景。

## 2. Java List介绍

Java是一种面向对象的语言，因此大多数问题都涉及对象以及与这些对象相关联的行为或操作。

此外，我们经常需要同时操作多个相同类型的对象，这就是集合发挥作用的地方。**Java [List](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html)是保证元素顺序并允许重复的[集合](https://www.baeldung.com/java-collections)的实现**。

## 3. List方法和用法

让我们看一下List接口中最重要的方法，看看我们如何使用它们。对于此示例，我们将使用ArrayList实现。

### 3.1 添加元素

让我们使用void add(E element)方法向列表中添加新元素：

```java
@Test
public void givenAFruitList_whenAddNewFruit_thenFruitIsAdded(){
    List fruits = new ArrayList();
    assertEquals("Unexpected number of fruits in the list, should have been 0", 0, fruits.size());
        
    fruits.add("Apple");
    assertEquals("Unexpected number of fruits in the list, should have been 1", 1, fruits.size());
}
```

### 3.2 检查列表是否包含元素

我们可以使用boolean contains(Object o)方法检查列表是否包含元素：

```java
@Test
public void givenAFruitList_whenContainsFruit_thenFruitIsInTheList(){
    List fruits = new ArrayList();
        
    fruits.add("Apple");
    assertTrue("Apple should be in the fruit list", fruits.contains("Apple"));
    assertFalse("Banana should not be in the fruit list", fruits.contains("Banana"));
}
```

### 3.3 检查列表是否为空

让我们使用boolean isEmpty()方法检查列表是否为空：

```java
@Test
public void givenAnEmptyFruitList_whenEmptyCheck_thenListIsEmpty(){
    List fruits = new ArrayList();
    assertTrue("Fruit list should be empty", fruits.isEmpty());
        
    fruits.add("Apple");
    assertFalse("Fruit list should not be empty", fruits.isEmpty());
}
```

### 3.4 迭代列表

如果我们想遍历列表，我们可以使用方法ListIterator listIterator()：

```java
@Test
public void givenAFruitList_whenIterateOverIt_thenFruitsAreInOrder(){
    List fruits = new ArrayList();
        
    fruits.add("Apple"); // fruit at index 0
    fruits.add("Orange");// fruit at index 1
    fruits.add("Banana");// fruit at index 2
    int index = 0;
    for (Iterator it = fruits.listIterator(); it.hasNext(); ) {
        String fruit = it.next();
        assertEquals("Fruits should be in order", fruits.get(index++), fruit);
    }
}
```

### 3.5 删除元素

让我们使用方法boolean remove(Object o)从列表中删除一个元素：

```java
@Test
public void givenAFruitList_whenRemoveFruit_thenFruitIsRemoved(){
    List fruits = new ArrayList();
        
    fruits.add("Apple"); 
    fruits.add("Orange");
    assertEquals("Unexpected number of fruits in the list, should have been 2", 2, fruits.size());
        
    fruits.remove("Apple");
    assertEquals("Unexpected number of fruits in the list, should have been 1", 1, fruits.size());
}
```

### 3.6 修改元素

让我们使用方法E set(int index, E element)修改指定索引处的列表元素：

```java
@Test
public void givenAFruitList_whenSetFruit_thenFruitIsUpdated(){
    List fruits = new ArrayList();
        
    fruits.add("Apple"); 
    fruits.add("Orange");
        
    fruits.set(0, "Banana");
    assertEquals("Fruit at index 0 should be Banana", "Banana", fruits.get(0));
}
```

### 3.7 获取列表大小

让我们使用方法int size()检索列表的大小：

```java
List fruits = new ArrayList();
        
fruits.add("Apple"); 
fruits.add("Orange");
assertEquals("Unexpected number of fruits in the list, should have been 2", 2, fruits.size());
```

### 3.8 排序列表

我们有很多方法对列表进行排序。在这里，让我们看看如何使用List接口中的方法default void sort(Comparator c)来做到这一点。

该方法需要一个比较器作为参数。让我们为它提供自然顺序比较器：

```java
@Test
public void givenAFruitList_whenSort_thenFruitsAreSorted(){
    List fruits = new ArrayList();
        
    fruits.add("Apple"); 
    fruits.add("Orange");
    fruits.add("Banana");
        
    fruits.sort(Comparator.naturalOrder());
        
    assertEquals("Fruit at index 0 should be Apple", "Apple", fruits.get(0));
    assertEquals("Fruit at index 1 should be Banana", "Banana", fruits.get(1));
    assertEquals("Fruit at index 2 should be Orange", "Orange", fruits.get(2));
}
```

### 3.9 创建子列表

我们可以通过向方法List subList(int fromIndex, int toIndex)提供fromIndex和toIndex参数来从列表创建子列表。我们需要在这里考虑toIndex是不包含的：

```java
@Test
public void givenAFruitList_whenSublist_thenWeGetASublist(){
    List fruits = new ArrayList();
        
    fruits.add("Apple"); 
    fruits.add("Orange");
    fruits.add("Banana");
        
    List fruitsSublist = fruits.subList(0, 2);
    assertEquals("Unexpected number of fruits in the sublist, should have been 2", 2, fruitsSublist.size());
        
    assertEquals("Fruit at index 0 should be Apple", "Apple", fruitsSublist.get(0));
    assertEquals("Fruit at index 1 should be Orange", "Orange", fruitsSublist.get(1));
}
```

### 3.10 使用列表元素创建数组

我们可以使用方法T[] toArray(T[] a)创建一个包含列表元素的数组：

```java
@Test
public void givenAFruitList_whenToArray_thenWeGetAnArray(){
    List fruits = new ArrayList();
        
    fruits.add("Apple"); 
    fruits.add("Orange");
    fruits.add("Banana");
        
    String[] fruitsArray = fruits.toArray(new String[0]);
    assertEquals("Unexpected number of fruits in the array, should have been 3", 3, fruitsArray.length);
        
    assertEquals("Fruit at index 0 should be Apple", "Apple", fruitsArray[0]);
    assertEquals("Fruit at index 1 should be Orange", "Orange", fruitsArray[1]);
    assertEquals("Fruit at index 2 should be Banana", "Banana", fruitsArray[2]);
}
```

## 4. List实现

让我们看一下List接口在Java中最常用的实现。

### 4.1 ArrayList

[ArrayList](https://www.baeldung.com/java-arraylist)是List接口的可调整大小的数组实现。它实现所有可选操作并允许所有元素，包括null。这个类大致等同于Vector，只是它是不同步的。

这是List接口使用最广泛的实现。

### 4.2 CopyOnWriteArrayList

[CopyOnWriteArrayList](https://www.baeldung.com/java-copy-on-write-arraylist)是ArrayList的线程安全变体。**此类中的所有可变操作(add、set等)都会创建底层数组的新副本**。

此实现用于其固有的线程安全功能。

### 4.3 LinkedList

[LinkedList](https://www.baeldung.com/java-linkedlist)是List和Deque接口的双向链表实现，它实现所有可选操作并允许所有元素(包括null)。

### 4.4 抽象List实现

我们这里有两个抽象实现，它们提供了List接口的骨架实现。这些有助于最大限度地减少扩展和自定义List所需的工作量：

-   AbstractList：为其内部状态保留一个“随机访问”数据存储(例如数组)
-   AbstractSequentialList：为其内部状态保留一个“顺序访问”数据存储(例如链表)

### 4.5 其他具体List实现

这里有两个更具体的实现值得讨论：

-   Vector：实现一个可增长的对象数组。也像数组一样，它包含可以使用整数索引访问的组件。这个类是同步的。因此，如果不需要线程安全的实现，建议[使用ArrayList代替Vector](https://www.baeldung.com/java-arraylist-vs-vector)。
-   [Stack](https://www.baeldung.com/java-stack)：表示对象的后进先出(LIFO)堆栈。它扩展了类Vector并提供了五个额外的操作，允许将向量视为堆栈。

Java还提供了几个特定的List实现，它们的行为类似于上面讨论的实现之一。

## 5. 总结

在本文中，我们探讨了Java List接口及其实现。当我们只关心元素顺序并允许重复时，List是首选集合类型。由于它们在内部处理增长，因此比数组更受欢迎。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-5)上获得。
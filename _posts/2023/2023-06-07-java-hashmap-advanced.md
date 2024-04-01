---
layout: post
title:  Java HashMap的底层
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将更详细地探讨Java集合框架中最流行的Map接口实现，从我们的[介绍性文章](https://www.baeldung.com/java-hashmap)停止的地方继续。

在我们开始实现之前，重要的是要指出主要的List和Set集合接口扩展了Collection，但Map没有。

简单地说，HashMap通过键存储值，并提供API以各种方式添加、检索和操作存储的数据。**实现是基于哈希表的原理**，初听起来有点复杂，但其实非常容易理解。

键值对存储在所谓的桶中，这些存储桶一起构成了所谓的表，该表实际上是一个内部数组。

**一旦我们知道存储或将要存储对象的键，存储和检索操作就会在常量时间内发生**，在一个大小合适的HashMap中为O(1)。

要了解HashMap在底层的工作原理，需要了解HashMap使用的存储和检索机制。我们将重点关注这些。

最后，**与HashMap相关的问题在面试中很常见**，因此这是准备面试的可靠方法。

## 2. put() API

要在HashMap中存储一个值，我们调用带有两个参数的put API；一个键和对应的值：

```java
V put(K key, V value);
```

当一个值被添加到Map的指定键时，键对象的hashCode() API被调用以检索所谓的初始哈希值。

为了看到它的实际效果，让我们创建一个对象作为键。我们将只创建一个属性用作哈希码来模拟哈希的第一阶段：

```java
public class MyKey {
    private int id;

    @Override
    public int hashCode() {
        System.out.println("Calling hashCode()");
        return id;
    }

    // constructor, setters and getters 
}
```

现在，我们可以使用此对象在HashMap中映射值：

```java
@Test
public void whenHashCodeIsCalledOnPut_thenCorrect() {
    MyKey key = new MyKey(1);
    Map<MyKey, String> map = new HashMap<>();
    map.put(key, "val");
}
```

上面的代码中没有发生太多事情，但要注意控制台输出。实际上hashCode方法被调用了：

```plaintext
Calling hashCode()
```

接下来，在内部调用HashMap的hash() API以使用初始哈希值计算最终哈希值。

这个最终的哈希值最终归结为内部数组中的索引或者我们所说的存储桶位置。

HashMap的hash函数如下所示：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这里我们应该注意的只是使用键对象的哈希码来计算最终的哈希值。

在put函数中，最终的哈希值是这样使用的：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

请注意，调用了内部putVal函数并将最终哈希值作为第一个参数。

有人可能想知道为什么在这个函数中再次使用键，因为我们已经使用它来计算过哈希值。

原因是**HashMap将键和值作为Map.Entry对象存储在存储桶位置中**。

如前所述，所有Java集合框架接口都扩展了Collection接口，但Map没有。对比我们前面看到的Map接口和Set接口的声明：

```java
public interface Set<E> extends Collection<E>
```

原因是**Map并不像其他集合那样存储单个元素，而是键值对的集合**。

因此Collection接口的泛型方法(如add，toArray)对Map没有意义。

**我们在最后三段中介绍的概念构成了最流行的Java集合框架面试问题之一**，因此值得理解。

HashMap的一个特殊属性是它接受空值和空键：

```java
@Test
public void givenNullKeyAndVal_whenAccepts_thenCorrect(){
    Map<String, String> map = new HashMap<>();
    map.put(null, null);
}
```

**当在put操作期间遇到null键时，它会自动分配最终哈希值0**，这意味着它将成为基础数组的第一个元素。

这也意味着当key为null时，没有哈希操作，因此不会调用键的hashCode API，最终避免了空指针异常。

在put操作期间，当我们使用之前已经使用过的键来存储值时，它会返回与该键关联的先前值：

```java
@Test
public void givenExistingKey_whenPutReturnsPrevValue_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("key1", "val1");

    String rtnVal = map.put("key1", "val2");

    assertEquals("val1", rtnVal);
}
```

否则，它返回null：

```java
@Test
public void givenNewKey_whenPutReturnsNull_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String rtnVal = map.put("key1", "val1");

    assertNull(rtnVal);
}
```

当put返回null时，也可能意味着与该键关联的先前值是null，不一定是新的键值映射：

```java
@Test
public void givenNullVal_whenPutReturnsNull_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String rtnVal = map.put("key1", null);

    assertNull(rtnVal);
}
```

containsKey API可用于区分我们将在下一节中看到的此类场景。

## 3. get API

要检索已存储在HashMap中的对象，我们必须知道存储它的键。我们调用get API并将键对象传递给它：

```java
@Test
public void whenGetWorks_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("key", "val");

    String val = map.get("key");

    assertEquals("val", val);
}
```

在内部，使用相同的哈希原则。调用键对象的hashCode() API获取初始哈希值：

```java
@Test
public void whenHashCodeIsCalledOnGet_thenCorrect() {
    MyKey key = new MyKey(1);
    Map<MyKey, String> map = new HashMap<>();
    map.put(key, "val");
    map.get(key);
}
```

这一次，MyKey的hashCode API被调用了两次；一次用于put一次用于get：

```plaintext
Calling hashCode()
Calling hashCode()
```

然后通过调用内部hash() API重新哈希该值以获得最终的哈希值。

正如我们在上一节中看到的，这个最终的哈希值最终归结为存储桶位置或内部数组的索引。

然后检索存储在该位置的值对象并将其返回给调用函数。

当返回值为null时，这可能意味着键对象未与HashMap中的任何值相关联：

```java
@Test
public void givenUnmappedKey_whenGetReturnsNull_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String rtnVal = map.get("key1");

    assertNull(rtnVal);
}
```

或者它可能只是意味着该键被显式映射到一个null实例：

```java
@Test
public void givenNullVal_whenRetrieves_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("key", null);
        
    String val=map.get("key");
        
    assertNull(val);
}
```

为了区分这两种情况，我们可以使用containsKey API，我们将键传递给它，当且仅当为HashMap中的指定键创建了映射时，它才返回true：

```java
@Test
public void whenContainsDistinguishesNullValues_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String val1 = map.get("key");
    boolean valPresent = map.containsKey("key");

    assertNull(val1);
    assertFalse(valPresent);

    map.put("key", null);
    String val = map.get("key");
    valPresent = map.containsKey("key");

    assertNull(val);
    assertTrue(valPresent);
}
```

对于上述测试中的两种情况，get API调用的返回值为null，但我们能够区分哪个是哪个。

## 4. HashMap中的集合视图

HashMap提供了三个视图，使我们能够将其键和值视为另一个集合。我们可以获得Map的所有键的集合：

```java
@Test
public void givenHashMap_whenRetrievesKeyset_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "tuyucheng");
    map.put("type", "blog");

    Set<String> keys = map.keySet();

    assertEquals(2, keys.size());
    assertTrue(keys.contains("name"));
    assertTrue(keys.contains("type"));
}
```

该集合由Map本身提供支持，因此**对集合所做的任何更改都会反映在Map中**：

```java
@Test
public void givenKeySet_whenChangeReflectsInMap_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "tuyucheng");
    map.put("type", "blog");

    assertEquals(2, map.size());

    Set<String> keys = map.keySet();
    keys.remove("name");

    assertEquals(1, map.size());
}
```

我们还可以获得值的集合视图：

```java
@Test
public void givenHashMap_whenRetrievesValues_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "tuyucheng");
    map.put("type", "blog");

    Collection<String> values = map.values();

    assertEquals(2, values.size());
    assertTrue(values.contains("tuyucheng"));
    assertTrue(values.contains("blog"));
}
```

就像键集一样，**在此集合中所做的任何更改都将反映在底层Map中**。

最后，我们可以获得Map中所有条目的集合视图：

```java
@Test
public void givenHashMap_whenRetrievesEntries_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "tuyucheng");
    map.put("type", "blog");

    Set<Entry<String, String>> entries = map.entrySet();

    assertEquals(2, entries.size());
    for (Entry<String, String> e : entries) {
        String key = e.getKey();
        String val = e.getValue();
        assertTrue(key.equals("name") || key.equals("type"));
        assertTrue(val.equals("tuyucheng") || val.equals("blog"));
    }
}
```

请记住，HashMap专用于包含无序元素，因此我们在测试foreach循环中的条目的键和值时假定任何顺序。

很多时候，你将像上一个示例一样在循环中使用集合视图，更具体地说是使用它们的迭代器。

请记住，**上述所有视图的迭代器都是快速失败的**。

如果对Map进行结构修改，则迭代器创建完成后，会抛出并发修改异常：

```java
@Test(expected = ConcurrentModificationException.class)
public void givenIterator_whenFailsFastOnModification_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "tuyucheng");
    map.put("type", "blog");

    Set<String> keys = map.keySet();
    Iterator<String> it = keys.iterator();
    map.remove("type");
    while (it.hasNext()) {
        String key = it.next();
    }
}
```

**唯一允许的结构修改是通过迭代器本身执行的remove操作**：

```java
public void givenIterator_whenRemoveWorks_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "tuyucheng");
    map.put("type", "blog");

    Set<String> keys = map.keySet();
    Iterator<String> it = keys.iterator();

    while (it.hasNext()) {
        it.next();
        it.remove();
    }

    assertEquals(0, map.size());
}
```

关于这些集合视图，最后要记住的是迭代的性能。这是HashMap与其对应的LinkedHashMap和TreeMap相比性能相当差的地方。

HashMap的迭代发生在最坏的O(n)情况下，其中n是其容量和条目数的总和。

## 5. HashMap性能

HashMap的性能受两个参数影响：初始容量和负载因子。容量是桶的数量或底层数组长度，初始容量只是创建时的容量。

负载因子(或LF)，简而言之，是在调整大小之前添加一些值后HashMap应该有多满的度量。

默认初始容量为16，默认负载因子为0.75。我们可以使用初始容量和LF的自定义值创建一个HashMap：

```java
Map<String,String> hashMapWithCapacity = new HashMap<>(32);
Map<String,String> hashMapWithCapacityAndLF = new HashMap<>(32, 0.5f);
```

Java团队设置的默认值已针对大多数情况进行了优化。但是，如果你需要使用自己的值(这很好)，则需要了解性能影响，以便你知道自己在做什么。

**当HashMap条目的数量超过LF和容量的乘积时，就会发生重哈希，即创建另一个内部数组，其大小是初始数组的两倍，并且所有条目都移动到新数组中的新存储桶位置**。

**较低的初始容量**可降低空间成本，但会**增加重哈希的频率**。重哈希显然是一个非常昂贵的过程。因此，通常，如果你预计会有很多条目，则应设置相当高的初始容量。

另一方面，如果你将初始容量设置得太高，你将付出迭代时间的代价。正如我们在上一节中看到的。

因此，**高初始容量适用于大量条目以及很少或没有迭代**。

**较低的初始容量适用于具有大量迭代的少数条目**。

## 6. HashMap中的碰撞

冲突/碰撞(或者更具体地说，HashMap中的哈希码冲突)是指**两个或多个键对象生成相同的最终哈希值**，并因此指向相同的存储桶位置或数组索引的情况。

之所以会出现这种情况，是因为根据equals和hashCode约定，**Java中两个不相等的对象可以具有相同的哈希码**。

由于底层数组的大小有限，即在调整大小之前，也可能发生这种情况。这个数组越小，发生冲突的可能性就越高。

也就是说，值得一提的是，Java实现了哈希码冲突解决技术，我们将使用一个示例来了解这一点。

**请记住，是键的哈希值决定了对象将存储在哪个桶中。因此，如果任意两个键的哈希码发生冲突，它们的条目仍将存储在同一个桶中**。

并且默认情况下，实现使用链表作为存储桶实现。

在发生冲突的情况下，初始恒定时间O(1) put和get操作将在线性时间O(n)内发生。这是因为在找到具有最终哈希值的桶位置后，将使用equals API将此位置的每个键与提供的键对象进行比较。

为了模拟这种冲突解决技术，让我们稍微修改一下我们之前的键对象：

```java
public class MyKey {
    private String name;
    private int id;

    public MyKey(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters

    @Override
    public int hashCode() {
        System.out.println("Calling hashCode()");
        return id;
    }

    // toString override for pretty logging

    @Override
    public boolean equals(Object obj) {
        System.out.println("Calling equals() for key: " + obj);
        // generated implementation
    }
}
```

请注意我们是如何简单地将id属性作为哈希码返回的-从而强制发生冲突。

另外，请注意我们在equals和hashCode实现中添加了日志语句-这样我们就可以准确知道逻辑何时被调用。

现在让我们继续存储和检索在某个时刻发生碰撞的一些对象：

```java
@Test
public void whenCallsEqualsOnCollision_thenCorrect() {
    HashMap<MyKey, String> map = new HashMap<>();
    MyKey k1 = new MyKey(1, "firstKey");
    MyKey k2 = new MyKey(2, "secondKey");
    MyKey k3 = new MyKey(2, "thirdKey");

    System.out.println("storing value for k1");
    map.put(k1, "firstValue");
    System.out.println("storing value for k2");
    map.put(k2, "secondValue");
    System.out.println("storing value for k3");
    map.put(k3, "thirdValue");

    System.out.println("retrieving value for k1");
    String v1 = map.get(k1);
    System.out.println("retrieving value for k2");
    String v2 = map.get(k2);
    System.out.println("retrieving value for k3");
    String v3 = map.get(k3);

    assertEquals("firstValue", v1);
    assertEquals("secondValue", v2);
    assertEquals("thirdValue", v3);
}
```

在上面的测试中，我们创建了三个不同的键-一个具有唯一id，另外两个具有相同的id。由于我们使用id作为初始哈希值，因此在使用这些键存储和检索数据时肯定会发生冲突。

除此之外，由于我们之前看到的冲突解决技术，我们希望每个存储的值都能被正确检索，因此给出最后三行中的断言。

当我们运行测试时，它应该通过，表明冲突已解决，我们将使用生成的日志记录来确认确实发生了冲突：

```plaintext
storing value for k1
Calling hashCode()
storing value for k2
Calling hashCode()
storing value for k3
Calling hashCode()
Calling equals() for key: MyKey [name=secondKey, id=2]
retrieving value for k1
Calling hashCode()
retrieving value for k2
Calling hashCode()
retrieving value for k3
Calling hashCode()
Calling equals() for key: MyKey [name=secondKey, id=2]
```

请注意，在存储操作期间，k1和k2仅使用哈希码成功映射到它们的值。

然而，k3的存储并不是那么简单，系统检测到它的存储桶位置已经包含了k2的映射。因此，使用equals比较来区分它们，并创建一个链表来包含这两个映射。

其键哈希到相同存储桶位置的任何其他后续映射将遵循相同的路线并最终替换链表中的某个节点，或者如果所有现有节点的equals比较返回false，则将其添加到列表的头部。

同样，在检索期间，对k3和k2进行equals比较，以确定应检索其值的正确键。

最后一点，从Java 8开始，在给定桶位置的冲突次数超过特定阈值后，链表在碰撞解决中动态替换为平衡二叉搜索树。

此更改提供了性能提升，因为在发生冲突的情况下，存储和检索发生在O(logn)中。

这部分在技术面试中很常见，尤其是在基本的存储和检索问题之后。

## 7. 总结

在本文中，我们探讨了Java Map接口的HashMap实现。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-1)上获得。
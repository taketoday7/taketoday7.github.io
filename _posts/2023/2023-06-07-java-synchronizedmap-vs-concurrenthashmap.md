---
layout: post
title:  Collections.synchronizedMap与ConcurrentHashMap
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将讨论[Collections.synchronizedMap()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#synchronizedMap(java.util.Map))和[ConcurrentHashMap](https://www.baeldung.com/java-concurrent-map)之间的区别。

此外，我们还将查看每个读取和写入操作的性能输出。

## 2. 差异

Collections.synchronizedMap()和ConcurrentHashMap都提供对数据集合的线程安全操作。

**[Collections](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html)实用程序类提供了对集合进行操作并返回包装集合的多态算法**。它的synchronizedMap()方法提供了线程安全的功能。

顾名思义，synchronizedMap()返回由我们在参数中提供的Map支持的同步Map。为了提供线程安全，synchronizedMap()允许通过返回的Map对后备Map进行所有访问。

**ConcurrentHashMap是在JDK 1.5中作为HashMap的增强功能引入的，它支持检索和更新的高并发性**。HashMap不是线程安全的，因此它可能会在线程争用期间导致不正确的结果。

ConcurrentHashMap类是线程安全的。因此，多个线程可以毫不复杂地操作单个对象。

**在ConcurrentHashMap中，读操作是非阻塞的，而写入操作对特定段或存储桶进行锁定**。默认的存储桶或并发级别为16，这意味着在锁定段或存储桶后，16个线程可以随时写入。

### 2.1 ConcurrentModificationException

对于像HashMap这样的对象，不允许执行并发操作。因此，如果我们尝试在迭代HashMap时更新它，我们将得到ConcurrentModificationException。使用synchronizedMap()时也会发生这种情况：

```java
@Test(expected = ConcurrentModificationException.class)
public void whenRemoveAndAddOnHashMap_thenConcurrentModificationError() {
    Map<Integer, String> map = new HashMap<>();
    map.put(1, "tuyucheng");
    map.put(2, "HashMap");
    Map<Integer, String> synchronizedMap = Collections.synchronizedMap(map);
    Iterator<Entry<Integer, String>> iterator = synchronizedMap.entrySet().iterator();
    while (iterator.hasNext()) {
        synchronizedMap.put(3, "Modification");
        iterator.next();
    }
}
```

但是，ConcurrentHashMap并非如此：

```java
Map<Integer, String> map = new ConcurrentHashMap<>();
map.put(1, "tuyucheng");
map.put(2, "HashMap");
 
Iterator<Entry<Integer, String>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    map.put(3, "Modification");
    iterator.next()
}
 
Assert.assertEquals(3, map.size());
```

### 2.2 null支持

Collections.synchronizedMap()和ConcurrentHashMap**以不同方式处理空键和空值**。

ConcurrentHashMap不允许在键或值中使用null：

```java
@Test(expected = NullPointerException.class)
public void allowNullKey_In_ConcurrentHasMap() {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    map.put(null, 1);
}
```

但是，**在使用Collections.synchronizedMap()时，空支持取决于输入Map**。当Collections.synchronizedMap()由HashMap或LinkedHashMap支持时，我们可以将一个null作为键和任意数量的null值，而如果我们使用TreeMap，我们可以有null值但不能有null键。

让我们断言我们可以对由HashMap支持的Collections.synchronizedMap()使用空键：

```java
Map<String, Integer> map = Collections
    .synchronizedMap(new HashMap<String, Integer>());
map.put(null, 1);
Assert.assertTrue(map.get(null).equals(1));
```

同样，我们可以验证Collections.synchronizedMap()和ConcurrentHashMap的值是否支持空值。

## 3. 性能比较

让我们比较一下ConcurrentHashMap与Collections.synchronizedMap()的性能。在这种情况下，我们使用开源框架[Java Microbenchmark Harness](https://www.baeldung.com/java-microbenchmark-harness)(JMH)以纳秒为单位比较这些方法的性能。

我们对这些Map上的随机读写操作进行了比较。让我们快速浏览一下我们的JMH基准测试代码：

```java
@Benchmark
public void randomReadAndWriteSynchronizedMap() {
    Map<String, Integer> map = Collections.synchronizedMap(new HashMap<String, Integer>());
    performReadAndWriteTest(map);
}

@Benchmark
public void randomReadAndWriteConcurrentHashMap() {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    performReadAndWriteTest(map);
}

private void performReadAndWriteTest(final Map<String, Integer> map) {
    for (int i = 0; i < TEST_NO_ITEMS; i++) {
        Integer randNumber = (int) Math.ceil(Math.random() * TEST_NO_ITEMS);
        map.get(String.valueOf(randNumber));
        map.put(String.valueOf(randNumber), randNumber);
    }
}
```

我们使用5次迭代和10个线程对1000个元素运行性能基准测试，让我们看看基准测试结果：

```text
Benchmark                                                     Mode  Cnt        Score        Error  Units
MapPerformanceComparison.randomReadAndWriteConcurrentHashMap  avgt  100  3061555.822 ±  84058.268  ns/op
MapPerformanceComparison.randomReadAndWriteSynchronizedMap    avgt  100  3234465.857 ±  60884.889  ns/op
MapPerformanceComparison.randomReadConcurrentHashMap          avgt  100  2728614.243 ± 148477.676  ns/op
MapPerformanceComparison.randomReadSynchronizedMap            avgt  100  3471147.160 ± 174361.431  ns/op
MapPerformanceComparison.randomWriteConcurrentHashMap         avgt  100  3081447.009 ±  69533.465  ns/op
MapPerformanceComparison.randomWriteSynchronizedMap           avgt  100  3385768.422 ± 141412.744  ns/op
```

上述结果表明**ConcurrentHashMap的性能优于Collections.synchronizedMap()**。

## 4. 何时使用

当数据一致性至关重要时，我们应该更倾向于Collections.synchronizedMap()，对于写操作远多于读操作的性能关键型应用程序，我们应该选择ConcurrentHashMap。

这是因为Collections.synchronizedMap()要求每个线程为读/写操作获取整个对象的锁。相比之下，ConcurrentHashMap允许线程在集合的不同段上获取锁，并同时进行修改。

## 5. 总结

在本文中，我们展示了ConcurrentHashMap和Collections.synchronizedMap()之间的区别，并使用简单的JMH基准测试展示了它们的性能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-3)上获得。
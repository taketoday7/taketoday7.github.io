---
layout: post
title:  HashSet中removeAll()的性能
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

[HashSet](https://www.baeldung.com/java-hashset)是一个用于存储唯一元素的集合。

在本教程中，我们将讨论java.util.HashSet类中removeAll()方法的性能。

## 2. HashSet.removeAll()

removeAll方法删除集合中包含的所有元素：

```java
Set<Integer> set = new HashSet<Integer>();
set.add(1);
set.add(2);
set.add(3);
set.add(4);

Collection<Integer> collection = new ArrayList<Integer>();
collection.add(1);
collection.add(3);

set.removeAll(collection);

Integer[] actualElements = new Integer[set.size()];
Integer[] expectedElements = new Integer[] { 2, 4 };
assertArrayEquals(expectedElements, set.toArray(actualElements));
```

结果，元素1和3将从集合中删除。

## 3. 内部实现和时间复杂度

removeAll()方法确定哪个更小-Set还是Collection。这是通过对Set和Collection调用size()方法来完成的。

**如果Collection的元素少于Set，则它以[时间复杂度](https://www.baeldung.com/java-algorithm-complexity)O(n)迭代指定的集合**。它还检查元素是否存在于时间复杂度为O(1)的Set中。如果该元素存在，则使用Set的remove()方法将其从Set中移除，这同样具有O(1)的时间复杂度。所以**整体时间复杂度是O(n)**。

**如果Set的元素少于Collection，则它使用O(n)迭代此Set**。然后它通过调用其contains()方法检查Collection中是否存在每个元素。如果存在这样的元素，则从Set中删除该元素。所以这取决于contains()方法的时间复杂度。

现在在这种情况下，如果集合是ArrayList，则contains()方法的时间复杂度是O(m)。因此，**从Set中删除ArrayList中存在的所有元素的总体时间复杂度为O(n * m)**。

如果集合再次为HashSet，则contains()方法的时间复杂度为O(1)。因此，**从Set中删除HashSet中存在的所有元素的总体时间复杂度为O(n)**。

## 4. 性能

要查看上述3种情况之间的性能差异，让我们编写一个简单的[JMH基准测试](https://www.baeldung.com/java-microbenchmark-harness)。

对于第一种情况，我们将初始化Set和Collection，Set中的元素多于Collection中的元素。在第二种情况下，我们将初始化Set和Collection，其中Collection中的元素多于Set中的元素。在第三种情况下，我们将初始化2个Set，其中第二个Set的元素数量多于第一个Set：

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5)
public class HashSetBenchmark {

    @State(Scope.Thread)
    public static class MyState {
        private Set employeeSet1 = new HashSet<>();
        private List employeeList1 = new ArrayList<>();
        private Set employeeSet2 = new HashSet<>();
        private List employeeList2 = new ArrayList<>();
        private Set<Employee> employeeSet3 = new HashSet<>();
        private Set<Employee> employeeSet4 = new HashSet<>();

        private long set1Size = 60000;
        private long list1Size = 50000;
        private long set2Size = 50000;
        private long list2Size = 60000;
        private long set3Size = 50000;
        private long set4Size = 60000;

        @Setup(Level.Trial)
        public void setUp() {
            // populating sets
        }
    }
}
```

之后，添加我们的基准测试：

```java
@Benchmark
public boolean given_SizeOfHashsetGreaterThanSizeOfCollection_whenRemoveAllFromHashSet_thenGoodPerformance(MyState state) {
    return state.employeeSet1.removeAll(state.employeeList1);
}

@Benchmark
public boolean given_SizeOfHashsetSmallerThanSizeOfCollection_whenRemoveAllFromHashSet_thenBadPerformance(MyState state) {
    return state.employeeSet2.removeAll(state.employeeList2);
}

@Benchmark
public boolean given_SizeOfHashsetSmallerThanSizeOfAnotherHashSet_whenRemoveAllFromHashSet_thenGoodPerformance(MyState state) {
    return state.employeeSet3.removeAll(state.employeeSet4);
}
```

结果如下：

```text
Benchmark                                              Mode  Cnt            Score            Error  Units
HashSetBenchmark.testHashSetSizeGreaterThanCollection  avgt   20      2700457.099 ±     475673.379  ns/op
HashSetBenchmark.testHashSetSmallerThanCollection      avgt   20  31522676649.950 ± 3556834894.168  ns/op
HashSetBenchmark.testHashSetSmallerThanOtherHashset    avgt   20      2672757.784 ±     224505.866  ns/op
```

**我们可以看到，当Collection作为参数传递给removeAll()方法，HashSet的元素少于Collection时，HashSet.removeAll()的性能非常糟糕。但是当另一个集合又是HashSet时，性能就很好**。

## 5. 总结

在本文中，我们看到了HashSet中removeAll()的性能。当Set的元素少于Collection的元素时，removeAll()的性能取决于集合的contains()方法的时间复杂度。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-3)上获得。
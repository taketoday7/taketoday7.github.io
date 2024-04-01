---
layout: post
title:  Java流数据的批处理
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在本教程中，我们将探讨如何在Java中完成流数据的批处理。我们将看到使用原生Java功能和一些第三方库的示例。

## 2. Stream数据的批处理是什么意思？

**Java中Stream数据的批处理是指将一个大的数据集划分为更小的、更易于管理的块并按顺序处理它们的做法**。在这个场景中，处理的数据源来自一个数据流。

出于多种原因，这可能是有利的，包括提高数据处理效率、处理可能无法一次放入内存中的非常庞大的数据集，以及提供一种使用多个处理器并行处理数据的机制。

但是，在实现批处理时可能会出现各种问题：

-   设置可接受的批大小：如果批大小太小，处理每个批的开销可能会变得很大。但是，如果批次大小太大，处理每个批的时间可能会太长，这会导致流处理管道出现延迟。
-   状态管理：为了跟踪中间结果或保证每个批次的处理与之前的批次一致，在采用批处理时通常需要保留批次之间的状态。使用分散系统的复杂性增加了状态管理的难度。
-   容错性：在批量处理大型数据集时，确保在发生故障时可以继续处理是至关重要的。这可能很困难，因为可能需要存储大量的中间状态才能恢复处理。

在本文中，为了简单明了，我们将只关注Java中Stream数据的批处理，而不关注如何解决上述问题。

## 3. 使用Java Stream API进行批处理

首先，我们必须注意我们将使用的一些关键概念。首先，我们有[Stream API](https://www.baeldung.com/java-8-streams-introduction)，这是Java 8中引入的一个主要特性。在Stream API中，我们将使用[Stream](https://www.baeldung.com/java-8-streams-introduction#1-stream-creation)类。

在这种情况下，我们需要考虑声明的数据流只能被调用一次。如果我们尝试对同一个数据流进行第二次操作，则会得到一个IllegalStateException。一个简单的例子向我们展示了这种行为：

```java
Stream<String> coursesStream = Stream.of("Java", "Frontend", "Backend", "Fullstack");
Stream<Integer> coursesStreamLength = coursesStream.map(String::length);
// we get java.lang.IllegalStateException
Stream<String> emphasisCourses = coursesStream.map(course -> course + "!");
```

其次，我们将使用函数式风格处理以下部分中的大多数示例。有些例子有副作用，我们必须以函数式编程风格尽量避免它们。

在构建代码示例之前，让我们定义我们的测试数据流、批大小和预期的批结果。

我们的数据流将是一个整数值流：

```java
Stream<Integer> data = IntStream.range(0, 34).boxed();
```

然后，我们的批大小为10：

```java
private final int BATCH_SIZE = 10;
```

最后，让我们定义预期批次：

```java
private final List<Integer> firstBatch = List.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9);
private final List<Integer> secondBatch = List.of(10, 11, 12, 13, 14, 15, 16, 17, 18, 19);
private final List<Integer> thirdBatch = List.of(20, 21, 22, 23, 24, 25, 26, 27, 28, 29);
private final List<Integer> fourthBatch = List.of(30, 31, 32, 33);
```

接下来，让我们看一些示例。

## 4. 使用Iterator

第一种方法使用[Iterator](https://www.baeldung.com/java-iterator)接口的自定义实现。我们定义了一个CustomBatchIterator类，我们可以在初始化Iterator的新实例时设置批大小。

让我们跳入代码：

```java
public class CustomBatchIterator<T> implements Iterator<List<T>> {
    private final int batchSize;
    private List<T> currentBatch;
    private final Iterator<T> iterator;
    
    public CustomBatchIterator(Iterator<T> sourceIterator, int batchSize) {
        this.batchSize = batchSize;
        this.iterator = sourceIterator;
    }
    
    @Override
    public List<T> next() {
        return currentBatch;
    }
    
    @Override
    public boolean hasNext() {
        prepareNextBatch();
        return currentBatch != null && !currentBatch.isEmpty();
    }
    
    private void prepareNextBatch() {
        currentBatch = new ArrayList<>(batchSize);
        while (iterator.hasNext() && currentBatch.size() < batchSize) {
            currentBatch.add(iterator.next());
        }
    }
}
```

在这里，我们在CustomBatchIterator类中覆盖了Iterator接口的hasNext()和next()方法。如果当前批次不为空，则hasNext()方法通过执行prepareNextBatch()方法准备下一批数据。我们需要使用next()方法来获取最新的信息。

prepareNextBatch()方法首先使用源迭代器中的元素填充当前批次，直到批次完成或源迭代器用完元素，以先发生者为准。currentBatch被初始化为一个空列表，其大小等于batchSize。

此外，我们将CustomBatchIterator构造函数声明为私有的。这可以防止CustomBatchIterator在其类范围之外被实例化。我们将添加一个静态batchStreamOf()方法以使CustomBatchIterator可用。

下一步是向我们的类添加两个静态方法：

```java
public class CustomBatchIterator<T> implements Iterator<List<T>> {

    // other methods

    public static <T> Stream<List<T>> batchStreamOf(Stream<T> stream, int batchSize) {
        return stream(new CustomBatchIterator<>(stream.iterator(), batchSize));
    }

    private static <T> Stream<T> stream(Iterator<T> iterator) {
        return StreamSupport.stream(Spliterators.spliteratorUnknownSize(iterator, ORDERED), false);
    }
}
```

我们的batchStreamOf()方法从数据流中生成批处理流。它通过实例化CustomBatchIterator类并将其传递给stream()方法来实现这一点，该方法从Iterator生成Stream。

我们的stream()方法使用Spliterators.spliteratorUnknownSize()方法从Iterator创建一个[Spliterator](https://www.baeldung.com/java-spliterator)(一种可以使用流探索的特殊迭代器)，然后将Spliterator提供给StreamSupport.stream()方法以构建流。

现在，是时候测试我们的实现了：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingSpliterator_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>();
    CustomBatchIterator.batchStreamOf(data, BATCH_SIZE).forEach(result::add);
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

在上面的测试中，我们将数据流和批大小传递给batchStreamOf()方法。然后，我们检查数据处理后是否有4批。

## 5. 使用Collection API

我们的下一个示例使用Collection API，并且比第一个示例相对更直接。

让我们看看我们的测试用例：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingCollectionAPI_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = data.collect(Collectors.groupingBy(it -> it / BATCH_SIZE))
        .values();
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

在此代码片段中，我们使用来自Java Stream API中的Collectors.groupingBy()方法，通过使用it -> it/BATCH_SIZE lambda表达式计算的键对数据流中的元素进行分组。lambda表达式将每个元素除以BATCH_SIZE，结果作为键返回。

然后，我们调用Map的values方法来检索元素列表的集合，并将其保存在result变量中。

**对于大型数据集，我们可以使用Stream中的[parallel()](https://www.baeldung.com/java-when-to-use-parallel-stream)方法。但是，我们需要考虑到执行顺序是我们无法控制的**。它可能会在我们每次运行程序时发生变化。

让我们使用parallel()检查我们的测试用例：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchParallelUsingCollectionAPI_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = data.parallel()
        .collect(Collectors.groupingBy(it -> it / BATCH_SIZE))
        .values();
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

## 6. RxJava

[RxJava](https://www.baeldung.com/rx-java)是ReactiveX的Java版本，它是一个使用可观察序列编写异步和基于事件的程序的库。我们可以将它与Stream API结合使用，以在Java中进行批处理。

首先，让我们在pom.xml文件中添加它的[依赖项](https://mvnrepository.com/artifact/io.reactivex.rxjava3/rxjava)：

```xml
<dependency>
    <groupId>io.reactivex.rxjava3</groupId>
    <artifactId>rxjava</artifactId>
    <version>3.1.5</version>
</dependency>
```

我们的下一步是实现测试用例：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingRxJavaV3_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>();
    Observable.fromStream(data)
        .buffer(BATCH_SIZE)
        .subscribe(result::add);
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

为了将数据流划分为可管理的块，此代码使用RxJava库中的buffer()运算符，每个块的大小由变量BATCH_SIZE确定。

此外，我们使用Observable.fromStream()方法从数据流创建一个Observable。我们以BATCH_SIZE作为输入调用Observable的buffer()方法。Observable元素被分类为我们选择大小的列表，每个列表作为流中的新元素发出。

结果是一个Observable，并在其上调用subscribe()方法，以result::add作为参数。这将创建对Observable的订阅，并且每次Observable发出一个元素时，都会调用result列表的add方法。在这种情况下，Observable的输出由聚合成集合的元素列表组成。

## 7. Vavr

[Vavr](https://www.baeldung.com/vavr)是一个函数式编程库，具有不可变集合和其他函数式数据结构。

在这种情况下，我们将其[依赖项](https://mvnrepository.com/artifact/io.vavr/vavr)添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>1.0.0-alpha-4</version>
</dependency>
```

现在，让我们看看实际的测试用例：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingVavr_thenFourBatchesAreObtained() {
    List<List<Integer>> result = Stream.ofAll(data)
        .toList()
        .grouped(BATCH_SIZE)
        .toList();
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

Stream.ofAll()方法使用Stream.ofAll()方法将数据集转换为流。最后，我们使用Stream的toList()方法将其转换为List。这个最终列表作为参数与值BATCH_SIZE一起传递给grouped()方法。此方法返回一个有序列表，其中包含从原始列表中获取的BATCH_SIZE元素，并在每个内部列表中复制一次。

**上述测试中的List类来自io.vavr.collection而不是来自java.util.List**。

## 8. Reactor

批处理的下一个选项是使用[Reactor](https://www.baeldung.com/reactor-core)库。除了批处理之外，Reactor是一个用于响应式编程的Java库，它还提供了多个用于处理流的运算符。**在这种情况下，我们将使用[Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)来进行批处理**。

对于此示例，让我们将[依赖项](https://mvnrepository.com/artifact/io.projectreactor/reactor-core)添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.5.1</version>
</dependency>
```

让我们实现我们的测试用例：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingReactor_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>();
    Flux.fromStream(data)
        .buffer(BATCH_SIZE)
        .subscribe(result::add);
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

要从java.util.stream.Stream对象创建Flux，我们使用Flux.fromStream()方法。当我们想使用Flux类提供的响应式运算符处理流的元素时，这很方便。

buffer()运算符用于将元素分批放入固定大小的列表中。Flux在发出新元素时被添加到当前列表中。当列表达到合适的大小时，Flux发出它，并形成一个新的列表。这对于批处理优化非常有用，例如减少数据库查询或网络请求的数量。

最后，subscribe()方法添加一个Flux订阅者。订阅者接收Flux发出的元素。接下来，它将它们添加到result对象中。subscribe()方法生成一个Subscription对象，该对象可用于调节数据流并在不再需要时取消订阅Flux。

## 9. Apache Commons

我们可以使用[Apache Commons Collections](https://www.baeldung.com/apache-commons-collection-utils)等强大的库来执行批处理。

让我们在pom.xml文件中添加它的[依赖项](https://mvnrepository.com/artifact/org.apache.commons/commons-collections4)：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

我们的测试实现很简单：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingApacheCommon_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>(ListUtils
        .partition(data.collect(Collectors.toList()), BATCH_SIZE));
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

partition()方法一个是Apache Commons ListUtils实用程序方法，它接收一个列表和一个大小。它生成一个List<List<\>>，每个内部List的最大大小为所提供的大小。我们可以注意到，**数据流在传递给partition()方法之前被转换为一个列表**。

## 10. Guava

接下来，我们有[Guava](https://www.baeldung.com/guava-guide)库。Guava提供了多种用于处理集合的实用方法，包括批处理。

让我们在pom.xml文件中添加[依赖项](https://mvnrepository.com/artifact/com.google.guava/guava)：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

现在，让我们看看我们的工作示例：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingGuava_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>();
    Iterators.partition(data.iterator(), BATCH_SIZE).forEachRemaining(result::add);
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

Iterators.partition()方法可以帮助将大型数据集分解成更小的块进行处理，例如并行分析数据或将其批量加载到数据库中。

我们使用Iterators.partition()方法将数据的迭代器拆分为一系列更小的迭代器。传递给Iterators.partition()方法的数据是我们数据流中的Iterator。此外，我们将BATCH_SIZE传递给它。

## 11. Cyclops

最后，我们有基于[jool](https://www.baeldung.com/jool)库的Cyclops库。Cyclops React是一个包含多个用于与流交互的运算符的库，其中也有一些用于批处理。

让我们将它的[依赖](https://mvnrepository.com/artifact/com.oath.cyclops/cyclops)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.oath.cyclops</groupId>
    <artifactId>cyclops</artifactId>
    <version>10.4.1</version>
</dependency>
```

让我们看看最后一个例子的代码：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingCyclops_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>();
    ReactiveSeq.fromStream(data)
        .grouped(BATCH_SIZE)
        .toList()
        .forEach(value -> result.add(value.collect(Collectors.toList())));
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

ReactiveSeq类是一种响应序列。此外，ReactiveSeq.fromStream()方法将数据流转换为响应序列。然后，数据被分组为BATCH_SIZE的批次。然后将处理后的数据收集到整数List的集合中。

但是，我们可以使用LazySeq获得惰性的函数式风格。在这种情况下，我们只需要将ReactiveSeq替换为LazySeq：

```java
@Test
public void givenAStreamOfData_whenIsProcessingInBatchUsingCyclopsLazy_thenFourBatchesAreObtained() {
    Collection<List<Integer>> result = new ArrayList<>();
    LazySeq.fromStream(data)
        .grouped(BATCH_SIZE)
        .toList()
        .forEach(value -> result.add(value.collect(Collectors.toList())));
    assertTrue(result.contains(firstBatch));
    assertTrue(result.contains(secondBatch));
    assertTrue(result.contains(thirdBatch));
    assertTrue(result.contains(fourthBatch));
}
```

## 12. 总结

在本文中，我们学习了几种在Java中完成Stream批处理的方法。我们探索了几种替代方案，从Java原生API到一些流行的库，如RxJava、Vavr和Apache Commons。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。

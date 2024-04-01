---
layout: post
title:  RoaringBitmap简介
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将了解RoaringBitmap。我们将使用一些对集合的基本操作作为对RoaringBitmap的示例。此外，我们将在Java中执行RoaringBitmap和BitSet之间的性能测试。

## 2. RoaringBitmap简介

由于其高性能和压缩比，RoaringBitmap数据结构通常用于分析、搜索和大数据项目。它背后的想法来自位图索引，这是一种有效表示数字数组的数据结构。它类似于Java [BitSet](https://www.baeldung.com/java-bitset)，但经过压缩。

压缩大整数集同时保持对单个元素的快速访问是RoaringBitmap的重要优势。RoaringBitmap在内部使用不同类型的容器来实现这一点。

## 3. RoaringBitmap的工作原理

RoaringBitmap是一组无符号整数，由不相交子集的容器组成。每个子集都有一个16位键索引，可以保存2^16范围内的值。这允许将无符号的32位整数存储为Java short，因为最坏的情况只需要16位来表示单个32位值。容器大小的选择还可以确保在最坏的情况下，容器仍然适合现代CPU的L1缓存。

下图表示RoaringBitmap结构的外观：

![](/assets/images/2023/javacollection/javaroaringbitmapintro01.png)

我们的整数的16个最高有效位是桶或块键。每个数据块代表区间(0 <= n< 2^16)中值范围的基数。此外，如果值范围内没有数据，则不会创建块。

下图是具有不同数据的RoaringBitmap示例：

![](/assets/images/2023/javacollection/javaroaringbitmapintro02.png)

在第一个块中，我们存储了2的前10个倍数。此外，在第二个块中，我们有100个从65536开始的连续整数。图片中的最后一个块具有131072到19660之间的偶数。

## 4. RoaringBitmap中的容器

RoaringBitmap中的容器主要分为三种-数组、位图和运行容器。根据分区集的特征，Run Container、Bitmap Container或Array Container是保存分区数据的容器的实现。

当我们向RoaringBitmap添加数据时，在内部，它会根据值是否适合容器键覆盖的范围来决定是创建一个新容器还是更改现有容器。

### 4.1 RoaringBitmap的数组容器

**数组容器不压缩数据，只容纳少量数据**。它占用的空间量与其保存的数据量成正比：每个都是两个字节。

数组容器使用的数据类型是short数据类型。**整数按排序顺序存储**。

另外，数组初始容量为4，最大元素数为4096，**数组容量是动态变化的**。但是RoaringBitmap在元素个数超过4096时，会在内部将数组容器转为位图容器。

让我们看一个将数据插入到RoaringBitmap中的数组容器的示例。我们有数字131090。16个最高有效位是0000000000000010，这是我们的键。低16位是0000000000010010。当我们将它转换为十进制时，它的值为18。现在，我们插入数据后，这是我们RoaringBitmap结构：

![](/assets/images/2023/javacollection/javaroaringbitmapintro03.png)

我们可以注意到，插入后，数组容器的基数对于最高16位代表的键为5。

### 4.2 RoaringBitmap的位图容器

位图容器是位集的经典实现。

RoaringBitmap使用了一个long数组来存储位图数据。数组容量恒定为1024，不像数组容器是一个动态扩展的数组。位图容器不需要找到位置。相反，它直接访问索引。

为了观察它是如何工作的，我们将使用一个简单的例子。我们将数字32786插入到RoaringBitmap中。前16位是0000000000000000。其余位是1000000000010010或十进制表示的32786。该值表示要在位图容器中设置的索引。让我们看看带有新信息的RoaringBitmap：

![](/assets/images/2023/javacollection/javaroaringbitmapintro04.png)

### 4.3 RoaringBitmap的运行容器

当位图的某个区域有大量干净的字时，运行长度编码(RLE)是最佳的容器选择。它使用short数据类型数组。

偶数索引处的值表示运行的开始，奇数索引处的值表示这些运行的长度。容器的基数是通过遍历完整的运行数组来计算的。

例如，下图向我们展示了一个包含连续整数序列的容器。然后，在RLE执行之后，容器只有四个值：

![](/assets/images/2023/javacollection/javaroaringbitmapintro05.png)

这些值表示为11后跟四个连续递增值和27后跟两个连续递增值。

这种压缩算法的工作原理取决于数据的紧凑程度或连续程度。如果我们有100个short都是连续的，它可以将它们从200字节压缩到4字节，但是如果它们都在不同的地方，编码后它会从200字节变成400字节。

## 5. RoaringBitmap中的联合

在简要介绍了RoaringBitmap之后，在我们进入代码示例之前，我们需要将[RoaringBitmap依赖项](https://search.maven.org/search?q=g:org.roaringbitmapANDa:RoaringBitmap)添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>org.roaringbitmap</groupId>
    <artifactId>RoaringBitmap</artifactId>
    <version>0.9.38</version>
</dependency>
```

集合的联合是测试RoaringBitmap的第一个操作。首先，让我们声明两个RoaringBitmap实例。第一个是A，第二个是B：

```java
@Test
void givenTwoRoaringBitmap_whenUsingOr_thenWillGetSetsUnion() {
    RoaringBitmap expected = RoaringBitmap.bitmapOf(1, 2, 3, 4, 5, 6, 7, 8);
    RoaringBitmap A = RoaringBitmap.bitmapOf(1, 2, 3, 4, 5);
    RoaringBitmap B = RoaringBitmap.bitmapOf(4, 5, 6, 7, 8);
    RoaringBitmap union = RoaringBitmap.or(A, B);
    assertEquals(expected, union);
}
```

在上面的代码中，我们声明了两个RoaringBitmap实例。我们使用RoaringBitmap提供的bitmapOf()[静态工厂方法](https://www.baeldung.com/java-constructors-vs-static-factory-methods)来创建实例。然后，我们使用or()方法执行集合并集操作。**在幕后，这个操作在设置的位图之间完成逻辑OR。这是一个线程安全的操作**。

## 6. RoaringBitmap中的交集

我们可以对集合执行的另一个有用的操作是[交集](https://www.baeldung.com/java-set-operations#2-the-intersection-of-sets)。

让我们针对交集问题实现我们的测试用例。与联合一样，交集操作在RoaringBitmap中非常简单：

```java
@Test
void givenTwoRoaringBitmap_whenUsingAnd_thenWillGetSetsIntersection() {
    RoaringBitmap expected = RoaringBitmap.bitmapOf(4, 5);
    RoaringBitmap A = RoaringBitmap.bitmapOfRange(1, 6);
    RoaringBitmap B = RoaringBitmap.bitmapOf(4, 5, 6, 7, 8);
    RoaringBitmap intersection = RoaringBitmap.and(A, B);
    assertEquals(expected, intersection);
}
```

我们使用来自RoaringBitmap类的另一个[静态工厂方法](https://www.baeldung.com/java-constructors-vs-static-factory-methods)在此测试用例中声明A集。bitmapOfRange()静态方法创建一个新的RoaringBitmap。**在底层，bitmapOfRange()方法创建一个新实例并使用add()方法将范围内的数据添加到RoaringBitmap**。在这种情况下，add()方法接收两个long值作为表示下限和上限的参数。下限是包含的。相比之下，上限被排除在结果集范围之外。add()方法接收两个int值作为参数，在当前API版本中已弃用。

然后，我们使用and()方法来执行我们的交集操作。**顾名思义，and()方法在两个集合之间执行逻辑AND操作**。此操作是线程安全的。

## 7. RoaringBitmap的区别

除了并集和交集，我们还有[集合运算的相对补集](https://www.baeldung.com/java-set-operations#4-the-relative-complement-of-sets)。

接下来，让我们使用RoaringBitmap构建我们的集合差集测试用例：

```java
@Test
void givenTwoRoaringBitmap_whenUsingAndNot_thenWillGetSetsDifference() {
    RoaringBitmap expected = RoaringBitmap.bitmapOf(1, 2, 3);
    RoaringBitmap A = new RoaringBitmap();
    A.add(1L, 6L);
    RoaringBitmap B = RoaringBitmap.bitmapOf(4, 5, 6, 7, 8);
    RoaringBitmap difference = RoaringBitmap.andNot(A, B);
    assertEquals(expected, difference);
}
```

与我们之前的代码示例一样，我们声明了两个集合A和B。对于这种情况，我们使用不同的方法来实例化A集。我们首先创建一个空的RoaringBitmap。然后，我们使用add()方法，与上一节中描述的bitmapOfRange()方法使用的方法相同。

**andNot()方法执行A和B之间的集合差**。从逻辑的角度来看，这个操作执行的是按位ANDNOT(差)运算。只要给定的位图保持不变，此操作就是线程安全的。

## 8. RoaringBitmap中的异或运算

此外，我们在RoaringBitmap中进行了XOR(异或)操作。此操作类似于[集合的相对补集](https://www.baeldung.com/java-set-operations#4-the-relative-complement-of-sets)，但结果集中省略了两个集合之间的公共元素。

我们使用xor()方法来执行此操作。让我们跳到我们的测试代码示例：

```java
@Test
void givenTwoRoaringBitmap_whenUsingXOR_thenWillGetSetsSymmetricDifference() {
    RoaringBitmap expected = RoaringBitmap.bitmapOf(1, 2, 3, 6, 7, 8);
    RoaringBitmap A = RoaringBitmap.bitmapOfRange(1, 6);
    RoaringBitmap B = RoaringBitmap.bitmapOfRange(4, 9);
    RoaringBitmap xor = RoaringBitmap.xor(A, B);
    assertEquals(expected, xor);
}
```

简而言之，RoaringBitmap类中的xor()方法执行的是按位异或运算，是线程安全的。

## 9. 比较BitSet的性能

此外，让我们在RoaringBitmap和Java BitSet之间构建一个简单的性能测试。对于每个集合类型，我们测试前面描述的操作：并集、交集、差集和异或。

我们使用[Java Microbenchmark Harness(JMH)](https://www.baeldung.com/java-microbenchmark-harness)来实现我们的性能测试。首先，我们需要将依赖项添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.36</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.36</version>
</dependency>
```

最新版本的[JMHCore](https://search.maven.org/search?q=g:org.openjdk.jmhANDa:jmh-core)和[JMH Annotation Processor](https://search.maven.org/search?q=g:org.openjdk.jmhANDa:jmh-generator-annprocess)依赖项可以在Maven Central中找到。

### 9.1 声明基准测试范围

接下来，我们在为测试设置初始条件时声明我们的类和集合：

```java
@State(Scope.Thread)
class BitSetsBenchmark {
    private RoaringBitmap rb1;
    private BitSet bs1;
    private RoaringBitmap rb2;
    private BitSet bs2;
    private final static int SIZE = 10_000_000;
}
```

最初，我们为每种类型声明了两组，BitSet和RoaringBitmap。然后，我们设置一个最大大小。SIZE变量是我们将用作集合大小的上限。

我们将在本教程中执行所有测试。**此外，我们使用Scope.Thread作为State和默认的Throughput基准模式**。

我们将在操作后生成一个新集合，以避免改变我们的输入数据结构。避免突变对于并发上下文很重要。这就是为什么，对于BitSet案例，我们将[克隆](https://www.baeldung.com/java-deep-copy#2-cloneable-interface)输入集，这样结果数据就不会改变输入集。

### 9.2 基准数据设置

接下来，让我们为测试设置数据：

```java
@Setup
public void setup() {
    rb1 = new RoaringBitmap();
    bs1 = new BitSet(SIZE);
    rb2 = new RoaringBitmap();
    bs2 = new BitSet(SIZE);
    for (int i = 0; i < SIZE / 2; i++) {
        rb1.add(i);
        bs1.set(i);
    }
    for (int i = SIZE / 2; i < SIZE; i++) {
        rb2.add(i);
        bs2.set(i);
    }
}
```

我们的两个BitSet集被初始化为SIZE。然后，对于第一个RoaringBitmap和BitSet，我们添加/设置值，最大为SIZE / 2，不包括在内。对于其他两个集合，我们将值从SIZE / 2添加到SIZE。

### 9.3 基准测试

最后，让我们编写测试代码。让我们从联合操作开始：

```java
@Benchmark
public RoaringBitmap roaringBitmapUnion() {
    return RoaringBitmap.or(rb1, rb2);
}

@Benchmark
public BitSet bitSetUnion() {
    BitSet result = (BitSet) bs1.clone();
    result.or(bs2);
    return result;
}
```

第二个操作是交集：

```java
@Benchmark
public RoaringBitmap roaringBitmapIntersection() {
    return RoaringBitmap.and(rb1, rb2);
}

@Benchmark
public BitSet bitSetIntersection() {
    BitSet result = (BitSet) bs1.clone();
    result.and(bs2);
    return result;
}
```

第三个是差集：

```java
@Benchmark
public RoaringBitmap roaringBitmapDifference() {
    return RoaringBitmap.andNot(rb1, rb2);
}

@Benchmark
public BitSet bitSetDifference() {
    BitSet result = (BitSet) bs1.clone();
    result.andNot(bs2);
    return result;
}
```

最后一个是异或运算：

```java
@Benchmark
public RoaringBitmap roaringBitmapXOR() {
    return RoaringBitmap.xor(rb1, rb2);
}

@Benchmark
public BitSet bitSetXOR() {
    BitSet result = (BitSet) bs1.clone();
    result.xor(bs2);
    return result;
}
```

### 9.4 基准测试结果

执行我们的基准测试后，得到以下结果：

```text
Benchmark                                    Mode  Cnt       Score       Error  Units
BitSetsBenchmark.bitSetDifference           thrpt   25    3890.694 ±   313.808  ops/s
BitSetsBenchmark.bitSetIntersection         thrpt   25    3542.387 ±   296.007  ops/s
BitSetsBenchmark.bitSetUnion                thrpt   25    3012.666 ±   503.821  ops/s
BitSetsBenchmark.bitSetXOR                  thrpt   25    2872.402 ±   348.099  ops/s
BitSetsBenchmark.roaringBitmapDifference    thrpt   25   12930.064 ±   527.289  ops/s
BitSetsBenchmark.roaringBitmapIntersection  thrpt   25  824167.502 ± 30176.431  ops/s
BitSetsBenchmark.roaringBitmapUnion         thrpt   25    6287.477 ±   250.657  ops/s
BitSetsBenchmark.roaringBitmapXOR           thrpt   25    6060.993 ±   607.562  ops/s
```

**我们可以注意到，RoaringBitmap比BitSet执行的性能更好**。尽管有这些结果，但我们需要考虑何时使用每种类型。

在某些情况下尝试使用压缩位图是一种浪费。例如，这可能是当我们有一个小型数据宇宙时。如果我们可以在不增加内存使用的情况下解压缩BitSet，那么压缩位图可能不适合我们。如果不需要压缩，BitSet可提供出色的速度。

## 10. 总结

在本文中，我们了解了RoaringBitmap数据结构。我们讨论了RoaringBitmap的一些操作。此外，我们在RoaringBitmap和BitSet之间进行了一些性能测试。结果，我们了解到前者的表现优于后者。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-5)上获得。
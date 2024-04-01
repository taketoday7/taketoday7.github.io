---
layout: post
title:  使用Java Stream将整数数组映射到字符串
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在本教程中，我们将了解如何使用[Java Stream](https://www.baeldung.com/java-streams)将Integer数组转换为String数组。

我们将根据我们是否拥有Integer数组或原始int值来比较我们需要采取的方法。对于Integer，我们将利用Stream<Integer\>和Integer从Object继承的方法进行转换。对于int，我们将使用专门的[IntStream](https://www.baeldung.com/java-intstream-convert)。

## 2. 从数组创建流

让我们从将数组转换为Stream开始。我们可以在这里对Integer和原始类型整数使用相同的方法，但返回类型会不同。如果我们有一个Integer数组，我们将得到一个Stream<Integer>：

```java
Integer[] integerArray = { 1, 2, 3, 4, 5 };
Stream<Integer> integerStream = Arrays.stream(integerArray);
```

如果我们改为原始类型整数数组，我们将得到一个IntStream：

```java
int[] intArray = { 1, 2, 3, 4, 5 };
IntStream intStream = Arrays.stream(intArray);
Java 8
```

IntStream为我们提供了一组我们可以使用的方法，稍后我们将使用这些方法进行类型转换。

## 3. 从Integer转换

准备好Stream<Integer\>后，我们可以继续将Integer转换为String。**由于Integer使我们可以访问Object的所有方法，因此我们可以将Object.toString()与map()一起使用**：

```java
String[] convertToStringArray(Integer[] input) {
    return Arrays.stream(input)
        .map(Object::toString)
        .toArray(String[]::new);
}
```

然后让我们使用convertToStringArray()来转换Integer数组并确认返回的String数组如我们所期望的那样出现：

```java
@Test
void whenConvertingIntegers_thenHandleStreamOfIntegers() {
    Integer[] integerNumbers = { 1, 2, 3, 4, 5 };
    String[] expectedOutput = { "1", "2", "3", "4", "5" };

    String[] strings = convertToStringArray(integerNumbers);

    Assert.assertArrayEquals(expectedOutput, strings);
}
```

## 4. 从原始类型整数转换

现在让我们看看如何处理以整数数组创建的IntStream。

### 4.1 返回数组

**准备好IntStream后，我们可以使用IntStream.mapToObj()进行转换**：

```java
String[] convertToStringArray(int[] input) {
    return Arrays.stream(input)
        .mapToObj(Integer::toString)
        .toArray(String[]::new);
}
```

mapToObj()方法使用我们提供的Integer.toString()方法返回一个对象值Stream。因此，在方法的那个阶段之后，我们有一个Stream<String\>可以使用，我们可以简单地使用toArray()收集内容。

然后我们可以再次检查使用convertToStringArray()是否为我们提供了一个匹配输入整数数组的String数组：

```java
@Test
void whenConvertingInts_thenHandleIntStream() {
    int[] intNumbers = { 1, 2, 3, 4, 5 };
    String[] expectedOutput = { "1", "2", "3", "4", "5" };

    String[] strings = convertToStringArray(intNumbers);

    Assert.assertArrayEquals(expectedOutput, strings);
}
```

此外，如果我们想在中途使用Integer类型带来的任何好处，我们可以使用boxed()：

```java
String[] convertToStringArrayWithBoxing(int[] input) {
    return Arrays.stream(input)
        .boxed()
        .map(Object::toString)
        .toArray(String[]::new);
}
```

### 4.2 返回单个字符串

另一个潜在的用例是将我们的整数数组转换为单个字符串。我们可以重用上面的大部分代码以及[Stream.collect()](https://www.baeldung.com/java-8-collectors)将Stream简化为String。collect()方法用途广泛，可以让我们将Stream终止为多种类型。在这里，我们将传递Collectors.joining(", ")，这样数组中的每个元素都将拼接成一个字符串，它们之间有一个逗号：

```java
String convertToString(int[] input){
    return Arrays.stream(input)
        .mapToObj(Integer::toString)
        .collect(Collectors.joining(", "));
}
```

然后我们可以测试返回的字符串是否符合我们的预期：

```java
@Test
void givenAnIntArray_whenUsingCollectorsJoining_thenReturnCommaSeparatedString(){
    int[] intNumbers = { 1, 2, 3, 4, 5 };
    String expectedOutput = "1, 2, 3, 4, 5";

    String string = convertToString(intNumbers);

    Assert.assertEquals(expectedOutput, string);
}
```

## 5. 总结

在本文中，我们学习了如何使用Java Streams将Integer或原始类型整数的数组转换为String。我们看到，在处理Integer时，我们需要期望一个Stream<Integer\>。但是，当改为处理原始类型整数时，我们期望IntStream。

然后我们研究了如何处理这两种Stream类型以得到一个String数组。map()方法可用于Stream<Integer\>，mapToObj()可用于IntStream。最后，我们看到了如何使用Collectors.joining()返回单个字符串。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。

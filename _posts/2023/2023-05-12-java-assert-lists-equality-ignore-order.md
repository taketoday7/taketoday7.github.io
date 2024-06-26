---
layout: post
title:  断言两个集合以不同顺序相等
category: assertion
copyright: assertion
excerpt: AssertJ
---

## 1. 概述

有时在编写单元测试时，我们需要对集合进行与顺序无关的比较，在这个教程中，我们介绍如何编写此类单元测试的不同示例。

## 2. 项目准备

根据[List#equals](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#equals(java.lang.Object))的Java文档，如果两个集合以相同的顺序包含相同的元素，则它们是相等的。因此，我们不能只使用equals方法，因为我们想做与顺序无关的比较。

在本教程中，我们使用以下三个集合作为我们测试的集合对象：

```java
public class OrderAgnosticListComparisonUnitTest {
    private final List<Integer> first = Arrays.asList(1, 3, 4, 6, 8);
    private final List<Integer> second = Arrays.asList(8, 1, 6, 3, 4);
    private final List<Integer> third = Arrays.asList(1, 3, 3, 6, 6);
}
```

## 3. 使用Junit

JUnit是Java生态系统中用于单元测试的知名框架，我们可以使用下面的逻辑来使用assertTrue和assertFalse 法比较两个集合的相等性。这里，我们检查两个集合的大小并检查第一个集合是否包含第二个集合的所有元素，反之亦然。尽管这种解决方案有效，但它的可读性并不高，因此现在我们看看一些替代方案：

```java
@Test
void whenTestingForOrderAgnosticEquality_ShouldBeTrue() {
	assertTrue(first.size() == second.size() && first.containsAll(second) && second.containsAll(first));
}
```

在第一个测试中，在我们检查两个集合中的元素是否相同之前，先比较了两个集合的大小，当这两个条件都返回true时，我们的测试将通过。

现在让我们来看一个失败的测试：

```java
@Test
void whenTestingForOrderAgnosticEquality_ShouldBeFalse() {
	assertFalse(first.size() == third.size() && first.containsAll(third) && third.containsAll(first));
```

相比之下，在这个版本的测试中，虽然两个集合的大小相同，但并非所有元素都匹配。

## 4. 使用AssertJ

AssertJ是一个开源社区驱动的库，用于在Java测试中编写流畅和丰富的断言。

要在maven项目中使用它，我们可以在pom.xml文件中添加assertj-core依赖项：

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.21.0</version>
</dependency>
```

让我们编写一个测试来比较相同元素和相同大小的两个集合实例的相等性：

```java
@Test
void whenTestingForOrderAgnosticEqualityBothList_ShouldBeEqual() {
	assertThat(first).hasSameElementsAs(second);
}
```

在这个例子中，我们首先验证是否以任何顺序包含给定集合的所有元素，而不包含其他元素；这种方法的主要限制是hasSameElementsAs方法忽略了重复项。

例如下面的例子：

```java
@Test
void whenTestingForOrderAgnosticEqualityBothList_ShouldNotBeEqual() {
	List<String> a = Arrays.asList("a", "a", "b", "c");
	List<String> b = Arrays.asList("a", "b", "c");
    
	assertThat(a).hasSameElementsAs(b);
}
```

在这个测试中，虽然两个集合有相同的元素，但它们的大小不相等，但是断言仍然是正确的，因为它忽略了重复项。为了让它正常断言为false，我们需要为两个集合添加大小检查：

```java
assertThat(a).hasSize(b.size()).hasSameElementsAs(b);
```

此时使用方法hasSameElementsAs()确实会按预期失败。

## 5. 使用Hamcrest

如果我们已经在使用Hamcrest或者想使用它来编写单元测试，这里是我们如何使用Matchers#containsInAnyOrder方法进行顺序不可知比较。

要在maven项目中使用Hamcrest，我们可以在pom.xml文件中添加hamcrest-all依赖项：

```xml
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>1.3</version>
</dependency>
```

下面是一个简单的测试：

```java
@Test
void whenTestingForOrderAgnosticEquality_ShouldBeEqual() {
	MatcherAssert.assertThat(first, Matchers.containsInAnyOrder(second.toArray()));
}
```

这里的containsInAnyOrder方法为Iterables创建了一个与顺序无关的匹配器，它与已检查的Iterable元素进行匹配。此测试匹配两个集合的元素，忽略集合中元素的顺序。

值得庆幸的是，这个解决方案不会出现与上一节中提到的相同的问题，因此我们不需要明确比较大小。

## 6. 使用Apache Commons

除了JUnit、Hamcrest、AssertJ之外，我们可以使用的另一个库是Apache CollectionUtils，它为涵盖广泛用例的常见操作提供工具方法，并帮助我们避免编写样板代码。

要在maven项目中使用它，我们可以在pom.xml文件中添加commons-collections4依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
    <scope>test</scope>
</dependency>
```

下面是一个使用CollectionUtils的测试：

```java
@Test
void whenTestingForOrderAgnosticEquality_ShouldBeTrueIfEqualOtherwiseFalse() {
	assertTrue(CollectionUtils.isEqualCollection(first, second));
	assertFalse(CollectionUtils.isEqualCollection(first, third));
}
```

如果给定的集合包含具有相同基数的完全相同的元素，则isEqualCollection方法返回true；否则，它返回false。

## 7. 总结

在本文中，我们介绍了如何断言两个List实例的相等性，其中两个集合的元素顺序不需要相同。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-assertions)上获得。
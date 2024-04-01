---
layout: post
title:  Java Stream：多个过滤器与复杂条件
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在本教程中，我们将比较过滤Java Stream的不同方法。首先，我们将看看哪种解决方案可以带来更具可读性的代码。之后，我们将从性能的角度比较解决方案。

## 2. 可读性

首先，我们将从可读性的角度比较这两个解决方案。对于本节中的代码示例，我们将使用Student类：

```java
public class Student {

    private String name;
    private int year;
    private List<Integer> marks;
    private Profile profile;

    // constructor getters and setters
}
```

我们的目标是根据以下三个规则过滤学生流：

-   profile必须是Profile.PHYSICS
-   marks和应大于3
-   mark平均值应大于50

### 2.1 多个过滤器

Stream API允许链接多个过滤器。我们可以利用这一点来满足所描述的复杂过滤标准。此外，如果我们想否定条件，我们可以使用not谓词。

这种方法可以产生干净且易于理解的代码：

```java
@Test
public void whenUsingMultipleFilters_dataShouldBeFiltered() {
    List<Student> filteredStream = students.stream()
        .filter(s -> s.getMarksAverage() > 50)
        .filter(s -> s.getMarks().size() > 3)
        .filter(not(s -> s.getProfile() == Student.Profile.PHYSICS))
        .collect(Collectors.toList());

    assertThat(filteredStream).containsExactly(mathStudent);
}
```

### 2.2 复杂条件下的单过滤器

另一种方法是使用具有更复杂条件的单个过滤器。

不幸的是，生成的代码将更难阅读：

```java
@Test
public void whenUsingSingleComplexFilter_dataShouldBeFiltered() {
    List<Student> filteredStream = students.stream()
        .filter(s -> s.getMarksAverage() > 50 
            && s.getMarks().size() > 3 
            && s.getProfile() != Student.Profile.PHYSICS)
        .collect(Collectors.toList());

    assertThat(filteredStream).containsExactly(mathStudent);
}
```

不过，我们可以通过将几个条件提取到一个单独的方法中来使其变得更好：

```java
public boolean isEligibleForScholarship() {
    return getMarksAverage() > 50
        && marks.size() > 3
        && profile != Profile.PHYSICS;
}
```

因此，我们将隐藏复杂的条件，并赋予过滤条件更多的含义：

```java
@Test
public void whenUsingSingleComplexFilterExtracted_dataShouldBeFiltered() {
    List<Student> filteredStream = students.stream()
        .filter(Student::isEligibleForScholarship)
        .collect(Collectors.toList());

    assertThat(filteredStream).containsExactly(mathStudent);
}
```

**这将是一个很好的解决方案，尤其是当我们可以将过滤器逻辑封装在我们的模型中时**。

## 3. 性能

我们已经看到使用多个过滤器可以提高代码的可读性。另一方面，这将意味着创建多个对象并且可能导致性能损失。为了演示这一点，我们将过滤不同大小的流并对它们的元素执行多次检查。

在此之后，我们将计算总处理时间(以毫秒为单位)并比较两个解决方案。此外，我们将在我们的测试中包含并行流和简单的旧for循环：

![](/assets/images/2023/javastream/javastreamsmultiplefiltersvscondition01.png)

**如我们所见，使用复杂条件会带来性能提升**。

但是，对于小样本量，差异可能并不明显。

## 4. 条件顺序

无论我们使用单个过滤器还是多个过滤器，如果未按最佳顺序执行检查，过滤都可能导致性能下降。

### 4.1 过滤掉许多元素的条件

假设我们有一个包含100个整数的流，并且我们希望找到小于20的偶数。

如果我们首先检查数字的奇偶性，我们最终会得到总共150次检查。这是因为每次都会评估第一个条件，而第二个条件只会对偶数进行评估。

```java
@Test
public void givenWrongFilterOrder_whenUsingMultipleFilters_shouldEvaluateManyConditions() {
    long filteredStreamSize = IntStream.range(0, 100).boxed()
        .filter(this::isEvenNumber)
        .filter(this::isSmallerThanTwenty)
        .count();

    assertThat(filteredStreamSize).isEqualTo(10);
    assertThat(numberOfOperations).hasValue(150);
}
```

另一方面，如果我们颠倒过滤器的顺序，我们只需要总共120次检查就可以正确地过滤流。**因此，应首先评估过滤掉大多数元素的条件**。

### 4.2 缓慢或繁重的条件

有些条件可能会很慢。例如，如果其中一个过滤器需要执行一些繁重的逻辑或通过网络进行外部调用。为了获得更好的性能，我们将尝试尽可能少地评估这些条件。**因此，我们将尝试仅在满足所有其他条件时才对它们进行评估**。

## 5. 总结

在本文中，我们分析了过滤Java Stream的不同方式。首先，我们从可读性的角度比较了这两种方法。我们发现多个过滤器可以提供更易于理解的过滤条件。

之后，我们从性能角度对解决方案进行了比较。我们了解到，使用复杂的条件并因此创建更少的对象将带来更好的整体性能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。

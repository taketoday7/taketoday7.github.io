---
layout: post
title:  使用jqwik进行基于属性的测试
category: test-lib
copyright: test-lib
excerpt: Jqwik
---

## 1. 简介

在本文中，我们将介绍基于属性的测试以及如何使用[jqwik](https://jqwik.net/)库在Java中执行此操作。

## 2. 参数化测试

在了解基于属性的测试之前，让我们简要介绍一下[参数化测试](https://www.baeldung.com/parameterized-tests-junit-5)。**在参数化测试中，我们可以编写单个测试函数，然后使用许多不同的参数调用它**。例如：

```java
@ParameterizedTest
@CsvSource({"4,2,2", "6,2,3", "6,3,2"})
void testIntegerDivision(int x, int y, int expected) {
    int answer = calculator.divide(x, y);
    assertEquals(expected, answer);
}
```

这使我们能够相对轻松地测试许多不同的输入集，并能够让我们确定一组适当的测试用例来查看和运行这些测试，接下来的挑战就变成了决定这些测试用例应该是什么。显然，我们无法测试每一个可能的值集-这是不可行的。相反，我们会尝试确定有趣的案例并进行测试。

对于这个例子，我们可以测试以下内容：

- 一些正常情况
- 将数字除以本身-总是得到“1”
- 将数字除以1-始终得到原始数字
- 正数除以负数-总是得到负数

等等。但是，这是假设我们可以想到所有这些情况。当我们没有想到某些案例呢？例如，如果我们将一个数字除以0会发生什么？我们没有想到要测试这一点，所以我们不知道结果会是什么。

## 3. 基于属性的测试

**相反，如果我们可以以编程方式生成测试输入怎么办**？

实现此目的的一个明显方法是进行参数化测试，在循环中生成输入：

```java
@ParameterizedTest
@MethodSource("provideIntegerInputs")
void testDivisionBySelf(int input) {
    int answer = calculator.divide(input, input);
    assertEquals(answer, 1);
}

private static Stream<Arguments> provideIntegerInputs() {
    return IntStream.rangeClosed(Integer.MIN_VALUE, Integer.MAX_VALUE)
        .mapToObj(i -> Arguments.of(i));
}
```

这保证可以找到任何边缘情况。然而，这样做的代价是巨大的。它将测试每一个可能的整数值-即4294967296个不同的测试用例。即使每次测试只需要1毫秒，运行也需要近50天。

**基于属性的测试是这个想法的变体。我们将根据我们定义的一组属性生成有趣的测试用例，而不是生成每个测试用例**。

例如，对于我们的除法示例，我们可能只测试-20到+20之间的每个数字。如果我们假设任何意外情况都在这个范围内，那么这将具有足够的代表性，同时也更容易管理。在这种情况下，我们的属性只是“介于-20和+20之间”：

```java
private static Stream<Arguments> provideIntegerInputs() {
    return IntStream.rangeClosed(-20, +20).mapToObj(i -> Arguments.of(i));
}
```

## 4. jqwik入门

**[jqwik](https://jqwik.net/)是一个Java测试库，为我们实现基于属性的测试**。它为我们提供了非常容易和高效地进行此类测试的工具，这包括生成数据集的能力，但它也与[JUnit 5](https://www.baeldung.com/junit-5)集成。

### 4.1 添加依赖项

为了使用jqwik，我们需要首先将其添加到我们的项目中：

```xml
<dependency>
    <groupId>net.jqwik</groupId>
    <artifactId>jqwik</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
</dependency>
```

最新版本可以在[Maven中央仓库](https://mvnrepository.com/artifact/net.jqwik/jqwik)中找到。

我们还需要确保在项目中正确设置JUnit 5，否则我们无法运行使用jqwik编写的测试。

### 4.2 第一次测试

现在我们已经设置了jqwik，让我们用它编写一个测试。我们将测试一个数字除以它本身是否返回1，就像我们之前所做的一样：

```java
@Property
public void divideBySelf(@ForAll int value) {
    int result = divide(value, value);
    assertEquals(result, 1);
}
```

仅此而已，那么我们做了什么？

首先，测试用@Property而不是@Test进行标注，这告诉JUnit使用jqwik运行器运行它。

接下来，我们有一个用@ForAll标注的测试参数，这告诉jqwik为该参数生成一组有趣的值，然后我们可以在测试中使用这些值。在这种情况下，我们没有任何约束，因此它只会从所有整数的集合中生成值。

如果我们运行这个会发生什么？

![](/assets/images/2023/test-lib/javajqwikpropertybasedtesting01.png)

jqwik运行了13个不同的测试，然后才找到一个失败的测试。失败的是输入“0”，这导致它抛出ArithmeticException。

**这立刻就在我们的代码中，在极少量的测试代码中发现了问题**。

## 5. 定义属性

我们已经了解了如何使用jqwik为我们的测试提供值。在本例中，我们的测试接收一个整数，对值的大小没有任何限制。

**jqwik可以为一组标准类型提供值-包括字符串、数字、布尔值、枚举和集合等**。所需要做的就是使用本身用@ForAll标注的参数编写我们的测试，这将告诉jqwik使用为此属性生成的不同值重复运行我们的测试。

我们可以拥有测试所需的任意多个属性：

```java
@Property
public void additionIsCommutative(@ForAll int a, @ForAll int b) {
    assertEquals(a + b, b + a);
}
```

我们甚至可以根据需要混合类型。

### 5.1 约束属性

**但是，我们通常需要对这些参数施加一些限制**。例如，当测试除法时，我们不能使用0作为分母，因此这需要成为我们测试中的一个约束。

我们可以编写测试来检测这些情况并跳过它们，但是，这仍将被视为测试用例。特别是，jqwik在决定测试通过之前仅运行有限数量的测试用例。如果我们短路这一点，那么我们就失去了使用属性测试的意义。

**相反，jqwik允许我们约束属性，以便生成的值都在范围内**。例如，我们可能想重复我们的除法测试，但仅限于正数：

```java
@Property
public void dividePositiveBySelf(@ForAll @Positive int value) {
    int result = divide(value, value);
    assertEquals(result, 1);
}
```

在这里，我们用@Positive标注了属性，这将导致jqwik将值限制为仅正值-即任何大于0的值。

jqwik附带了一组可应用于标准属性类型的约束-例如：

- 是否包含空值
- 数字的最小值和最大值-无论是整数还是其他值
- 字符串和集合的最小和最大长度
- 字符串中允许的字符

### 5.2 自定义约束

通常，标准约束足以满足我们的需求。但是，在某些情况下，我们可能需要更加规范。**jqwik使我们能够编写自己的生成函数，这些函数可以执行我们需要的任何操作**。

例如，我们知道0是唯一不能用于除法的情况，其他所有数字都应该可以正常工作。但是，jqwik的标准注解不允许我们生成除一个值之外的所有内容，我们能做的最好的事情就是从整个范围生成数字。

因此，我们将自己生成数字。这将使我们能够从整个整数范围生成除0之外的任何数字：

```java
@Property
public void divideNonZeroBySelf(@ForAll("nonZeroNumbers") int value) {
    int result = divide(value, value);
    assertEquals(result, 1);
}

@Provide
Arbitrary<Integer> nonZeroNumbers() {
    return Arbitraries.integers().filter(v -> v != 0);
}
```

为了实现这一目标，我们在测试类中编写了一个新方法，用@Provide标注，返回Arbitrary<Integer\>。然后，我们在@ForAll注解上指示使用此方法生成，而不是仅从参数类型的整个可能值集中生成。这样做意味着jqwik现在将使用我们的生成方法而不是默认的生成方法。

如果我们尝试这样做，我们会发现我们从整个整数值范围中获得了大范围的正数和负数，而0永远不会出现-因为我们明确地将其过滤掉。

### 5.3 假设

**有时我们需要拥有多个属性，但它们之间存在约束**。例如，我们可能想要测试一个大数除以一个小数是否总是返回大于1的结果。我们可以轻松地为每个数定义属性，但不能根据一个属性来定义另一个属性。

相反，这可以通过使用假设来完成。我们告诉我们的测试，我们假设一些前提条件，只有当该前提条件通过时，我们才会运行测试：

```java
@Property
public void divideLargeBySmall(@ForAll @Positive int a, @ForAll @Positive int b) {
    Assume.that(a > b);

    int result = divide(a, b);
    assertTrue(result >= 1);
}
```

运行这个测试将会通过，但让我们看看输出：

![](/assets/images/2023/test-lib/javajqwikpropertybasedtesting02.png)

**我们尝试了1000次，但是，我们实际上只检查了其中的498个。这是因为生成的测试中有502个未满足假设，因此未运行**。

jqwik有一个可配置的最大丢弃率，这是丢弃的测试与尝试的测试的比率，默认设置为“5”。这意味着，如果我们的假设对于每次尝试都拒绝超过5个案例，那么测试将被视为失败，因为没有足够的可行测试数据。稍后我们将看到如何更改这些设置。

## 6. 结果缩小

jqwik可以非常有效地找到我们的代码未通过测试的情况。但是，如果生成的案例相当模糊，这并不总是有用。

**相反，jqwik将尝试将任何失败案例“缩小”到最小案例**，这将帮助我们更直接地找到需要考虑的任何边缘情况。

让我们尝试一个看似简单的案例，对数字进行平方应该产生大于输入的结果。

```java
@Property
public void square(@ForAll @Positive int a) {
    int result = a * a;
    assertTrue(result >= a);
}
```

但是，如果我们运行此测试，则会失败：

![](/assets/images/2023/test-lib/javajqwikpropertybasedtesting03.png)

在这里我们有一些额外的信息。“Original Sample”是第一次测试失败的生成值，“Shrunk Sample”是jqwik随后将其缩小到但仍然失败的样本。

但为什么这会失败呢？这没有什么不寻常的，那么我们自己来尝试一下吧。46341的平方等于2147488281，这恰好比整数所能容纳的要大。但是，46340平方适合该范围。因此，我们确实在这里发现了一个令人惊讶的优势-我们无法对46341进行平方并将结果存储在整数字段中。

## 7. 重新运行测试

**如果我们的测试失败，我们通常会想要修复错误，然后重新运行测试。但是，如果生成的测试用例是随机的，则重新运行测试可能不会遇到与之前相同的失败情况**。

jqwik为我们提供了一些工具来帮助解决这个问题。默认情况下，运行之前失败的测试将从失败的确切情况开始，然后生成新的条件，这为我们提供了对之前失败案例的非常快速的反馈。还可以将其配置为再次运行全套测试，但重新使用相同的随机种子。这意味着所有测试用例的运行都与以前完全相同-包括通过和失败的测试用例。

但是，这两种情况都需要一些本地存储。详细信息默认存储在本地文件系统上的文件中-.jqwik-database，这意味着如果CI构建中的测试失败，则存储的详细信息将在运行之间丢失。

另一种选择是我们可以显式地将固定种子传递到测试中，这将导致测试完全像随机选择种子一样运行。测试运行的输出还为我们提供了所使用的种子，因此我们可以将其复制到本地测试运行中以准确重现它的作用。

## 8. 配置测试

**jqwik附带了一些开箱即用的合理默认设置，其中许多我们已经看到过**。例如，它将运行1000次迭代来查找任何失败的案例。在许多情况下，这完全没问题。但是，如果我们需要对此进行一些更改，也可以做到。

其中许多可以直接在@Property注解上配置：

```java
@Property(tries = 5000, shrinking = ShrinkingMode.OFF, generation = GenerationMode.RANDOMIZED)
```

该注解现在将：

- 运行测试5000次迭代而不是默认迭代
- 禁用结果缩小
- 指示更倾向随机生成属性而不是详尽无遗

除此之外，我们还可以配置假设的丢弃率、如何重新运行失败测试的详细信息以及如何以完全相同的方式随机生成值的详细信息。

我们可以通过在测试类上使用@PropertyDefaults注解来执行完全相同的操作，然后它将配置此类中的每个测试方法。许多设置也可以在位于类路径根目录的junit-platform.properties文件中配置-例如，在src/test/resources中。这样做将为整个项目配置默认值，而不仅仅是一组测试。

## 9. 总结

在这里我们看到了jqwik库的介绍，这只是这个库可以实现的一小部分，但希望足以看到属性测试可以给我们的项目带来的力量。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/jqwik)上找到。
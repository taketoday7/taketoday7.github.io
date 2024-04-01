---
layout: post
title:  使用OAT技术的灰盒测试
category: test
copyright: test
excerpt: 灰盒测试
---

## 1. 概述

灰盒测试有助于我们建立足够的测试覆盖率，而无需测试每个可能的情况。

在本教程中，我们将研究该方法以及如何使用正交数组测试(OAT)技术来练习它。

最后，我们将确定使用灰盒测试的优点和缺点。

## 2. 什么是灰盒测试？

首先，让我们比较一下[白盒和黑盒测试](https://www.baeldung.com/cs/testing-white-box-vs-black-box)方法，然后了解灰盒测试。

白盒测试是指测试我们完全已知的算法的一部分。因此，我们可以测试该算法的所有路径。正因为如此，白盒测试可能会产生大量的测试场景。

黑盒测试意味着测试应用程序的外部视角。换句话说，我们对实现的算法一无所知，而且更难测试它的所有路径。因此，我们专注于验证有限数量的测试场景。

灰盒测试使用有限的信息，通常在白盒测试中可用。然后，它使用黑盒测试技术来生成具有可用信息的测试场景。

因此，我们最终得到的测试场景少于白盒测试。但是，**这些场景涵盖的功能比黑盒测试更多**。

因此，**灰盒测试是黑盒测试技术和白盒测试知识的结合**。

## 3. 进行灰盒测试

在本节中，我们将使用OAT技术对佣金计算器演示应用程序进行灰盒测试。

### 3.1 创建被测系统

在测试之前，让我们首先创建一个应用程序，根据四个属性来计算销售人员的平均佣金：

-   销售人员级别：L1、L2或L3
-   合同类型：全职委托、承包商或自由职业者
-   资历：初级、中级、高级
-   销售额的影响：低、中、高

为此，让我们创建SalaryCommissionPercentageCalculator类来满足上述要求：

```java
public class SalaryCommissionPercentageCalculator {
    public BigDecimal calculate(Level level, Type type, Seniority seniority, SalesImpact impact) {
        return BigDecimal.valueOf(DoubleStream.of(
                                level.getBonus(),
                                type.getBonus(),
                                seniority.getBonus(),
                                impact.getBonus(),
                                type.getBonus())
                        .average()
                        .orElse(0))
                .setScale(2, RoundingMode.CEILING);
    }

    public enum Level {
        L1(0.06), L2(0.12), L3(0.2);
        private double bonus;

        Level(double bonus) {
            this.bonus = bonus;
        }

        public double getBonus() {
            return bonus;
        }
    }

    public enum Type {
        FULL_TIME_COMMISSIONED(0.18), CONTRACTOR(0.1), FREELANCER(0.06)

        // bonus field, constructor and getter
    }

    public enum Seniority {
        JR(0.8), MID(0.13), SR(0.19)

        // bonus field, constructor and getter
    }

    public enum SalesImpact {
        LOW(0.06), MEDIUM(0.12), HIGH(0.2)

        // bonus field, constructor and getter
    }
}
```

上面的代码定义了4个枚举来映射销售人员的属性。每个枚举都包含一个bonus字段，表示每个属性的佣金百分比。

calculate()方法使用[double原始类型流](https://www.baeldung.com/java-8-primitive-streams)来计算所有百分比的平均值。

最后，我们使用[BigDecimal](https://www.baeldung.com/java-bigdecimal-biginteger)类的setScale()方法将平均结果四舍五入到小数点后两位。

### 3.2 OAT技术简介

OAT技术基于[Dr. Genichi Taguchi](https://en.wikipedia.org/wiki/Genichi_Taguchi)提出的田口设计实验。该实验允许我们仅考虑所有输入组合的一个子集来测试变量之间的交互作用。

这个想法是在建立实验时只考虑变量值之间的双因素相互作用，而忽略重复的相互作用。**这意味着每个变量的值与实验子集中的另一个变量的值精确交互一次**。当我们构建测试场景时，这一点将变得清晰。

变量及其值用于构造正交数组。**正交数组是一个数字数组，其中每行代表一个唯一的变量组合**。这些列表示可以采用多个值之一的单个变量。

我们可以将正交数组表示为val^var，其中val是它们假定的值的数量，var是输入变量的数量。在我们的例子中，我们有4个变量，每个变量假设3个值。因此，val等于3，var等于4。

最后，正确的正交数组是3^4，在Taguchi的设计中也称为“L9：3级4因子”。

### 3.3 获取正交数组

正交数组的计算可能过于复杂且计算量大。出于这个原因，OAT测试的设计者通常使用已映射数组的列表。因此，我们可以使用该[数组目录](https://www.york.ac.uk/depts/maths/tables/orthogonal.htm)来找到正确的数组。在我们的例子中，提供的目录中的正确数组是L9 3级4因子数组：

| Scenario # | var 1 | var 2 | var 3 | var 4 |
| ---------- | ----- | ----- | ----- | ----- |
| 1          | val 1 | val 1 | val 1 | val 1 |
| 2          | val 1 | val 2 | val 3 | val 2 |
| 3          | val 1 | val 3 | val 2 | val 3 |
| 4          | val 2 | val 1 | val 3 | val 3 |
| 5          | val 2 | val 2 | val 2 | val 1 |
| 6          | val 2 | val 3 | val 1 | val 2 |
| 7          | val 3 | val 1 | val 2 | val 2 |
| 8          | val 3 | val 2 | val 1 | val 3 |
| 9          | val 3 | val 3 | val 3 | val 1 |

上表包含我们添加到3^4正交数组的两个额外标头。第一行的标题定义变量，而第一列定义测试场景编号。

值得注意的是，**在所有场景中，两个值只交互一次**。我们不会在其他场景中重复同一对值。例如，var1=val1和var2=val1对仅出现在第一个测试场景中。

### 3.4 映射变量及其值

现在，我们必须按照变量在代码中出现的相同顺序将变量及其值替换到我们的正交数组中。因此，例如，var1对应于定义的第一个枚举Level，其中Level下面的val0是它的第一个值L1。

映射所有变量后，我们得到下面的填充表：

| Scenario # | Level | Type                   | Seniority | SalesImpact |
| ---------- | ----- | ---------------------- | --------- | ----------- |
| 1          | L1    | FULL_TIME_COMMISSIONED | JR        | LOW         |
| 2          | L1    | CONTRACTOR             | SR        | MEDIUM      |
| 3          | L1    | FREELANCER             | MID       | HIGH        |
| 4          | L2    | FULL_TIME_COMMISSIONED | SR        | HIGH        |
| 5          | L2    | CONTRACTOR             | MID       | LOW         |
| 6          | L2    | FREELANCER             | JR        | MEDIUM      |
| 7          | L3    | FULL_TIME_COMMISSIONED | MID       | MEDIUM      |
| 8          | L3    | CONTRACTOR             | JR        | HIGH        |
| 9          | L3    | FREELANCER             | SR        | LOW         |


上表的每一行对应一个测试场景，使用每个对应单元格的值。

### 3.5 配置JUnit 5

本文的主要重点是使用OAT灰盒技术进行灰盒测试。因此，为简单起见，我们将使用简单的单元测试来说明它。

首先，我们需要在项目中[配置JUnit 5](https://www.baeldung.com/junit-5)。为此，让我们将其最新的依赖项[junit-jupiter-engine](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

### 3.6 创建测试类

让我们定义SalaryCommissionPercentageCalculatorUnitTest类：

```java
class SalaryCommissionPercentageCalculatorUnitTest {
    private SalaryCommissionPercentageCalculator testTarget = new SalaryCommissionPercentageCalculator();

    @ParameterizedTest
    @MethodSource("provideReferenceTestScenarioTable")
    void givenReferenceTable_whenCalculateAverageCommission_thenReturnExpectedResult(Level level,
                                                                                     Type type, Seniority seniority, SalesImpact impact, double expected) {
        BigDecimal got = testTarget.calculate(level, type, seniority, impact);
        assertEquals(BigDecimal.valueOf(expected), got);
    }

    private static Stream<Arguments> provideReferenceTestScenarioTable() {
        return Stream.of(
                Arguments.of(L1, FULL_TIME_COMMISSIONED, JR, LOW, 0.26),
                Arguments.of(L1, CONTRACTOR, SR, MEDIUM, 0.12),
                Arguments.of(L1, FREELANCER, MID, HIGH, 0.11),
                Arguments.of(L2, FULL_TIME_COMMISSIONED, SR, HIGH, 0.18),
                Arguments.of(L2, CONTRACTOR, MID, LOW, 0.11),
                Arguments.of(L2, FREELANCER, JR, MEDIUM, 0.24),
                Arguments.of(L3, FULL_TIME_COMMISSIONED, MID, MEDIUM, 0.17),
                Arguments.of(L3, CONTRACTOR, JR, HIGH, 0.28),
                Arguments.of(L3, FREELANCER, SR, LOW, 0.12)
        );
    }
}
```

为了了解发生了什么，让我们分解代码。

测试方法使用[JUnit 5 @ParameterizedTest](https://www.baeldung.com/parameterized-tests-junit-5)和@MethodSource标注来使用方法作为输入数据提供者。

provideReferenceTestScenarioTable()提供了3.4节的正交数组中相同的数据，作为Arguments流。每个Argument.of()调用对应一个测试场景和calculate()调用的预期结果。

最后，我们使用提供的参数调用calculate()并使用assertEquals()将实际结果与预期结果进行比较。

## 4. 灰盒测试的优缺点

在我们的示例中，calculate()输入的排列总数为81。**我们使用OAT将这个数字减少到9，同时保持良好的测试覆盖率**。

如果输入大小变得太大，尝试所有输入组合可能会变得困难。例如，在具有10个变量和10个值的系统中，场景总数为10e10。测试如此多的场景是不切实际的，我们可以通过使用OAT来减小这个数字，避免输入的[组合爆炸](https://en.wikipedia.org/wiki/Combinatorial_explosion)。

因此，**OAT技术的主要优点是在不损失测试覆盖率的情况下提高了测试代码的可维护性和开发速度**。

另一方面，**OAT技术和灰盒测试通常具有无法涵盖所有可能的输入排列的缺点**。因此，我们可能会错过一个基本的测试场景或一个有问题的边缘情况。

## 5. 总结

在本文中，我们研究了OAT技术以了解灰盒测试。

使用这种技术，我们大大减少了测试场景的数量。但是，我们必须正确评估何时使用它，因为我们可能会错过重要的边缘情况。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-techniques)上获得。
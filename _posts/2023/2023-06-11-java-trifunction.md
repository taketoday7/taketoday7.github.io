---
layout: post
title:  Java中的TriFunction接口
category: java
copyright: java
excerpt: Java函数式编程
---

## 1. 概述

在本文中，我们将定义一个TriFunction[函数式接口](https://www.baeldung.com/java-8-functional-interfaces)，它表示接收三个参数并计算结果的函数。稍后，我们还将看到一个使用[Vavr](https://www.baeldung.com/vavr)库的内置Function3的示例。

## 2. 创建我们自己的TriFunction接口

从版本8开始，Java定义了[BiFunction](https://www.baeldung.com/java-bifunction-interface)函数式接口。它表示接收两个参数并计算其结果的函数。为了允许函数组合，它还提供了一个andThen()方法，该方法将另一个Function应用于BiFunction的结果。

**类似地，我们将定义我们的TriFunction接口并为其提供andThen()方法**：

```java
@FunctionalInterface
public interface TriFunction<T, U, V, R> {

    R apply(T t, U u, V v);

    default <K> TriFunction<T, U, V, K> andThen(Function<? super R, ? extends K> after) {
        Objects.requireNonNull(after);
        return (T t, U u, V v) -> after.apply(apply(t, u, v));
    }
}
```

让我们看看如何使用这个接口。我们将定义一个接收三个Integer的函数，将前两个操作数相乘，然后相加最后一个操作数：

```java
static TriFunction<Integer, Integer, Integer, Integer> multiplyThenAdd = (x, y, z) -> x * y + z;
```

请注意，只有当前两个操作数的乘积低于[Integer最大值](https://www.baeldung.com/cs/max-int-java-c-python)时，此方法的结果才是准确的。

例如，我们可以使用andThen()方法来定义一个TriFunction：

-   首先，将multiplyThenAdd()应用于参数
-   然后，应用一个Function，该函数将计算整数的欧几里得除以10的商应用于上一步的结果

```java
static TriFunction<Integer, Integer, Integer, Integer> multiplyThenAddThenDivideByTen = multiplyThenAdd.andThen(x -> x / 10);
```

现在，我们可以编写一些快速测试来检查我们的TriFunction是否按预期运行：

```java
@Test
void whenMultiplyThenAdd_ThenReturnsCorrectResult() {
    assertEquals(25, multiplyThenAdd.apply(2, 10, 5));
}

@Test
void whenMultiplyThenAddThenDivideByTen_ThenReturnsCorrectResult() {
    assertEquals(2, multiplyThenAddThenDivideByTen.apply(2, 10, 5));
}
```

最后一点，TriFunction的操作数可以是多种类型。例如，我们可以定义一个TriFunction，它根据布尔条件将Integer转换为String或返回另一个给定的String：

```java
static TriFunction<Integer, String, Boolean, String> convertIntegerOrReturnStringDependingOnCondition = (myInt, myStr, myBool) -> {
    if (Boolean.TRUE.equals(myBool)) {
        return myInt != null ? myInt.toString() : "";
    } else {
        return myStr;
    }
};
```

## 3. 使用Vavr的Function3

**Vavr库已经定义了一个具有我们想要的行为的Function3接口**。首先，让我们将Vavr[依赖项](https://search.maven.org/search?q=io.vavr)添加到我们的项目中：

```xml
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.4</version>
</dependency>
```

我们现在可以用它重新定义multiplyThenAdd()和multiplyThenAddThenDivideByTen()方法：

```java
static Function3<Integer, Integer, Integer, Integer> multiplyThenAdd = (x, y, z) -> x * y + z;

static Function3<Integer, Integer, Integer, Integer> multiplyThenAddThenDivideByTen = multiplyThenAdd.andThen(x -> x / 10);
```

如果我们需要定义最多有8个参数的函数，使用Vavr是一个不错的选择。Function4、Function5、...Function8确实已经在库中定义了。

## 4. 总结

在本教程中，我们为接收3个参数的函数实现了自己的函数式接口。我们还强调了Vavr库包含此类函数的实现。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-function)上获得。
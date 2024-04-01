---
layout: post
title:  Java中的函数
category: java
copyright: java
excerpt: Java函数式编程
---

## 1. 概述

在本教程中，我们将演示如何在Java中创建函子。首先，让我们来看看关于术语“函子”性质的一些细节。然后我们将查看一些代码示例，说明如何在Java中使用它。

## 2. 什么是函子？

“函子”一词来自数学领域，特别是来自一个称为“范畴论”的子领域。在计算机编程中，函子可以被认为是一个实用程序类，它允许我们映射包含在特定上下文中的值。此外，它还表示两个类别之间的结构保留映射。

两个定律支配函子：

-   Identity：当一个函子被一个恒等函数映射时，这个函数返回与它传递的参数相同的值，我们需要得到初始的函子(容器及其内容保持不变)。
-   Composition/Associativity：当函子用于映射两个部分的组合时，它应该与一个接一个地映射一个函数相同。

## 3. 函数式编程中的函子

**函子是一种用于函数式编程的设计模式，其灵感来自范畴论中使用的定义。它使泛型能够在其内部应用函数而不影响泛型的结构**。在像[Scala](https://www.baeldung.com/scala/functors-functional-programming)这样的编程语言中，我们可以找到函子的很多用途。

## 4. Java中的函子

Java和大多数其他当代编程语言不包含任何被认为是合适的内置函子等价物。但是，从Java 8开始，函数式编程元素被引入到该语言中。[函数式编程](https://www.baeldung.com/java-functional-programming)的思想在Java编程语言中还是比较新颖的。

函子可以使用java.util.function包中的Function接口在Java中实现。下面是Java中Functor类的示例，它采用Function对象并将其应用于值：

```java
public class Functor<T> {
    private final T value;

    public Functor(T value) {
        this.value = value;
    }

    public <R> Functor<R> map(Function<T, R> mapper) {
        return new Functor<>(mapper.apply(value));
    }
    // getter
}
```

正如我们所注意到的，map()方法负责执行操作。对于新类，我们定义了一个最终value属性。该属性是函数将被应用的地方。此外，我们需要一种方法来比较值。让我们把这个函数添加到Functor类中：

```java
public class Functor<T> {
    // Definitions
    boolean eq(T other) {
        return value.equals(other);
    }
    // Getter
}
```

在此示例中，Functor类是泛型的，因为它接收一个类型参数T，该参数指定存储在类中的值的类型。map方法接收一个Function对象，该对象接收类型T的值并返回类型R的值。然后map方法通过将函数应用于原始值并返回它来创建一个新的Functor对象。

下面是一个如何使用这个函子类的例子：

```java
@Test
public void whenProvideAValue_ShouldMapTheValue() {
    Functor<Integer> functor = new Functor<>(5);
    Function<Integer, Integer> addThree = (num) -> num + 3;
    Functor<Integer> mappedFunctor = functor.map(addThree);
    assertEquals(8, mappedFunctor.getValue());
}
```

## 5. 定律验证器

因此，我们需要进行测试。在我们的第一种方法之后，让我们使用Functor类来演示函子定律。首先是定律法。在这种情况下，我们的代码片段是：

```java
@Test
public void whenApplyAnIdentityToAFunctor_thenResultIsEqualsToInitialValue() {
    String value = "tuyucheng";
    // Identity
    Functor<String> identity = new Functor<>(value).map(Function.identity());
    assertTrue(identity.eq(value));
}
```

在刚刚给出的示例中，我们使用了Function类中可用的identity方法。结果Functor返回的值不受影响，并且与作为参数传入的值保持相同。这种行为是遵守身份法的证据。

下一步是应用第二定律。在开始实施之前，我们需要定义一些假设。

-   f是将类型T和R相互映射的函数。
-   g是将类型R和U相互映射的函数。

之后，我们准备好实现我们的测试来证明组合/结合律，这是我们实现的一些代码：

```java
@Test
public void whenApplyAFunctionToOtherFunction_thenResultIsEqualsBetweenBoth() {
    int value = 100;
    Function<Integer, String> f = Object::toString;
    Function<String, Long> g = Long::valueOf;
    Functor<Long> left = new Functor<>(value).map(f).map(g);
    Functor<Long> right = new Functor<>(value).map(f.andThen(g));
    assertTrue(left.eq(100L));
    assertTrue(right.eq(100L));
}
```

从我们的代码片段中，我们定义了两个标记为f和g的函数。之后，我们使用两种不同的映射策略构建两个Functor，一个名为left，另一个名为right。两个Functor最终产生相同的输出。结果，我们对第二定律的实现得到了成功的应用。

## 6. Java 8之前的函子

到目前为止，我们已经看到了使用Java 8中引入的java.util.function.Function接口的代码示例。假设我们使用的是早期版本的Java，在那种情况下，我们可以使用类似的接口或创建我们自己的函数接口来表示一个接收单个参数并返回结果的函数。

另一方面，我们可以利用枚举的功能设计一个Functor。虽然这不是最佳答案，但它确实符合Functor法则，也许最重要的是，它完成了工作。让我们定义我们的EnumFunctor类：

```java
public enum EnumFunctor {
    PLUS {
        public int apply(int a, int b) {
            return a + b;
        }
    }, MINUS {
        public int apply(int a, int b) {
            return a - b;
        }
    }, MULTIPLY {
        public int apply(int a, int b) {
            return a * b;
        }
    }, DIVIDE {
        public int apply(int a, int b) {
            return a / b;
        }
    };

    public abstract int apply(int a, int b);
}
```

在此示例中，对每个常量值调用apply方法，使用两个整数作为参数。该方法执行必要的数学运算并返回结果。另外，本例中使用了abstract关键字，表明apply过程不是在Enum本身实现的，而是必须由各个常量值来实现。现在，让我们测试我们的实现：

```java
@Test
public void whenApplyOperationsToEnumFunctors_thenGetTheProperResult() {
    assertEquals(15, EnumFunctor.PLUS.apply(10, 5));
    assertEquals(5, EnumFunctor.MINUS.apply(10, 5));
    assertEquals(50, EnumFunctor.MULTIPLY.apply(10, 5));
    assertEquals(2, EnumFunctor.DIVIDE.apply(10, 5));
}
```

## 7. 总结

在本文中，我们首先描述了函子是什么。然后，我们进入其定律的定义。之后，我们在Java 8中实现了一些代码示例来演示函子的使用。此外，我们通过示例演示了两个函子定律。最后，我们简要说明了如何在Java 8之前的Java版本中使用函子，并提供了一个使用枚举的示例。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-functional)上获得。
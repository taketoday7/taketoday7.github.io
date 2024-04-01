---
layout: post
title:  Java什么时候抛出ExceptionInInitializerError？
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在这个快速教程中，我们将了解导致Java抛出[ExceptionInInitializerError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ExceptionInInitializerError.html)异常实例的原因。

我们将从一些理论开始。然后我们将在实践中看到这个异常的几个例子。

## 2. ExceptionInInitializerError

**ExceptionInInitializerError表示[静态初始化器](https://www.baeldung.com/java-static)中发生了意外异常**。基本上，当我们看到这个异常时，我们应该知道Java未能评估静态初始化块或实例化静态变量。

事实上，每当静态初始化器中发生任何异常时，Java都会自动将该异常包装在ExceptionInInitializerError类的实例中。这样，它还维护对实际异常的引用作为根本原因。

现在我们知道了这个异常背后的基本原理，让我们在实践中看看它。

## 3. 静态初始化块

要有一个失败的静态块初始化器，我们要有意地将一个整数除以零：

```java
public class StaticBlock {

    private static int state;

    static {
        state = 42 / 0;
    }
}
```

现在，如果我们触发类初始化：

```java
new StaticBlock();
```

然后，我们会看到以下异常：

```plaintext
java.lang.ExceptionInInitializerError
    at cn.tuyucheng.taketoday...(ExceptionInInitializerErrorUnitTest.java:18)
Caused by: java.lang.ArithmeticException: / by zero
    at cn.tuyucheng.taketoday.StaticBlock.<clinit>(ExceptionInInitializerErrorUnitTest.java:35)
    ... 23 more
```

如前所述，Java抛出ExceptionInInitializerError异常的同时维护对根本原因的引用：

```java
assertThatThrownBy(StaticBlock::new)
    .isInstanceOf(ExceptionInInitializerError.class)
    .hasCauseInstanceOf(ArithmeticException.class);
```

另外值得一提的是，<clinit\>方法是JVM中的[类初始化方法](https://www.baeldung.com/jvm-init-clinit-methods#clinit)。

## 4. 静态变量初始化

如果Java未能初始化静态变量，也会发生同样的事情：

```java
public class StaticVar {

    private static int state = initializeState();

    private static int initializeState() {
        throw new RuntimeException();
    }
}
```

同样，如果我们触发类初始化过程：

```java
new StaticVar();
```

然后出现同样的异常：

```plaintext
java.lang.ExceptionInInitializerError
    at cn.tuyucheng.taketoday...(ExceptionInInitializerErrorUnitTest.java:11)
Caused by: java.lang.RuntimeException
    at cn.tuyucheng.taketoday.StaticVar.initializeState(ExceptionInInitializerErrorUnitTest.java:26)
    at cn.tuyucheng.taketoday.StaticVar.<clinit>(ExceptionInInitializerErrorUnitTest.java:23)
    ... 23 more
```

与静态初始化块类似，也保留了异常的根本原因：

```java
assertThatThrownBy(StaticVar::new)
    .isInstanceOf(ExceptionInInitializerError.class)
    .hasCauseInstanceOf(RuntimeException.class);
```

## 5. 受检异常

作为[Java语言规范(JLS-11.2.3)](https://docs.oracle.com/javase/specs/jls/se14/html/jls-11.html#jls-11.2.3)的一部分，我们不能在静态初始化块或静态变量初始化程序中抛出[受检异常](https://www.baeldung.com/java-exceptions#1checked-exceptions)。例如，如果我们尝试这样做：

```java
public class NoChecked {
    static {
        throw new Exception();
    }
}
```

编译器将因以下编译错误而失败：

```plaintext
java: initializer must be able to complete normally
```

**作为约定，当我们的静态初始化逻辑抛出受检异常时，我们应该将可能的受检异常包装在ExceptionInInitializerError实例中**：

```java
public class CheckedConvention {

    private static Constructor<?> constructor;

    static {
        try {
            constructor = CheckedConvention.class.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
}
```

如上所示，getDeclaredConstructor()方法抛出一个受检异常。因此，我们捕获了受检的异常并按照惯例对其进行了包装。

**由于我们已经明确地返回了一个ExceptionInInitializerError异常的实例，因此Java不会将这个异常包装在另一个ExceptionInInitializerError实例中**。

但是，如果我们抛出任何其他非受检的异常，Java将抛出另一个ExceptionInInitializerError：

```java
static {
    try {
        constructor = CheckedConvention.class.getConstructor();
    } catch (NoSuchMethodException e) {
        throw new RuntimeException(e);
    }
}
```

在这里，我们将受检的异常包装在非受检的异常中。由于这个非受检的异常不是ExceptionInInitializerError的实例，因此Java将再次包装它，从而导致以下意外的堆栈跟踪：

```plaintext
java.lang.ExceptionInInitializerError
	at cn.tuyucheng.taketoday.exceptionininitializererror...
Caused by: java.lang.RuntimeException: java.lang.NoSuchMethodException: ...
Caused by: java.lang.NoSuchMethodException: cn.tuyucheng.taketoday.CheckedConvention.<init>()
	at java.base/java.lang.Class.getConstructor0(Class.java:3427)
	at java.base/java.lang.Class.getConstructor(Class.java:2165)
```

如上所示，如果我们遵循约定，那么堆栈跟踪将比这更清晰。

### 5.1 OpenJDK

最近，这个约定甚至用在OpenJDK源代码本身中。例如，下面是[AtomicReference](https://github.com/openjdk/jdk/blob/b87302ca99ff30a03e311ab1c0f524684ed37596/src/java.base/share/classes/java/util/concurrent/atomic/AtomicReference.java#L51)如何使用这种方法：

```java
public class AtomicReference<V> implements java.io.Serializable {
    private static final VarHandle VALUE;

    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup();
            VALUE = l.findVarHandle(AtomicReference.class, "value", Object.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    private volatile V value;

    // omitted
}
```

## 6. 总结

在本教程中，我们了解了导致Java抛出ExceptionInInitializerError异常实例的原因。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-2)上获得。
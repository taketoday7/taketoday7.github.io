---
layout: post
title:  IllegalArgumentException还是NullPointerException？
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在我们编写应用程序时做出的决定中，有很多是关于何时抛出[异常](https://www.baeldung.com/java-exceptions)以及抛出哪种类型的异常。

在这个快速教程中，我们将解决当有人将null参数传递给我们的方法之一时抛出哪个异常的问题：IllegalArgumentException或[NullPointerException](https://www.baeldung.com/java-14-nullpointerexception)。

## 2. IllegalArgumentException

首先，让我们看看抛出IllegalArgumentException的论据。

让我们创建一个简单的方法，在传递null时抛出IllegalArgumentException：

```java
public void processSomethingNotNull(Object myParameter) {
    if (myParameter == null) {
        throw new IllegalArgumentException("Parameter 'myParameter' cannot be null");
    }
}
```

现在，让我们继续讨论支持IllegalArgumentException的论据。

### 2.1 Javadoc是这样说的

当我们阅读[IllegalArgumentException的Javadoc](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/IllegalArgumentException.html)时，它说**这是在将非法或不适当的值传递给方法时使用的**。如果我们的方法不期望null对象，我们可以认为它是非法或不合适的，这将是我们抛出的适当异常。

### 2.2 它符合开发人员的期望

接下来，让我们考虑一下，当我们在应用程序中看到堆栈跟踪时，作为开发人员，我们是如何思考的。我们收到NullPointerException的一个非常常见的场景是当我们不小心尝试访问空对象时。在这种情况下，我们将尽可能深入堆栈，看看我们引用的什么是null。

当我们得到一个IllegalArgumentException时，我们可能会假设我们将错误的东西传递给了一个方法。在这种情况下，我们将在堆栈中查找我们正在调用的最底部的方法，并从那里开始调试。如果我们考虑这种思维方式，IllegalArgumentException将使我们更接近发生错误的堆栈。

### 2.3 其他论点

在我们继续讨论NullPointerException的论点之前，让我们先看几个支持IllegalArgumentException的小点。一些开发人员认为只有JDK应该抛出NullPointerException。正如我们将在下一节中看到的，Javadoc不支持这个理论。另一个论点是使用IllegalArgumentException更一致，因为这是我们用于其他非法参数值的方法。

## 3. NullPointerException

接下来，让我们考虑NullPointerException的论点。

让我们创建一个抛出NullPointerException的示例：

```java
public void processSomethingElseNotNull(Object myParameter) {
    if (myParameter == null) {
        throw new NullPointerException("Parameter 'myParameter' cannot be null");
    }
}
```

### 3.1 Javadoc是这样说的

根据[NullPointerException的Javadoc](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/NullPointerException.html)，**NullPointerException用于尝试在需要对象的地方使用null**。如果我们的方法参数不打算为null，那么我们可以合理地将其视为必需的对象并抛出NullPointerException。

### 3.2 与JDK API一致

让我们花点时间考虑一下我们在开发过程中调用的许多常见JDK方法。如果我们提供null，他们中的很多都会抛出NullPointerException。此外，如果我们传入null，Objects.requireNonNull()会抛出NullPointerException。根据[Objects文档](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Objects.html#requireNonNull(T))，它主要用于验证参数。

除了抛出NullPointerException的JDK方法之外，我们还可以找到从集合API中的方法抛出特定异常类型的其他示例。**如果索引超出列表大小，ArrayList.addAll(index, Collection)抛出IndexOutOfBoundsException，如果集合为null则抛出NullPointerException**。这是两个非常具体的异常类型，而不是更通用的IllegalArgumentException。

我们可以将IllegalArgumentException视为适用于我们没有可用的更具体的异常类型的情况。

## 4. 总结

正如我们在本教程中看到的，这是一个没有明确答案的问题。这两个异常的文档似乎重叠，因为当单独使用时，它们听起来都很合适。基于开发人员进行调试的方式以及在JDK方法本身中看到的模式，双方还有其他令人信服的论据。

无论我们选择哪种异常，我们都应该在整个应用程序中保持一致。此外，我们可以通过向异常构造函数提供有意义的信息来使我们的异常更有用。例如，如果我们在异常消息中提供参数名称，我们的应用程序将更容易调试。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-3)上获得。

---
layout: post
title:  Java异常面试问题(+答案)
category: interview
copyright: interview
excerpt: Java Exception
---

## 1. 概述

异常是每个Java开发人员都应该熟悉的基本主题，本文提供了一些在面试中可能会出现的问题的答案。

## 2. 问题

### Q1. 什么是异常？

异常是程序执行过程中发生的意外事件，它会扰乱程序指令的正常流程。

### Q2. throw和throws关键字的目的是什么？

throws关键字用于指定方法在执行期间可能引发异常，它在调用方法时强制执行显式异常处理：

```java
public void simpleMethod() throws Exception {
    // ...
}
```

throw关键字允许我们抛出一个异常对象来中断程序的正常流程。当程序无法满足给定条件时，最常使用这种方法：

```java
if (task.isTooComplicated()) {
    throw new TooComplicatedException("The task is too complicated");
}
```

### Q3. 如何处理异常？

通过使用try-catch-finally语句：

```java
try {
    // ...
} catch (ExceptionType1 ex) {
    // ...
} catch (ExceptionType2 ex) {
    // ...
} finally {
    // ...
}
```

可能发生异常的代码块包含在try块中，此块也称为“受保护”代码。

如果发生异常，则执行与抛出的异常相匹配的catch块，否则，将忽略所有catch块。

finally块始终在try块退出后执行，无论其内部是否抛出异常。

### Q4. 如何捕获多个异常？

可以通过三种方式在代码块中处理多个异常。

第一种是使用可以处理所有抛出的异常类型的catch块：

```java
try {
    // ...
} catch (Exception ex) {
    // ...
}
```

你应该记住，推荐的做法是使用尽可能准确的异常处理程序。

过于宽泛的异常处理程序会使你的代码更容易出错，捕获未预料到的异常，并导致程序出现意外行为。

第二种方法是实现多个catch块：

```java
try {
    // ...
} catch (FileNotFoundException ex) {
    // ...
} catch (EOFException ex) {
    // ...
}
```

注意，如果异常有继承关系；子类型必须在前，父类型在后。如果我们不这样做，就会导致编译错误。

第三种是catch多个异常：

```java
try {
    // ...
} catch (FileNotFoundException | EOFException ex) {
    // ...
}
```

这个特性，在Java 7中首次引入；减少代码重复，更易于维护。

### Q5. 受检异常和非受检异常有什么区别？

受检异常必须在try-catch块中处理或在throws子句中声明；而非受检异常不需要处理或声明。

受检异常和非受检异常也分别称为编译时异常和运行时异常。

除了Error、RuntimeException及其子类指示的异常之外，所有异常都是受检异常。

### Q6. 异常和错误有什么区别？

异常是表示可以从中恢复的条件的事件，而错误表示通常无法从中恢复的外部情况。

JVM抛出的所有错误都是Error或其子类之一的实例，比较常见的包括：

-   OutOfMemoryError：当JVM由于内存不足而无法分配更多对象并且垃圾回收器无法提供更多可用对象时抛出
-   StackOverflowError：当线程的堆栈空间用完时发生，通常是因为应用程序递归太深
-   ExceptionInInitializerError：表示在评估静态初始化程序期间发生了意外异常
-   NoClassDefFoundError：当类加载器试图加载一个类的定义但找不到它时抛出，通常是因为在类路径中找不到所需的class文件
-   UnsupportedClassVersionError：当JVM尝试读取class文件并确定文件中的版本不受支持时发生，通常是因为该文件是使用较新版本的Java生成的

尽管可以使用try语句处理错误，但这不是推荐的做法，因为无法保证程序在抛出错误后能够可靠地执行任何操作。

### Q7. 执行下面的代码块会抛出什么异常？

```java
Integer[][] ints = { { 1, 2, 3 }, { null }, { 7, 8, 9 } };
System.out.println("value = " + ints[1][1].intValue());
```

它抛出ArrayIndexOutOfBoundsException，因为我们试图访问大于数组长度的位置。

### Q8. 什么是异常链接？

在为响应另一个异常而引发异常时发生，这使我们能够发现我们提出的问题的完整历史：

```java
try {
    task.readConfigFile();
} catch (FileNotFoundException ex) {
    throw new TaskException("Could not perform task", ex);
}
```

### Q9. 什么是堆栈跟踪以及它与异常有何关系？

堆栈跟踪提供了从应用程序开始到发生异常点所调用的类和方法的名称。

这是一个非常有用的调试工具，因为它使我们能够准确地确定异常在应用程序中被抛出的位置以及导致它的原始原因。

### Q10. 为什么要对异常进行子类化？

如果异常类型不是由Java平台中已经存在的异常类型表示的，或者如果你需要向客户端代码提供更多信息以更精确地处理它，那么你应该创建一个自定义异常。

决定使用受检或非受检自定义异常完全取决于业务案例。但是，根据经验；如果可以预期使用你的异常的代码可以从中恢复，则创建一个受检的异常，否则使用非受检异常。

此外，你应该继承与你要抛出的异常密切相关的最具体的异常子类。如果没有这样的类，则选择Exception作为父类。

### Q11. 异常有哪些优点？

传统的错误检测和处理技术通常会导致意大利面条式代码难以维护和阅读。但是，异常使我们能够将应用程序的核心逻辑与发生意外情况时的操作细节分开。

此外，由于JVM向后搜索调用堆栈以找到任何对处理特定异常感兴趣的方法；我们获得了在调用堆栈中向上传播错误的能力，而无需编写额外的代码。

此外，由于程序中抛出的所有异常都是对象，因此可以根据其类层次结构对它们进行分组或分类。这允许我们通过在catch块中指定异常的超类来在单个异常处理程序中捕获一组异常。

### Q12. 你能在Lambda表达式的主体中抛出任何异常吗？

当使用Java已经提供的标准函数接口时，你只能抛出非受检的异常，因为标准函数接口在方法签名中没有“throws”子句：

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> {
    if (i == 0) {
        throw new IllegalArgumentException("Zero not allowed");
    }
    System.out.println(Math.PI / i);
});
```

但是，如果你使用的是自定义函数接口，则可能会抛出受检异常：

```java
@FunctionalInterface
public static interface CheckedFunction<T> {
    void apply(T t) throws Exception;
}
```

```java
public void processTasks(List<Task> taks, CheckedFunction<Task> checkedFunction) {
    for (Task task : taks) {
        try {
            checkedFunction.apply(task);
        } catch (Exception e) {
            // ...
        }
    }
}

processTasks(taskList, t -> {
    // ...
    throw new Exception("Something happened");
});
```

### Q13. 重写抛出异常的方法需要遵循哪些规则？

有几条规则规定了必须如何在继承上下文中声明异常。

当父类方法不抛出任何异常时，子类方法不能抛出任何受检异常，但它可以抛出任何非受检异常。

以下是一个示例代码：

```java
class Parent {
    void doSomething() {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IllegalArgumentException {
        // ...
    }
}
```

以下示例将无法编译，因为重写方法抛出未在被重写方法中声明的受检异常：

```java
class Parent {
    void doSomething() {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IOException {
        // Compilation error
    }
}
```

当父类方法抛出一个或多个受检异常时，子类方法可以抛出任何非受检异常；所有、没有或声明的受检异常的子集，甚至更多，只要它们具有相同或更窄的范围。

这是成功遵循以上规则的示例代码：

```java
class Parent {
    void doSomething() throws IOException, ParseException {
        // ...
    }

    void doSomethingElse() throws IOException {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IOException {
        // ...
    }

    void doSomethingElse() throws FileNotFoundException, EOFException {
        // ...
    }
}
```

请注意，这两种方法都遵循规则。第一个抛出的异常比被重写的方法少，而第二个抛出的异常更多，但它们的范围更窄(即FileNotFoundException和EOFException是IOException的子类)。

但是，如果我们尝试抛出父类方法未声明的受检异常，或者抛出范围更广的异常；我们会得到一个编译错误：

```java
class Parent {
    void doSomething() throws FileNotFoundException {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IOException {
        // Compilation error
    }
}
```

当父类方法具有声明非受检异常的throws子句时，子类方法可以不抛出任何或任意数量的非受检异常，即使它们不相关。

这是一个遵循规则的示例：

```java
class Parent {
    void doSomething() throws IllegalArgumentException {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws ArithmeticException, BufferOverflowException {
        // ...
    }
}
```

### Q14. 以下代码可以编译吗？

```java
void doSomething() {
    // ...
    throw new RuntimeException(new Exception("Chained Exception"));
}
```

可以。当链接异常时，编译器只关心链中的第一个异常，并且由于它检测到一个非受检的异常，因此我们不需要添加throws子句。

### Q15. 有什么方法可以从没有throws子句的方法中抛出受检异常？

我们可以利用编译器执行的类型擦除，让它认为我们正在抛出一个非受检的异常，而实际上；我们抛出一个受检异常：

```java
public <T extends Throwable> T sneakyThrow(Throwable ex) throws T {
    throw (T) ex;
}

public void methodWithoutThrows() {
    this.<RuntimeException>sneakyThrow(new Exception("Checked Exception"));
}
```

## 3. 总结

在本文中，我们探讨了一些可能出现在Java开发人员技术面试中的有关异常的问题。这不是一个详尽的列表，它应该只是作为进一步研究的开始。


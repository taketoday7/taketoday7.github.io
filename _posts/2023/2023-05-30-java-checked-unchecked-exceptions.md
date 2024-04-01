---
layout: post
title:  Java中的受检和非受检异常
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

[Java异常](https://www.baeldung.com/java-exceptions)分为两大类：**受检异常和非受检异常**。

在本教程中，我们将提供一些有关如何使用它们的代码示例。

## 2. 受检异常

通常，受检异常表示程序无法控制的错误。例如，如果输入文件不存在，[FileInputStream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileInputStream.html#(java.io.File))的构造函数将抛出FileNotFoundException。

**Java在编译时验证受检异常**。

因此，我们应该使用[throws](https://www.baeldung.com/java-throw-throws)关键字来声明一个受检异常：

```java
private static void checkedExceptionWithThrows() throws FileNotFoundException {
    File file = new File("not_existing_file.txt");
    FileInputStream stream = new FileInputStream(file);
}
```

我们还可以使用try-catch块来处理受检异常：

```java
private static void checkedExceptionWithTryCatch() {
    File file = new File("not_existing_file.txt");
    try {
        FileInputStream stream = new FileInputStream(file);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```

Java中一些[常见的受检异常](https://www.baeldung.com/java-common-exceptions)是IOException、SQLException和ParseException。

Exception类是受检异常的超类，因此我们可以通过扩展Exception来[创建自定义受检异常](https://www.baeldung.com/java-new-custom-exception)：

```java
public class IncorrectFileNameException extends Exception {
    public IncorrectFileNameException(String errorMessage) {
        super(errorMessage);
    }
}
```

## 3. 非受检异常

如果一个程序抛出一个非受检异常，它反映了程序逻辑内部的一些错误。

例如，如果我们将一个数除以0，Java将抛出ArithmeticException：

```java
private static void divideByZero() {
    int numerator = 1;
    int denominator = 0;
    int result = numerator / denominator;
}
```

**Java不会在编译时验证非受检的异常**。此外，我们不必在带有throws关键字的方法中声明非受检的异常。虽然上面的代码在编译时没有任何错误，但它会在运行时抛出ArithmeticException。

Java中一些[常见的非受检异常](https://www.baeldung.com/java-common-exceptions)NullPointerException、ArrayIndexOutOfBoundsException和IllegalArgumentException。

[RuntimeException](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/RuntimeException.html)类是所有非受检异常的超类，因此我们可以通过扩展RuntimeException来[创建自定义非受检异常](https://www.baeldung.com/java-new-custom-exception)：

```java
public class NullOrEmptyException extends RuntimeException {
    public NullOrEmptyException(String errorMessage) {
        super(errorMessage);
    }
}
```

## 4. 何时使用受检异常和非受检异常

在Java中使用异常是一种很好的做法，这样我们就可以将错误处理代码与常规代码分开。但是，我们需要决定抛出哪种类型的异常。[Oracle Java文档](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)提供了有关何时使用受检异常和非受检异常的指南：

> “如果可以合理地预期客户端从异常中恢复，则将其设为受检异常。如果客户端无法执行任何操作来从异常中恢复，则将其设为非受检异常。”

例如，在我们打开一个文件之前，我们可以先验证输入的文件名。如果用户输入的文件名无效，我们可以抛出自定义受检异常：

```java
if (!isCorrectFileName(fileName)) {
    throw new IncorrectFileNameException("Incorrect filename : " + fileName );
}
```

这样，我们就可以通过接受另一个用户输入的文件名来恢复系统。

但是，如果输入的文件名是一个空指针或者是一个空字符串，则意味着我们的代码有一些错误。在这种情况下，我们应该抛出一个非受检异常：

```java
if (fileName == null || fileName.isEmpty())  {
    throw new NullOrEmptyException("The filename is null or empty.");
}
```

## 5. 总结

在本文中，我们讨论了受检异常和非受检异常之间的区别。我们还提供了一些代码示例来说明何时使用受检或非受检的异常。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-1)上获得。

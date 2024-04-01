---
layout: post
title:  Java全局异常处理程序
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将重点介绍Java中的全局异常处理程序。我们将首先讨论异常和异常处理的基础知识。然后我们将全面了解全局异常处理程序。

要了解有关一般异常的更多信息，请查看[Java中的异常处理](https://www.baeldung.com/java-exceptions)。

## 2. 什么是异常？

**异常是在运行时或编译时代码序列中出现的异常情况**。当程序违反Java编程语言的语义约束时，就会出现这种异常情况。

**编译期间发生的异常是受检异常**。这些异常是Exception类的直接子类，必须在代码中处理这些异常。

**另一种类型的异常是非受检的异常，编译器在编译时不会检查这些异常**。这些异常是扩展Exception类的RuntimeException类的直接子类。

此外，没有必要在代码中处理运行时异常。

## 3. 异常处理程序

Java是一种健壮的编程语言。使其健壮的核心功能之一是异常处理框架。这意味着程序可以在出错时优雅地退出，而不仅仅是崩溃。

**每当发生异常时，都会通过JVM或执行代码的方法构造一个Exception对象**。该对象包含有关异常的信息。**异常处理就是处理这个异常对象的一种方式**。

### 3.1 try-catch块

在下面的示例中，try块包含可能抛出异常的代码。catch块包含处理此异常的逻辑。

catch块捕获try块中的代码引发的Exception对象：

```java
String string = "01, , 2010";
DateFormat format = new SimpleDateFormat("MM, dd, yyyy");
Date date;
try {
    date = format.parse(string);
} catch (ParseException e) {
    System.out.println("ParseException caught!");
}
```

### 3.2 throw和throws关键词

或者，该方法也可以选择抛出异常而不是处理它。这意味着处理Exception对象的逻辑是在其他地方编写的。

通常，调用方法会在这种情况下处理异常：

```java
public class ExceptionHandler {

    public static void main(String[] args) {

        String strDate = "01, , 2010";
        String dateFormat = "MM, dd, yyyy";

        try {
            Date date = new DateParser().getParsedDate(strDate, dateFormat);
        } catch (ParseException e) {
            System.out.println("The calling method caught ParseException!");
        }
    }
}

class DateParser {

    public Date getParsedDate(String strDate, String dateFormat) throws ParseException {
        DateFormat format = new SimpleDateFormat(dateFormat);

        try {
            return format.parse(strDate);
        } catch (ParseException parseException) {
            throw parseException;
        }
    }
}
```

接下来，我们将考虑全局异常处理程序，作为处理异常的通用方法。

## 4. 全局异常处理器

RuntimeException的实例是可选处理的。因此，它仍然为在运行时获取长堆栈跟踪留出一个窗口。为了处理这个问题，**Java提供了UncaughtExceptionHandler接口**。Thread类包含此作为内部类。

除了这个接口，**Java 1.5版本还在Thread类中引入了一个静态方法setDefaultUncaughtExceptionHandler()**。此方法的参数是一个实现UncaughtExceptionHandler接口的处理程序类。

此外，该接口声明了方法uncaughtException(Thread t, Throwable e)。当给定的线程t由于给定的未捕获异常e而终止时，它将被调用。实现类实现这个方法并定义用于处理这些未捕获异常的逻辑。

让我们考虑以下在运行时抛出ArithmeticException的示例。我们定义了实现接口UncaughtExceptionHandler的类Handler。

该类实现了uncaughtException()方法并在其中定义了处理未捕获异常的逻辑：

```java
public class GlobalExceptionHandler {

    public static void main(String[] args) {
        Handler globalExceptionHandler = new Handler();
        Thread.setDefaultUncaughtExceptionHandler(globalExceptionHandler);
        new GlobalExceptionHandler().performArithmeticOperation(10, 0);
    }

    public int performArithmeticOperation(int num1, int num2) {
        return num1 / num2;
    }
}

class Handler implements Thread.UncaughtExceptionHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(Handler.class);

    public void uncaughtException(Thread t, Throwable e) {
        LOGGER.info("Unhandled exception caught!");
    }
}
```

这里，当前正在执行的线程就是主线程。因此，它的实例连同引发的异常一起传递给方法uncaughtException()。然后类Handler处理这个异常。

这同样适用于未处理的受检异常，让我们也看一个简单的例子：

```java
public static void main(String[] args) throws Exception {
    Handler globalExceptionHandler = new Handler();
    Thread.setDefaultUncaughtExceptionHandler(globalExceptionHandler);
    Path file = Paths.get("");
    Files.delete(file);
}
```

在这里，Files.delete()方法抛出一个受检的IOException，该IOException由main()方法签名进一步抛出。Handler也会捕获此异常。

通过这种方式，UncaughtExceptionHandler有助于在运行时管理未处理的异常。**但是，它打破了在接近原点的地方捕获和处理异常的想法**。

## 5. 总结

在本文中，我们了解了异常是什么，以及处理它们的基本方法。此外，我们确定全局异常处理程序是Thread类的一部分，它处理未捕获的运行时异常。

然后，我们看到了一个抛出运行时异常并使用全局异常处理程序处理它的示例程序。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-2)上获得。
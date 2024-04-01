---
layout: post
title:  如何在Java中获取对象的大小
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

与我们可以使用sizeof()方法以字节为单位获取对象大小的C/C++不同，在Java中没有真正等效的此类方法。

在本文中，我们将演示如何获取特定对象的大小。

## 2. Java中的内存消耗

虽然Java中没有sizeof运算符，但实际上我们不需要。所有基本类型都有标准大小，并且通常没有填充字节或对齐字节。不过，这并不总是直截了当的。

**虽然基本类型必须表现得像它们具有官方大小一样，但JVM可以在内部以任何它喜欢的方式存储数据，具有任意数量的填充或开销**。它可以选择将boolean[]存储在64位长块中，如BitSet，在堆栈上分配一些临时对象或优化一些完全不存在的变量或方法调用，用常量替换它们，等等...但是，只要程序给出相同的结果，完全没问题。

还要考虑到硬件和操作系统缓存的影响(我们的数据可能在每个缓存级别上重复)，**这意味着我们只能粗略地预测RAM消耗**。

### 2.1 对象、引用和包装类

**现代64位JDK的最小对象大小为16字节**，因为对象具有12字节的标头，填充为8字节的倍数。在32位JDK中，开销为8字节，填充为4字节的倍数。

**在堆边界小于32Gb(-Xmx32G)的32位平台和64位平台上，引用的典型大小为4字节**，对于高于32Gb的边界，引用的大小为8字节。

这意味着64位JVM通常需要多30-50%的堆空间。

特别需要注意的是，**包装类型、数组、String和其他容器(如多维数组)的内存消耗很大，因为它们会增加一定的开销**。例如，当我们将int primitive(仅占用4个字节)与占用16个字节的Integer对象进行比较时，我们看到有300%的内存开销。

## 3. 使用Instrumentation估计对象大小

在Java中估算对象大小的一种方法是使用Java 5中引入的[Instrumentation接口](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)的getObjectSize(Object)方法。

正如我们在Javadoc文档中看到的那样，该方法提供了指定对象大小的“特定于实现的近似值”。值得注意的是，在单个JVM调用期间，存在潜在的开销，并且值可能不同。

**此方法只支持所考虑对象本身的大小估计，而不支持它引用的对象的大小估计**。要估计对象的总大小，我们需要一个代码来遍历这些引用并计算估计的大小。

### 3.1 创建检测代理

为了调用Instrumentation.getObjectSize(Object)来获取对象的大小，**我们需要首先能够访问Instrumentation的实例**。我们需要使用检测代理，有两种方法可以做到这一点，如[java.lang.instrument包的文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html)中所述。

**可以通过命令行指定Instrumentation代理，或者我们可以将其与已经运行的JVM一起使用**。我们将重点关注第一个方法。

**要通过命令行指定检测代理，我们需要实现重载的premain方法**，该方法将在使用检测时首先由JVM调用。除此之外，我们需要公开一个静态方法才能访问Instrumentation.getObjectSize(Object)。

现在让我们创建InstrumentationAgent类：

```java
public class InstrumentationAgent {
    private static volatile Instrumentation globalInstrumentation;

    public static void premain(final String agentArgs, final Instrumentation inst) {
        globalInstrumentation = inst;
    }

    public static long getObjectSize(final Object object) {
        if (globalInstrumentation == null) {
            throw new IllegalStateException("Agent not initialized.");
        }
        return globalInstrumentation.getObjectSize(object);
    }
}
```

**在为此代理创建JAR之前，我们需要确保其中包含一个简单的MANIFEST.MF文件**：

```shell
Premain-class: cn.tuyucheng.taketoday.objectsize.InstrumentationAgent
```

现在我们可以创建一个包含MANIFEST.MF文件的代理JAR，一种方法是通过命令行：

```shell
javac InstrumentationAgent.java
jar cmf MANIFEST.MF InstrumentationAgent.jar InstrumentationAgent.class
```

### 3.2 示例类

让我们通过创建一个包含示例对象的类来查看此操作，该类将使用我们的代理类：

```java
public class InstrumentationExample {

    public static void printObjectSize(Object object) {
        System.out.println("Object type: " + object.getClass() + ", size: " + InstrumentationAgent.getObjectSize(object) + " bytes");
    }

    public static void main(String[] arguments) {
        String emptyString = "";
        String string = "Estimating Object Size Using Instrumentation";
        String[] stringArray = {emptyString, string, "cn.tuyucheng.taketoday"};
        String[] anotherStringArray = new String[100];
        List<String> stringList = new ArrayList<>();
        StringBuilder stringBuilder = new StringBuilder(100);
        int maxIntPrimitive = Integer.MAX_VALUE;
        int minIntPrimitive = Integer.MIN_VALUE;
        Integer maxInteger = Integer.MAX_VALUE;
        Integer minInteger = Integer.MIN_VALUE;
        long zeroLong = 0L;
        double zeroDouble = 0.0;
        boolean falseBoolean = false;
        Object object = new Object();

        class EmptyClass {
        }
        EmptyClass emptyClass = new EmptyClass();

        class StringClass {
            public String s;
        }
        StringClass stringClass = new StringClass();

        printObjectSize(emptyString);
        printObjectSize(string);
        printObjectSize(stringArray);
        printObjectSize(anotherStringArray);
        printObjectSize(stringList);
        printObjectSize(stringBuilder);
        printObjectSize(maxIntPrimitive);
        printObjectSize(minIntPrimitive);
        printObjectSize(maxInteger);
        printObjectSize(minInteger);
        printObjectSize(zeroLong);
        printObjectSize(zeroDouble);
        printObjectSize(falseBoolean);
        printObjectSize(Day.TUESDAY);
        printObjectSize(object);
        printObjectSize(emptyClass);
        printObjectSize(stringClass);
    }

    public enum Day {
        MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
    }
}
```

为此，**我们需要在运行我们的应用程序时包括–javaagent选项以及代理JAR的路径**：

```shell
VM Options: -javaagent:"path_to_agent_directory\InstrumentationAgent.jar"
```

运行类的输出将向我们显示估计的对象大小：

```plaintext
Object type: class java.lang.String, size: 24 bytes
Object type: class java.lang.String, size: 24 bytes
Object type: class [Ljava.lang.String;, size: 32 bytes
Object type: class [Ljava.lang.String;, size: 416 bytes
Object type: class java.util.ArrayList, size: 24 bytes
Object type: class java.lang.StringBuilder, size: 24 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Long, size: 24 bytes
Object type: class java.lang.Double, size: 24 bytes
Object type: class java.lang.Boolean, size: 16 bytes
Object type: class cn.tuyucheng.taketoday.objectsize.InstrumentationExample$Day, size: 24 bytes
Object type: class java.lang.Object, size: 16 bytes
Object type: class cn.tuyucheng.taketoday.objectsize.InstrumentationExample$1EmptyClass, size: 16 bytes
Object type: class cn.tuyucheng.taketoday.objectsize.InstrumentationExample$1StringClass, size: 16 bytes
```

## 4. 总结

在本文中，我们描述了Java中特定类型如何使用内存，JVM如何存储数据，并强调了可能影响总内存消耗的因素。然后我们演示了如何在实践中获取Java对象的估计大小。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-1)上获得。
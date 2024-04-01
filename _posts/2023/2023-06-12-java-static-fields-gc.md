---
layout: post
title:  静态字段和垃圾回收
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

**在本教程中，我们将了解[垃圾回收](https://www.baeldung.com/jvm-garbage-collectors)如何处理静态字段**。此外，我们还将涉及[类加载](https://www.baeldung.com/java-classloaders)和类对象等主题。在本文之后，我们将更好地理解类、类加载器和静态字段之间的联系以及垃圾回收器如何处理它们。

## 2. Java垃圾回收概述

**Java提供了一个非常好的自动内存管理功能**。在大多数情况下，这种方法不如手动有效。但是，它有助于避免难以调试的问题并减少样板代码。此外，随着垃圾回收的改进，这个过程变得越来越好。因此，我们应该回顾一下垃圾回收器的工作原理，以及我们的应用程序中有哪些垃圾。

### 2.1 垃圾对象

**[引用计数](https://www.baeldung.com/java-gc-cyclic-references#reference-counting)是识别垃圾对象最直接、最直观的方法**。这种方法允许我们检查当前对象是否有任何对它的引用。然而，这种方法有一些缺点，其中最重要的是循环引用。

**处理循环引用的方法之一是跟踪**。当对象与应用程序的[垃圾回收根](https://www.baeldung.com/java-gc-roots#types-of-gc-roots)没有任何链接时，它们就会变成垃圾。

### 2.2 静态字段和类对象

**在Java中，一切都是对象，包括类的定义**。它们包含有关类、方法和静态字段值的所有元信息。因此，所有静态字段都是受尊重的class对象的引用。**因此，在类对象存在并被应用程序引用之前，静态字段将不符合垃圾回收条件**。

同时，所有加载的类都有对用于加载该特定类的类加载器的引用。这样，我们可以跟踪加载的类。

在这种情况下，我们有一个引用层次结构。**类加载器保留对所有加载类的引用**。同时，类存储对各自类加载器的引用。在这种情况下，我们有双向引用。每当我们实例化一个新对象时，它都会持有对其类定义的引用。因此，我们有以下层次结构：

![](/assets/images/2023/javajvm/javastaticfieldsgc01.png)

**在我们的应用程序具有对类的引用之前，我们不能卸载它**。让我们检查一下我们需要什么才能使类定义符合垃圾回收的条件。首先，应用程序不应引用类的实例。这很重要，因为所有实例都引用了它们的类。其次，这个类的类加载器应该不能从应用程序中获得。最后，类本身在应用程序中不应有任何引用。

## 3. 垃圾回收静态字段示例

让我们创建一个示例，让垃圾回收器删除我们的静态字段。**JVM支持对扩展类加载器和系统类加载器加载的类进行类卸载**。但是，这将很难重现，我们将为此使用自定义类加载器，因为我们将对其进行更多控制。

### 3.1 自定义类加载器

首先，让我们创建自己的CustomClassloader将从我们应用程序的资源文件夹中加载一个类。为了让我们的类加载器工作，我们应该覆盖loadClass(String name)方法：

```java
public class CustomClassloader extends ClassLoader {

    public static final String PREFIX = "cn.tuyucheng.taketoday.classloader";

    public CustomClassloader(ClassLoader parent) {
        super(parent);
    }

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        if (name.startsWith(PREFIX)) {
            return getClass(name);
        } else {
            return super.loadClass(name);
        }
    }

    // ...
}
```

在此实现中，我们使用getClass方法，该方法隐藏了从资源加载类的复杂性：

```java
private Class<?> getClass(String name) {
    String fileName = name.replace('.', File.separatorChar) + ".class";
    try {
        byte[] byteArr = IOUtils.toByteArray(getClass().getClassLoader().getResourceAsStream(fileName));
        Class<?> c = defineClass(name, byteArr, 0, byteArr.length);
        resolveClass(c);
        return c;
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

### 3.2 文件夹结构

**为了正常工作，我们的自定义类应该在我们的类路径范围之外**。这样，它就不会被[系统类加载器](https://www.baeldung.com/java-system-gc#garbagecollection)检测。唯一处理这个特定类的类加载器将是我们的CustomClassloader。

我们文件夹的结构如下所示：

![](/assets/images/2023/javajvm/javastaticfieldsgc02.png)

### 3.3 静态字段持有者

我们将使用一个自定义类来充当静态字段的持有者，在定义了类加载器的实现之后，我们可以使用它来上传我们准备好的类。这是一个简单的类：

```java
public class GarbageCollectedStaticFieldHolder {

    private static GarbageCollectedInnerObject garbageCollectedInnerObject = new GarbageCollectedInnerObject("Hello from a garbage collected static field");

    public void printValue() {
        System.out.println(garbageCollectedInnerObject.getMessage());
    }
}
```

### 3.4 静态字段类

GarbageCollectedInnerObject将表示我们想要变成垃圾的对象。为了简单和方便，此类定义在与GarbageCollectedStaticFieldHolder相同的文件中。此类包含一条消息，并且还重写了finalize()方法。**尽管[finalize()](https://www.baeldung.com/java-finalize)方法已被弃用并且有很多缺点，但它可以让我们可视化垃圾回收器何时移除对象**。我们仅将此方法用于演示目的。下面我们的静态字段类：

```java
class GarbageCollectedInnerObject {

    private final String message;

    public GarbageCollectedInnerObject(final String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    @Override
    protected void finalize() {
        System.out.println("The object is garbage now");
    }
}
```

### 3.5 上传类

现在我们可以上传并实例化我们的类了。创建实例后，我们可以确保类已上传，对象已创建，静态字段包含所需信息：

```java
private static void loadClass() {
    try {
        final String className = "cn.tuyucheng.taketoday.classloader.GarbageCollectedStaticFieldHolder";
        CustomClassloader loader = new CustomClassloader(Main.class.getClassLoader());
        Class<?> clazz = loader.loadClass(className);
        Object instance = clazz.getConstructor().newInstance();
        clazz.getMethod(METHOD_NAME).invoke(instance);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

这个方法应该创建我们特殊类的实例并输出消息：

```text
Hello from a garbage collected static field
```

### 3.6 垃圾回收实践

现在让我们启动我们的应用程序并尝试删除垃圾：

```java
public static void main(String[] args) throws InterruptedException {
    loadClass();
    System.gc();
    Thread.sleep(1000);
}
```

调用loadClass()方法后，该方法中的所有变量，即类加载器、我们的类和实例，都将超出范围并失去与垃圾回收根的连接。也可以将null分配给引用，但使用范围的选项更清晰：

```java
public static void main(String[] args) throws InterruptedException {
    CustomClassloader loader;
    Class<?> clazz;
    Object instance;
    try {
        final String className = "cn.tuyucheng.taketoday.classloader.GarbageCollectedStaticFieldHolder";
        loader = new CustomClassloader(GarbageCollectionNullExample.class.getClassLoader());
        clazz = loader.loadClass(className);
        instance = clazz.getConstructor().newInstance();
        clazz.getMethod(METHOD_NAME).invoke(instance);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
    loader = null;
    clazz = null;
    instance = null;
    System.gc();
    Thread.sleep(1000);
}
```

尽管这段代码存在一些问题，但它在大多数情况下都能正常工作。**主要问题是我们不能在Java中强制进行垃圾回收，并且[System.gc()](https://www.baeldung.com/java-system-gc#systemgc)的调用不能保证垃圾回收一定会发**生。然而，在大多数JVM实现中，这将触发[Major GC](https://www.baeldung.com/java-choosing-gc-algorithm)。因此，我们应该在输出中看到以下几行：

```text
Hello from a garbage collected static field 
The object is garbage now
```

这个输出告诉我们垃圾回收器删除了静态字段。垃圾回收器还删除了类加载器、持有者的类、静态字段和连接的对象。

### 3.7 不使用System.gc()的示例

我们还可以更自然地触发垃圾回收，这样方式会更稳定。但是，它需要更多的周期来调用垃圾回收器：

```java
public static void main(String[] args) {
    while (true) {
        loadClass();
    }
}
```

这里我们使用相同的loadClass()方法，但我们不调用System.gc()，并且垃圾回收器会在内存不足时触发，因为我们在无限循环中加载类。

## 4. 总结

本文向我们介绍了垃圾回收在Java中如何针对类和静态字段进行工作。我们创建了一个自定义类加载器并将其用于我们的示例。此外，我们了解了类加载器、类及其静态字段之间的联系。为了进一步理解，值得阅读文本中链接的文章。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-2)上获得。
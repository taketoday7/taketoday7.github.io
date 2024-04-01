---
layout: post
title:  列出JVM中加载的所有类
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在本教程中，我们将学习不同的技术来列出JVM中[加载的所有类](https://www.baeldung.com/java-classloaders)。例如，我们可以加载JVM的堆转储或将正在运行的应用程序连接到各种工具并列出该工具中加载的所有类。此外，还有各种库可以以编程方式完成此操作。

我们将探讨非编程化和编程化方法。

## 2. 非编程化方法

### 2.1 使用VM参数

列出所有加载的类的最直接方法是将其记录在控制台输出或文件中。

我们将使用以下JVM参数运行Java应用程序：

```shell
java <app_name> --verbose:class
```

```text
[Opened /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Object from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.io.Serializable from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.Comparable from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.CharSequence from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.String from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.reflect.AnnotatedElement from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.reflect.GenericDeclaration from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.reflect.Type from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
[Loaded java.lang.Class from /Library/Java/JavaVirtualMachines/jdk1.8.0_241.jdk/Contents/Home/jre/lib/rt.jar] 
...............................
```

对于Java 9，我们将使用-Xlog JVM参数来将加载的类记录到文件中：

```shell
java <app_name> -Xlog:class+load=info:classloaded.txt
```

### 2.2 使用堆转储

我们将看到不同的工具如何使用JVM[堆转储](https://www.baeldung.com/java-heap-dump-capture)来提取类加载信息。但是，首先，我们将使用以下命令生成堆转储：

```shell
jmap -dump:format=b,file=/opt/tmp/heapdump.bin <app_pid>
```

可以在各种工具中打开上面的堆转储以获得不同的指标。

在Eclipse中，我们将在[Eclipse内存分析器](https://www.eclipse.org/mat/)中加载堆转储文件heapdump.bin并使用直方图接口：

![](/assets/images/2023/javajvm/jvmlistallclassesloaded01.png)

现在，我们将在[Java VisualVM](https://visualvm.github.io/)中打开堆转储文件heapdump.bin并使用按实例或大小划分的类选项：

![](/assets/images/2023/javajvm/jvmlistallclassesloaded02.png)

### 2.3 JProfiler

[JProfiler](https://www.baeldung.com/java-profilers)是顶级Java应用程序分析器之一，具有一组丰富的功能来查看不同的指标。

**在[JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)中，我们可以附加到正在运行的JVM或加载堆转储文件并获取所有与JVM相关的指标，包括所有加载类的名称**。

我们将使用附加进程功能让JProfiler连接到正在运行的应用程序ListLoadedClass：

![](/assets/images/2023/javajvm/jvmlistallclassesloaded03.png)

然后，我们将获取应用程序的快照并使用它来加载所有类：

![](/assets/images/2023/javajvm/jvmlistallclassesloaded04.png)

下面，我们可以使用Heap Walker功能查看加载类实例计数的名称：

![](/assets/images/2023/javajvm/jvmlistallclassesloaded05.png)

## 3. 编程化方法

### 3.1 Instrumentation API

Java提供了[Instrumentation API](https://www.baeldung.com/java-list-classes-class-loader)，这有助于获取有关应用程序的有价值的指标。首先，我们必须创建并加载一个Java代理，以将Instrumentation接口的实例获取到应用程序中。Java代理是一种检测在JVM上运行的程序的工具。

然后，我们需要调用Instrumentation方法[getInitiatedClasses(Classloader loader)](https://docs.oracle.com/en/java/javase/14/docs/api/java.instrument/java/lang/instrument/Instrumentation.html#getInitiatedClasses(java.lang.ClassLoader))来获取由特定类加载器类型加载的所有类。

### 3.2 Google Guava

我们将看到Guava库如何使用当前类加载器获取加载到JVM中的所有类的列表。

让我们首先将[Guava依赖项](https://search.maven.org/search?q=g:com.google.guavaANDa:guava)添加到我们的Maven项目中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

我们将使用当前类加载器实例初始化ClassPath对象：

```java
ClassPath classPath = ClassPath.from(ListLoadedClass.class.getClassLoader());
Set<ClassInfo> classes = classPath.getAllClasses();
Assertions.assertTrue(4 < classes.size());
```

### 3.3 反射API

我们将使用[Reflections](https://www.baeldung.com/reflections-library)库来扫描当前类路径并允许我们在运行时查询它。

让我们首先将[reflections依赖项](https://search.maven.org/search?q=g:org.reflectionsANDa:reflections)添加到我们的Maven项目中：

```xml
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.10.2</version>
</dependency>
```

现在，我们将研究示例代码，它返回包下的一组类：

```java
Reflections reflections = new Reflections(packageName, new SubTypesScanner(false));
Set<Class> classes = reflections.getSubTypesOf(Object.class)
    .stream()
    .collect(Collectors.toSet());
Assertions.assertEquals(4, classes.size());
```

## 4. 总结

在本文中，我们学习了列出JVM中加载的所有类的各种方法。首先，我们了解了如何使用VM参数记录已加载类的列表。

然后，我们探讨了各种工具如何加载堆转储或连接到JVM以显示各种指标，包括加载的类。最后，我们介绍了一些Java库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-2)上获得。
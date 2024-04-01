---
layout: post
title:  列出在特定类加载器中加载的所有类
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在本教程中，我们将分析使用[Java Instrumentation API](https://www.baeldung.com/java-instrumentation)列出Java中特定类加载器加载的所有类的技术。我们还将了解如何创建和加载Java代理以获取Instrumentation实例并调用所需的方法来完成我们的任务。

## 2. Java中的类加载器

类加载器是JRE(Java运行时环境)的组成部分，他们的工作是**将类动态加载到Java虚拟机中**。换句话说，它们会在应用程序需要时按需将类加载到内存中。有关[Java类加载器](https://www.baeldung.com/java-classloaders)的文章讨论了它们的不同类型，并详细介绍了它们的工作原理。

## 3. 使用Instrumentation API

**Instrumentation接口提供了[getInitiatedClasses(Classloader loader)](https://docs.oracle.com/en/java/javase/14/docs/api/java.instrument/java/lang/instrument/Instrumentation.html#getInitiatedClasses(java.lang.ClassLoader))方法，可以调用该方法返回一个数组，该数组包含由特定加载器加载的所有类**。让我们看看这是如何工作的。

首先，我们需要创建并加载一个代理来获取Instrumentation接口的实例。Java代理是一种工具，用于检测在JVM(Java虚拟机)上运行的程序。

换句话说，它可以添加或修改方法的字节码以收集数据。我们将需要一个代理来获取Instrumentation实例的句柄并调用所需的方法。

有多种方法可以[创建和加载代理](https://www.baeldung.com/java-instrumentation)。在本教程中，我们将选择使用premain方法和-javaagent选项的静态加载方法。

### 3.1 创建Java代理

**要创建Java代理，我们需要定义premain方法，Instrumentation实例将在代理加载时传递到该方法**。现在让我们创建ListLoadedClassesAgent类：

```java
public class ListLoadedClassesAgent {

    private static Instrumentation instrumentation;

    public static void premain(String agentArgs, Instrumentation instrumentation) {
        ListLoadedClassesAgent.instrumentation = instrumentation;
    }
}
```

### 3.2 定义listLoadedClasses方法

除了定义代理之外，我们还将定义并公开一个静态方法来返回给定的类加载器已加载的类。

请注意，**如果我们将具有null值的类加载器传递给getInitiatedClasses方法，它会返回由引导类加载器加载的类**。

让我们看看实际的代码：

```java
public static Class<?>[] listLoadedClasses(String classLoaderType) {
    return instrumentation.getInitiatedClasses(getClassLoader(classLoaderType));
}

private static ClassLoader getClassLoader(String classLoaderType) {
    ClassLoader classLoader = null;
    switch (classLoaderType) {
        case "SYSTEM":
            classLoader = ClassLoader.getSystemClassLoader();
            break;
        case "EXTENSION":
            classLoader = ClassLoader.getSystemClassLoader().getParent();
            break;
        case "BOOTSTRAP":
            break;
        default:
            break;
    }
    return classLoader;
}
```

请注意，如果我们使用的是Java 9或更高版本，则可以使用getPlatformClassLoader方法。这将列出平台类加载器加载的类。在这种情况下，switch语句还将包含：

```java
case "PLATFORM":
    classLoader = ClassLoader.getPlatformClassLoader();
    break;
```

### 3.3 创建代理Manifest文件

现在，让我们创建一个manifest文件MANIFEST.MF，其中包含适合我们的代理运行的属性，包括：

```manifest
Premain-Class: cn.tuyucheng.taketoday.loadedclasslisting.ListLoadedClassesAgent
```

代理JAR文件的完整清单属性列表可在[java.lang.instrument](https://docs.oracle.com/en/java/javase/14/docs/api/java.instrument/java/lang/instrument/package-summary.html)包的官方文档中找到。

### 3.4 加载代理并运行应用程序

现在让我们加载代理并运行应用程序。首先，我们需要代理JAR文件，其中包含一个包含Premain-Class信息的manifest文件。此外，我们需要应用程序JAR文件以及包含Main-Class信息的manifest文件。包含main方法的Launcher类将启动我们的应用程序。然后我们将能够打印由不同类型的类加载器加载的类：

```java
public class Launcher {

    public static void main(String[] args) {
        printClassesLoadedBy("BOOTSTRAP");
        printClassesLoadedBy("SYSTEM");
        printClassesLoadedBy("EXTENSION");
    }

    private static void printClassesLoadedBy(String classLoaderType) {
        System.out.println(classLoaderType + " ClassLoader : ");
        Class<?>[] classes = ListLoadedClassesAgent.listLoadedClasses(classLoaderType);
        Arrays.asList(classes)
                .forEach(clazz -> System.out.println(clazz.getCanonicalName()));
    }
}
```

接下来，让我们静态[加载Java代理](https://www.baeldung.com/java-instrumentation#loading-a-java-agent)并启动我们的应用程序：

```java
java -javaagent:agent.jar -jar app.jar
```

运行上述命令后，我们将看到输出：

```text
BOOTSTRAP ClassLoader :
java.lang.ClassValue.Entry[]
java.util.concurrent.ConcurrentHashMap.Segment
java.util.concurrent.ConcurrentHashMap.Segment[]
java.util.StringTokenizer
..............
SYSTEM ClassLoader : 
java.lang.Object[]
java.lang.Object[][]
java.lang.Class
java.lang.Class[]
..............
EXTENSION ClassLoader :
byte[]
char[]
int[]
int[][]
short[]
```

## 4. 总结

在本教程中，我们了解了列出在特定类加载器中加载的所有类的技术。

首先，我们创建了Java代理。之后，我们定义了使用Java Instrumentation API列出加载类的方法。最后，我们创建了代理manifest文件、加载代理并运行我们的应用程序。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-2)上获得。
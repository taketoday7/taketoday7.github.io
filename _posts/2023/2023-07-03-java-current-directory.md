---
layout: post
title:  在Java中获取当前工作目录
category: java
copyright: java
excerpt: Java OS
---

## 1. 概述

在Java中获取当前工作目录是一项简单的任务，但不幸的是，JDK中没有可用的直接API来执行此操作。

在本教程中，我们将学习如何使用java.lang.System、java.io.File、java.nio.file.FileSystems和java.nio.file.Paths获取Java中的当前工作目录。

## 2. System

让我们从使用System#getProperty的标准解决方案开始，假设我们当前的工作目录名称在整个代码中都是Tuyucheng：

```java
static final String CURRENT_DIR = "Tuyucheng";
@Test
void whenUsingSystemProperties_thenReturnCurrentDirectory() {
    String userDirectory = System.getProperty("user.dir");
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

我们使用Java内置的属性键user.dir从System的属性映射中获取当前工作目录，此解决方案适用于所有JDK版本。

## 3. File

让我们看看另一个使用java.io.File的解决方案：

```java
@Test
void whenUsingJavaIoFile_thenReturnCurrentDirectory() {
    String userDirectory = new File("").getAbsolutePath();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

在这里，File#getAbsolutePath在内部使用System#getProperty来获取目录名称，类似于我们的第一个解决方案。这是获取当前工作目录的非标准解决方案，并且适用于所有JDK版本。

## 4. FileSystems

另一个有效的替代方法是使用新的java.nio.file.FileSystems API：

```java
@Test
void whenUsingJavaNioFileSystems_thenReturnCurrentDirectory() {
    String userDirectory = FileSystems.getDefault()
        .getPath("")
        .toAbsolutePath()
        .toString();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

该解决方案使用新的[Java NIO API](https://www.baeldung.com/java-nio-2-file-api)，并且仅适用于**JDK 7或更高版本**。

## 5. Paths

最后，让我们看看使用java.nio.file.Paths API获取当前目录的更简单的解决方案：

```java
@Test
void whenUsingJavaNioPaths_thenReturnCurrentDirectory() {
    String userDirectory = Paths.get("")
        .toAbsolutePath()
        .toString();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

在这里，Paths#get在内部使用FileSystem#getPath来获取路径。它使用新的[Java NIO API](https://www.baeldung.com/java-nio-2-file-api)，因此该解决方案仅适用于**JDK 7或更高版本**。

## 6. 总结

在本教程中，我们探讨了四种在Java中获取当前工作目录的不同方法。前两个解决方案适用于所有版本的JDK，而后两个解决方案仅适用于JDK 7或更高版本。

我们建议使用System解决方案，因为它高效且直接，我们可以通过将此API调用包装在静态实用程序方法中并直接访问它来简化。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-os)上获得。
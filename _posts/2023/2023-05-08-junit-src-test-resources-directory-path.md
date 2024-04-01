---
layout: post
title:  在JUnit中获取/src/test/resources目录的路径
category: unittest
copyright: unittest
excerpt: JUnit 5类路径获取
---

## 1. 概述

有时在单元测试期间，我们可能需要从类路径中读取文件或将文件传递给被测对象。我们还可能在src/test/resources目录中有一个文件，其中包含[WireMock](https://www.baeldung.com/introduction-to-wiremock)等库可以使用的stub数据。

在本教程中，我们将学习**如何读取/src/test/resources目录的路径**。

## 2. Maven依赖

首先，我们需要将JUnit 5添加到我们的Maven依赖项中：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version>
</dependency>
```

最新版本的JUnit 5可以在[Maven Central](https://central.sonatype.com/artifact/org.junit.jupiter/junit-jupiter-engine/5.9.3)上找到。

## 3. 使用java.io.File

**最简单的方法是使用java.io.File类的实例**通过调用getAbsolutePath()方法来读取/src/test/resources目录：

```java
@Test
void givenResourcePath_whenReadAbsolutePathWithFile_thenAbsolutePathEndsWithDirectory() {
    String path = "src/test/resources";

    File file = new File(path);
    String absolutePath = file.getAbsolutePath();

    LOGGER.debug(absolutePath);
    assertTrue(absolutePath.endsWith("src" + File.separator + "test" + File.separator + "resources"));
}
```

请注意，**此路径是相对于当前工作目录的，即项目目录**。

让我们看一下在Windows上运行测试时的示例输出：

![](/assets/images/2023/unittest/junit5resourcespath01.png)

## 4. 使用java.nio.file.Path

**接下来，我们可以使用Java 7中引入的Path类**。

首先，我们需要调用静态工厂方法Paths.get()。然后我们将Path转换为File。最后，我们只需要调用getAbsolutePath()，如前面的例子所示：

```java
@Test
void givenResourcePath_whenReadAbsolutePathWithPaths_thenAbsolutePathEndsWithDirectory() {
    Path resourceDirectory = Paths.get("src", "test", "resources");

    String absolutePath = resourceDirectory.toFile().getAbsolutePath();

    LOGGER.debug(absolutePath);
    assertTrue(absolutePath.endsWith("src" + File.separator + "test" + File.separator + "resources"));
}
```

输出结果与上一个测试相同。

## 5. 使用ClassLoader

最后，我们还可以使用ClassLoader类：

```java
@Test
void givenResourceFile_whenReadResourceWithClassLoader_thenPathEndWithFilename() {
    String resourceName = "example_resource.txt";

    ClassLoader classLoader = getClass().getClassLoader();
    File file = new File(classLoader.getResource(resourceName).getFile());
    String absolutePath = file.getAbsolutePath();

    LOGGER.debug(absolutePath);
    assertTrue(absolutePath.endsWith(File.separator + "example_resource.txt"));
}
```

让我们看一下输出：

```text
D:\workspace\intellij\taketoday-tutorial4j\software.test\junit-5-basics\target\test-classes\example_resource.txt
```

请注意，这一次，我们有一个junit-5-basics\target\test-classes\example_resource.txt文件。当我们将结果与以前的方法进行比较时，它会有所不同。

这是因为**ClassLoader在类路径中查找资源**。在Maven中，编译后的类和资源放在/target/目录中。这就是为什么这一次，我们得到的是一个类路径资源的路径。

## 6. 总结

在这篇简短的文章中，我们讨论了如何在JUnit 5中读取/src/test/resources目录。

根据我们的需要，我们可以使用多种方法来实现我们的目标：File、Paths或ClassLoader类。

与往常一样，所有示例都可以在我们的[GitHub项目](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)中找到。
---
layout: post
title:  自执行JAR中Main清单属性的重要性
category: java
copyright: java
excerpt: Java Jar
---

## 1. 概述

每个可执行的Java类都必须包含一个main方法。简而言之，此方法是应用程序的起点。

要从自执行JAR文件运行我们的main方法，我们必须创建一个适当的manifest文件并将其与我们的代码一起打包。这个manifest文件必须有一个main清单属性，它定义了包含我们的main方法的类的路径。

在本教程中，我们将展示如何**将一个简单的Java类打包为一个自执行的JAR**，并演示main清单属性对于成功执行的重要性。

## 2. 执行没有Main Manifest属性的JAR

为了更加实用，我们将展示一个没有适当清单属性的失败执行示例。

让我们编写一个带有main方法的简单Java类：

```java
public class AppExample {
    public static void main(String[] args) {
        System.out.println("AppExample executed!");
    }
}
```

要将我们的示例类打包到JAR存档中，我们必须转到操作系统的shell并编译它：

```shell
javac -d . AppExample.java
```

然后我们可以把它打包成一个JAR：

```bash
jar cvf example.jar cn/tuyucheng/taketoday/manifest/AppExample.class
```

我们的example.jar将包含一个默认清单文件。现在我们可以尝试执行JAR：

```shell
java -jar example.jar
```

执行将失败并出现错误：

```plaintext
nomainmanifest attribute, in example.jar
```

## 3. 使用Main Manifest属性执行JAR

正如我们所看到的，JVM无法找到我们的main清单属性。因此，它找不到包含main方法的主类。

让我们将适当的清单属性与我们的代码一起包含到JAR中，我们需要创建一个包含单行的MANIFEST.MF文件：

```manifest
Main-Class: manifest.cn.tuyucheng.taketoday.AppExample
```

我们的manifest现在包含我们编译的AppExample.class的类路径。因为我们已经编译了我们的示例类，所以没有必要再做一次。

我们将把它与我们的manifest文件打包在一起：

```shell
jar cvmf MANIFEST.MF example.jar cn/tuyucheng/taketoday/manifest/AppExample.class
```

这次JAR按预期执行并输出：

```plaintext
AppExample executed!
```

## 4. 总结

在这篇简短的文章中，我们展示了如何将一个简单的Java类打包为一个自执行的JAR，并且我们在两个简单的示例中展示了main清单属性的重要性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jar)上获得。
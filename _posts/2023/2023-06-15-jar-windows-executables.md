---
layout: post
title:  从Java创建Jar可执行文件和Windows可执行文件指南
category: java
copyright: java
excerpt: Java Jar
---

## 1. 概述

在本教程中，我们将首先学习如何将Java程序打包到可执行的Java ARchive(JAR)文件中。然后，我们将了解如何使用该可执行JAR生成Microsoft Windows支持的可执行文件。

**我们将使用Java附带的jar命令行工具来创建JAR文件。然后，我们将学习使用jpackage工具(在Java 16及更高版本中作为jdk.jpackage提供)来生成可执行文件**。

## 2. jar和jpackage命令的基础知识

[JAR文件](https://www.baeldung.com/java-create-jar)是已编译的Java类文件和其他资源的容器，它基于流行的ZIP文件格式。

可执行JAR文件也是一个JAR文件，但也包含一个主类。主类在manifest文件中引用，我们将在稍后讨论。

**为了运行以JAR格式交付的应用程序，我们必须拥有Java运行时环境(JRE)**。

**与JAR文件不同，特定于平台的可执行文件可以在其构建的平台上本地运行**。例如，该平台可以是Microsoft Windows、Linux或Apple macOS。

**为了获得良好的最终用户体验，最好为客户提供特定于平台的可执行文件**。

### 2.1 jar命令

创建JAR文件的一般语法是：

```shell
jar cf jar-file input-file(s)
```

让我们来看看在使用jar命令创建新存档时可以使用的一些选项：

-   c指定我们要创建一个JAR文件
-   f指定我们希望输出到一个文件
-   m用于包含来自现有清单文件的清单信息
-   jar-file是我们希望生成的JAR文件的名称，JAR文件通常带有.jar扩展名，但这不是必需的
-   input-file(s)是我们要包含在我们的JAR文件中的以空格分隔的文件名列表，通配符\*也可以在这里使用

创建JAR文件后，我们通常会检查它的内容。要查看JAR文件包含的内容，我们使用以下语法：

```shell
jar tf jar-file
```

这里，t表示我们要列出JAR文件的内容。f选项表示我们要检查的JAR文件是在命令行上指定的。

### 2.2 jpackage命令

**[jpackage](https://www.baeldung.com/java14-jpackage)命令行工具帮助我们为模块化和非模块化Java应用程序生成可安装的软件包**。

它使用[jlink](https://www.baeldung.com/jlink)命令为我们的应用程序生成Java运行时镜像。结果，我们获得了针对特定平台的自包含应用程序包。

由于应用程序包是为目标平台构建的，因此该系统必须包含以下内容：

-   应用程序本身
-   JDK
-   打包工具需要的软件。**对于Windows，jpackage需要WiX 3.0或更高版本**

下面是[jpackage](https://www.baeldung.com/java14-jpackage)命令的常用形式：

```shell
jpackage --input . --main-jar MyAppn.jar
```

## 3. 创建可执行文件

现在让我们创建一个可执行的JAR文件。准备就绪后，我们将着手生成Windows可执行文件。

### 3.1 创建可执行JAR文件

创建可执行JAR非常简单。我们首先需要一个Java项目，其中至少有一个带有main()方法的类。我们为示例创建了一个名为MySampleGUIAppn的Java类。

第二步是创建清单文件，让我们将清单文件创建为MySampleGUIAppn.mf：

```manifest
Manifest-Version: 1.0
Main-Class: MySampleGUIAppn
```

我们必须确保此清单文件末尾有换行符，它才能正常工作。

清单文件准备就绪后，我们将创建一个可执行JAR：

```shell
jar cmf MySampleGUIAppn.mf MySampleGUIAppn.jar MySampleGUIAppn.class MySampleGUIAppn.java
```

让我们查看我们创建的JAR的内容：

```shell
jar tf MySampleGUIAppn.jar
```

这是一个示例输出：

```text
META-INF/
META-INF/MANIFEST.MF
MySampleGUIAppn.class
MySampleGUIAppn.java
```

接下来，我们可以通过CLI或GUI运行我们的JAR可执行文件。

让我们在命令行上运行它：

```shell
java -jar MySampleGUIAppn.jar
```

在GUI中，我们只需双击相关的JAR文件即可，这应该像任何其他应用程序一样正常启动它。

### 3.2 创建Windows可执行文件

现在我们的可执行JAR已准备就绪并可以运行，让我们为示例项目生成一个Windows可执行文件：

```shell
jpackage --input . --main-jar MySampleGUIAppn.jar
```

此命令需要一段时间才能完成。完成后，它会在当前工作文件夹中生成一个exe文件。可执行文件的文件名将与清单文件中提到的版本号拼接，我们将能够像启动任何其他Windows应用程序一样启动它。

以下是我们可以与jpackage命令一起使用的一些特定于Windows的选项：

-   –type：指定msi而不是默认的exe格式
-   –win-console：使用控制台窗口启动我们的应用程序
-   –win-shortcut：在Windows开始菜单中创建一个快捷方式文件
-   –win-dir-chooser：让最终用户指定自定义目录来安装可执行文件
-   –win-menu–win-menu-group：让最终用户在开始菜单中指定自定义目录

## 4. 总结

在本文中，我们介绍了有关JAR文件和可执行JAR文件的一些基础知识。我们还了解了如何将Java程序转换为JAR可执行文件，然后再转换为Microsoft Windows支持的可执行文件。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jar)上获得。
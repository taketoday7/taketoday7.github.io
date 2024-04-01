---
layout: post
title:  Maven Reactor
category: maven
copyright: maven
excerpt: Maven
---

## 1. 简介

在本教程中，我们将快速了解Maven Reactor的基本概念及其在[Maven](https://www.baeldung.com/maven)生态系统中的位置。

我们将从Maven Reactor的介绍开始。接下来，我们将设置一个具有模块间依赖关系的[多模块Maven项目](https://www.baeldung.com/maven-multi-module)的简单示例，并查看Reactor的运行情况以整理构建依赖关系。我们将介绍一些可以微调Maven Reactor行为的可用标志。最后，我们将总结使用Reactor的一些好处。

## 2. Maven Reactor基础知识

**Maven Reactor是Maven的内置部分，负责管理项目依赖和构建。它负责Maven构建的执行，并确保项目以正确的顺序构建以满足依赖关系。Maven Reactor的真正好处可以在具有许多模块间依赖关系的复杂多模块项目中得到体现**。 

**Reactor使用[有向无环图(DAG)](https://www.baeldung.com/cs/dag-applications)来确定项目的构建顺序**。

它作为Maven核心的一部分执行以下功能：

-   收集所有可用于构建的模块
-   将项目组织成适当的构建顺序
-   依次执行选定的项目

## 3. 示例用例

让我们考虑一个涉及开发用于管理患者信息的基于Web的应用程序的项目。该项目由三个模块组成：

1.  patient-web模块：该模块用作应用程序的用户界面
2.  patient-data模块：该模块处理所有数据库CRUD操作
3.  patient-domain模块：该模块包含应用程序使用的域实体

在这个项目中，patient-web模块依赖于其他两个模块，因为它从持久存储中检索和显示数据。另一方面，patient-data模块依赖于patient-domain模块，因为它需要访问域实体以执行CRUD操作。请务必注意，patient-domain模块独立于其他两个模块。

### 3.1 Maven设置

为了实现我们的简单示例，让我们创建一个名为sample-reactor的多模块项目，它包含三个模块。这些模块中的每一个都将用于前面提到的目的：

![](/assets/images/2023/maven/javamavenreactor01.png)

此时，让我们来看看项目POM：

```xml
<artifactId>maven-reactor</artifactId>
<version>1.0.0</version>
<name>maven-reactor</name>
<packaging>pom</packaging>
<modules>
    <module>patient-web</module>
    <module>patient-data</module>
    <module>patient-domain</module>
</modules>
```

**本质上，我们在这里定义了一个多模块项目，所有三个模块都在项目pom.xml的<module\>..</module\>标签内声明**。

现在，让我们看一下patient-data模块的POM：

```xml
<artifactId>cn.tuyucheng.taketoday</artifactId>
<name>patient-data</name>
<parent>
    <groupId>cn.tuyucheng.taketoday</groupId>
    <artifactId>maven-reactor</artifactId>
    <version>1.0.0</version>
</parent>
<dependencies>
    <dependency>
        <groupId>cn.tuyucheng.taketoday</groupId>
        <artifactId>patient-domain</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

**在这里，我们可以看到patient-data依赖于patient-domain**。

对于我们的用例，我们假设patient-domain模块独立于其余模块并且可以独立构建。它的POM看起来像这样：

```xml
<artifactId>patient-domain</artifactId>
<name>patient-domain</name>
    <parent>
        <groupId>cn.tuyucheng.taketoday</groupId>
        <artifactId>maven-reactor</artifactId>
        <version>1.0.0</version>
    </parent>
</project>
```

**最后，patient-web应该同时依赖于patient-data和patient-domain**：

```xml
<artifactId>patient-web</artifactId>
<version>1.0.0</version>
<name>patient-web</name>
    <parent>
        <groupId>cn.tuyucheng.taketoday</groupId>
        <artifactId>maven-reactor</artifactId>
        <version>1.0.0</version>
    </parent>
<dependencies>
    <dependency>
        <groupId>cn.tuyucheng.taketoday</groupId>
        <artifactId>patient-data</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>cn.tuyucheng.taketoday</groupId>
        <artifactId>patient-domain</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

### 3.2 Maven Reactor实践

要查看Reactor的运行情况，让我们转到项目的父目录(maven-reactor)并执行mvn clean install。

此时，Maven将使用Reactor执行以下任务：

1.  收集项目中的所有可用模块(在本例中为patient-web、patient-data和patient-domain)
2.  **根据模块的依赖关系确定构建模块的正确顺序(在这种情况下，patient-domain必须在patient-data之前构建，而patient-data必须在patient-web之前构建)**
3.  以正确的顺序构建每个模块，确保正确解决依赖关系

以下是成功构建的Reactor构建顺序：

![](/assets/images/2023/maven/javamavenreactor02.png)

## 4. 配置Reactor

虽然默认情况下Reactor是Maven的一部分，但我们仍然可以通过使用几个[命令行开关](https://www.baeldung.com/maven-arguments)来修改它的行为。这些开关被认为是必不可少的，因为它们允许我们控制Reactor如何构建我们的项目。一些需要考虑的基本开关是：

-   –resume-from：允许我们从特定项目恢复Reactor，以防它在构建过程中失败
-   –also-make：在Reactor中构建指定的项目及其任何依赖项
-   -also-make-dependents：构建指定的项目和任何依赖它们的项目
-   **–fail-fast：每当模块构建失败时立即停止整个构建(默认)**
-   –fail-at-end：即使特定模块构建失败，此选项也会继续Reactor构建，并在最后报告所有失败的模块
-   –non-recursive：使用这个选项，我们可以禁用Reactor构建，只构建当前目录下的项目，即使项目的pom声明了其他模块

通过使用这些选项，我们可以微调Reactor的行为，并按照我们的需要构建我们的项目。

## 5. 总结

在本文中，我们快速了解了使用Maven Reactor作为Apache Maven生态系统的一部分来构建多模块复杂项目的好处，它消除了开发人员解决依赖关系和构建顺序的责任，并减少了构建时间。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/maven.modules)上获得。
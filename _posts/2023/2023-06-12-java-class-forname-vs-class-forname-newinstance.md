---
layout: post
title:  Class.forName()和Class.forName().newInstance()之间的区别
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在Java中，类的动态加载涉及在运行时而不是编译时将类加载到Java虚拟机中。当我们在编译时不知道类名的情况下，或者当类加载基于用户输入或系统属性时，这种方法被证明是有益的。

有几种方法可以在Java中动态加载类，包括Class.forName()方法、ClassLoader API和依赖注入框架。

在本文中，我们将研究Class.forName()和Class.forName().newInstance()方法，它们在Java应用程序中很常用，对于开发人员来说是必不可少的。

## 2. Class.forName()方法

**Class.forName()方法将类的完全限定名称作为参数，并返回与加载的类关联的Class对象**。在执行期间，它会尝试定位、加载和链接类或接口。当我们想要获取有关类的信息(例如字段和方法)时，通常会使用它。但是，我们应该注意，**该类未初始化，调用它的方法是不可能的**。

让我们编写一个使用Class.forName()方法的快速测试：

```java
@Test
public void whenUseClassForNameCreatedInstanceOfClassClass() throws... {
   Class instance = Class.forName("cn.tuyucheng.taketoday.loadclass.MyClassForLoad");
   assertInstanceOf(Class.class, instance, "instance should be of Class.class");
}
```

同样重要的是要记住，**动态加载方法不会验证请求的类对其调用者的可访问性**，从而导致与加载、链接或初始化相关的潜在异常。正确处理它们对于避免意外行为和潜在崩溃至关重要：

-   LinkageError：如果链接失败
-   ExceptionInInitializerError：如果初始化失败
-   ClassNotFoundException：如果找不到类

## 3. Class.forName().newInstance()方法

当我们想要创建一个类的实例时，Class.forName().newInstance()方法是合适的。**此方法也将类的完全限定名称作为参数，并返回此Class对象表示的类的新实例**。这就像使用空参数列表为MyClassForLoad调用new运算符。

让我们看一个在单元测试中使用Class.forName().newInstance()方法的示例：

```java
@Test
public void whenUseClassForNameWithNewInstanceCreatedInstanceOfTargetClass throws... () {
   Object instance = Class.forName("cn.tuyucheng.taketoday.loadclass.MyClassForLoad").newInstance();
   assertInstanceOf(MyClassForLoad.class, instance, "instance should be of MyClassForLoad.class");
}
```

请记住，Class.forName().newInstance()可能会抛出以下几种异常之一：

-   IllegalAccessException：如果类或其空参构造函数不可访问
-   InstantiationException：如果此类表示抽象类、接口、数组类、原始类型或void
-   ExceptionInInitializerError：如果此方法引发的初始化失败
-   SecurityException：当调用代码没有足够的权限访问指定的类时

但是，请务必注意**newInstance()自Java 9以来已被弃用**，因为它会传播空参构造函数抛出的任何异常，包括受检的异常。此方法有效地绕过了编译器执行的编译时异常检查。因此，它可能会导致意外错误或故障，并导致代码库健壮性和可维护性降低。

**建议调用getDeclaredConstructor().newInstance()构造来避免此问题**：

```java
Object instance = Class.forName("cn.tuyucheng.taketoday.loadclass.MyClassForLoad").getDeclaredConstructor().newInstance();
```

## 4. 总结

开发人员必须深入了解Java中动态类加载的工作原理，它提供了许多有助于提高Java应用程序的性能、可维护性和可伸缩性的好处。

在本文中，我们了解到Class.forName()和Class.forName().newInstance()是Java中的两个关键方法，它们允许我们在运行时加载类。这些方法之间的区别在于它们做什么和返回什么。 

**Class.forName()动态加载一个类并返回一个Class对象，而Class.forName().newInstance()加载一个类并创建加载的类的实例**。

了解这些方法及其差异至关重要，因为它们经常用于现实场景中： 

-   插件开发
-   依赖注入框架，如Spring或Guice
-   基于正在使用的数据库加载JDBC驱动程序
-   根据入参加载类

最后，通过了解Class.forName()和Class.forName().newInstance()之间的区别，开发人员可以就如何在他们的Java项目中动态加载类做出明智的决定，从而产生更高效的代码。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-3)上获得。
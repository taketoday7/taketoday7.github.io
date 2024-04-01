---
layout: post
title:  Java 9迁移问题和解决方案
category: java
copyright: java
excerpt: Java 9
---

## 1. 概述

Java平台曾经有一个整体式架构，将所有包捆绑为一个单元。

在Java 9中，随着[Java平台模块系统(JPMS)](https://www.baeldung.com/java-9-modularity)或简称模块的引入，相关的包被归类到模块下，**模块取代包成为复用的基本单位**。

在这个简短的教程中，我们将介绍在将现有应用程序迁移到Java 9时可能会遇到的一些与模块相关的问题。

## 2. 简单示例

让我们来看一个简单的Java 8应用程序，它包含4个方法，这4个方法在Java 8下有效，但在未来的版本中具有挑战性。我们将使用这些方法来了解迁移到Java 9的影响。

第一个方法**获取应用程序中引用的JCE提供者的名称**：

```java
private static void getCrytpographyProviderName() {
    LOGGER.info("1. JCE Provider Name: {}\n", new SunJCE().getName());
}
```

第二个方法**列出堆栈跟踪中类的名称**：

```java
private static void getCallStackClassNames() {
    StringBuffer sbStack = new StringBuffer();
    int i = 0;
    Class<?> caller = Reflection.getCallerClass(i++);
    do {
        sbStack.append(i + ".").append(caller.getName())
            .append("\n");
        caller = Reflection.getCallerClass(i++);
    } while (caller != null);
    LOGGER.info("2. Call Stack:\n{}", sbStack);
}
```

第三个方法**将Java对象转换为XML**：

```java
private static void getXmlFromObject(Book book) throws JAXBException {
    Marshaller marshallerObj = JAXBContext.newInstance(Book.class).createMarshaller();
    marshallerObj.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);

    StringWriter sw = new StringWriter();
    marshallerObj.marshal(book, sw);
    LOGGER.info("3. Xml for Book object:\n{}", sw);
}
```

最后一个方法**使用来自JDK内部库的sun.misc.BASE64Encoder将字符串编码为Base64**：

```java
private static void getBase64EncodedString(String inputString) {
    String encodedString = new BASE64Encoder().encode(inputString.getBytes());
    LOGGER.info("4. Base Encoded String: {}", encodedString);
}
```

让我们从main方法调用所有方法：

```java
public static void main(String[] args) throws Exception {
    getCrytpographyProviderName();
    getCallStackClassNames();
    getXmlFromObject(new Book(100, "Java Modules Architecture"));
    getBase64EncodedString("Java");
}
```

当我们在Java 8中运行这个应用程序时，我们得到以下信息：

```bash
> java -jar target\pre-jpms.jar
[INFO] 1. JCE Provider Name: SunJCE

[INFO] 2. Call Stack:
1.sun.reflect.Reflection
2.cn.tuyucheng.taketoday.prejpms.App
3.cn.tuyucheng.taketoday.prejpms.App

[INFO] 3. Xml for Book object:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="100">
    <title>Java Modules Architecture</title>
</book>

[INFO] 4. Base Encoded String: SmF2YQ==
```

通常，Java版本保证向后兼容性，但JPMS改变了其中的一些。

## 3. 在Java 9中执行

现在，让我们在Java 9中运行这个应用程序：

```bash
>java -jar target\pre-jpms.jar
[INFO] 1. JCE Provider Name: SunJCE

[INFO] 2. Call Stack:
1.sun.reflect.Reflection
2.cn.tuyucheng.taketoday.prejpms.App
3.cn.tuyucheng.taketoday.prejpms.App

[ERROR] java.lang.NoClassDefFoundError: javax/xml/bind/JAXBContext
[ERROR] java.lang.NoClassDefFoundError: sun/misc/BASE64Encoder
```

我们可以看到前两个方法运行良好，而后两个方法失败了。**让我们通过分析应用程序的依赖关系来调查失败的原因**。我们将使用Java 9附带的jdeps工具：

```bash
>jdeps target\pre-jpms.jar
   cn.tuyucheng.taketoday.prejpms            -> com.sun.crypto.provider               JDK internal API (java.base)
   cn.tuyucheng.taketoday.prejpms            -> java.io                               java.base
   cn.tuyucheng.taketoday.prejpms            -> java.lang                             java.base
   cn.tuyucheng.taketoday.prejpms            -> javax.xml.bind                        java.xml.bind
   cn.tuyucheng.taketoday.prejpms            -> javax.xml.bind.annotation             java.xml.bind
   cn.tuyucheng.taketoday.prejpms            -> org.slf4j                             not found
   cn.tuyucheng.taketoday.prejpms            -> sun.misc                              JDK internal API (JDK removed internal API)
   cn.tuyucheng.taketoday.prejpms            -> sun.reflect                           JDK internal API (jdk.unsupported)
```

该命令的输出给出：

-   第一列包含我们应用程序中所有包的列表
-   第二列包含我们应用程序中所有依赖项的列表
-   依赖项在Java 9平台中的位置-**这可以是模块名称，或内部JDK API，或者第三方库没有对应的模块**

## 4. 弃用的模块

现在让我们尝试解决第一个错误java.lang.NoClassDefFoundError: javax/xml/bind/JAXBContext。

根据依赖项列表，我们知道java.xml.bind包属于java.xml.bind模块，这似乎是一个有效的模块。那么，让我们看看[这个模块的官方文档](https://docs.oracle.com/javase/9/docs/api/java.xml.bind-summary.html)吧。

官方文档说java.xml.bind模块已弃用，将在未来的版本中删除。因此，默认情况下该模块不会加载到类路径中。

但是，**Java提供了一种通过使用–add-modules选项按需加载模块的方法**。所以，让我们继续尝试一下：

```bash
>java --add-modules java.xml.bind -jar target\pre-jpms.jar
...
INFO 3. Xml for Book object:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="100">
    <title>Java Modules Architecture</title>
</book>
...
```

我们可以看到执行成功了。此解决方案快速简便，但不是最佳解决方案。

作为长期解决方案，我们应该使用Maven添加[依赖项](https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api)作为第三方库：

```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.4.0-b180725.0427</version>
</dependency>
```

## 5. JDK内部API

现在让我们看看第二个错误java.lang.NoClassDefFoundError: sun/misc/BASE64Encoder。

从依赖列表可以看出sun.misc包是一个JDK内部API。

顾名思义，内部API就是私有代码，仅供在JDK内部使用。

在我们的示例中，**内部API似乎已从JDK中删除**。让我们使用–jdk-internals选项来检查替代API是什么：

```bash
>jdeps --jdk-internals target\pre-jpms.jar
...
JDK Internal API                         Suggested Replacement
----------------                         ---------------------
com.sun.crypto.provider.SunJCE           Use java.security.Security.getProvider(provider-name) @since 1.3
sun.misc.BASE64Encoder                   Use java.util.Base64 @since 1.8
sun.reflect.Reflection                   Use java.lang.StackWalker @since 9
```

我们可以看到Java 9推荐使用java.util.Base64而不是sun.misc.Base64Encoder。因此，我们的应用程序必须更改代码才能在Java 9中运行。

请注意，我们在应用程序中使用了另外两个内部API，Java平台建议对其进行替换，但我们没有收到任何错误：

-   **一些内部API(如sun.reflect.Reflection)被认为对平台至关重要，因此被添加到特定于JDK的jdk.unsupported模块中**。这个模块默认在Java 9的类路径中可用。
-   **com.sun.crypto.provider.SunJCE等内部API仅在某些Java实现中提供**，只要使用它们的代码在同一个实现上运行，它就不会抛出任何错误。

在此示例中的所有情况下，我们都使用内部API，这不是推荐的做法。因此，长期的解决方案是将它们替换为平台提供的合适的公共API。

## 6. 总结

在本文中，我们看到了Java 9中引入的模块系统如何导致一些使用已弃用API或内部API的旧应用程序出现迁移问题。

我们还看到了如何对这些错误应用短期和长期修复。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-pre-jpms)上获得。
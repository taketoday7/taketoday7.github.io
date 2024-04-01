---
layout: post
title:  Spring Boot属性前缀必须采用规范形式
category: springboot
copyright: springboot
excerpt: Spring Boot Properties
---

## 1. 概述

在这个快速教程中，我们将仔细研究[Spring Boot](https://www.baeldung.com/spring-boot)错误“Reason: Canonical names should be kebab-case (‘-‘ separated), lowercase alpha-numeric characters, and must start with a letter”。

首先，我们将阐明Spring Boot中出现此错误的主要原因。然后，我们将**通过一个实际示例深入探讨如何重现和解决该问题**。

## 2. 问题陈述

首先，让我们了解一下错误消息的含义。“**Canonical names should be kebab-case**”只是告诉我们规范名称(规范名称是指唯一标识属性的属性名称)应该是短横线大小写。

为了确保一致性，@ConfigurationProperties注解的prefix参数中使用的命名约定应遵循短横线大小写。

例如：

```java
@ConfigurationProperties(prefix = "my-example")
```

在上面的代码片段中，前缀my-example应遵守短横线大小写约定。

## 3. Maven依赖

由于这是一个基于Maven的项目，因此我们将必要的依赖项添加到pom.xml中：

```xml
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter</artifactId> 
    <version>3.0.5</version>
</dependency>
```

为了重现该问题，[spring-boot-starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-maven-plugin)是唯一需要的依赖。

## 4. 重现错误

### 4.1 应用程序配置

如果你不熟悉配置属性，可以通过阅读[Spring Boot中的配置属性指南](https://www.baeldung.com/configuration-properties-in-spring-boot)来更好的理解。

让我们注册所需的组件：

```java
@Configuration
@ConfigurationProperties(prefix = "customProperties")
public class MainConfiguration {
    String name;

    // getters and setters
}
```

然后，我们需要向application.properties文件添加自定义属性：

```properties
custom-properties.name="Tuyucheng"
```

application.properties位于src/main/resources下：

```text
|   pom.xml
+---src
|   +---main
|   |   +---java
|   |   |   \---cn
|   |   |       \---tuyucheng
|   |   |           \---taketoday
|   |   |               ...
|   |   |               ...
|   |   \---resources
|   |           application.properties
```

现在，让我们通过在项目根文件夹中执行mvn spring-boot:run命令来运行示例Spring Boot应用程序，看看会发生什么：

```shell
$ mvn spring-boot:run
...
...
***************************
APPLICATION FAILED TO START
***************************

Description:

Configuration property name 'customProperties' is not valid:

    Invalid characters: 'P'
    Bean: mainConfiguration
    Reason: Canonical names should be kebab-case ('-' separated), lowercase alpha-numeric characters and must start with a letter

Action:

Modify 'customProperties' so that it conforms to the canonical names requirements.
```

如上所示，我们收到一条错误消息**Modify ‘customProperties’ so that it conforms to the canonical names requirements**。此错误消息表明当前用于customProperties的命名约定不遵循Spring设置的命名约定，换句话说，需要更改名称customProperties以符合Spring中命名属性的要求。

## 5. 修复错误

我们需要更改属性前缀：

```java
@ConfigurationProperties(prefix = "customProperties")
```

为短横线大小写前缀：

```java
@ConfigurationProperties(prefix = "custom-properties")
```

在属性中，我们可以保留任何样式，并且可以很好地访问它们。

## 6. 短横线大小写的优点

在访问这些属性时使用短横线大小写的主要优点是我们可以使用以下任何样式：

- camelCaseLikeThis
- PascalCaseLikeThis
- snake_case_like_this
- kebab-case-like-this

在properties文件中并使用短横线大小写访问它们。

```java
@ConfigurationProperties(prefix = "custom-properties")
```

将能够访问以下任何属性：

```properties
customProperties.name="Tuyucheng"
CustomProperties.name="Tuyucheng"
custom_properties.name="Tuyucheng"
custom-properties.name="Tuyucheng"
```

## 7. 总结

在本教程中，我们了解到Spring Boot支持多种格式，包括属性名称中的驼峰命名法、蛇形命名法和短横线命名法，但鼓励我们以短横线命名法规范地访问它们，从而减少因命名约定不一致而导致错误或混乱的可能性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-properties-4)上获得。
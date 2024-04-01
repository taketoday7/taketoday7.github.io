---
layout: post
title:  如何将application.properties转换为Spring Boot的application.yml
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将学习如何将从[Spring Initializer](https://start.spring.io/)下载新的[Spring Boot](https://www.baeldung.com/spring-boot)项目时得到的默认application.properties文件转换为更具可读性的application.yml文件。

## 2. Properties和YML文件的区别

在直接进入主题之前，让我们以代码的形式看看这两种文件格式之间的区别。

**在application.properties文件中，属性存储为单行配置**。Spring Boot生成properties作为默认文件：

```properties
spring.datasource.url=jdbc:h2:mem:testDB
spring.datasource.username=user
spring.datasource.password=testpwd
```

另一方面，我们可以创建一个application.yml。这是一个基于YML的文件，与properties相比，在具有分层数据时更易于阅读：

```yaml
spring:
    datasource:
        url: jdbc:h2:mem:testDB
        username: user
        password: testpwd
```

**正如我们所看到的，在[基于YML的配置](https://www.baeldung.com/spring-boot-yaml-vs-properties)的帮助下，我们不再需要添加重复的前缀(spring.datasource)**。

## 3. 将Properties转换为YML，反之亦然

### 3.1 IntelliJ插件

如果我们使用IntelliJ作为IDE来运行Spring Boot应用程序，我们可以通过安装以下插件来进行转换：

[![Properties-To-YML-Intellij](https://www.baeldung.com/wp-content/uploads/2023/06/Plugin-properties-to-yml-1024x735.png)](https://www.baeldung.com/wp-content/uploads/2023/06/Plugin-properties-to-yml.png)

我们需要转到File > Settings > Plugins > Install “Convert YAML and Properties file”。

安装插件后，我们：

-   右键单击application.properties文件
-   选择“Convert YAML and Properties file”选项，自动将文件转换为application.yml

我们也能够将其转换回来。

### 3.2 在线网站工具

除了安装插件，我们还可以直接将配置从我们的代码库复制粘贴到[Mageddo](https://mageddo.com/tools/yaml-converter)转换器网站。

出于安全考虑，**请确保我们不会在第三方网站上输入转换密码**：

[![Mageddo-Properties-To-YML](https://www.baeldung.com/wp-content/uploads/2023/06/properties-to-yaml-2-1024x373.png)](https://www.baeldung.com/wp-content/uploads/2023/06/properties-to-yaml-2.png)

我们将代码放在相应的properties/YML部分中，并将其转换为另一种格式。

## 4. 总结

在本教程中，我们介绍了properties和yml文件之间的区别，并学习了如何使用突出显示的各种工具和插件将application.properties文件转换为application.yml，反之亦然。
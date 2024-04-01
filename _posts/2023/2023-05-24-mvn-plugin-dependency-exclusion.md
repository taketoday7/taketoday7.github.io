---
layout: post
title:  排除Maven插件中的依赖项
category: maven
copyright: maven
excerpt: Maven
---

## 1. 概述

在Maven中排除依赖是一个常见的操作。但是，在处理Maven插件时会变得更加困难。

## 2. 什么是依赖排除

Maven管理依赖项的传递性。这意味着**Maven可以通过我们添加的依赖自动添加所有需要的依赖**。在某些情况下，**这种传递性会迅速增加依赖项的数量，因为它增加了级联依赖项**。

例如，如果我们有A → B → C → D这样的依赖关系，那么A将依赖于B、C和D。如果A只使用B的一小部分，不需要C，那么可以告诉Maven忽略A中的B → C依赖。

因此，A将只依赖于B而不再依赖于C和D。这称为依赖排除。

## 3. 排除传递依赖

我们可以使用<exclusions\>元素排除子依赖项，该元素包含一组对特定依赖项的排除项。**简而言之，我们只需要在POM文件的<dependency\>元素中添加一个<exclusions\>元素**。

例如，让我们考虑commons-text依赖的例子，并假设我们的项目只使用来自commons-text的代码，不需要commons-lang子依赖。

我们将能够从我们项目的commons-text传递链中排除commons-lang依赖，只需在我们项目的POM文件中的commons-text依赖声明中添加一个<exclusions\>部分，如下所示：

```xml
<project>
    <!-- ... -->
    <dependencies>
        <!-- ... -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-text</artifactId>
            <version>1.1</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.commons</groupId>
                    <artifactId>commons-lang3</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <!-- ... -->
</project>
```

因此，如果我们用上面的POM重建一个项目，我们会看到commons-text库被集成到我们的项目中，而没有commons-lang库。

## 4. 从插件中排除传递依赖

到目前为止，Maven不支持从插件中排除直接依赖项，并且已经公开了包含此新功能的[问题](https://issues.apache.org/jira/browse/MNG-6222)。在本章中，**我们将讨论一种解决方法，通过用虚拟对象覆盖它来排除Maven插件的直接依赖**。

假设我们必须排除Maven Surefire插件的JUnit 4.7依赖项。

首先，我们必须创建一个虚拟模块，它必须是我们项目根POM的一部分。该模块将仅包含一个如下所示的POM文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.maven.surefire</groupId>
    <artifactId>surefire-junit47</artifactId>
    <version>dummy</version>
</project>
```

接下来，我们需要在我们希望停用依赖项的地方调整我们的子POM。为此，我们必须将虚拟版本的依赖项添加到Maven Surefire插件声明中：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>${surefire-version}</version>
            <configuration>
                <runOrder>alphabetical</runOrder>
                <threadCount>1</threadCount>
                <properties>
                    <property>
                        <name>junit</name>
                        <value>false</value>
                    </property>
                </properties>
            </configuration>
            <dependencies>
                <dependency>
                    <!-- Deactivate JUnit 4.7 engine by overriding it with an empty dummy -->
                    <groupId>org.apache.maven.surefire</groupId>
                    <artifactId>surefire-junit47</artifactId>
                    <version>dummy</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

最后，一旦我们构建了我们的项目，我们将看到Maven Surefire插件的JUnit 4.7依赖项没有包含在项目中。

## 5. 总结

在本快速教程中，我们解释了依赖排除以及如何使用<exclusions\>元素排除传递依赖。此外，我们公开了一种解决方法，通过用虚拟对象覆盖插件来排除插件中的直接依赖项。 

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/maven.modules)上获得。
---
layout: post
title:  查找包含给定类的所有Jar
category: java
copyright: java
excerpt: Java Jar
---

## 1. 概述

在本文中，我们将学习查找包含特定类的所有jar。我们将使用两种不同的方法来演示这一点，即基于命令和基于程序。

## 2. 基于命令

在这种方法中，我们将使用shell命令来识别本地Maven仓库中具有ObjectMapper类的所有jar。让我们首先编写一个脚本来识别jar中的类，**该脚本使用[jar](https://docs.oracle.com/en/java/javase/11/tools/jar.html)和[grep](https://man7.org/linux/man-pages/man1/grep.1.html)命令来打印相应的jar**：

```shell
jar -tf $1 | grep $2 && echo "Found in : $1"
```

这里$1是jar文件路径，$2是类名。对于这种情况，类名将始终为com.fasterxml.jackson.databind.ObjectMapper。让我们将上述命令保存在bash文件findJar.sh中。之后，我们将在本地Maven仓库上运行以下[find](https://www.baeldung.com/linux/find-command)命令，使用findJar.sh来获取生成的jars：

```text
$ find ~/.m2/repository -type f -name '*.jar' -exec ./findJar.sh {} com.fasterxml.jackson.databind.ObjectMapper \;
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$1.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$2.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$3.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$DefaultTypeResolverBuilder.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$DefaultTyping.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper.class
Found in : <strong>/home/user/.m2/repository/com/spotify/docker-client/8.16.0/docker-client-8.16.0-shaded.jar</strong>
com/fasterxml/jackson/databind/ObjectMapper$1.class
com/fasterxml/jackson/databind/ObjectMapper$2.class
com/fasterxml/jackson/databind/ObjectMapper$3.class
com/fasterxml/jackson/databind/ObjectMapper$DefaultTypeResolverBuilder.class
com/fasterxml/jackson/databind/ObjectMapper$DefaultTyping.class
com/fasterxml/jackson/databind/ObjectMapper.class
Found in : <strong>/home/user/.m2/repository/com/fasterxml/jackson/core/jackson-databind/2.12.3/jackson-databind-2.12.3.jar</strong>
```

## 3. 基于程序

在基于程序的方法中，**我们将编写一个Java类来在Java类路径中查找ObjectMapper类**。我们可以显示jar，如下所示的程序：

```java
public class App {
    public static void main(String[] args) {
        Class klass = ObjectMapper.class;
        URL path = klass.getProtectionDomain().getCodeSource().getLocation();
        System.out.println(path);
    }
}
```

输出：

```text
file:/Users/home/.m2/repository/com/fasterxml/jackson/core/jackson-databind/2.12.3/jackson-databind-2.12.3.jar
```

在这里我们看到每个Class类都有getProtectionDomain().getCodeSource().getLocation()，此方法提供所需类所在的jar文件。因此，我们可以使用它来获取包含该类的jar文件。

## 4. 总结

在本文中，我们介绍了从jars列表中查找类的基于命令和程序的方法。

首先，我们从一个说明性的例子开始。之后，我们探索了一种基于命令的方法来从本地Maven仓库中识别给定的类。然后，在第二种方法中，我们学会了编写一个程序，从类路径中查找运行时使用的jar来实例化类。

这两种方法都很有效，但它们都有自己的用例。
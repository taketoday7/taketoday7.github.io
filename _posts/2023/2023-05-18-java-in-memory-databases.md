---
layout: post
title:  内存数据库列表
category: springdata
copyright: springdata
excerpt: Spring Data
---

## 1. 概述

内存数据库依赖于系统内存，而不是磁盘空间来存储数据。因为内存访问比磁盘访问快，因此这些数据库自然也更快。

当然，我们只能在不需要持久化数据或者为了更快地执行测试的应用和场景中使用内存数据库。它们通常作为嵌入式数据库运行，这意味着它们在进程启动时创建并在进程结束时丢弃，这对于测试来说非常方便，因为你不需要设置外部数据库。

在以下部分中，**我们将了解一些最常用于Java环境的内存数据库以及它们各自所需的配置**。

## 2. H2数据库

H2是一个用Java编写的开源数据库，支持嵌入式和独立数据库的标准SQL。它非常快并且包含在一个只有大约1.5MB的JAR包中。

### 2.1 Maven依赖

要在应用程序中使用H2数据库，我们需要添加以下依赖项：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
```

最新版本的[H2数据库](https://central.sonatype.com/artifact/com.h2database/h2/2.1.212)可以从Maven Central下载。

### 2.2 配置

要连接到H2内存数据库，我们可以使用带有协议mem的连接字符串，后跟数据库名称。driverClassName、URL、username和password属性可以放在一个.properties文件中，以供我们的应用程序读取：

```properties
driverClassName=org.h2.Driver
url=jdbc:h2:mem:myDb;DB_CLOSE_DELAY=-1
username=sa
password=sa
```

这些属性确保myDb数据库在应用程序启动时自动创建。

默认情况下，当与数据库的连接关闭时，数据库也会关闭。如果我们希望数据库在JVM运行时一直存在，我们可以指定属性DB_CLOSE_DELAY=-1。

如果我们在Hibernate中使用数据库，我们还需要指定Hibernate方言：

```properties
hibernate.dialect=org.hibernate.dialect.H2Dialect
```

H2数据库定期维护并在[h2database.com](http://www.h2database.com/html/main.html)上提供更详细的文档。

## 3. HSQLDB(HyperSQL数据库)

HSQLDB是一个开源项目，也是用Java编写的，代表关系型数据库。它遵循SQL和JDBC标准，并支持存储过程和触发器等SQL功能。

它可以在内存模式下使用，也可以配置为使用磁盘存储。

### 3.1 Maven依赖

要使用HSQLDB开发应用程序，我们需要以下Maven依赖项：

```xml
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.3.4</version>
</dependency>
```

你可以在Maven Central上找到最新版本的[HSQLDB](https://central.sonatype.com/artifact/org.hsqldb/hsqldb/2.7.1)。

### 3.2 配置

我们需要的连接属性具有以下格式：

```properties
driverClassName=org.hsqldb.jdbc.JDBCDriver
url=jdbc:hsqldb:mem:myDb
username=sa
password=sa
```

这确保数据库将在启动时自动创建，在应用程序期间驻留在内存中，并在进程结束时被删除。

HSQLDB的Hibernate方言属性是：

```properties
hibernate.dialect=org.hibernate.dialect.HSQLDialect
```

JAR文件还包含带有GUI的数据库管理器。可以在[hsqldb.org](http://hsqldb.org/)网站上找到更多信息。

## 4. Apache Derby数据库

Apache Derby是另一个开源项目，包含由Apache软件基金会创建的关系型数据库管理系统。

Derby基于SQL和JDBC标准，主要用作嵌入式数据库，但也可以使用Derby Network Server框架以客户端-服务器模式运行。

### 4.1 Maven依赖

要在应用程序中使用Derby数据库，我们需要添加以下Maven依赖项：

```xml
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derby</artifactId>
    <version>10.13.1.1</version>
</dependency>
```

可以在Maven Central上找到最新版本的[Derby数据库](https://central.sonatype.com/artifact/org.apache.derby/derby/10.15.2.0)。

### 4.2 配置

连接字符串使用memory协议：

```properties
driverClassName=org.apache.derby.jdbc.EmbeddedDriver
url=jdbc:derby:memory:myDb;create=true
username=sa
password=sa
```

对于要在启动时自动创建的数据库，我们必须在连接字符串中指定create=true。默认情况下，数据库在JVM退出时关闭并删除。

如果将数据库与Hibernate一起使用，我们需要定义方言：

```properties
hibernate.dialect=org.hibernate.dialect.DerbyDialect
```

你可以在[db.apache.org/derby](https://db.apache.org/derby/)阅读更多关于Derby数据库的信息。

## 5. SQLite数据库

SQLite是一个只在嵌入式模式下运行的SQL数据库，要么在内存中，要么保存为文件。它是用C语言编写的，但也可以与Java一起使用。

### 5.1 Maven依赖

要使用SQLite数据库，我们需要添加JDBC驱动程序JAR：

```xml
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.16.1</version>
</dependency>
```

[sqlite-jdbc](https://central.sonatype.com/artifact/org.xerial/sqlite-jdbc/3.41.0.0)依赖项可以从Maven Central下载。

### 5.2 配置

连接属性使用org.sqlite.JDBC驱动程序类和连接字符串的memory协议：

```properties
driverClassName=org.sqlite.JDBC
url=jdbc:sqlite:memory:myDb
username=sa
password=sa
```

如果myDb数据库不存在，这将自动创建它。

目前，Hibernate没有为SQLite提供方言，尽管将来很可能会提供。如果你想将SQLite与Hibernate一起使用，你必须创建你自己的HibernateDialect类。

要了解有关SQLite的更多信息，请访问[sqlite.org](https://www.sqlite.org/index.html)。

## 6. Spring Boot中的内存数据库

Spring Boot使内存数据库的使用变得特别容易-因为它可以自动为H2、HSQLDB和Derby创建配置。

要在Spring Boot中使用三种类型之一的数据库，我们需要做的就是将其依赖项添加到pom.xml中。当框架扫描到类路径中有关的依赖项时，它会自动配置数据库。

## 7. 总结

在本文中，我们快速了解了Java生态系统中最常用的内存数据库及其基本配置。尽管它们对测试很有用，但请记住，在许多情况下，它们不提供与原始数据库完全相同的功能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-data-modules)上获得。
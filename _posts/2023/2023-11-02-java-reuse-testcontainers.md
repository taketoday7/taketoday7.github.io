---
layout: post
title:  如何在Java中重用测试容器
category: springboot
copyright: springboot
excerpt: Spring Boot TestContainers
---

## 1. 概述

在本教程中，我们将学习在设置本地开发和测试环境时如何重用[Testcontainers](https://www.baeldung.com/docker-test-containers)。

首先，我们必须确保在应用程序停止或测试套件完成时不会关闭容器。之后，我们将讨论特定于Testcontainer的配置，并将讨论使用Testcontainers Desktop的好处。最后，我们需要记住，[重用Testcontainers](https://java.testcontainers.org/features/reuse/)是一项实验性功能，尚未准备好在CI管道中使用。

## 2. 确保Testcontainer没有停止

为我们的单元测试启用Testcontainers的一个简单方法是通过@Testcontainers和@Container注解来利用其专用的[JUnit 5扩展](https://www.baeldung.com/junit-5-extensions)。

让我们编写一个测试来启动Spring Boot应用程序并允许它连接到在[Docker](https://www.baeldung.com/ops/docker-guide)容器中运行的MongoDB数据库：

```java
@Testcontainers
@SpringBootTest
class ReusableContainersLiveTest {

    @Container
    static MongoDBContainer mongoDBContainer = new MongoDBContainer(DockerImageName.parse("mongo:4.0.10"));

    // dynamic properties and test cases
}
```

**然而，自动启动MongoDBContainer的Testcontainer的JUnit 5扩展也会在测试后将其关闭。因此，让我们删除@Testcontainers和@Container注解，并手动启动容器**：

```java
@SpringBootTest
class ReusableContainersLiveTest {
    static MongoDBContainer mongoDBContainer = new MongoDBContainer(DockerImageName.parse("mongo:4.0.10"));

    @BeforeAll
    static void beforeAll() {
        mongoDBContainer.start();
    }

    // dynamic properties and test cases
}
```

另一方面，我们可能会在本地开发期间使用[Spring Boot对Testcontainers的内置支持](https://www.baeldung.com/spring-boot-built-in-testcontainers)。在这种情况下，我们不会使用JUnit 5扩展，并且此步骤是不必要的。

## 3. 管理Testcontainer生命周期

现在，我们可以完全控制容器的生命周期。我们可以将应用程序配置为重用现有的Testcontainer，并且可以从终端手动停止它。

### 3.1 withReuse()方法

**我们可以使用流式API的withReuse()方法将Testcontainer标记为可重用**：

```java
static MongoDBContainer mongoDBContainer = new MongoDBContainer(DockerImageName.parse("mongo:4.0.10"))
    .withReuse(true);
```

当我们第一次运行测试时，我们将看到有关启动MongoDBContainer的常见Testcontainers日志，这通常需要几秒钟：

```shell
23:56:42.383 [main] INFO tc.mongo:4.0.10 - Creating container for image: mongo:4.0.10
23:56:42.892 [main] INFO tc.mongo:4.0.10 - Container mongo:4.0.10 is starting: d5fa298bf6...
23:56:45.470 [main] INFO tc.mongo:4.0.10 - Container mongo:4.0.10 started in PT3.11239S
```

测试完成后，我们应该能够看到容器仍在运行。例如，我们可以使用docker ps命令从终端进行检查：

[![img](https://www.baeldung.com/wp-content/uploads/2023/10/reusing_testcontainers-300x136.png)](https://www.baeldung.com/wp-content/uploads/2023/10/reusing_testcontainers.png)

此外，当我们重新运行测试时，只要配置不更改，容器就会被重用。因此，容器设置时间显著减少：

```shell
00:12:23.859 [main] INFO tc.mongo:4.0.10 - Creating container for image: mongo:4.0.10
00:12:24.190 [main] INFO tc.mongo:4.0.10 - Reusing container with ID: d5fa298b... and hash: 0702144b...
00:12:24.191 [main] INFO tc.mongo:4.0.10 - Reusing existing container (d5fa298b...) and not creating a new one
00:12:24.398 [main] INFO tc.mongo:4.0.10 - Container mongo:4.0.10 started in PT0.5555088S
```

最后，重用的数据库包含先前插入的文档。虽然这可能对本地开发有用，但对测试可能有害。如果我们需要重新开始，我们可以在每次测试之前简单地清除集合。

### 3.2 测试容器配置

**在某些情况下，可能会出现警告，指出“Reuse was requested but the environment doesn’t support the reuse of containers”**。当我们的本地Testcontainers配置中禁用重用时，就会发生这种情况：

```shell
00:23:09.461 [main] INFO tc.mongo:4.0.10 - Creating container for image: mongo:4.0.10
00:23:09.463 [main] WARN tc.mongo:4.0.10 - Reuse was requested but the environment does not support the reuse of containers
To enable reuse of containers, you must set 'testcontainers.reuse.enable=true' in a file located at C:\Users\Emanuel Trandafir\.testcontainers.properties
00:23:09.544 [main] INFO tc.mongo:4.0.10 - Container mongo:4.0.10 is starting: 903dd52d7...
```

要修复它，我们可以简单地编辑.testcontainers.properties文件并将reuse设置为enabled：

[![img](https://www.baeldung.com/wp-content/uploads/2023/10/enable_tc_reuse_from_properties-300x110.png)](https://www.baeldung.com/wp-content/uploads/2023/10/enable_tc_reuse_from_properties.png)

### 3.3 停止容器

我们可以随时从终端手动停止Docker容器。为此，我们只需运行[docker stop](https://www.baeldung.com/ops/docker-stop-vs-kill)命令，然后运行容器ID。应用程序的未来执行会启动一个新的Docker容器。

## 4. Testcontainers Desktop

**我们可以安装[Testcontainer Desktop](https://testcontainers.com/desktop/)应用程序来轻松管理测试容器的生命周期和配置**。

该应用程序需要身份验证，但我们可以使用GitHub帐户轻松登录。登录后，我们将在工具栏中看到“Testcontainers”图标。如果我们点击它，我们将有几个选项可供选择：

[![img](https://www.baeldung.com/wp-content/uploads/2023/10/testcontainers_desktop-1-249x300.png)](https://www.baeldung.com/wp-content/uploads/2023/10/testcontainers_desktop-1.png)

现在，执行前面演示的步骤就像单击按钮一样轻松。例如，我们可以通过Preferences > Enable reusable containers轻松启用或禁用可重用容器。此外，如果需要更多调试，我们可以在关闭之前终止容器或冻结它们。

## 5. 总结

在本文中，我们学习了如何在Java中重用Testcontainers。我们发现JUnit 5可能会在执行完成之前尝试关闭容器，我们通过手动启动容器而不是依赖Testcontainers的JUnit 5扩展来避免这种情况。

之后，我们讨论了withReuse()方法和其他特定于Testcontainer的配置。最后，我们安装了Testcontainers桌面应用程序，并且我们看到了在管理Testcontainers生命周期时它如何成为双重检查配置的宝贵资产。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-3-testcontainers)上获得。
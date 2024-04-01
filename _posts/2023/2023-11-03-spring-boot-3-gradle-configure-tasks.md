---
layout: post
title:  在Spring Boot 3中配置Gradle任务
category: springboot
copyright: springboot
excerpt: Spring Boot 3 Gradle
---

## 1. 简介

Spring Boot Gradle插件在Gradle中提供Spring Boot支持，它允许我们打包可执行的JAR或WAR文件，运行Spring Boot应用程序，并使用spring-boot-dependencies提供的依赖管理。Spring Boot 3 Gradle插件需要Gradle 7.x(7.5或更高版本)或8.x，并且可以与Gradle的配置缓存一起使用。

在本教程中，我们将学习Spring Boot 3 Gradle插件任务配置。Spring Boot 3 Gradle插件中有多个可用的Gradle任务，我们将使用一个简单的[Spring Boot](https://www.baeldung.com/spring-boot-start)应用程序来演示配置一些任务。我们不会出于演示目的向Spring Boot应用程序添加任何安全或数据功能。言归正传，现在让我们更详细地讨论定义和配置任务。

## 2. 配置bootJar Gradle任务

在Spring Boot 3 Gradle插件中，Gradle任务相对于之前的版本进行了改进。一些常见的Gradle任务包括bootJar、bootWar、bootRun和bootBuildImage。让我们深入了解bootJar并了解如何配置bootJar任务。

**要配置bootJar任务，我们必须将bootJar配置块添加到build.gradle文件中**：

```groovy
tasks.named("bootJar") {
    launchScript{
        enabled = true
    }
    enabled = true
    archiveFileName = "tuyucheng.${archiveExtension.get()}"
}
```

该配置块为bootJar任务设置了几个选项。

属性launchScript生成打包在生成的JAR中的启动脚本，这允许像任何其他命令一样运行JAR。例如，无需显式使用java -jar <jarname\>我们就可以使用jarname或./jarname来运行JAR。要禁用bootjar任务，我们将属性enabled设置为false，默认情况下它设置为true。

我们可以**使用archiveFileName属性定义输出JAR名称**。现在，我们已准备好运行bootJar任务：

```shell
gradlew bootJar
```

这会在build/libs文件夹中生成一个完全可执行的JAR。在我们的例子中，JAR名称为tuyucheng.jar。

## 3. 分层JAR生成

Spring Boot Gradle插件提供对构建分层JAR的支持，这有助于减少内存使用并促进关注点分离。

让我们将bootJar任务配置为使用分层架构，我们将JAR分为两层：application层和springBoot层：

```groovy
bootJar {
    layered {
        enabled = true
        application {
            layer = 'application'
            dependencies {
                // Add any dependencies that should be included in the application layer
            }
        }
        springBoot {
            layer = 'spring-boot'
        }
    }
}
```

**本例中启用了分层功能，并定义了两层：application和spring-boot**。application层包含应用程序代码和任何指定的依赖项，而spring-boot层包含Spring Boot框架及其依赖项。

接下来，让我们使用bootJar任务构建Spring Boot应用程序：

```shell
./gradlew bootJar
```

这将在build/libs目录中创建一个名为{projectName}-{projectVersion}-layers.jar的分层JAR文件。

当我们在分层架构中将应用程序代码与Spring Boot框架代码分离时，我们可以获得更快的启动时间和更低的内存使用量。此外，正如在我们的分层JAR文件中一样，我们为应用程序和框架提供了单独的层。因此，我们可以跨多个应用程序共享框架层，这样可以减少代码重复和资源。

## 4. 配置bootBuildImage任务

现在让我们使用任务bootBuildImage来构建我们的Docker镜像，**新插件使用Cloud Native Buildpacks(CNB)创建OCI镜像**。

**任务bootBuildImage需要访问Docker Daemon**。默认情况下，它将通过本地连接与Docker守护进程进行通信。这适用于所有支持的平台上的[Docker引擎](https://docs.docker.com/install/)，无需任何特定配置。我们可以使用DOCKER_HOST、DOCKER_TLS_VERIFY、DOCKER_CERT_PATH等环境变量更改默认值。此外，我们可以选择使用插件配置不同的属性。

让我们在build.gradle中添加一个具有自定义配置的典型bootBuildImage任务：

```groovy
tasks.named("bootBuildImage") {
    imageName = 'tuyucheng:latest'
}
```

接下来，让我们运行bootBuildImage命令：

```shell
gradlew.bat bootBuildImage
```

让我们确保我们的Docker服务在我们的操作系统上启动并运行，Docker适用于所有主要操作系统，无论是Windows、Linux还是macOS。运行bootBuildImage任务的结果是，我们在Docker环境中获得了一个镜像。让我们列出本地环境中可用的Docker镜像来验证我们新构建的镜像：

```shell
docker images
```

现在，准备运行我们的容器：

```shell
docker run -p 8080:8080 tuyucheng:latest
```

-p 8080:8080将我们的主机端口8080映射到容器端口8080。默认情况下，Spring Boot在容器内的端口8080上运行我们的应用程序，容器将其公开以供外部映射。bootBuildImage任务中还有其他几个可用的配置选项，我们可以将它们用于不同的功能。

现在让我们在浏览器中导航到http://localhost:8080/hello来验证输出。

## 5. 总结

在本文中，我们介绍了一些Spring Boot 3 Gradle插件任务，这些任务比以前的版本有许多改进。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-gradle-2)上获得。
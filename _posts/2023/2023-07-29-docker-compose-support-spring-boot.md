---
layout: post
title:  Spring Boot 3中的Docker Compose支持
category: springboot
copyright: springboot
excerpt: Spring Boot 3
---

## 1. 概述

[Spring Boot 3](https://www.baeldung.com/spring-boot-3-spring-6-new)具有一些新功能，例如将我们的应用程序构建为[GraalVM本机镜像](https://www.baeldung.com/spring-native-intro)或Java 17基线。然而，另一个相关的支持是[Docker Compose](https://www.baeldung.com/ops/docker-compose)。

在本教程中，我们将介绍如何将Docker Compose工作流与Spring Boot 3集成。

## 2. Spring Boot 3 Docker Compose支持提供什么？

通常，我们会基于docker-compose.yml运行docker-compose up来启动容器，并运行docker-compose down来停止容器。现在，我们可以将这些Docker Compose命令委托给Spring Boot 3。当Spring Boot应用程序启动或停止时，它还将管理我们的容器。

**此外，它还内置了对多种服务的管理，例如SQL数据库、MongoDB、Cassandra等。因此，我们可能不需要在应用程序资源文件中复制配置类或属性**。

最后，我们将看到我们可以将此支持与自定义Docker镜像和Docker Compose配置文件一起使用。

## 3. 设置

我们需要Docker Compose和Spring Boot 3来探索这一新支持。

### 3.1 Docker Compose

[Docker Compose](https://docs.docker.com/compose/install/)需要已安装[Docker](https://docs.docker.com/engine/install/)引擎，它们很容易安装，但根据操作系统的不同可能会有所不同。

Docker在我们的主机中作为服务运行。通过Docker镜像，我们可以将容器作为系统中的轻量级进程运行。我们可以将一个镜像视为最小Linux内核之上的多个镜像层。

### 3.2 Spring Boot 3

有多种方法可以设置Spring Boot 3项目。例如，我们可以使用3.1.0版本的[Spring initializer](https://start.spring.io/)。但是，我们始终需要Spring Boot 3 Starter库来用于项目中包含的依赖项。

首先，我们添加一个[parent](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-parent) POM：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <relativePath />
</parent>
```

我们希望为我们的应用程序使用REST端点，因此我们需要[Web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)依赖项；

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

我们将连接到一个示例数据库。有多种开箱即用的支持，我们将使用[MongoDB](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

为了检查我们的应用程序是否已启动并运行，我们将使用[Spring Boot Actuator](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator)进行检查：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

最后，我们将添加[Docker Compose](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-docker-compose)依赖项。如果我们想使用其他项目功能但排除Docker Compose支持，我们可以将[optional](https://www.baeldung.com/maven-optional-dependency)标签设置为true：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-docker-compose</artifactId>
    <version>3.1.1</version>
</dependency>
```

如果我们使用Gradle，我们可能会考虑使用[Spring Boot Gradle插件](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/)来进行类似BOM的依赖管理。

## 4. 包含Docker Compose的Spring Boot 3应用程序启动

我们将使用MongoDB数据库创建一个Spring Boot 3应用程序，**一旦我们在启动时有了spring-boot-docker-compose依赖项，我们的应用程序就会启动docker-compose.yml文件中的所有服务**。

### 4.1 Docker Compose文件

首先，我们创建一个docker-compose.yml文件：

```yaml
version: '3.8'
services:
    db:
        image: mongo:latest
        ports:
            - '27017:27017'
        volumes:
            - db:/data/db
volumes:
    db:
        driver:
            local
```

### 4.2 Spring Profile

我们需要告诉Spring Boot 3 Docker Compose文件的名称及其路径，我们可以将其添加到application-{profile}属性或YAML文件中。我们将使用docker-compose Spring profile。因此，我们将创建一个application-docker-compose.yml配置文件：

```yaml
spring:
    docker:
        compose:
            enabled: true
            file: docker-compose.yml
```

### 4.3 数据库配置

我们不需要数据库配置，Docker Compose支持将创建一个默认的。但是，我们仍然可以使用profile添加[MongoDB配置](https://www.baeldung.com/spring-data-mongodb-tutorial)，例如：

```java
@Profile("!docker-compose")
```

这样，我们就可以选择是否使用Docker Compose支持。如果我们不使用profile，应用程序将期望数据库已经在运行。

### 4.4 模型

然后，我们为常规项目创建一个简单的Document类：

```java
@Document("item")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Item {

    @Id
    private String id;
    private String name;
    private int quantity;
    private String category;
}
```

### 4.5 控制器

最后，让我们定义一个具有一些CRUD操作的控制器：

```java
@RestController
@RequestMapping("/item")
@RequiredArgsConstructor
public class ItemController {
    // ...
    @PostMapping(consumes = APPLICATION_JSON_VALUE)
    public ResponseEntity<Item> save(final @RequestBody Item item) {
        return ResponseEntity.ok(itemRepository.save(item));
    }
    // other endpoints
}
```

## 5. 应用测试

我们可以通过从最喜欢的IDE或命令行启动主Spring Boot 3类来启动应用程序。

### 5.1 应用程序启动

让我们记住指定Spring profile。例如，从命令行，我们可以使用[Spring Boot Maven插件](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)：

```shell
mvn spring-boot:run -Pdocker-compose -Dspring-boot.run.profiles=docker-compose
```

我们还添加了专用的Maven构建profile(-Pdocker-compose)，以防其他profile存在。

现在，如果我们执行docker ps，我们将看到MongoDb容器正在运行：

```shell
CONTAINER ID   IMAGE             COMMAND                  CREATED        STATUS            PORTS                                           NAMES
77a9667b291a   mongo:latest      "docker-entrypoint.s…"   21 hours ago   Up 10 minutes     0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   classes-db-1
```

我们现在可以对我们的应用程序进行一些实时测试。

### 5.2 应用程序检查

我们可以使用Actuator端点检查我们的应用程序是否已启动并正在运行：

```shell
curl --location 'http://localhost:8080/actuator/health'
```

如果一切正常，我们应该得到200状态：

```json
{
    "status": "UP"
}
```

对于数据库检查，让我们在端点http://localhost:8080/item处使用POST调用添加一些元素。例如，让我们看一个curl Post请求：

```shell
curl --location 'http://localhost:8080/item' \
--header 'Content-Type: application/json' \
--data '{
    "name" : "Tennis Ball",
    "quantity" : 5,
    "category" : "sport"
}'
```

我们将收到包含生成的元素ID的响应：

```json
{
    "id": "64b117b6a805f7296d8412d9",
    "name": "Tennis Ball",
    "quantity": 5,
    "category": "sport"
}
```

### 5.3 应用程序关闭

最后，关闭Spring Boot 3应用程序也会停止我们的容器。我们可以通过执行docker ps-a看到：

```shell
CONTAINER ID   IMAGE             COMMAND                  CREATED        STATUS                     PORTS     NAMES
77a9667b291a   mongo:latest      "docker-entrypoint.s…"   22 hours ago   Exited (0) 5 seconds ago             classes-db-1
```

## 6. Docker Compose支持功能

让我们快速说明最相关的Docker Compose支持[功能](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#features.docker-compose)。

### 6.1 服务连接

此支持将在启动时自动发现多种服务，我们已经看到了MongoDB。但是，还有其他一些，例如[Redis](https://www.baeldung.com/spring-data-redis-tutorial)或[ElasticSearch](https://www.baeldung.com/spring-data-elasticsearch-tutorial)。服务连接将查找并使用本地映射的端口，我们可以跳过配置类或属性，这是由Spring Boot使用[ConnectionDetails](https://spring.io/blog/2023/06/19/spring-boot-31-connectiondetails-abstraction)抽象来完成的。

### 6.2 自定义镜像

我们可以通过应用label来使用自定义Docker镜像：

```yaml
version: '3.8'
services:
    db:
        image: our-custom-mongo-image
        ports:
            - '27017:27017'
        volumes:
            - db:/data/db
        labels:
            org.springframework.boot.service-connection: mongo
volumes:
    db:
        driver:
            local
```

### 6.3 等待容器准备就绪

有趣的是，Spring Boot 3将自动检查容器的准备情况。容器可能需要一些时间才能完全准备好，因此，这个功能让我们可以使用[healthcheck](https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck)命令来查看容器是否准备就绪。

### 6.4 激活Docker Compose Profile

我们可以在运行时在不同的Docker Compose [profile](https://docs.docker.com/compose/profiles/)之间切换。我们的服务定义可能很复杂，因此我们可能想要选择启用哪个profile，例如，如果我们处于调试或生产环境中，我们可以通过使用配置属性来实现这一点：

```properties
spring.docker.compose.profiles.active=myprofile
```

## 7. Docker Compose支持的好处

在生产环境中，我们的Docker服务可能分布在不同的实例上。因此，在这种情况下，我们可能不需要这种支持。但是，我们可以激活从docker-compose.yml定义加载的Spring profile，仅用于本地开发。

**这种支持与我们的IDE很好地集成，我们不会在命令行上来回跳转来启动和停止Docker服务**。

从3.1版本开始支持。总体而言，已经有了一些很好的功能，例如多服务连接、服务准备就绪的默认检查以及使用Docker Compose profile的可能性。

## 8. 总结

在本文中，我们介绍了Spring Boot 3.1.0中新的Docker Compose支持，我们了解了如何使用它设置和创建Spring Boot 3应用程序。

继Spring Boot易于开发的特点之后，这种支持也很方便，并且已经具备了很好的特性。在启动和停止应用程序时，Spring Boot 3管理Docker服务的生命周期。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-3)上找到。
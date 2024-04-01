---
layout: post
title:  不使用Spring Boot的Spring Boot Actuator
category: spring
copyright: spring
excerpt: Spring Actuator
---

## 1. 概述

[Spring Boot](https://spring.io/projects/spring-boot)项目提供了有助于创建基于Spring的独立应用程序并支持云原生开发的功能。因此，它是对[Spring Framework的扩展](https://www.baeldung.com/spring-vs-spring-boot)，非常有用。

有时，我们不想使用Spring Boot，例如在将Spring Framework集成到Jakarta EE应用程序中时，但是，我们仍然希望从指标和健康检查等生产就绪功能中受益，即所谓的“可观测性”。(我们可以在文章“[Spring Boot 3中的可观测性](https://www.baeldung.com/spring-boot-3-observability)”中找到详细信息。)

可观测性功能由[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)提供，它是Spring Boot的一个子项目。在本文中，我们将了解如何将Actuator集成到不使用Spring Boot的应用程序中。

## 2. 项目配置

排除Spring Boot时，我们需要处理应用程序打包和服务器运行时配置，并且需要自行[外部化配置](https://docs.spring.io/spring-boot/docs/3.0.6/reference/html/features.html#features.external-config)。Spring Boot提供的那些功能对于在我们基于Spring的应用程序中使用Actuator不是必需的。而且，虽然我们确实需要项目依赖项，但我们不能使用Spring Boot的[Starter依赖项](https://www.baeldung.com/spring-boot-starters)(在本例中为spring-boot-starter-actuator)。除此之外，我们还需要将必要的bean添加到应用程序上下文中。

我们可以手动或使用自动配置来完成此操作。由于Actuator的配置非常复杂并且没有详细记录，因此我们应该更喜欢自动配置。这是我们需要Spring Boot的一部分，因此我们不能完全排除Spring Boot。

### 2.1 添加项目依赖

要集成Actuator，我们需要[spring-boot-actuator-autoconfigure](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-actuator-autoconfigure/3.0.6)依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator-autoconfigure</artifactId>
    <version>3.0.6</version>
</dependency>
```

这还将包括spring-boot-actuator、spring-boot和spring-boot-autoconfigure作为传递依赖项。

### 2.2 启用自动配置

然后，我们启用自动配置。这可以通过向应用程序的配置添加@EnableAutoConfiguration轻松完成：

```java
@EnableAutoConfiguration
// ... @ComponentScan, @Import or any other application configuration
public class AppConfig {
    // ...
}
```

我们应该意识到这可能会影响整个应用程序，因为如果类路径中有更多自动配置类，这也会自动配置框架的其他部分。

### 2.3 启用端点

默认情况下，仅启用健康端点。Actuator的自动配置类使用配置属性。例如，WebEndpointAutoConfiguration使用映射到具有“management.endpoints.web”前缀的属性的WebEndpointProperties。要启用所有端点，我们需要

```properties
management.endpoints.web.exposure.include=*
```

这些属性必须对上下文可用-例如，通过将它们放入application.properties文件并使用@PropertySource标注我们的配置类：

```java
@EnableAutoConfiguration
@PropertySource("classpath:application.properties")
// ... @ComponentScan, @Import or any other application configuration
public class AppConfig {
}
```

### 2.4 测试项目配置

现在，我们已准备好调用Actuator端点。我们可以使用此属性启用健康详细信息：

```properties
management.endpoint.health.show-details=always
```

我们可以[实现自定义健康端点](https://www.baeldung.com/spring-boot-health-indicators)：

```java
@Configuration
public class ActuatorConfiguration {

    @Bean
    public HealthIndicator sampleHealthIndicator() {
        return Health.up()
                .withDetail("info", "Sample Health")
                ::build;
    }
}
```

然后，调用“{url_to_project}/actuator/health”会生成如下输出：

![](/assets/images/2023/spring/springbootactuatorwithoutspringboot01.png)

## 3.总结

在本教程中，我们了解了如何在非启动应用程序中集成Spring Boot Actuator。像往常一样，所有代码实现都可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-modules/spring-actuator)上找到。
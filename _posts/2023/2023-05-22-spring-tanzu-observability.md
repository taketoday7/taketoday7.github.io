---
layout: post
title:  Spring的可观测性
category: springboot
copyright: springboot
excerpt: Spring Boot WaveFront
---

## 1. 概述

在文本中，我们将介绍如何使用Wavefront的Tanzu Observability构建一个Spring Boot项目并将其配置为将指标发送到免费增值集群。

## 2. Maven依赖

首先，让我们将wavefront-spring-boot-starter添加到项目POM中：

```xml
<dependency>
    <groupId>com.wavefront</groupId>
    <artifactId>wavefront-spring-boot-starter</artifactId>
    <scope>runtime</scope>
    <version>3.0.1</version>
</dependency>
```

最新版本的依赖项可以在[Maven Central](https://central.sonatype.com/artifact/com.wavefront/wavefront-spring-boot-starter/3.0.1)上找到。

## 3. 开箱即用的可观测性

按照以下步骤启动项目并通过Wavefront自动将多个自动配置的指标发送到Tanzu Observability。

1.  在启动服务之前，配置项目以便可以识别应用程序和服务发送的指标。打开application.properties文件并添加以下内容：

    ```properties
    management.wavefront.application.name=spring-boot-wavefront
    management.wavefront.application.service-name=getting-started
    ```

> 上述属性将集成配置为使用spring-boot-wavefront应用程序和getting-started服务将指标发送到Wavefront的Tanzu Observability，一个应用程序可以包含任意数量的微服务。

2.  通过调用WaveFrontDemoApplication的main方法从IDE运行应用程序，你会看到以下内容：

    ```shell
    2023-05-22T17:06:00.966+08:00  INFO 28752 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
    2023-05-22T17:06:01.014+08:00  INFO 28752 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
    2023-05-22T17:06:01.026+08:00  INFO 28752 --- [           main] c.t.taketoday.WaveFrontDemoApplication   : Started WaveFrontDemoApplication in 4.169 seconds (process running for 4.963)
    
    A Wavefront account has been provisioned successfully and the API token has been saved to disk.
    
    To share this account, make sure the following is added to your configuration:
    
    	management.wavefront.api-token=236ca873-9346-44c7-976c-38e622c0a53f
    	management.wavefront.uri=https://wavefront.surf
    
    Connect to your Wavefront dashboard using this one-time use link:
    https://wavefront.surf/us/H5cdWLmmVh
    ```

这会自动完成以下操作：

-   在没有任何额外信息的情况下，系统会自动为你提供一个免费增值集群上的帐户。
-   为你创建API令牌。
-   为了让你访问免费增值集群上的仪表板，应用程序启动时会记录一个一次性使用链接。链接以https://wavefront.surf开头，将此链接复制到你最喜欢的浏览器中并探索开箱即用的Spring Boot仪表板：

![](/assets/images/2023/springboot/springtanzuobservability01.png)

> 显示数据需要稍等一会，当你可以在Wavefront的Tanzu Observability中看到数据时，请确保过滤器中的应用程序和服务名称与你在application.properties文件中配置的名称相匹配。
> ![](/assets/images/2023/springboot/springtanzuobservability02.png)

## 4. 创建一个简单的控制器

接下来，你可以创建一个简单的控制器来查看如何自动检测HTTP流量：

```java
@RestController
public class DemoController {

    @GetMapping("/")
    public String home() {
        return "Hello World";
    }
}
```

1.  重新启动应用程序并从浏览器多次访问[http://localhost:8080](http://localhost:8080)。

2.  你会注意到仪表板上有一个额外的HTTP部分。此功能称为条件仪表板，可让你根据过滤器显示部分。
    ![](/assets/images/2023/springboot/springtanzuobservability03.png)

3.  访问[http://localhost:8080/does-not-exist](http://localhost:8080/does-not-exist)以触发客户端404错误(可选)。

## 5. 总结

在本文中，我们创建了一个Web应用程序，用于将指标发送到Wavefront的Tanzu Observability。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-wavefront)上获得。
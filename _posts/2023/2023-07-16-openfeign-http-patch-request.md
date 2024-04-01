---
layout: post
title:  使用OpenFeign设置HTTP PATCH请求
category: springcloud
copyright: springcloud
excerpt: OpenFeign
---

## 1. 概述

通过REST API更新对象时，最好使用[PATCH](https://www.baeldung.com/http-put-patch-difference-spring)方法，这允许我们使用我们希望更改的字段执行部分更新。当现有资源需要完全更改时，我们还可以使用PUT方法。

在本教程中，我们将学习如何在[OpenFeign](https://www.baeldung.com/spring-cloud-openfeign)中设置HTTP PATCH方法。在[Feign](https://www.baeldung.com/intro-to-feign)客户端中测试PATCH方法时，我们还会看到意外错误。

最后，我们将介绍根本原因并解决问题。

## 2. Spring Boot中的示例应用程序

假设我们需要构建一个简单的微服务来调用下游服务来进行部分更新。

### 2.1 Maven依赖项

首先，我们将添加[spring-boot-starter-web](https://central.sonatype.com/search?q=spring-boot-starter-web)和[spring-cloud-starter-openfeign](https://central.sonatype.com/search?q=spring-boot-starter-openfeign)依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 2.2 实现Feign客户端

现在，让我们使用[Spring Web注解](https://www.baeldung.com/spring-mvc-annotations)在Feign中实现PATCH方法。

首先，让我们用一些属性来建模User类：

```java
public class User {
    private String userId;
    private String userName;
    private String email;
}
```

[使用OpenFeign设置HTTP PATCH请求](https://tuyucheng7.github.io/springcloud/2023/07/16/.html)

```java
@FeignClient(name = "user-client", url = "http://localhost:8082/api/user")
public interface UserClient {
    @RequestMapping(value = "{userId}", method = RequestMethod.PATCH)
    User updateUser(@PathVariable(value = "userId") String userId, @RequestBody User user);
}
```

在上面的PATCH方法中，我们传递User对象，其中只有要更新的必填字段以及userId字段。它比发送整个资源表示更方便，节省了一些网络带宽，并避免了跨不同字段对同一对象进行多个更新时的争用。

相反，如果我们使用PUT请求，则必须传递完整的资源表示来替换现有资源。

## 3. 对Feign客户端进行测试

现在让我们通过模拟HTTP调用来实现UserClient的测试用例。

### 3.1 设置WireMock服务器

为了进行实验，我们需要使用Mock框架来模拟我们正在调用的服务。

首先，让我们添加[WireMockServer](https://central.sonatype.com/search?q=wiremock-jre8) Maven依赖项：

```xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.35.0</version>
    <scope>test</scope>
</dependency>
```

然后，让我们配置并启动WireMockServer：

```java
WireMockServer wireMockServer = new WireMockServer(8082);
configureFor("localhost", 8082);
wireMockServer.start();
```

WireMockServer在Feign客户端配置使用的同一主机和端口上启动。

### 3.2 存根PATCH API

我们将模拟PATCH方法来测试更新用户API：

```java
String updatedUserResponse = """
      {
          "userId": 100001,
          "userName": "name",
          "email": "updated-email@mail.in"
      }""";
stubFor(patch(urlEqualTo("/api/user/".concat(USER_ID)))
    .willReturn(aResponse().withStatus(HttpStatus.OK.value())
    .withHeader("Content-Type", "application/json")
    .withBody(updatedUserResponse)));
```

### 3.3 测试PATCH请求

为了进行测试，我们将User对象传递给UserClient，其中包含更新所需的字段。

现在，让我们完成测试并验证更新功能：

```java
User user = new User();
user.setUserId("100001");
user.setEmail("updated-email@mail.in");
User updatedUser = userClient.updateUser("100001", user);

assertEquals(user.getUserId(), updatedUser.getUserId());
assertEquals(user.getEmail(), updatedUser.getEmail());
```

预计上述测试应该通过。但是，我们会从Feign客户端收到意外错误：

```text
feign.RetryableException: Invalid HTTP method: PATCH executing PATCH http://localhost:8082/api/user/100001
        at feign.FeignException.errorExecuting(FeignException.java:268)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:131)
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:91)
	at feign.ReflectiveFeign$FeignInvocationHandler.invoke(ReflectiveFeign.java:100)
	at jdk.proxy2/jdk.proxy2.$Proxy80.updateUser(Unknown Source)
	at cn.tuyucheng.taketoday.cloud.openfeign.patcherror.client.UserClientUnitTest.givenUserExistsAndIsValid_whenUpdateUserCalled_thenReturnSuccess(UserClientUnitTest.java:64)
	...
```

接下来，让我们详细调查该错误。

### 3.4 无效HTTP方法错误的原因

**上述错误信息表明请求的HTTP方法无效**。不过，根据[HTTP标准](https://datatracker.ietf.org/doc/html/rfc5789)，PATCH方法是有效的。

我们可以从错误消息中看到它是由ProtocolException类引起并从HttpURLConnection类传播的：

```text
Caused by: java.net.ProtocolException: Invalid HTTP method: PATCH
	at java.base/java.net.HttpURLConnection.setRequestMethod(HttpURLConnection.java:489)
	at java.base/sun.net.www.protocol.http.HttpURLConnection.setRequestMethod(HttpURLConnection.java:598)
	at feign.Client$Default.convertAndSend(Client.java:170)
	at feign.Client$Default.execute(Client.java:104)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:119)
```

事实证明，默认的HTTP客户端使用HttpURLConnection类来建立HTTP连接。HttpURLConnection有一个setRequestMethod方法来设置请求方法。

不幸的是，**HttpURLConnection类无法将PATCH方法识别为有效类型**。

## 4. 修复PATCH方法错误

为了修复该错误，我们将添加受支持的HTTP客户端依赖项。此外，我们将通过添加配置来覆盖默认的HTTP客户端。

### 4.1 添加OkHttpClient依赖项

让我们包含[feign-okhttp](https://central.sonatype.com/search?q=feign-okhttp)依赖项：

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

我们应该注意，任何其他受支持的HTTP客户端(例如Apache HttpClient)也可以工作。

### 4.2 启用OkHttpClient

OkHttpClient类将PATCH方法视为有效类型，并且不会引发任何异常。

让我们使用以下配置启用OkHttpClient类：

```properties
feign.okhttp.enabled=true
```

最后，我们将重新运行测试并验证PATCH方法是否有效。现在，我们没有从Feign客户端收到任何错误：

```text
UserClientUnitTest.givenUserExistsAndIsValid_whenUpdateUserCalled_thenReturnSuccess: 1 total, 1 passed
```

## 5. 总结

在本文中，我们学习了如何在OpenFeign中使用PATCH方法，我们在测试时还发现了意外错误并了解了根本原因。

我们还使用OkHttpClient实现解决了该问题。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-cloud-modules/spring-cloud-openfeign-2)上找到。
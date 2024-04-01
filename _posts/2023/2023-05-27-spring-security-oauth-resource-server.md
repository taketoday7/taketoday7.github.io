---
layout: post
title:  使用Spring Security 5的OAuth 2.0资源服务器
category: springsecurity
copyright: springsecurity
excerpt: Spring Security OAuth 2.0
---

## 1. 概述

在本教程中，我们将学习如何**使用Spring Security 5设置OAuth 2.0资源服务器**。

我们将使用JWT以及不透明令牌(Spring Security支持的两种不记名令牌)来执行此操作。

在进入实现和代码示例之前，我们将首先建立一些背景知识。

## 2. 一点背景

### 2.1 什么是JWT和不透明令牌？

JWT或[JSON Web Token](https://tools.ietf.org/html/rfc7519)是一种以广泛接受的JSON格式安全传输敏感信息的方法。包含的信息可能是关于用户的，也可以是关于令牌本身的信息，例如令牌的到期时间和颁发者。

另一方面，不透明令牌，顾名思义，就其携带的信息而言是不透明的。令牌只是指向存储在授权服务器中的信息的标识符；它通过服务器端的自省得到验证。

### 2.2 什么是资源服务器？

在OAuth 2.0的上下文中，**资源服务器是一种通过OAuth令牌保护资源的应用程序**。这些令牌由授权服务器颁发，通常颁发给客户端应用程序。资源服务器的工作是在向客户端提供资源之前验证令牌。

令牌的有效性由以下几个因素决定：

-   此令牌是否来自配置的授权服务器？
-   是否没有过期？
-   这个资源服务器是它的目标受众吗？
-   令牌是否具有访问所请求资源所需的权限？

为了形象化这一点，让我们看一下[授权代码流](https://tools.ietf.org/html/rfc6749#section-1.3.1)的序列图，并查看所有的参与者：

![](/assets/images/2023/springsecurity/springsecurityoauthresourceserver01.png)

正如我们在第8步中看到的，当客户端应用程序调用资源服务器的API来访问受保护的资源时，它首先会去授权服务器验证请求的Authorization:Bearer标头中包含的令牌，然后响应客户端。

**第9步是我们将在本教程中重点关注的内容**。

那么现在让我们进入代码部分。我们将使用Keycloak设置一个授权服务器，一个验证JWT令牌的资源服务器，另一个验证不透明令牌的资源服务器，以及几个JUnit测试来模拟客户端应用程序并验证响应。

## 3. 授权服务器

首先，我们将设置一个授权服务器，即颁发令牌的东西。

**为此，我们将使用[嵌入在Spring Boot应用程序中的Keycloak](https://www.baeldung.com/keycloak-embedded-in-spring-boot-app)**。Keycloak是一种开源身份和访问管理解决方案。由于我们在本教程中重点是资源服务器，因此我们不会对其进行更深入的探讨。

我们的嵌入式Keycloak服务器定义了两个客户端，fooClient和barClient，对应于我们的两个资源服务器应用程序。

## 4. 资源服务器-使用JWT

我们的资源服务器将有四个主要组件：

-   模型：要保护的资源
-   API：一个用于公开资源的REST控制器
-   安全配置：一个类，用于定义API公开的受保护资源的访问控制
-   application.yml：一个声明属性的配置文件，包括关于授权服务器的信息

### 4.1 Maven依赖项

主要是，我们需要[spring-boot-starter-oauth2-resource-server](https://search.maven.org/search?q=spring-boot-starter-oauth2-resource-server)，Spring Boot资源服务器支持的Starter。这个Starter默认包含Spring Security，所以我们不需要显式添加它：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    <version>2.7.5</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

除此之外，我们还将添加Web支持。

出于演示目的，我们将在Apache的[commons-lang3](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)库的帮助下随机生成资源，而不是从数据库中获取资源。

### 4.2 模型

为简单起见，我们将使用Foo(一个POJO)作为我们受保护的资源：

```java
public class Foo {
    private long id;
    private String name;

    // constructor, getters and setters
}
```

### 4.3 API

这是我们的Rest控制器，使Foo可用于操作：

```java
@RestController
@RequestMapping(value = "/foos")
public class FooController {

    @GetMapping(value = "/{id}")
    public Foo findOne(@PathVariable Long id) {
        return new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));
    }

    @GetMapping
    public List findAll() {
        List fooList = new ArrayList();
        fooList.add(new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4)));
        fooList.add(new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4)));
        fooList.add(new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4)));
        return fooList;
    }

    @ResponseStatus(HttpStatus.CREATED)
    @PostMapping
    public void create(@RequestBody Foo newFoo) {
        logger.info("Foo created");
    }
}
```

很明显，我们有GET所有FOO、根据id GET一个FOO和POST一个FOO三个端点。

### 4.4 安全配置

在这个配置类中，我们将为我们的资源定义访问级别：

```java
@Configuration
public class JWTSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests(authz -> authz.antMatchers(HttpMethod.GET, "/foos/**")
                        .hasAuthority("SCOPE_read")
                        .antMatchers(HttpMethod.POST, "/foos")
                        .hasAuthority("SCOPE_write")
                        .anyRequest()
                        .authenticated())
                .oauth2ResourceServer(oauth2 -> oauth2.jwt());
        return http.build();
    }
}
```

任何拥有具有read作用域的访问令牌的人都可以获取Foos。为了POST一个新的Foo，他们的令牌应该有一个write作用域。

此外，**我们将使用oauth2ResourceServer() [DSL](https://docs.spring.io/spring-integration/docs/5.1.0.M1/reference/html/java-dsl.html)添加对jwt()的调用，以在此处指示我们的服务器支持的令牌类型**。

### 4.5 application.yml

在应用程序属性中，除了通常的端口号和上下文路径外，**我们还需要定义授权服务器颁发者URI的路径，以便资源服务器可以发现其[提供程序配置](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)**：

```yaml
server:
    port: 8081
    servlet:
        context-path: /resource-server-jwt

spring:
    security:
        oauth2:
            resourceserver:
                jwt:
                    issuer-uri: http://localhost:8083/auth/realms/tuyucheng
```

资源服务器使用此信息来验证来自客户端应用程序的JWT令牌，如序列图的步骤9所示。

要使用issuer-uri属性进行此验证，授权服务器必须已启动并正在运行。否则，资源服务器不会启动。

如果我们需要独立启动它，那么我们可以提供jwk-set-uri属性来指向授权服务器的公开公钥的端点：

```yaml
jwk-set-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/certs
```

这就是让我们的服务器验证JWT令牌所需的全部内容。

### 4.6 测试

为了进行测试，我们将设置一个JUnit。为了执行此测试，我们需要启动并运行授权服务器和资源服务器。

让我们验证是否可以在我们的测试中使用read作用域的令牌从resource-server-jwt获取Foos：

```java
@Test
public void givenUserWithReadScope_whenGetFooResource_thenSuccess() {
    String accessToken = obtainAccessToken("read");

    Response response = RestAssured.given()
        .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
        .get("http://localhost:8081/resource-server-jwt/foos");
    assertThat(response.as(List.class)).hasSizeGreaterThan(0);
}
```

在上面的代码中，在第3行，我们从授权服务器获取具有read作用域的访问令牌，涵盖了序列图中的步骤1到7。

步骤8由RestAssured的get()调用执行。步骤9由资源服务器使用我们看到的配置执行，并且对我们作为用户是透明的。

## 5. 资源服务器-使用不透明令牌

接下来，让我们看看我们的资源服务器处理不透明令牌的相同组件。

### 5.1 Maven依赖项

为了支持不透明令牌，我们需要额外的[oauth2-oidc-sdk](https://search.maven.org/search?q=oauth2-oidc-sdk)依赖项：

```xml
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>oauth2-oidc-sdk</artifactId>
    <version>8.19</version>
    <scope>runtime</scope>
</dependency>
```

### 5.2 模型和控制器

对于这一个，我们将添加一个Bar资源：

```java
public class Bar {
    private long id;
    private String name;

    // constructor, getters and setters
}
```

我们还将有一个BarController，其端点类似于我们之前的FooController。

### 5.3 application.yml

在这里的application.yml中，我们需要添加一个与授权服务器的内省端点相对应的introspection-uri。如前所述，这是验证不透明令牌的方式：


```yaml
server:
    port: 8082
    servlet:
        context-path: /resource-server-opaque

spring:
    security:
        oauth2:
            resourceserver:
                opaque:
                    introspection-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/token/introspect
                    introspection-client-id: barClient
                    introspection-client-secret: barClientSecret
```

### 5.4 安全配置

为Bar资源保持类似于Foo的访问级别，**此配置类也使用oauth2ResourceServer() DSL调用opaqueToken()以指示不透明令牌类型的使用**：

```java
@Configuration
public class OpaqueSecurityConfig {

    @Value("${spring.security.oauth2.resourceserver.opaque.introspection-uri}")
    String introspectionUri;

    @Value("${spring.security.oauth2.resourceserver.opaque.introspection-client-id}")
    String clientId;

    @Value("${spring.security.oauth2.resourceserver.opaque.introspection-client-secret}")
    String clientSecret;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests(authz -> authz.antMatchers(HttpMethod.GET, "/bars/**")
                        .hasAuthority("SCOPE_read")
                        .antMatchers(HttpMethod.POST, "/bars")
                        .hasAuthority("SCOPE_write")
                        .anyRequest()
                        .authenticated())
                .oauth2ResourceServer(oauth2 -> oauth2.opaqueToken
                        (token -> token.introspectionUri(this.introspectionUri)
                                .introspectionClientCredentials(this.clientId, this.clientSecret)));
        return http.build();
    }
}
```

在这里，我们还将指定与我们将使用的授权服务器客户端相对应的客户端凭据。我们之前在application.yml中定义了这些。

### 5.5 测试

我们将为不透明的基于令牌的资源服务器设置一个JUnit，类似于我们为JWT所做的。

在这种情况下，我们将检查write作用域的访问令牌是否可以将Bar POST到resource-server-opaque：

```java
@Test
public void givenUserWithWriteScope_whenPostNewBarResource_thenCreated() {
    String accessToken = obtainAccessToken("read write");
    Bar newBar = new Bar(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));

    Response response = RestAssured.given()
        .contentType(ContentType.JSON)
        .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
        .body(newBar)
        .log()
        .all()
        .post("http://localhost:8082/resource-server-opaque/bars");
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED.value());
}
```

如果我们得到CREATED状态，则意味着资源服务器成功验证了不透明令牌并为我们创建了Bar。

## 6. 总结

在本文中，我们学习了如何配置基于Spring Security的资源服务器应用程序来验证JWT以及不透明令牌。

正如我们所看到的，通过最少的设置，Spring可以无缝地验证颁发者的令牌并将资源发送给请求方(在我们的例子中是JUnit测试)。

与往常一样，源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules/spring-security-oauth/oauth-resource-server)上找到。
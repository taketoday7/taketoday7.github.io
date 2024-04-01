---
layout: post
title:  使用Spring Security OAuth2进行简单的单点登录
category: springsecurity
copyright: springsecurity
excerpt: Spring Security SSO
---

## 1. 概述

在本教程中，我们将讨论如何**使用Spring Security OAuth和Spring Boot以及使用Keycloak作为授权服务器来实现[SSO(单点登录)](https://www.baeldung.com/cs/sso-guide)**。

我们将使用4个独立的应用程序：

-   授权服务器：作为中央身份验证机制
-   资源服务器：Foo的提供者
-   两个客户端应用程序：使用SSO的应用程序

简而言之，当用户试图通过一个客户端应用程序访问资源时，他们将被重定向到首先通过授权服务器进行身份验证。Keycloak将让用户登录，并且在登录第一个应用的同时，如果使用相同的浏览器访问第二个客户端应用，则用户无需再次输入其凭据。

我们将使用OAuth2中的Authorization Code授权类型来驱动身份验证委托。

**我们将在Spring Security 5中使用OAuth堆栈**。如果你想使用Spring Security OAuth遗留堆栈，请查看之前的文章：[使用Spring Security OAuth2(遗留堆栈)进行简单单点登录](https://www.baeldung.com/sso-spring-security-oauth2-legacy)

根据[迁移指南](https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide#changes-in-approach-1)：

>  Spring Security将此功能称为OAuth 2.0登录，而Spring Security OAuth将其称为SSO

## 延伸阅读：

## [Spring Security 5 - OAuth2登录](https://www.baeldung.com/spring-security-5-oauth2-login)

了解如何在Spring Security 5中使用OAuth2使用Facebook、Google或其他凭据对用户进行身份验证。

[阅读更多](https://www.baeldung.com/spring-security-5-oauth2-login)→

## [Spring Security OAuth2中的新功能 - 验证Claim](https://www.baeldung.com/spring-security-oauth-2-verify-claims)

Spring Security OAuth中新Claim验证支持的快速实用介绍。

[阅读更多](https://www.baeldung.com/spring-security-oauth-2-verify-claims)→

## [使用Spring Social的二级Facebook登录](https://www.baeldung.com/facebook-authentication-with-spring-security-and-social)

快速了解在标准表单登录Spring应用程序旁边实现Facebook驱动的身份验证。

[阅读更多](https://www.baeldung.com/facebook-authentication-with-spring-security-and-social)→

## 2. 授权服务器

以前，Spring Security OAuth堆栈提供了将授权服务器设置为Spring应用程序的可能性。

然而，OAuth堆栈已被Spring弃用，现在我们将使用Keycloak作为我们的授权服务器。

**所以这一次，我们将授权服务器设置为Spring Boot应用程序中的[嵌入式Keycloak服务器](https://www.baeldung.com/keycloak-embedded-in-a-spring-boot-application)**。

在我们的[预配置](https://www.baeldung.com/keycloak-embedded-in-spring-boot-app#keycloak-config)中，**我们将定义两个客户端，ssoClient-1和ssoClient-2**，每个客户端应用程序一个。

## 3. 资源服务器

接下来，我们需要一个资源服务器或REST API，它将为我们提供我们的客户端应用程序将使用的Foos。

它基本上与我们[之前用于Angular客户端应用程序](https://www.baeldung.com/rest-api-spring-oauth2-angular#resource-server)的功能相同。

## 4. 客户端应用程序

现在让我们看看我们的Thymeleaf客户端应用程序；当然，我们将使用Spring Boot来最小化配置。

请记住，**我们需要其中的两个来演示单点登录功能**。

### 4.1 Maven依赖项

首先，我们需要在pom.xml中添加以下依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor.netty</groupId>
    <artifactId>reactor-netty</artifactId>
</dependency>
```

**为了包括我们需要的所有客户端支持，包括安全性，我们只需要添加spring-boot-starter-oauth2-client**。此外，由于旧的RestTemplate将被弃用，我们将使用WebClient，这就是我们添加spring-webflux和reactor-netty的原因。

### 4.2 安全配置

接下来，最重要的部分，我们第一个客户端应用程序的安全配置：

```java
@EnableWebSecurity
public class UiSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/login**")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .oauth2Login();
        return http.build();
    }

    @Bean
    WebClient webClient(ClientRegistrationRepository clientRegistrationRepository,
                        OAuth2AuthorizedClientRepository authorizedClientRepository) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2 =
                new ServletOAuth2AuthorizedClientExchangeFilterFunction(clientRegistrationRepository, authorizedClientRepository);
        oauth2.setDefaultOAuth2AuthorizedClient(true);
        return WebClient.builder()
                .apply(oauth2.oauth2Configuration())
                .build();
    }
}
```

**此配置的核心部分是oauth2Login()方法，用于启用Spring Security的OAuth 2.0登录支持**。由于我们使用的是Keycloak，它默认是Web应用程序和RESTful Web服务的单点登录解决方案，因此我们不需要为SSO添加任何进一步的配置。

最后，我们还定义了一个WebClient bean作为一个简单的HTTP客户端来处理发送到我们的资源服务器的请求。

这是application.yml：

```yaml
spring:
    security:
        oauth2:
            client:
                registration:
                    custom:
                        client-id: ssoClient-1
                        client-secret: ssoClientSecret-1
                        scope: read,write
                        authorization-grant-type: authorization_code
                        redirect-uri: http://localhost:8082/ui-one/login/oauth2/code/custom
                provider:
                    custom:
                        authorization-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/auth
                        token-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/token
                        user-info-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/userinfo
                        user-name-attribute: preferred_username
    thymeleaf:
        cache: false

server:
    port: 8082
    servlet:
        context-path: /ui-one

resourceserver:
    api:
        project:
            url: http://localhost:8081/sso-resource-server/api/foos/
```

在这里，spring.security.oauth2.client.registration是注册客户端的根命名空间。我们定义了一个注册ID为custom的客户端。然后我们定义了它的client-id，client-secret，scope，authorization-grant-type和redirect-uri，当然，这应该和我们的授权服务器定义的一样。

之后，我们定义了我们的服务提供者或授权服务器，再次使用相同的custom id，并列出了其不同的URI供Spring Security使用。这就是我们需要定义的全部内容，**框架会为我们无缝地完成整个登录过程，包括重定向到Keycloak**。

另请注意，在我们的示例中，我们推出了授权服务器，但当然我们也可以使用其他第三方提供商，例如[Facebook或GitHub](https://www.baeldung.com/spring-security-5-oauth2-login)。

### 4.3 控制器

现在让我们在客户端应用程序中实现我们的控制器，以从我们的资源服务器请求Foos：

```java
@Controller
public class FooClientController {

    @Value("${resourceserver.api.url}")
    private String fooApiUrl;

    @Autowired
    private WebClient webClient;

    @GetMapping("/foos")
    public String getFoos(Model model) {
        List<FooModel> foos = this.webClient.get()
                .uri(fooApiUrl)
                .retrieve()
                .bodyToMono(new ParameterizedTypeReference<List<FooModel>>() {})
                .block();
        model.addAttribute("foos", foos);
        return "foos";
    }
}
```

正如我们所看到的，我们这里只有一种方法可以将资源分发给foos模板。**我们不必添加任何登录代码**。

### 4.4 前端

现在，让我们看一下客户端应用程序的前端配置。我们不打算在这里重点讨论，主要是因为我们[已经在网站上进行了介绍](https://www.baeldung.com/spring-thymeleaf-3)。

我们这里的客户端应用程序有一个非常简单的前端；这是index.html：

```html
<a class="navbar-brand" th:href="@{/foos/}">Spring OAuth Client Thymeleaf - 1</a>
<label>Welcome !</label> <br /> <a th:href="@{/foos/}">Login</a>
```

还有foos.html：

```html
<a class="navbar-brand" th:href="@{/foos/}">Spring OAuth Client Thymeleaf -1</a>
Hi, <span sec:authentication="name">preferred_username</span>

<h1>All Foos:</h1>
<table>
    <thead>
    <tr>
        <td>ID</td>
        <td>Name</td>
    </tr>
    </thead>
    <tbody>
    <tr th:if="${foos.empty}">
        <td colspan="4">No foos</td>
    </tr>
    <tr th:each="foo : ${foos}">
        <td><span th:text="${foo.id}"> ID </span></td>
        <td><span th:text="${foo.name}"> Name </span></td>
    </tr>
    </tbody>
</table>
```

foos.html页面需要对用户进行身份验证。**如果未经身份验证的用户试图访问foos.html，他们将首先被重定向到Keycloak的登录页面**。

### 4.5 第二个客户端应用程序

我们将使用另一个client_id ssoClient-2配置第二个应用程序Spring OAuth Client Thymeleaf - 2。

它与我们刚刚描述的第一个应用程序基本相同。

**只有application.yml将有所不同，在其spring.security.oauth2.client.registration中包含不同的client_id、client_secret和redirect_uri**：

```yaml
spring:
    security:
        oauth2:
            client:
                registration:
                    custom:
                        client-id: ssoClient-2
                        client-secret: ssoClientSecret-2
                        scope: read,write
                        authorization-grant-type: authorization_code
                        redirect-uri: http://localhost:8084/ui-two/login/oauth2/code/custom
```

而且，当然，我们还需要有一个不同的服务器端口，以便我们可以并行运行它们：

```yaml
server:
    port: 8084
    servlet:
        context-path: /ui-two
```

最后，我们将调整前端HTML的标题为Spring OAuth Client Thymeleaf – 2而不是Spring OAuth Client Thymeleaf - 1，以便我们可以区分两者。

## 5. 测试SSO行为

要测试SSO行为，让我们运行我们的应用程序。

为此，我们需要启动并运行所有4个启动应用程序-授权服务器、资源服务器和两个客户端应用程序。

现在让我们打开一个浏览器，比如Chrome，并使用凭据john@test.com/123登录到[Client-1](http://localhost:8082/ui-one)。接下来，在另一个窗口或选项卡中，点击[Client-2](http://localhost:8084/ui-two)的URL。单击登录按钮后，我们将立即被重定向到Foos页面，绕过身份验证步骤。

类似地，如果用户首先登录到Client-2，则他们无需输入Client-1的用户名/密码。

## 6. 总结

在本教程中，我们重点介绍了使用Spring Security OAuth2和使用Keycloak作为身份提供者的Spring Boot实现单点登录。

与往常一样，可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules/spring-security-oauth/oauth-sso)上找到完整的源代码。
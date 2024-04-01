---
layout: post
title:  Spring Security OAuth授权服务器
category: springsecurity
copyright: springsecurity
excerpt: Spring Security OAuth
---

## 1. 概述

[OAuth](https://oauth.net/)是一个描述授权过程的开放标准。它可用于授权用户访问API。例如，REST API可以将访问权限限制为只有具有适当角色的注册用户。

OAuth授权服务器负责对用户进行身份验证并颁发包含用户数据和适当访问策略的访问令牌。

在本教程中，我们将使用[Spring Security OAuth授权服务器](https://github.com/spring-projects/spring-authorization-server#spring-authorization-server)项目实现一个简单的OAuth应用程序。

在此过程中，我们将创建一个客户端-服务器应用程序，该应用程序将从REST API获取Taketoday文章列表。客户端服务和服务器服务都需要OAuth身份验证。

## 2. 授权服务器实现

我们将从查看OAuth授权服务器配置开始。它将作为文章资源和客户端服务器的身份验证源。

### 2.1 依赖项

首先，我们需要在pom.xml文件中添加一些依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>0.2.0</version>
</dependency>
```

### 2.2 配置

现在我们将通过在application.yml文件中设置server.port属性来配置我们的授权服务器将运行的端口：

```yaml
server:
    port: 9000
```

然后我们可以转到Spring bean配置。首先，我们需要一个@Configuration类，在其中创建一些特定于OAuth的bean。第一个将是客户服务的存储库。在我们的示例中，我们将有一个客户端，使用RegisteredClient构建器类创建：

```java
@Configuration
@Import(OAuth2AuthorizationServerConfiguration.class)
public class AuthorizationServerConfig {
    
    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
                .clientId("articles-client")
                .clientSecret("{noop}secret")
                .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
                .redirectUri("http://127.0.0.1:8080/login/oauth2/code/articles-client-oidc")
                .redirectUri("http://127.0.0.1:8080/authorized")
                .scope(OidcScopes.OPENID)
                .scope("articles.read")
                .build();
        return new InMemoryRegisteredClientRepository(registeredClient);
    }
}
```

上面配置的属性包括：

-   客户端ID：Spring将使用它来识别哪个客户端正在尝试访问资源
-   客户端密钥：客户端和服务器已知的密钥，提供两者之间的信任
-   身份验证方法：在我们的例子中，我们将使用基本身份验证，它只是一个用户名和密码
-   授权授予类型：我们希望允许客户端同时生成授权码和刷新令牌
-   重定向URI：客户端将在基于重定向的流程中使用它
-   作用域：此参数定义客户端可能拥有的权限。在我们的例子中，我们将拥有所需的OidcScopes.OPENID和我们自定义的articles.read

接下来，我们将配置一个bean来应用默认的OAuth安全性并生成一个默认的表单登录页面：

```java
@Bean
@Order(Ordered.HIGHEST_PRECEDENCE)
public SecurityFilterChain authServerSecurityFilterChain(HttpSecurity http) throws Exception {
    OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
    return http.formLogin(Customizer.withDefaults()).build();
}
```

每个授权服务器都需要其令牌的签名密钥，以在安全域之间保持适当的边界。让我们生成一个2048字节的RSA密钥：

```java
@Bean
public JWKSource<SecurityContext> jwkSource() {
    RSAKey rsaKey = generateRsa();
    JWKSet jwkSet = new JWKSet(rsaKey);
    return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
}

private static RSAKey generateRsa() {
    KeyPair keyPair = generateRsaKey();
    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
    return new RSAKey.Builder(publicKey)
        .privateKey(privateKey)
        .keyID(UUID.randomUUID().toString())
        .build();
}

private static KeyPair generateRsaKey() {
    KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
    keyPairGenerator.initialize(2048);
    return keyPairGenerator.generateKeyPair();
}
```

除签名密钥外，每个授权服务器还需要有一个唯一的颁发者URL。我们将通过创建ProviderSettings bean将其设置为端口9000上http://auth-server的本地主机别名：

```java
@Bean
public ProviderSettings providerSettings() {
    return ProviderSettings.builder()
        .issuer("http://auth-server:9000")
        .build();
}
```

此外，我们将在/etc/hosts文件中添加一个“127.0.0.1 auth-server”条目。这允许我们在本地机器上运行客户端和授权服务器，并避免两者之间的会话cookie覆盖问题。

然后我们将使用@EnableWebSecurity标注的配置类启用Spring Web安全模块：

```java
@EnableWebSecurity
public class DefaultSecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests(authorizeRequests -> authorizeRequests.anyRequest().authenticated())
                .formLogin(withDefaults());
        return http.build();
    }

    // ...
}
```

在这里，我们调用authorizeRequests.anyRequest().authenticated()来要求对所有请求进行身份验证。我们还通过调用formLogin(defaults())方法提供基于表单的身份验证。

最后，我们将定义一组用于测试的示例用户。在本例中，我们将创建一个仅包含一个管理员用户的存储库：

```java
@Bean
UserDetailsService users() {
    UserDetails user = User.withDefaultPasswordEncoder()
        .username("admin")
        .password("password")
        .build();
    return new InMemoryUserDetailsManager(user);
}
```

## 3. 资源服务器

现在我们将创建一个资源服务器，该服务器将从GET端点返回文章列表。端点应仅允许针对我们的OAuth服务器进行身份验证的请求。

### 3.1 依赖项

首先，我们将包含所需的依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    <version>2.5.4</version>
</dependency>
```

### 3.2 配置

在开始实现代码之前，我们应该在application.yml文件中配置一些属性。第一个是服务器端口：

```yaml
server:
    port: 8090
```

接下来，是时候进行安全配置了。我们需要使用我们之前在ProviderSettings bean中配置的主机和端口为我们的身份验证服务器设置正确的URL：

```yaml
spring:
    security:
        oauth2:
            resourceserver:
                jwt:
                    issuer-uri: http://auth-server:9000
```

现在我们可以设置我们的Web安全配置。同样，我们要明确声明对文章资源的每个请求都应该经过授权并具有适当的articles.read权限：

```java
@EnableWebSecurity
public class ResourceServerConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.mvcMatcher("/articles/**")
                .authorizeRequests()
                .mvcMatchers("/articles/**")
                .access("hasAuthority('SCOPE_articles.read')")
                .and()
                .oauth2ResourceServer()
                .jwt();
        return http.build();
    }
}
```

如此处所示，我们还调用了oauth2ResourceServer()方法，该方法将根据application.yml配置来配置OAuth服务器连接。

### 3.3 文章控制器

最后，我们将创建一个REST控制器，它将在GET /articles端点下返回一个文章列表：

```java
@RestController
public class ArticlesController {

    @GetMapping("/articles")
    public String[] getArticles() {
        return new String[] { "Article 1", "Article 2", "Article 3" };
    }
}
```

## 4. API客户端

对于最后一部分，我们将创建一个REST API客户端，该客户端将从资源服务器获取文章列表。

### 4.1 依赖项

首先，我们将包含必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
    <version>5.3.9</version>
</dependency>
<dependency>
    <groupId>io.projectreactor.netty</groupId>
    <artifactId>reactor-netty</artifactId>
    <version>1.0.9</version>
</dependency>
```

### 4.2 配置

正如我们之前所做的那样，我们将定义一些用于身份验证的配置属性：

```yaml
server:
    port: 8080

spring:
    security:
        oauth2:
            client:
                registration:
                    articles-client-oidc:
                        provider: spring
                        client-id: articles-client
                        client-secret: secret
                        authorization-grant-type: authorization_code
                        redirect-uri: "http://127.0.0.1:8080/login/oauth2/code/{registrationId}"
                        scope: openid
                        client-name: articles-client-oidc
                    articles-client-authorization-code:
                        provider: spring
                        client-id: articles-client
                        client-secret: secret
                        authorization-grant-type: authorization_code
                        redirect-uri: "http://127.0.0.1:8080/authorized"
                        scope: articles.read
                        client-name: articles-client-authorization-code
                provider:
                    spring:
                        issuer-uri: http://auth-server:9000
```

现在我们将创建一个WebClient实例来对我们的资源服务器执行HTTP请求。我们将使用标准实现，只需添加一个OAuth授权过滤器：

```java
@Bean
WebClient webClient(OAuth2AuthorizedClientManager authorizedClientManager) {
    ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2Client = new ServletOAuth2AuthorizedClientExchangeFilterFunction(authorizedClientManager);
    return WebClient.builder()
        .apply(oauth2Client.oauth2Configuration())
        .build();
}
```

WebClient需要一个OAuth2AuthorizedClientManager作为依赖项。让我们创建一个默认实现：

```java
@Bean
OAuth2AuthorizedClientManager authorizedClientManager(ClientRegistrationRepository clientRegistrationRepository, OAuth2AuthorizedClientRepository authorizedClientRepository) {

    OAuth2AuthorizedClientProvider authorizedClientProvider = OAuth2AuthorizedClientProviderBuilder.builder()
        .authorizationCode()
        .refreshToken()
        .build();
    DefaultOAuth2AuthorizedClientManager authorizedClientManager = new DefaultOAuth2AuthorizedClientManager(clientRegistrationRepository, authorizedClientRepository);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

    return authorizedClientManager;
}
```

最后，我们将配置Web安全：

```java
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests(authorizeRequests -> authorizeRequests.anyRequest().authenticated())
                .oauth2Login(oauth2Login -> oauth2Login.loginPage("/oauth2/authorization/articles-client-oidc"))
                .oauth2Client(withDefaults());
        return http.build();
    }
}
```

在这里，以及在其他服务器中，我们需要对每个请求进行身份验证。此外，我们需要配置登录页面URL(在.yml配置中定义)和OAuth客户端。

### 4.3 文章客户端控制器

最后，我们可以创建数据访问控制器。我们将使用之前配置的WebClient向我们的资源服务器发送HTTP请求：

```java
@RestController
public class ArticlesController {

    private WebClient webClient;

    @GetMapping(value = "/articles")
    public String[] getArticles(@RegisteredOAuth2AuthorizedClient("articles-client-authorization-code") OAuth2AuthorizedClient authorizedClient) {
        return this.webClient
                .get()
                .uri("http://127.0.0.1:8090/articles")
                .attributes(oauth2AuthorizedClient(authorizedClient))
                .retrieve()
                .bodyToMono(String[].class)
                .block();
    }
}
```

在上面的示例中，我们以OAuth2AuthorizedClient类的形式从请求中获取OAuth授权令牌。它由Spring使用带有正确标识的@RegisterdOAuth2AuthorizedClient注解自动绑定。在我们的例子中，它是从我们之前在.yml文件中配置的article-client-authorization-code中提取的。

此授权令牌进一步传递给HTTP请求。

### 4.4 访问文章列表

现在，当我们进入浏览器并尝试访问http://127.0.0.1:8080/articles页面时，我们将自动重定向到http://auth-server:9000/login URL下的OAuth服务器登录页面:

![](/assets/images/2023/springsecurity/springsecurityoauthauthserver01.png)

提供正确的用户名和密码后，授权服务器会将我们重定向回请求的URL，即文章列表。

对文章端点的进一步请求不需要登录，因为访问令牌将存储在cookie中。

## 5. 总结

在本文中，我们学习了如何设置、配置和使用Spring Security OAuth授权服务器。

与往常一样，完整的源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules/spring-security-oauth/oauth-authorization-server)上找到。
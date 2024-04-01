---
layout: post
title:  Spring Boot应用程序中的共享密钥身份验证
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

身份验证是设计安全微服务的基本方面。我们可以通过多种方式实现身份验证，例如使用基于用户的凭据、证书或基于令牌的方式。

在本教程中，我们将学习如何为服务到服务通信设置身份验证。我们将使用Spring Security实现该解决方案。

## 2. 自定义认证介绍

使用身份提供者或密码数据库可能并不总是可行的，因为私有微服务不需要基于用户的交互。但是，我们仍然应该保护应用程序免受任何无效请求的侵害，而不是仅仅依赖网络安全。

在这种情况下，我们可以通过使用自定义共享密钥标头来设计一种简单的身份验证技术。应用程序将根据预配置的请求标头验证请求。

我们还应该在应用程序中启用TLS以保护网络上的共享密钥。

我们可能还需要确保一些端点无需任何身份验证即可工作，例如健康检查或错误端点。

## 3. 示例应用

假设我们需要构建一个包含一些REST API的微服务。

### 3.1 Maven依赖项

首先，我们将创建一个Spring Boot Web项目并包含一些Spring依赖项。

让我们添加[spring-boot-starter-web](https://central.sonatype.com/search?q=spring-boot-starter-web)、[spring-boot-starter-security](https://central.sonatype.com/search?q=spring-boot-starter-security)和[spring-boot-starter-test](https://central.sonatype.com/search?q=spring-boot-starter-test)依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 3.2 实现REST控制器

我们的应用程序有两个端点，一个端点可通过共享密钥标头访问，另一个端点可供网络中的所有人访问。

首先，让我们使用/hello端点实现APIController类：

```java
@GetMapping(path = "/api/hello")
public String hello(){
    return "hello";
}
```

然后，我们将在HealthCheckController类中实现health端点：

```java
@GetMapping(path = "/health")
public String getHealthStatus() {
   return "OK";
}
```

## 4. 使用Spring Security实现自定义认证

[Spring Security](https://www.baeldung.com/security-spring)提供了几个内置的过滤器类来实现身份验证。我们还可以覆盖内置过滤器类或使用身份验证提供程序来实现自定义解决方案。

我们将应用程序配置为将AuthenticationFilter注册到过滤器链中。

### 4.1 实现身份验证过滤器

要实现基于标头的身份验证，我们可以使用RequestHeaderAuthenticationFilter类。RequestHeaderAuthenticationFilter是一个预认证过滤器，它从请求标头中获取主体。与任何预身份验证方案一样，我们需要将身份验证证明转换为具有角色的用户。

RequestHeaderAuthenticationFilter使用请求标头设置Principal对象。在内部，它将使用请求标头中的Principal和Credential创建一个PreAuthenticatedAuthenticationToken对象，并将令牌传递给身份验证管理器。

让我们在SecurityConfig类中添加RequestHeaderAuthenticationFilter bean： 

```java
@Bean
public RequestHeaderAuthenticationFilter requestHeaderAuthenticationFilter() {
    RequestHeaderAuthenticationFilter filter = new RequestHeaderAuthenticationFilter();
    filter.setPrincipalRequestHeader("x-auth-secret-key");
    filter.setExceptionIfHeaderMissing(false);
    filter.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/api/**"));
    filter.setAuthenticationManager(authenticationManager());

    return filter;
}
```

在上面的代码中，x-auth-header-key标头被添加为Principal对象。此外，还包括AuthenticationManager对象以委托实际的身份验证。

我们应该注意，**过滤器是为与/api/**路径匹配的端点启用的**。

### 4.2 设置身份验证管理器

现在，我们将创建AuthenticationManager并传递一个自定义的AuthenticationProvider对象，稍后我们将创建该对象：

```java
@Bean
protected AuthenticationManager authenticationManager() {
    return new ProviderManager(Collections.singletonList(requestHeaderAuthenticationProvider));
}
```

### 4.3 配置身份验证提供程序

要实现自定义身份验证提供程序，我们将实现AuthenticationProvider接口。

让我们覆盖AuthenticationProvider接口中定义的authenticate方法：

```java
public class RequestHeaderAuthenticationProvider implements AuthenticationProvider {

    @Value("${api.auth.secret}")
    private String apiAuthSecret;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String authSecretKey = String.valueOf(authentication.getPrincipal());

        if(StringUtils.isBlank(authSecretKey) || !authSecretKey.equals(apiAuthSecret)) {
            throw new BadCredentialsException("Bad Request Header Credentials");
        }

        return new PreAuthenticatedAuthenticationToken(authentication.getPrincipal(), null, new ArrayList<>());
    }
}
```

在上面的代码中，authSecretKey值与Principal匹配。如果标头无效，该方法将抛出BadCredentialsException。

身份验证成功后，它将返回完全经过身份验证的PreAuthenticatedAuthenticationToken对象。PreAuthenticatedAuthenticationToken对象可以被视为基于角色授权的用户。

此外，我们需要重写AuthenticationProvider接口中定义的supports方法：

```java
@Override
public boolean supports(Class<?> authentication) {
    return authentication.equals(PreAuthenticatedAuthenticationToken.class);
}
```

supports方法检查此身份验证提供程序支持的Authentication类类型。

### 4.4 使用Spring Security配置过滤器

要在应用程序中启用Spring Security，我们将添加@EnableWebSecurity注解。此外，我们需要创建一个SecurityFilterChain对象。

此外，Spring Security默认启用[CORS](https://www.baeldung.com/spring-cors)和[CSRF](https://www.baeldung.com/spring-security-csrf)保护。由于此应用程序只能由内部微服务访问，因此我们将禁用CORS和CSRF保护。

让我们在SecurityFilterChain中包含上面的RequestHeaderAuthenticationFilter：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.cors().and()
                .csrf()
                .disable()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .addFilterAfter(requestHeaderAuthenticationFilter(), HeaderWriterFilter.class)
                .authorizeHttpRequests()
                .antMatchers("/api/**").authenticated();

        return http.build();
    }
}
```

我们应该注意到**会话管理设置为STATELESS**，因为应用程序是在内部访问的。

### 4.5 从身份验证中排除健康端点

使用antMatcher的permitAll方法，我们可以从身份验证和授权中排除任何公共端点。

让我们在上面的filterChain方法中添加/health端点以从身份验证中排除：

```java
.antMatchers("/health").permitAll()
.and()
.exceptionHandling().authenticationEntryPoint((request, response, authException) -> response.sendError(HttpServletResponse.SC_UNAUTHORIZED));
```

我们应该注意，**异常处理配置为包括用于返回401 Unauthorized状态的authenticationEntryPoint**。

## 5. 为API实施集成测试

使用[TestRestTemplate](https://www.baeldung.com/spring-boot-testresttemplate)，我们将为端点实现集成测试。

首先，让我们通过将有效的x-auth-secret-key标头传递给/hello端点来实现测试：

```java
HttpHeaders headers = new HttpHeaders();
headers.add("x-auth-secret-key", "test-secret");

ResponseEntity<String> response = restTemplate.exchange(new URI("http://localhost:8080/app/api"), HttpMethod.GET, new HttpEntity<>(headers), String.class);

assertEquals(HttpStatus.OK, response.getStatusCode());
assertEquals("hello", response.getBody());
```

然后，让我们通过传递一个无效的标头来实现测试：

```java
HttpHeaders headers = new HttpHeaders();
headers.add("x-auth-secret-key", "invalid-secret");

ResponseEntity<String> response = restTemplate.exchange(new URI("http://localhost:8080/app/api"), HttpMethod.GET, new HttpEntity<>(headers), String.class);
assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
```

最后，我们将在不添加任何标头的情况下测试/health端点：

```java
HttpHeaders headers = new HttpHeaders();
ResponseEntity<String> response = restTemplate.exchange(new URI(HEALTH_CHECK_ENDPOINT), HttpMethod.GET, new HttpEntity<>(headers), String.class);

assertEquals(HttpStatus.OK, response.getStatusCode());
assertEquals("OK", response.getBody());
```

正如预期的那样，身份验证适用于所需的端点。/health端点无需标头身份验证即可访问。

## 6. 总结

在本文中，我们了解了使用带有共享秘密身份验证的自定义标头如何帮助确保服务到服务通信的安全。

我们还了解了如何结合使用RequestHeaderAuthenticationFilter和自定义身份验证提供程序来实现基于共享密钥的标头身份验证。

与往常一样，可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules/spring-security-web-boot-5)上找到示例代码。
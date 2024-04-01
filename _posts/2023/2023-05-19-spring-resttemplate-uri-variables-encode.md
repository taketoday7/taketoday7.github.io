---
layout: post
title:  RestTemplate上URI变量的编码
category: springweb
copyright: springweb
excerpt: Spring Web
---

## 一、概述

在本教程中，我们将学习如何在 Spring 的[RestTemplate](https://www.baeldung.com/rest-template)上对 URI 变量进行编码。

我们面临的常见编码问题之一是当我们有一个包含加号 ( + )的 URI 变量时。例如，如果我们有一个值为 http://localhost:8080/api/v1/plus+sign 的URI 变量，加号将被编码为空格，这可能会导致意外的服务器响应。

让我们看看解决这个问题的几种方法。

## 2.项目设置

我们将创建一个使用RestTemplate调用 API 的小项目。

### 2.1. Spring Web 依赖

让我们从将 [Spring Web Starter](https://search.maven.org/search?q=g:org.springframework.boot a:spring-boot-starter-web)依赖项添加到我们的pom.xml开始：![img]()

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

```

或者，我们可以使用[Spring Initializr](https://start.spring.io/)生成项目并添加依赖项。

### 2.2. RestTemplate Bean

接下来，我们将创建一个RestTemplate bean：![img]()

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

```

## 3.API调用

让我们创建一个调用公共 API http://httpbin.org/get的服务类。

API 返回带有请求参数的 JSON 响应。例如，如果我们在浏览器上调用 URL https://httpbin.org/get?parameter=springboot，我们会得到这样的响应：

```json
{
  "args": {
    "parameter": "springboot"
  },
  "headers": {
  },
  "origin": "",
  "url": ""
}

```

这里， args 对象包含请求参数。为简洁起见，省略了其他值。

### 3.1. 服务等级

让我们创建一个调用 API 并返回参数键值的服务 类 ：

```java
@Service
public class HttpBinService {
    private final RestTemplate restTemplate;

    public HttpBinService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public String get(String parameter) {
        String url = "http://httpbin.org/get?parameter={parameter}";
        ResponseEntity<Map> response = restTemplate.getForEntity(url, Map.class, parameter);
        Map<String, String> args = (Map<>) response.getBody().get("args");
        return args.get("parameter");
    }
}
```

get ()方法调用指定的 URL，将响应解析为 Map ，并检索args 对象中 字段参数 的值。

### 3.2. 测试

让我们用两个参数测试我们的服务类——springboot和 spring +boot——并检查响应是否符合预期：

```java
@SpringBootTest
class HttpBinServiceTest {
    @Autowired
    private HttpBinService httpBinService;

    @Test
    void givenWithoutPlusSign_whenGet_thenSameValueReturned() throws JsonProcessingException {
        String parameterWithoutPlusSign = "springboot";
        String responseWithoutPlusSign = httpBinService.get(parameterWithoutPlusSign);
        assertEquals(parameterWithoutPlusSign, responseWithoutPlusSign);
    }

    @Test
    void givenWithPlusSign_whenGet_thenSameValueReturned() throws JsonProcessingException {
        String parameterWithPlusSign = "spring+boot";
        String responseWithPlusSign = httpBinService.get(parameterWithPlusSign);
        assertEquals(parameterWithPlusSign, responseWithPlusSign);
    }
}

```

如果我们运行测试，我们会看到第二个测试失败了。响应是 spring boot而不是spring+boot。

## 4. 将拦截器与RestTemplate一起使用

我们可以使用拦截器对 URI 变量进行编码。

让我们创建一个实现ClientHttpRequestInterceptor 接口的类：![img]()

```java
public class UriEncodingInterceptor implements ClientHttpRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        HttpRequest encodedRequest = new HttpRequestWrapper(request) {
            @Override
            public URI getURI() {
                URI uri = super.getURI();
                String escapedQuery = uri.getRawQuery().replace("+", "%2B");
                return UriComponentsBuilder.fromUri(uri)
                  .replaceQuery(escapedQuery)
                  .build(true).toUri();
            }
        };
        return execution.execute(encodedRequest, body);
    }
}
```

我们已经实现了intercept() 方法。此方法将在RestTemplate发出每个请求之前执行。 

让我们分解一下代码：

-   我们创建了一个新的 HttpRequest 对象来包装原始请求。
-   对于这个包装器，我们覆盖了 getURI() 方法来编码 URI 变量。 在这种情况下，我们将查询字符串中的加号替换为 %2B 。
-   使用 UriComponentsBuilder，我们创建一个新的 URI 并将查询字符串替换为编码的查询字符串。
-    我们从将替换原始请求的intercept()方法返回编码请求 。

### 4.1. 添加拦截器

接下来，我们需要将拦截器添加到RestTemplate bean：

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setInterceptors(Collections.singletonList(new UriEncodingInterceptor()));
        return restTemplate;
    }
}

```

如果我们再次运行测试，我们会看到它通过了。

[拦截器](https://www.baeldung.com/spring-rest-template-interceptor)提供了更改我们想要的请求的任何部分的灵活性。它们对于复杂的场景很有用，例如添加额外的标头或对请求中的字段执行更改。

对于像我们的示例这样更简单的任务，我们还可以使用DefaultUriBuilderFactory 来更改编码。接下来让我们看看如何做到这一点。

## 5. 使用DefaultUriBuilderFactory

另一种对 URI 变量进行编码的方法是更改  RestTemplate内部使用的DefaultUriBuilderFactory对象。

默认情况下，URI 构建器首先对整个 URL 进行编码，然后分别对值进行编码。我们将创建一个新的DefaultUriBuilderFactory对象并将编码模式设置为VALUES_ONLY。这将编码限制为仅值。

然后我们可以使用 setUriTemplateHandler()方法 在我们的RestTemplate bean 中设置新的DefaultUriBuilderFactory对象。

让我们用它来创建一个新的RestTemplate bean：![img]()

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        DefaultUriBuilderFactory defaultUriBuilderFactory = new DefaultUriBuilderFactory();
        defaultUriBuilderFactory.setEncodingMode(DefaultUriBuilderFactory.EncodingMode.VALUES_ONLY);
        restTemplate.setUriTemplateHandler(defaultUriBuilderFactory);
        return restTemplate;
    }
}

```

这是对 URI 变量进行编码的另一种选择。同样，如果我们运行测试，我们会看到它通过了。

## 六，结论

在本文中，我们了解了如何在RestTemplate请求中对 URI 变量进行编码。我们看到了执行此操作的两种方法 — 使用拦截器和更改DefaultUriBuilderFactory 对象。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-web-modules)上获得。
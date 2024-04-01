---
layout: post
title:  Spring Boot中的RestClient指南
category: springboot
copyright: springboot
excerpt: RestClient
---

## 1. 简介

RestClient是Spring Framework 6.1 M2中引入的同步HTTP客户端，它取代了[RestTemplate](https://www.baeldung.com/rest-template)。同步HTTP客户端以阻塞方式发送和接收HTTP请求和响应，这意味着它会等待每个请求完成，然后再继续处理下一个请求。

在本文中，我们将探讨RestClient提供的功能以及它与RestTemplate的比较。

## 2. RestClient和RestTemplate

顾名思义，RestTemplate是建立在模板设计模式之上的。它是一种行为设计模式，在方法中定义算法的骨架，允许子类为某些步骤提供特定的实现。虽然它是一个强大的模式，但它会产生重载的需求，这可能会带来不便。

**为了改进这一点，RestClient提供了流式的API**。流式的API是一种设计模式，它允许方法链接，通过顺序调用对象上的方法，使代码更具可读性和表现力，通常不需要中间变量。

让我们从创建一个基本的RestClient开始：

```java
RestClient restClient = RestClient.create();
```

## 3. 使用HTTP请求方法简单获取

与RestTemplate或任何其他REST客户端类似，**RestClient允许我们使用请求方法进行HTTP调用**。让我们逐步了解创建、检索、修改和删除资源的不同HTTP方法。

我们将对一个基本的Article类进行操作：

```java
public class Article {
    Integer id;
    String title;
    // constructor and getters
}
```

### 3.1 使用GET检索资源

**我们使用GET HTTP方法从Web服务器上的指定资源请求和检索数据，而不对其进行修改**。它主要用于Web应用程序中的只读操作。

对于初学者，让我们获取一个简单的String作为响应，而不对我们的自定义类进行任何序列化：

```java
String result = restClient.get()
    .uri(uriBase + "/articles")
    .retrieve()
    .body(String.class);
```

### 3.2 使用POST创建资源

**我们使用POST HTTP方法将数据提交到Web服务器上的资源，通常是为了在Web应用程序中创建新记录或资源**。与检索数据的GET方法不同，POST设计用于发送要由服务器处理的数据，例如在提交Web表单时。

URI应该定义我们想要处理的资源。

让我们将ID等于1的简单文章发送到我们的服务器：

```java
Article article = new Article(1, "How to use RestClient");
ResponseEntity<Void> response = restClient.post()
    .uri(uriBase + "/articles")
    .contentType(APPLICATION_JSON)
    .body(article)
    .retrieve()
    .toBodilessEntity();
```

因为我们指定了“APPLICATION_JSON”内容类型，Article类的实例将由Jackson库在后台自动序列化为JSON。在此示例中，我们使用toBodilessEntity()方法忽略响应正文。POST端点不需要而且通常也不会返回任何有效负载。

### 3.3 使用PUT更新资源

接下来，**我们将介绍用于使用提供的数据更新或替换现有资源的PUT HTTP方法**。它通常用于修改Web应用程序中的现有实体或其他资源。通常，我们需要指定更新的资源，以确保完全替换。

让我们修改我们在上一段中创建的文章。我们提供的URI应标识我们要更改的资源：

```java
Article article = new Article(1, "How to use RestClient even better");
ResponseEntity<Void> response = restClient.put()
    .uri(uriBase + "/articles/1")
    .contentType(APPLICATION_JSON)
    .body(article)
    .retrieve()
    .toBodilessEntity();
```

与上一段类似，我们依靠RestClient来序列化我们的有效负载并忽略响应。

### 3.4 使用DELETE删除资源

**我们使用DELETE HTTP方法来请求从Web服务器中删除指定的资源**。与GET端点类似，我们通常不为请求提供任何有效负载，而是依赖于URI中编码的参数：

```java
ResponseEntity<Void> response = restClient.delete()
    .uri(uriBase + "/articles/1")
    .retrieve()
    .toBodilessEntity();
```

## 4. 反序列化响应

我们经常希望序列化请求并将响应反序列化到某个我们可以有效操作的类。**RestClient具有执行JSON到对象转换的能力，这是由Jackson库提供支持的功能**。

此外，由于消息转换器的共享利用，我们可以使用RestTemplate支持的所有数据类型。

让我们通过ID检索一篇文章并将其序列化为Article类的实例：

```java
Article article = restClient.get()
    .uri(uriBase + "/articles/1")
    .retrieve()
    .body(Article.class);
```

当我们想要获取像List这样的泛型类的实例时，指定主体的类会稍微复杂一些。例如，如果我们想获取所有文章，我们将获取List<Article\>对象。在这种情况下，我们可以使用ParameterizedTypeReference抽象类来告诉RestClient我们将获得什么对象。

我们甚至不需要指定泛型类型，Java会为我们推断类型：

```java
List<Article> articles = restClient.get()
    .uri(uriBase + "/articles")
    .retrieve()
    .body(new ParameterizedTypeReference<>() {});
```

## 5. 使用Exchange解析响应

**RestClient包含exchange()方法，用于通过授予对底层HTTP请求和响应的访问权限来处理更高级的情况**。在这种情况下，库不会应用默认处理程序，我们必须自己处理状态。

假设当数据库中没有文章时，我们正在通信的服务返回204状态码。由于这种稍微不标准的行为，我们希望以特殊的方式处理它。当状态码等于204时，我们将抛出一个ArticleNotFoundException异常；当状态码不等于200时，我们将抛出一个更通用的异常：

```java
List<Article> article = restClient.get()
    .uri(uriBase + "/articles")
    .exchange((request, response) -> {
        if (response.getStatusCode().isSameCodeAs(HttpStatusCode.valueOf(204))) {
            throw new ArticleNotFoundException();
        } else if (response.getStatusCode().isSameCodeAs(HttpStatusCode.valueOf(200))) {
            return objectMapper.readValue(response.getBody(), new TypeReference<>() {});
        } else {
            throw new InvalidArticleResponseException();
        }
    });
```

因为我们在这里使用原始响应，所以我们还需要使用[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)自己反序列化响应的主体。

## 6. 错误处理

默认情况下，当RestClient在HTTP响应中遇到4xx或5xx状态码时，它会引发一个异常，该异常是RestClientException的子类。**我们可以通过实现状态处理程序来覆盖此行为**。

让我们编写一个在找不到文章时抛出自定义异常的程序：

```java
Article article = restClient.get()
    .uri(uriBase + "/articles/1234")
    .retrieve()
    .onStatus(status -> status.value() == 404, (request, response) -> {
        throw new ArticleNotFoundException(response);
    })
    .body(Article.class);
```

## 7. 从RestTemplate构建RestClient

RestClient是RestTemplate的后继者，在较旧的代码库中，我们很可能会遇到使用RestTemplate的实现。

幸运的是，使用旧RestTemplate的配置创建RestClient实例非常简单：

```java
RestTemplate oldRestTemplate;
RestClient restClient = RestClient.create(oldRestTemplate);
```

## 8. 总结

在本文中，我们研究了RestClient类，它是作为同步HTTP客户端的RestTemplate的继承者。我们学习了如何将其流式的API用于简单和复杂的用例。我们开始汇总所有HTTP方法，然后转向响应序列化和错误处理主题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-3)上获得。
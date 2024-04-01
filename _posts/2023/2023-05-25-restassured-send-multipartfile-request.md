---
layout: post
title:  使用RestAssured发送MultipartFile请求
category: test-lib
copyright: test-lib
excerpt: Rest Assured
---

## 1. 概述

在本教程中，我们将使用[RestAssured](https://www.baeldung.com/rest-assured-tutorial)库向服务器发送multipart请求。这对于在Spring中测试[multipart控制器](https://www.baeldung.com/sprint-boot-multipart-requests)或针对已部署的服务器编写集成测试非常有用。

## 2. 什么是Multipart请求？

multipart请求是一种HTTP POST请求，**它们允许在单个请求中发送各种文件或数据**。

在multipart请求中，数据被分成多个部分。每个部分都有一个名称，并以其自己的一组标头开头，标头指示其包含的数据类型。每个部分之间的数据和边界都被编码。

## 3. 设置

让我们设置我们将使用的库。

### 3.1 RestAssured测试库

**RestAssured是一个基于Java的库，它提供了一种特定于域的语言，用于为RESTful Web服务编写自动化测试**。

让我们首先将[RestAssured](https://mvnrepository.com/artifact/io.rest-assured/rest-assured)库添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.3.0</version>
    <scope>test</scope>
</dependency>
```

### 3.2 设置Wiremock服务器

我们将通过RestAssured发送multipart请求。在实际情况下，这些请求将到达我们要测试的目标服务器。为了我们的示例，我们将用mock服务器替换这个真实服务器，并为此目的使用Wiremock。

**[WireMock](https://www.baeldung.com/introduction-to-wiremock)是一个用于mock Web服务的开源库**。它允许我们创建用于测试客户端-服务器交互的存根。

让我们将[Wiremock](https://mvnrepository.com/artifact/com.github.tomakehurst/wiremock)库添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock</artifactId>
    <version>2.27.2</version>
    <scope>test</scope>
</dependency>
```

我们现在可以在[JUnit](https://www.baeldung.com/junit-5)测试中配置Wiremock服务器。我们将使服务器在每个测试方法[之前](https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall)启动并在每个测试方法之后停止：

```java
private WireMockServer wireMockServer;

@BeforeEach
void startServer() {
    wireMockServer = new WireMockServer();
    wireMockServer.start();
}

@AfterEach
void stopServer() {
    wireMockServer.stop();
}
```

## 4. 使用RestAssured发送文件

我们将快速设置我们的模mock服务器，然后开始编写我们的RestAssured测试。

### 4.1 文件处理

让我们在test/resources目录中创建一个文件tuyucheng.txt。**为了准备发送文件，让我们编写两个实用程序方法**：

-   给定文件名，getFile()检索相应的[File](https://www.baeldung.com/reading-file-in-java)对象
-   getFileContent()读取文件内容

以下是我们的方法：

```java
File getFile(String fileName) throws IOException {
    return new ClassPathResource(fileName).getFile();
}

String getFileContent(String fileName) throws IOException {
    return new String(Files.readAllBytes(Paths.get(getFile(fileName).getPath())));
}
```

### 4.2 存根服务器创建

我们会将tuyucheng.txt文件发送到URL /upload。我们将file设置为multipart请求中文件的控件名称。

首先，**我们将创建一个存根来接收这样的请求**。此外，我们将检查Content-Type标头是否为multipart/form-data。当请求满足所有这些条件时，我们将以200响应状态进行响应：

```java
stubFor(post(urlEqualTo("/upload"))
    .withHeader("Content-Type", containing("multipart/form-data"))
    .withRequestBody(containing("file"))
    .withRequestBody(containing(getFileContent("tuyucheng.txt")))
    .willReturn(aResponse().withStatus(200)));
```

### 4.3 RestAssured测试请求

现在是时候关注我们将发送到服务器的multipart请求了。**使用RestAssured，请求规范遵循given-when-then范式**。

首先，我们将使用multipart()方法来添加multipart。让我们看看它的参数：

-   file，与请求中的文件关联的控件名称
-   文件内容

然后，我们将使用post()方法发出HTTP POST请求。它的参数是/upload目标URL。最后，我们将使用statusCode()方法设置预期的响应状态：

```java
given()
    .multiPart("file", getFile("/tuyucheng.txt"))
    .when()
    .post("/upload")
    .then()
    .statusCode(200);
```

现在我们可以根据之前的存根测试这个请求：状态码是正确的！

可以通过重复调用multipart()方法向测试请求添加更多文件。比如我们可以新增一个文件helloworld.txt，并在请求中将对应的部分命名为helloworld：

```java
given() 
    .multiPart("file", getFile("/tuyucheng.txt"))
    .multiPart("helloworld", getFile("/helloworld.txt"))
    .when() 
    .post("/upload") 
    .then() 
    .statusCode(200);
```

## 5. 构建我们自己的Multipart规范

有时，我们希望在请求中提供更详细的multipart。让我们看看如何做到这一点。

### 5.1 存根服务器更新

让我们快速更新我们的Wiremock存根。这一次，我们希望请求包含一个名为file的multipart。该文件的名称将为file.txt。请求的Content-Type标头将为text/plain，正文将包含File content。

**我们将使用MultipartValuePatternBuilder来完成我们的mock服务器规范**：

```java
MultipartValuePatternBuilder multipartValuePatternBuilder = aMultipart()
    .withName("file")
    .withHeader("Content-Disposition", containing("file.txt"))
    .withBody(equalTo("File content"))
    .withHeader("Content-Type", containing("text/plain"));

stubFor(post(urlEqualTo("/upload"))
    .withMultipartRequestBody(multipartValuePatternBuilder)
    .willReturn(aResponse().withStatus(200)));
```

### 5.2 Multipart规范

现在让我们跳到编写我们的测试请求。**得益于RestAssured库的MultipartSpecification类，我们可以设置控件名称**，只要文件的名称、MIME类型、字符集和内容即可。控件名称标识服务器端的部分，而文件名由发送方设置。

因此，我们将构建一个简单的示例，其中：

-   该部分将被称为file
-   文件的名称是file.txt
-   文件的MIME类型是text/plain

因此，请求的Content-Type是text/plain而不是multipart/form-data。尽管如此，这仍然是一个multipart请求，因为内容是在一个部分中编码的。此外，我们不需要使用磁盘上的现有文件。为了展示这一点，我们将从String动态生成内容：

```java
MultiPartSpecification multiPartSpecification = new MultiPartSpecBuilder("File content".getBytes())
    .fileName("file.txt")
    .controlName("file")
    .mimeType("text/plain")
    .build();
```

现在，我们可以通过直接使用重载方法multipart()将MultipartSpecification作为参数来更新我们的RestAssured请求规范：

```java
given()
    .multiPart(multiPartSpecification)
    .when()
    .post("/upload")
    .then()
    .statusCode(200);
```

我们现在知道如何为我们的RestAssured multipart请求带来更多的粒度。

## 6. 总结

在本文中，我们使用RestAssured将multipart请求发送到模拟服务器。

我们看到了如何发送仅包含内容和控件名称的文件，然后切换到构建更复杂的部分。

与往常一样，代码在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/rest-assured)上可用。
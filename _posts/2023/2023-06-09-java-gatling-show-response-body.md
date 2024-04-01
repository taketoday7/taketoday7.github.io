---
layout: post
title:  如何使用Gatling显示完整的HTTP响应主体
category: load
copyright: load
excerpt: Gatling
---

## 1. 概述

[Gatling](https://www.baeldung.com/introduction-to-gatling)是一种流行的用Scala编写的负载测试工具，可以帮助我们在本地和云机器上创建高性能、压力和[负载测试](https://www.baeldung.com/load-test-a-website-with-gatling)。此外，它还广泛用于测试HTTP服务器。默认情况下，Gatling专注于捕获和分析响应时间、错误率等性能指标，而不会显示完整的HTTP响应主体。

在本教程中，我们将学习如何使用Gatling显示完整的HTTP响应主体。这对于在负载测试期间理解和调试服务器响应非常有用。

## 2. 项目设置

在本教程中，我们将使用[Gatling](https://www.baeldung.com/gatling-distributed-perf-testing) Maven插件来运行Gatling脚本。为此，我们需要将插件添加到pom.xml中：

```xml
<plugin>
    <groupId>io.gatling</groupId>
    <artifactId>gatling-maven-plugin</artifactId>
    <version>4.3.0</version>
    <configuration>
        <includes>
            <include>cn.tuyucheng.taketoday.gatling.http.FetchSinglePostSimulation</include>
            <include>cn.tuyucheng.taketoday.gatling.http.FetchSinglePostSimulationLog</include>
        </includes>
        <runMultipleSimulations>true</runMultipleSimulations>
    </configuration>
</plugin>
```

我们将插件配置为运行多个模拟。此外，我们还需要[Gatling app](https://mvnrepository.com/artifact/io.gatling/gatling-app)和[Gatling highcharts](https://mvnrepository.com/artifact/io.gatling.highcharts/gatling-charts-highcharts)依赖项：

```xml
<dependency>
    <groupId>io.gatling</groupId>
    <artifactId>gatling-app</artifactId>
    <version>3.9.5</version>
</dependency>
<dependency>
    <groupId>io.gatling.highcharts</groupId>
    <artifactId>gatling-charts-highcharts</artifactId>
    <version>3.9.2</version>
    <scope>test</scope>
</dependency>
```

我们的目标是在控制台和日志文件中显示从示例API端点https://jsonplaceholder.typicode.com/posts/1获得的HTTP响应主体。

## 3. 使用Gatling显示完整的HTTP响应主体

让我们编写一个简单的[Gatling模拟](https://www.baeldung.com/gatling-load-testing-rest-endpoint)类，它向https://jsonplaceholder.typicode.com/posts/1发出[HTTP请求](https://www.baeldung.com/java-http-request)：

```java
public class FetchSinglePostSimulation extends Simulation {

    public FetchSinglePostSimulation() {
        HttpProtocolBuilder httpProtocolBuilder = http.baseUrl("https://jsonplaceholder.typicode.com");

        ScenarioBuilder scn = scenario("Display Full HTTP Response Body")
                .exec(http("GET Request")
                        .get("/posts/1")
                        .check(status().is(200))
                        .check(bodyString().saveAs("responseBody")))
                .exec(session -> {
                    System.out.println("Response Body:");
                    System.out.println(session.getString("responseBody"));
                    return session;
                });
        setUp(scn.injectOpen(atOnceUsers(1))).protocols(httpProtocolBuilder);
    }
}
```

在上面的代码中，我们定义了一个名为FetchSinglePostSimulation的新类，它扩展了Gatling库中的Simulation类。

接下来，我们创建一个HttpProtocolBuilder对象并将HTTP请求的基本URL设置为https://jsonplaceholder.typicode.com/。

然后，我们定义一个ScenarioBuilder对象，该对象有助于定义一系列模拟场景。首先，我们使用exec()方法启动HTTP请求。接下来，我们指定该请求是对/posts/1端点的GET请求。

此外，我们检查响应的HTTP状态代码是否为200。最后，我们使用check()方法将响应主体作为字符串保存在会话变量responseBody中。

此外，我们启动一个将Session对象作为输入的自定义操作。然后，我们将responseBody的值打印到控制台。最后，我们返回会话对象。

此外，模拟是通过一次将一个用户注入场景并使用我们之前创建的httpProtocolBuilder对象配置HTTP协议来设置的。

要运行模拟，让我们打开终端并切换到项目的根目录。然后，让我们运行Gatling test命令：

```shell
mvn gatling:test
```

该命令生成模拟报告，并将响应正文输出到控制台。这是测试的响应主体：

![](/assets/images/2023/load/javagatlingshowresponsebody01.png)

上图显示了模拟报告的响应。

让我们更进一步，将响应记录在文件中而不是将其输出到控制台。首先，让我们编写一个方法来处理文件创建：

```java
void writeFile(String fileName, String content) throws IOException {
    try(BufferedWriter writer = new BufferedWriter(new FileWriter(fileName, true))){
        writer.write(content);
        writer.newLine();
    }
}
```

在上面的方法中，我们创建了BufferedWriter和FileWriter的新实例，以将[HTTP响应主体写入文本文件](https://www.baeldung.com/java-write-to-file)。

最后，让我们修改自定义操作，将响应主体写入文件，而不是输出到控制台：

```java
public class FetchSinglePostSimulationLog extends Simulation {

    public FetchSinglePostSimulationLog() {
        HttpProtocolBuilder httpProtocolBuilder = http.baseUrl("https://jsonplaceholder.typicode.com");

        ScenarioBuilder scn = scenario("Display Full HTTP Response Body")
                .exec(http("GET Request")
                        .get("/posts/1")
                        .check(status().is(200))
                        .check(bodyString().saveAs("responseBody")))
                .exec(session -> {
                    String responseBody = session.getString("responseBody");
                    try {
                        writeFile("response_body.log", responseBody);
                    } catch (IOException e) {
                        System.err.println("error writing file");
                    }
                    return session;
                });
        setUp(scn.injectOpen(atOnceUsers(1))).protocols(httpProtocolBuilder);
    }
}
```

我们通过调用writeFile()方法并添加文件名response_body.log和HTTP响应正文作为参数来修改自定义操作，writeFile()方法执行将响应记录到文件的操作。

## 4. 总结

在本文中，我们学习了如何使用Gatling在控制台上显示完整的HTTP响应主体。此外，我们还了解了如何将响应记录到文件而不是将其输出到控制台。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/gatling-java)上获得。
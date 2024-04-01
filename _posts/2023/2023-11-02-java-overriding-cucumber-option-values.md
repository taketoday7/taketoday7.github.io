---
layout: post
title: 覆盖Cucumber选项值
category: bdd
copyright: bdd
excerpt: Cucumber
---

## 1. 概述

**在本教程中，我们将学习三种不同的方法来覆盖[Cucumber](https://www.baeldung.com/cucumber-spring-integration)选项值**。从优先级的角度来看，Cucumber将解析并覆盖以下选项：

- 系统属性、环境变量和cucumber.properties文件
- @CucumberOptions注解
- CLI参数

**为了展示每种方法，我们将运行一个包含两个场景的简单feature文件，并覆盖Cucumber tags选项**。

## 2. 设置

在使用每种方法之前，我们需要进行一些初始设置。首先，让我们添加[cucumber-java](https://mvnrepository.com/artifact/io.cucumber/cucumber-java)、[cucumber-junit](https://mvnrepository.com/artifact/io.cucumber/cucumber-junit)、[cucumber-spring](https://mvnrepository.com/artifact/io.cucumber/cucumber-spring)和[junit-vintage-engine](https://mvnrepository.com/artifact/org.junit.vintage/junit-vintage-engine)依赖：

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>7.14.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>7.14.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>7.14.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
```

接下来，让我们实现一个包含两个端点的简单控制器：

```java
@RestController
public class HealthCheckController {

    @GetMapping(path = "/v1/status", produces = APPLICATION_JSON_VALUE)
    public HttpStatus getV1Status() {
        return ResponseEntity.ok().build().getStatusCode();
    }

    @GetMapping(path = "/v2/status", produces = APPLICATION_JSON_VALUE)
    public HttpStatus getV2Status() {
        return ResponseEntity.ok().build().getStatusCode();
    }
}
```

现在，我们可以添加feature文件和两个场景：

```gherkin
Feature: status endpoints can be verified
    @v1
    Scenario: v1 status is healthy
        When the client calls /v1/status
        Then the client receives 200 status code

    @v2
    Scenario: v2 status is healthy
        When the client calls /v2/status
        Then the client receives 200 status code

```

最后，让我们添加所需的Cucumber胶水代码：

```java
@When("^the client calls /v1/status")
public void checkV1Status() throws Throwable {
    executeGet("http://localhost:8082/v1/status");
}

@When("^the client calls /v2/status")
public void checkV2Status() throws Throwable {
    executeGet("http://localhost:8082/v2/status");
}

@Then("^the client receives (\\d+) status code$")
public void verifyStatusCode(int statusCode) throws Throwable {
    final HttpStatus currentStatusCode = latestResponse.getStatusCode();
    assertThat(currentStatusCode.value(), is(statusCode));
}
```

默认情况下，如果没有另外指定，Cucumber会运行所有场景。让我们通过运行测试来验证此行为：

```shell
mvn test
```

正如预期的那样，这两个场景都会被执行：

```shell
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
```

下面，我们开始讨论覆盖Cucumber选项值的三种不同方法。

## 3. 使用cucumber.properties文件

第一种方法从系统属性、环境变量和cucumber.properties文件加载Cucumber选项。

让我们将cucumber.filter.tags属性添加到cucumber.properties文件中：

```properties
cucumber.filter.tags=@v1
```

这次运行测试将仅执行@v1场景：

```shell
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

## 4. 使用@CucumberOptions注解

第二种方法使用@CucumberOptions注解，让我们添加tags字段：

```java
@CucumberOptions(tags = "@v1")
```

再次运行测试将仅执行@v1场景：

```shell
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

**请注意，此方法将覆盖使用系统属性、环境变量和cucumber.properties文件中的值提供的任何值**。

## 5. 使用CLI参数

最后一种方法使用Cucumber CLI运行程序和tags参数。确保使用正确的类路径，以便它包含已编译的类和资源以及所有依赖项(包括具有测试范围的依赖项)：

```shell
java -cp ... io.cucumber.core.cli.Main --tags @v1
```

正如预期，仅执行@v1场景：

```shell
1 Scenarios (1 passed)
2 Steps (2 passed)
```

**请注意，此方法将覆盖通过cucumber.properties文件或@CucumberOptions注解提供的任何值**。

## 6. 总结

在本文中，我们学习了三种覆盖Cucumber选项值的方法。

第一种方法考虑作为系统属性、环境变量和cucumber.properties文件中的值提供的选项。第二种方法考虑作为@CucumberOptions注释中的字段提供的选项，并覆盖第一种方法提供的任何选项。最后一个方法使用CLI参数并覆盖使用前面任何方法提供的选项。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/spring-cucumber)上获得。
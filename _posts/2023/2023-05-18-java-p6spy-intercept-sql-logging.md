---
layout: post
title:  使用P6Spy拦截SQL日志记录
category: springboot
copyright: springboot
excerpt: P6Spy
---

## 1. 简介

在本教程中，我们将讨论[P6Spy](https://github.com/p6spy/p6spy)，**这是一个开源的免费库，可用于拦截Java应用程序中的SQL日志**。

在本文的第一部分，我们将讨论依赖这个外部库而不是仅仅为JPA或Hibernate启用SQL日志记录的主要优势，以及我们可以将该库集成到我们的应用程序中的不同方式。然后我们将展示一个使用P6Spy的[Spring Boot应用程序](https://www.baeldung.com/spring-boot)的简单示例，以查看一些最重要的可用配置。

## 2. P6Spy安装

P6Spy需要[安装在应用服务器上](https://p6spy.readthedocs.io/en/latest/install.html)。通常，将应用程序JAR放在类路径中并方便地配置驱动程序和JDBC连接就足够了。

使用P6Spy的另一种方法是通过与我们应用程序的现有代码集成，假设对代码进行小的更改是可以接受的。在下一节中，我们将看到一个示例，说明如何通过自动配置将P6Spy集成到Spring Boot应用程序中。

[p6spy-spring-boot-starter](https://mvnrepository.com/artifact/com.github.gavlyukovskiy/p6spy-spring-boot-starter)是一个Spring Boot启动器，提供与P6Spy和其他数据库监控库的集成。多亏了这个库，启用P6Spy日志记录就像在类路径上添加一个jar一样简单。使用Maven，只需在POM.xml中添加以下代码片段：

```xml
<dependency>
    <groupId>com.github.gavlyukovskiy</groupId>
    <artifactId>p6spy-spring-boot-starter</artifactId>
    <version>1.9.0</version>
</dependency>
```

如果我们要配置日志记录，我们需要在资源文件夹中添加一个名为“spy.properties”的文件：

```properties
appender=com.p6spy.engine.spy.appender.FileLogger
logfile=database.log
append=true
logMessageFormat=com.p6spy.engine.spy.appender.CustomLineFormat
customLogMessageFormat=%(currentTime)|%(executionTime)|%(category)|%(sqlSingleLine)
```

在这种情况下，我们将P6Spy配置为以自定义格式以附加模式将信息记录在名为“database.log”的文件中。这些只是其中的一些配置；其他的记录在[项目网站](https://p6spy.readthedocs.io/en/latest/configandusage.html)上。

## 3. 日志示例

要查看日志记录，我们需要运行一些查询。让我们向我们的应用程序添加几个简单的端点：

```java
@RestController
@RequestMapping("student")
public class StudentController {
    @Autowired
    private StudentRepository repository;
    @RequestMapping("/save")
    public Long save() {
        return repository.save(new Student("Pablo", "Picasso")).getId();
    }
    @RequestMapping("/find/{name}")
    public List<Student> getAll(@PathVariable String name) {
        return repository.findAllByFirstName(name);
    }
}
```

假设应用程序在端口8080上公开，现在让我们使用CURL访问这些端点：

```shell
curl http://localhost:8080/student/save
curl http://localhost:8080/student/find/Pablo
```

**我们现在可以看到在项目目录中创建了一个名为“database.log”的文件，其内容如下**：

```log
1683396972301|0|statement|select next value for student_seq
1683396972318|0|statement|insert into student (first_name, last_name, id) values ('Pablo', 'Picasso', 1)
1683396972320|0|commit|
1683396990989|0|statement|select s1_0.id,s1_0.first_name,s1_0.last_name from student s1_0 where s1_0.first_name='Pablo'
```

如果我们使用PreparedStatements并手动管理提交和回滚，日志记录也将起作用。让我们在应用程序中添加另一个控制器来测试此行为：

```java
@RestController
@RequestMapping("jdbc")
public class JDBCController {
    @Autowired
    private DataSource dataSource;

    @RequestMapping("/commit")
    public List<Map<String, String>> select() {
        List<Map<String, String>> results = new ArrayList<>();
        try {
            Connection connection = dataSource.getConnection();
            Statement statement = connection.createStatement();
            statement.executeQuery("SELECT * FROM student");
        } catch (Exception e) {
            throw new IllegalStateException(e);
        }
        return results;
    }

    @RequestMapping("/rollback")
    public List<Map<String, String>> rollback() {
        List<Map<String, String>> results = new ArrayList<>();
        try (Connection connection = dataSource.getConnection()) {
            connection.rollback();
        } catch (Exception e) {
            throw new IllegalStateException(e);
        }
        return results;
    }

    @RequestMapping("/query-error")
    public void error() {
        try (Connection connection = dataSource.getConnection();
             Statement statement = connection.createStatement()) {
            statement.execute("SELECT UNDEFINED()");
        } catch (Exception ignored) {
        }
    }
}
```

然后，因此，我们使用curl请求访问了以下端点：

```shell
curl http://localhost:8080/jdbc/commit
curl http://localhost:8080/jdbc/rollback
curl http://localhost:8080/jdbc/query-error
```

结果，我们将在“database.log”文件中看到以下日志：

```log
1683448381083|0|statement|SELECT * FROM student
1683448381087|0|commit|
1683448386586|0|rollback|
1683448388604|3|statement|SELECT UNDEFINED()
```

## 4. 总结

在本文中，我们看到了依赖外部第三方库(例如P6Spy)来记录数据库查询的多种优势。例如，我们尝试过的特殊配置使我们摆脱了嘈杂的邻居问题(想象一下充满查询的日志控制台)。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-3-observation)上获得。
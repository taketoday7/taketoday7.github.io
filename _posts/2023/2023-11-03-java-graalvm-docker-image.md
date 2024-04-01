---
layout: post
title:  创建GraalVM Docker镜像
category: springboot
copyright: springboot
excerpt: GraalVM
---

## 1. 简介

[GraalVM](https://www.baeldung.com/spring-native-intro)使用其AOT编译器将Java应用程序编译为机器可执行文件，这些可执行文件直接在目标计算机中执行，无需使用即时(JIT)编译器。GraalVM生成的二进制文件更小，启动时间更快，并且无需任何预热即可提供峰值性能。此外，这些可执行文件比在JVM上运行的应用程序具有更低的内存占用和CPU。

Docker允许我们将软件组件打包到Docker镜像中并作为Docker容器运行。Docker容器包含应用程序运行所需的一切，包括应用程序代码、运行时、系统工具和库。

在本教程中，我们将讨论创建Java应用程序的GraalVM本机镜像。然后我们将讨论如何使用此本机镜像作为Docker镜像并将其作为Docker容器运行。

## 2. 什么是原生镜像？

原生镜像是一种将Java代码提前编译为本机可执行文件的技术，此本机可执行文件仅包含需要在运行时执行的代码。这包括应用程序类、标准库类、语言运行时和来自JDK的静态链接本机代码。

本机镜像生成器(native-image)扫描应用程序类和其他元数据以创建特定于操作系统和架构的二进制文件。native-image工具执行静态应用程序代码分析，以确定应用程序运行时可访问的类和方法。然后它将所需的类、方法和资源编译为二进制可执行文件。

## 3. 原生镜像的好处

本机镜像可执行文件有几个优点：

- 由于本机镜像生成器仅编译运行时所需的资源，因此可执行文件的大小很小
- 本机可执行文件具有极快的启动时间，因为它们直接在目标计算机中执行，无需JIT编译器
- 由于仅打包所需的应用程序资源，因此提供较小的攻击面
- 可用于打包轻量级容器镜像(例如Docker镜像)，以实现快速高效的部署

## 4. 构建GraalVM原生镜像

在本节中，我们将为Spring Boot应用程序构建GraalVM本机镜像。首先，我们需要安装GraalVM并设置JAVA_HOME环境变量。其次，使用[Spring Web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web/3.1.4)和GraalVM Native Support依赖项创建一个Spring Boot应用程序：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.1.4</version>
</dependency>
```

我们还需要添加以下[插件](https://mvnrepository.com/artifact/org.graalvm.buildtools/native-maven-plugin)来支持GraalVM：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.graalvm.buildtools</groupId>
            <artifactId>native-maven-plugin</artifactId>
            <version>0.9.27</version>
        </plugin>
    </plugins>
</build>
```

该应用程序包含一个示例REST控制器：

```java
@RestController
class HelloController {
	
    @GetMapping
    public String hello() {
	return "Hello GraalVM";
    }
}
```

让我们使用Maven命令构建本机可执行文件：

```shell
$mvn -Pnative native:compile
```

native-maven-plugin构建GraalVM本机镜像。由于GraalVM本机镜像编译器执行静态代码分析，因此与常规Java应用程序编译相比，构建时间较长。

以下是GraalVM编译的输出：

```shell
========================================================================================================================
GraalVM Native Image: Generating 'springboot-graalvm-docker' (executable)...
========================================================================================================================
<strong>[1/8] Initializing... (42.7s @ 0.15GB)</strong>
Java version: 17.0.8+9-LTS, vendor version: Oracle GraalVM 17.0.8+9.1
Graal compiler: optimization level: 2, target machine: x86-64-v3, PGO: ML-inferred
C compiler: gcc (linux, x86_64, 11.3.0)
Garbage collector: Serial GC (max heap size: 80% of RAM)

// Omitted for clarity

<strong>[2/8] Performing analysis... [******] (234.6s @ 1.39GB)</strong>
15,543 (90.25%) of 17,222 types reachable
25,854 (67.59%) of 38,251 fields reachable
84,701 (65.21%) of 129,883 methods reachable
4,906 types, 258 fields, and 4,984 methods registered for reflection
64 types, 70 fields, and 55 methods registered for JNI access
4 native libraries: dl, pthread, rt, z
[3/8] Building universe... (14.7s @ 2.03GB)
[4/8] Parsing methods... [*******] (55.6s @ 2.05GB)
[5/8] Inlining methods... [***] (4.9s @ 2.01GB)
[6/8] Compiling methods... [**********
[6/8] Compiling methods... [*******************] (385.2s @ 3.02GB)
[7/8] Layouting methods... [****] (14.0s @ 2.00GB)
[8/8] Creating image... [*****] (30.7s @ 2.72GB)
48.81MB (58.93%) for code area: 48,318 compilation units
30.92MB (37.33%) for image heap: 398,288 objects and 175 resources
3.10MB ( 3.75%) for other data
82.83MB in total

// Omitted for clarity

Finished generating 'springboot-graalvm-docker' in 13m 7s.

// Omitted for clarity
```

在上面的编译输出中，有以下几个关键点：

- 编译使用GraalVM Java编译器来编译应用程序
- 编译器对类型、字段和方法进行可达性检查
- 接下来，它构建本机执行并显示可执行文件大小和编译所需的时间

成功构建后，我们可以在target目录中找到可用的本机可执行文件，该可执行文件可以在命令行中执行。

## 5. 构建Docker镜像

在本节中，我们将为上一步中生成的本机可执行文件开发一个Docker镜像。

让我们创建以下Dockerfile：

```dockerfile
FROM ubuntu:jammy
COPY target/springboot-graalvm-docker /springboot-graalvm-docker
CMD ["/springboot-graalvm-docker"]
```

接下来，让我们使用以下命令构建Docker镜像：

```shell
$docker build -t springboot-graalvm-docker .
```

成功构建后，我们可以注意到springboot-graalvm-dockerDocker镜像可用：

```shell
$docker images | grep springboot-graalvm-docker
```

我们可以使用以下命令执行该镜像：

```shell
$docker run -p 8080:8080 springboot-graalvm-docker
```

上面的命令启动了容器，我们可以注意到Spring Boot的启动日志：

```shell
// Ommited for clarity
***  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization <strong>completed in 14 ms</strong>
***  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
***  INFO 1 --- [           main] c.t.t.g.GraalvmDockerImageApplication    : Started GraalvmDockerImageApplication in 0.043 seconds (process running for 0.046)
```

应用程序将在43毫秒内启动，我们可以通过访问以下命令来访问REST端点：

```shell
$curl localhost:8080
```

它显示以下输出：

```text
Hello GraalVM
```

## 6. 总结

在本文中，我们为GraalVM本机可执行文件构建Docker镜像。

我们开始讨论GraalVM原生镜像及其优势，它对于需要首次启动和低内存占用的用例非常有用。接下来，我们使用GraalVM本机镜像编译器生成Spring Boot应用程序的本机可执行文件。最后，我们使用本机可执行文件开发了一个Docker镜像，并使用该镜像启动了一个Docker容器。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-graalvm-docker)上获得。
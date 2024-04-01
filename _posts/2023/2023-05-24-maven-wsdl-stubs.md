---
layout: post
title:  使用Maven生成WSDL存根
category: maven
copyright: maven
excerpt: Maven
---

## 1. 简介

在本教程中，我们将展示如何配置[JAX-WS Maven插件](https://www.mojohaus.org/jaxws-maven-plugin/)以从WSDL(Web服务描述语言)文件生成Java类。因此，我们将能够使用生成的类轻松调用Web服务。

## 2. 配置我们的Maven插件

首先，让我们在pom.xml文件的构建部分中包含带有wsimport目标的JAX-WS Maven插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>jaxws-maven-plugin</artifactId>
            <version>2.6</version>
            <executions>
                <execution>
                    <goals>
                        <goal>wsimport</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

简而言之，**wsimport目标生成用于JAX-WS客户端和服务的JAX-WS可移植工件**。该工具读取WSDL文件并生成Web服务开发、部署和调用所需的所有工件。

### 2.1 WSDL目录配置

在我们的Maven插件部分，**wsdlDirectory配置属性通知插件我们的WSDL文件所在的位置**。在这个例子中，我们将告诉插件获取我们项目的src/main/resources目录中的所有WSDL文件：

```xml
<plugin>
    ...
    <configuration>
        <wsdlDirectory>${project.basedir}/src/main/resources/</wsdlDirectory>
    </configuration>
</plugin>
```

### 2.2 WSDL目录特定文件配置

此外，我们可以使用wsdlFiles配置属性来定义生成类时要考虑的WSDL文件列表：

```xml
<plugin>
    ...
    <configuration>
        <wsdlDirectory>${project.basedir}/src/main/resources/</wsdlDirectory>
        <wsdlFiles>
            <wsdlFile>file1.wsdl</wsdlFile>
            <wsdlFile>file2.wsdl</wsdlFile>
            ...
        </wsdlFiles>
    </configuration>
</plugin>
```

但是，**当未设置wsdlFiles属性时，则将考虑wsdlDirectory属性指定的目录中的所有文件**。

### 2.3 WSDL URL配置

或者，我们可以配置插件的wsdlUrl配置属性：

```xml
<plugin>
    ...
    <configuration>
        <wsdlUrls>
            <wsdlUrl>http://localhost:8888/ws/country?wsdl</wsdlUrl>
            ...
        </wsdlUrls>
    </configuration>
</plugin>
```

要使用此选项，**托管WSDL文件URL的服务器必须已启动并正在运行，以便我们的插件可以读取它**。

### 2.4 配置生成的类目录

接下来，在packageName属性中，我们可以设置生成的类包名称，并在sourceDestDir中设置输出目录：

```xml
<plugin>
    ...
    <configuration>
        <packageName>cn.tuyucheng.taketoday.soap.ws.client</packageName>
        <sourceDestDir>
            ${project.build.directory}/generated-sources/
        </sourceDestDir>
    </configuration>
</plugin>
```

因此，我们使用wsdlDirectory选项的插件配置的最终版本是：

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>jaxws-maven-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <goals>
                <goal>wsimport</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <wsdlDirectory>${project.basedir}/src/main/resources/</wsdlDirectory>
        <packageName>cn.tuyucheng.taketoday.soap.ws.client</packageName>
        <sourceDestDir>
            ${project.build.directory}/generated-sources/
        </sourceDestDir>
    </configuration>
</plugin>
```

## 3. 运行JAX-WS插件

最后，配置好插件后，我们可以使用Maven生成类并检查输出日志：

```shell
mvn clean install
```

```shell
[INFO] --- jaxws-maven-plugin:2.6:wsimport (default) @ jaxws ---
[INFO] Processing: file:/D:/workspace/intellij/taketoday-tutorial4j/maven.modules/maven-plugins/jaxws/src/main/resources/country.wsdl
[INFO] jaxws:wsimport args: [-keep, -s, 'D:\workspace\intellij\taketoday-tutorial4j\maven.modules\maven-plugins\jaxws\target\generated-sources', -d, 'D:\workspace\intellij\taketoday-tutorial4j\maven.module
s\maven-plugins\jaxws\target\classes', -encoding, UTF-8, -Xnocompile, -p, cn.tuyucheng.taketoday.soap.ws.client, "file:/D:/workspace/intellij/taketoday-tutorial4j/maven.modules/maven-plugins/jaxws/src/main/resources/country.wsdl"]

parsing WSDL...
Generating code...
```

## 4. 检查生成的类

运行我们的插件后，我们可以检查在sourceDestDir属性中配置的文件夹target/generated-sources中的输出。

生成的类可以在packageName属性中配置的cn.tuyucheng.taketoday.soap.ws.client中找到：

```text
cn.tuyucheng.taketoday.soap.ws.client.Country.java
cn.tuyucheng.taketoday.soap.ws.client.CountryService.java  
cn.tuyucheng.taketoday.soap.ws.client.CountryServiceImplService.java
cn.tuyucheng.taketoday.soap.ws.client.Currency.java
cn.tuyucheng.taketoday.soap.ws.client.ObjectFactory.java
```

## 5. 总结

在本文中，我们了解了如何使用JAX-WS插件从WSDL文件生成Java类。因此，我们现在能够创建Web服务客户端并使用生成的类来调用我们的服务。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/maven.modules/maven-plugins/jaxws)上找到。
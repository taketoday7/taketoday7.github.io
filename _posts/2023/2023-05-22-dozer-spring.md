---
layout: post
title:  Spring Boot中使用Dozer
category: springboot
copyright: springboot
excerpt: Dozer
---

## 1. 概述

Dozer是一个Java Bean之间的映射器，它递归地将数据从一个对象复制到另一个对象。

有了这个工具之后，**我们将一个对象的所有属性值转给另一个对象时，就不需要再去写重复的调用setter和getter方法**。

最重要的是，Dozer可以确保来自数据库的内部域对象不会渗入外部表示层或外部消费者，它还可以将域对象映射到外部API调用，反之亦然。

## 2. 为什么要使用映射框架

映射框架在分层架构中作用很大，我们可以通过封装对特定数据对象的更改与将这些对象传播到其他层(即外部服务数据对象、域对象、数据传输对象、内部服务数据对象)来创建抽象层。映射框架非常适合在负责将数据从一个数据对象映射到另一个数据对象的Mapper类型类中使用。

对于分布式系统架构而言，副作用是域对象在不同系统之间的传递。那么，我们不希望内部域对象暴露在外部，也不允许外部域对象渗入我们的系统。

数据对象之间的映射传统上是通过在对象之间复制数据的手动编码值对象组装器(或转换器)来解决的。Dozer作为一个强大、通用、灵活、可重用和可配置的开源映射框架，节省了开发人员开发某种自定义映射器框架带来的时间成本。

## 3. Maven依赖

为了简化使用方式，Dozer提供了[dozer-spring-boot-starter](https://central.sonatype.com/artifact/com.github.dozermapper/dozer-spring-boot-starter/6.5.2)，因此让我们将其添加到我们项目的POM中：

```xml
<dependency>
    <groupId>com.github.dozermapper</groupId>
    <artifactId>dozer-spring-boot-starter</artifactId>
    <version>6.5.2</version>
</dependency>
```

## 4. 实体类

首先，让我们创建项目中使用到的实体类：

```java
@Data
public class UserDTO {
    private String userId;
    private String userName;
    private int userAge;
    private String address;
    private String birthday;
}
```

```java
@Data
public class UserEntity {
    private String id;
    private String name;
    private int age;
    private String address;
    private Date birthday;
}
```

## 5. Dozer配置

接下来，让我们在resources/dozer/目录下创建dozer的全局配置文件global.dozer.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mappings xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns="http://dozermapper.github.io/schema/bean-mapping"
          xsi:schemaLocation="http://dozermapper.github.io/schema/bean-mapping
                              http://dozermapper.github.io/schema/bean-mapping.xsd">
    <configuration>
        <date-format>yyyy-MM-dd</date-format>
    </configuration>
</mappings>
```

然后，在resources/dozer/目录下创建dozer的映射文件biz.dozer.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mappings xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns="http://dozermapper.github.io/schema/bean-mapping"
          xsi:schemaLocation="http://dozermapper.github.io/schema/bean-mapping
                             http://dozermapper.github.io/schema/bean-mapping.xsd">

    <mapping date-format="yyyy-MM-dd">
        <class-a>cn.tuyucheng.taketoday.dozer.entity.UserEntity</class-a>
        <class-b>cn.tuyucheng.taketoday.dozer.dto.UserDTO</class-b>
        <field>
            <a>id</a>
            <b>userId</b>
        </field>
        <field>
            <a>name</a>
            <b>userName</b>
        </field>
        <field>
            <a>age</a>
            <b>userAge</b>
        </field>
    </mapping>
</mappings>
```

或者，我们可以通过map-id属性为映射指定一个id：

```xml
<mapping date-format="yyyy-MM-dd" map-id="user">
    <class-a>cn.tuyucheng.taketoday.dozer.entity.UserEntity</class-a>
    <class-b>cn.tuyucheng.taketoday.dozer.dto.UserDTO</class-b>
    <field>
        <a>id</a>
        <b>userId</b>
    </field>
    <field>
        <a>name</a>
        <b>userName</b>
    </field>
    <field>
        <a>age</a>
        <b>userAge</b>
    </field>
</mapping>
```

为了自动配置Dozer，我们需要创建一个DozerBeanMapperFactoryBean类型的bean：

```java
@SpringBootApplication
public class DozerApp {

    public static void main(String[] args) {
        SpringApplication.run(DozerApp.class, args);
    }

    @Bean
    public DozerBeanMapperFactoryBean dozerMapper(@Value("classpath:dozer/*.xml") Resource[] resources) throws IOException {
        DozerBeanMapperFactoryBean dozerBeanMapperFactoryBean = new DozerBeanMapperFactoryBean();
        dozerBeanMapperFactoryBean.setMappingFiles(resources);
        return dozerBeanMapperFactoryBean;
    }
}
```

## 6. 单元测试

最后，让我们编写一些测试来确保一切正常工作：

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = DozerApp.class)
class DozerIntegrationTest {

    @Autowired
    private Mapper mapper;

    @Test
    void givenDTO_whenMapperToEntityUsingDozer_thenCorrect() {
        UserDTO userDTO = new UserDTO();
        userDTO.setUserId("100");
        userDTO.setUserName("Doe");
        userDTO.setUserAge(2);
        userDTO.setAddress("Sweden");
        userDTO.setBirthday("2020-07-04");

        UserEntity entity = mapper.map(userDTO, UserEntity.class);

        assertThat(entity).isNotNull().satisfies(e -> {
            assertThat(e.getId()).isEqualTo("100");
            assertThat(e.getName()).isEqualTo("Doe");
            assertThat(e.getAge()).isEqualTo(2);
            assertThat(e.getAddress()).isEqualTo("Sweden");
            assertThat(e.getBirthday()).isEqualTo("2020-07-04");
        });
    }

    @Test
    void givenDTOAndEntityOfInitialId_whenMapperToEntityUsingDozer_thenEntityIdShouldBeOverridden() {
        UserDTO userDTO = new UserDTO();
        userDTO.setUserId("100");
        userDTO.setUserName("Doe");
        userDTO.setUserAge(5);
        userDTO.setAddress("Sweden");
        userDTO.setBirthday("2017-07-04");

        UserEntity entity = new UserEntity();
        entity.setId("200");

        assertThat(entity.getId()).isEqualTo("200");

        mapper.map(userDTO, entity);

        assertThat(entity).isNotNull().satisfies(e -> {
            assertThat(e.getId()).isEqualTo("100"); // overridden by userDTO
            assertThat(e.getName()).isEqualTo("Doe");
            assertThat(e.getAge()).isEqualTo(5);
            assertThat(e.getAddress()).isEqualTo("Sweden");
            assertThat(e.getBirthday()).isEqualTo("2017-07-04");
        });
    }

    @Test
    void givenDTO_whenMapperToEntityUsingSpecifiedMapId_thenCorrect() {
        UserDTO userDTO = new UserDTO();
        userDTO.setUserId("100");
        userDTO.setUserName("Doe");
        userDTO.setUserAge(3);
        userDTO.setAddress("Sweden");

        UserEntity entity = new UserEntity();

        assertThat(entity.getName()).isNull();
        assertThat(entity.getAge()).isEqualTo(0);

        mapper.map(userDTO, entity, "user"); // specify the map-id as user

        assertThat(entity).isNotNull().satisfies(e -> {
            assertThat(e.getId()).isEqualTo("100");
            assertThat(e.getName()).isEqualTo("Doe");
            assertThat(e.getAge()).isEqualTo(3);
            assertThat(e.getAddress()).isEqualTo("Sweden");
        });
    }
}
```

## 7. 总结

在本文中，我们介绍了Dozer并且演示了如何使用Dozer将DTO映射到实体类。该库可以简化很多不必要的工作，但不幸的是，Dozer已经停更并且不再维护。如果你想使用一个活跃的库，那么MapStruct是一个不错的选择。

与往常一样，所有代码都可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-libraries-3)上找到。
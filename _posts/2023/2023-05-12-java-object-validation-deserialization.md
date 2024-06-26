---
layout: post
title:  反序列化后的对象验证
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将了解如何使用Java的验证 API 在反序列化后验证对象。

## 2. 手动触发验证

Java 的[bean 验证](https://www.baeldung.com/javax-validation)API在[JSR 380](https://jcp.org/en/jsr/detail?id=380)中定义。它的一个常见用途是[在 Spring 控制器中使用](https://www.baeldung.com/spring-boot-bean-validation)[@Valid](https://www.baeldung.com/spring-boot-bean-validation)注解参数。但是，在本文中，我们将重点关注控制器之外的验证。

首先，让我们编写一个方法来验证对象的内容是否符合其验证约束。为此，我们将从默认验证器工厂获取验证器。然后，我们将对对象应用validate()方法。此方法返回一[组](https://www.baeldung.com/java-set-operations#1-what-is-a-set)ConstraintViolation。ConstraintViolation封装了一些关于验证错误的提示。为了简单起见，我们将抛出一个ConstraintViolationException以防出现任何验证问题：

```java
<T> void validate(T t) {
    Set<ConstraintViolation<T>> violations = validator.validate(t);
    if (!violations.isEmpty()) {
        throw new ConstraintViolationException(violations);
    }
}
```

如果我们在对象上调用此方法，如果对象不遵守任何验证约束，它将抛出异常。可以在具有附加约束的现有对象上的任何时候调用此方法。

## 3.将验证纳入反序列化过程

我们现在的目标是将验证合并到反序列化过程中。具体来说，我们将覆盖[Jackson](https://www.baeldung.com/jackson)的反序列化器以在反序列化后立即执行验证。这将确保任何时候我们反序列化一个对象，如果它不合规，我们不允许任何进一步的处理。

首先，我们需要覆盖默认的BeanDeserializer。BeanDeserializer是一个可以反序列化对象的类。我们将要调用基本的反序列化方法，然后将我们的 validate() 方法应用于创建的实例。我们的 BeanDeserializerWithValidation 看起来像这样：

```java
public class BeanDeserializerWithValidation extends BeanDeserializer {

    protected BeanDeserializerWithValidation(BeanDeserializerBase src) {
        super(src);
    }

    @Override
    public Object deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {
        Object instance = super.deserialize(p, ctxt);
        validate(instance);
        return instance;
    }

}
```

下一步是实现我们自己的BeanDeserializerModifier。这将允许我们使用BeanDeserializerWithValidation中定义的行为来改变反序列化过程：

```java
public class BeanDeserializerModifierWithValidation extends BeanDeserializerModifier {

    @Override
    public JsonDeserializer<?> modifyDeserializer(DeserializationConfig config, BeanDescription beanDesc, JsonDeserializer<?> deserializer) {
        if (deserializer instanceof BeanDeserializer) {
            return new BeanDeserializerWithValidation((BeanDeserializer) deserializer);
        }

        return deserializer;
    }

}
```

最后，我们需要创建一个[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)并将我们的BeanDeserializerModifier注册为一个Module。模块是扩展 Jackson 默认功能的一种方式。让我们把它包装在一个方法中：

```java
ObjectMapper getObjectMapperWithValidation() {
    SimpleModule validationModule = new SimpleModule();
    validationModule.setDeserializerModifier(new BeanDeserializerModifierWithValidation());
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(validationModule);
    return mapper;
}
```

## 4. 示例用法：从文件中读取和验证对象

我们现在将展示一个如何使用自定义ObjectMapper的小示例。首先，让我们定义一个Student对象。一个学生有一个名字。名称长度必须在 5 到 10 个字符之间：

```java
public class Student {

    @Size(min = 5, max = 10, message = "Student's name must be between 5 and 10 characters")
    private String name;

    public String getName() {
        return name;
    }

}
```

现在让我们创建一个包含有效Student对象的[JSON表示的](https://www.baeldung.com/java-json)validStudent.json文件：

```java
{
  "name": "Daniel"
}
```

我们将在InputStream中读取该文件的内容。首先，让我们定义将InputStream解析为Student对象并同时对其进行验证的方法。为此，我们要使用我们的ObjectMapper：

```java
Student readStudent(InputStream inputStream) throws IOException {
    ObjectMapper mapper = getObjectMapperWithValidation();
    return mapper.readValue(inputStream, Student.class);
}
```

我们现在可以编写一个测试，我们将：

-   首先将文件内容读入InputStream
-   将InputStream转换为Student对象
-   检查Student对象的内容是否与预期一致

这个测试看起来像这样：

```java
@Test
void givenValidStudent_WhenReadStudent_ThenReturnStudent() throws IOException {
    InputStream inputStream = getClass().getClassLoader().getResourceAsStream(("validStudent.json");
    Student result = readStudent(inputStream);
    assertEquals("Daniel", result.getName());
}
```

同样，我们可以创建一个invalid.json文件，其中包含名称少于 5 个字符的Student的 JSON 表示形式：

```java
{
  "name": "Max"
}
```

现在我们需要调整测试以检查是否确实抛出了 ConstraintViolationException。此外，我们可以检查错误消息是否正确：

```java
@Test
void givenStudentWithInvalidName_WhenReadStudent_ThenThrows() {
    InputStream inputStream = getClass().getClassLoader().getResourceAsStream("invalidStudent.json");
    ConstraintViolationException constraintViolationException = assertThrows(ConstraintViolationException.class, () -> readStudent(inputStream));
    assertEquals("name: Student's name must be between 5 and 10 characters", constraintViolationException.getMessage());
}
```

## 5.总结

在本文中，我们了解了如何覆盖 Jackson 的配置以在反序列化后立即验证对象。因此，我们可以保证以后不可能对无效对象进行处理。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-validation-advanced)上获得。
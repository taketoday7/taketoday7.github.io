---
layout: post
title:  在没有equals()方法的情况下断言两个对象相等
category: assertion
copyright: assertion
excerpt: Assertion
---

## 1. 概述

有时，我们没有能力重写类中的equals()方法。尽管如此，我们仍然希望将一个对象与另一个对象进行比较，以检查它们是否相同。

在本教程中，我们将学习几种在不使用[equals()](https://www.baeldung.com/java-equals-method-operator-difference)方法的情况下测试两个对象是否相等的方法。

## 2. 示例类

首先，让我们创建本文中将使用Person和Address类：

```java
public class Person {

    private Long id;
    private String firstName;
    private String lastName;
    private Address address;

    // getters and setters
}

public class Address {

    private Long id;
    private String city;
    private String street;
    private String country;

    // getters and setters
}
```

我们没有重写类中的equals()方法，因此，在确定相等性时将执行Object类中给出的默认实现。换句话说，Java在检查相等时会检查两个引用是否指向同一个对象。

## 3. 使用AssertJ

[AssertJ](https://www.baeldung.com/introduction-to-assertj)库提供了一种使用递归比较来比较对象的方法。使用内省，它确定应该比较哪些字段和值。

首先，要使用AssertJ库，让我们在pom.xml中添加[assertj-core](https://mvnrepository.com/artifact/org.assertj/assertj-core)依赖项：

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.21.0</version>
    <scope>test</scope>
</dependency>
```

要检查两个Person实例中的字段是否包含相同的值，我们将在调用isEqualTo()方法之前使用usingRecursiveComparison()方法：

```java
Person expected = new Person(1L, "Jane", "Doe");
Person actual = new Person(1L, "Jane", "Doe");

assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
```

此外，该算法获取实际对象的字段，然后将它们与预期对象的相应字段进行比较。但是，比较不能以对称方式进行。预期的对象可以有比实际对象更多的字段。

此外，我们可以使用ignoringFields()方法忽略某个字段：

```java
Person expected = new Person(1L, "Jane", "Doe");
Person actual = new Person(2L, "Jane", "Doe");

assertThat(actual)
    .usingRecursiveComparison()
    .ignoringFields("id")
    .isEqualTo(expected);
```

此外，当我们想要比较复杂的对象时，它可以有效地工作：

```java
Person expected = new Person(1L, "Jane", "Doe");
Address address1 = new Address(1L, "New York", "Sesame Street", "United States");
expected.setAddress(address1);

Person actual = new Person(1L, "Jane", "Doe");
Address address2 = new Address(1L, "New York", "Sesame Street", "United States");
actual.setAddress(address2);

assertThat(actual)
    .usingRecursiveComparison()
    .isEqualTo(expected);
```

## 4. 使用Hamcrest

[Hamcrest](https://www.baeldung.com/java-junit-hamcrest-guide)库使用反射来检查两个对象是否包含相同的属性。此外，它创建一个匹配器来检查实际对象是否包含与预期对象相同的值。

首先，让我们在pom.xml中添加[Hamcrest](https://mvnrepository.com/artifact/org.hamcrest/hamcrest)依赖项：

```xml
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
```

现在，让我们调用samePropertyValuesAs()方法并传递预期的对象：

```java
Person expected = new Person(1L, "Jane", "Doe");
Person actual = new Person(1L, "Jane", "Doe");

MatcherAssert.assertThat(actual, samePropertyValuesAs(expected));
```

与前面的示例类似，我们可以传递要忽略的字段的名称，它们将从预期和实际对象中删除。

但是，在幕后，Hamcrest使用反射从某些字段中获取值。检查是否相等时，将在每个字段上调用equals()方法。

也就是说，如果我们使用复杂对象，上面的代码将不起作用，因为我们也没有覆盖Address类中的equals()方法。因此，它将检查两个地址引用是否指向内存中的同一个对象。因此，断言将失败。

如果我们想比较复杂的对象，我们需要分别比较它们：

```java
Person expected = new Person(1L, "Jane", "Doe");
Address address1 = new Address(1L, "New York", "Sesame Street", "United States");
expected.setAddress(address1);

Person actual = new Person(1L, "Jane", "Doe");
Address address2 = new Address(1L, "New York", "Sesame Street", "United States");
actual.setAddress(address2);

MatcherAssert.assertThat(actual, samePropertyValuesAs(expected, "address"));
MatcherAssert.assertThat(actual.getAddress(), samePropertyValuesAs(expected.getAddress()));
```

在这里，我们首先在第一个断言中排除了address字段，然后在第二个断言中对其进行了比较。

## 5. 使用Apache Commons Lang3

现在，让我们看看如何使用[Apache Commons](https://www.baeldung.com/java-commons-lang-3)库检查是否相等。

我们将在pom.xml中添加[Apache Commons Lang3](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
    <scope>test</scope>
</dependency>
```

### 5.1 ReflectionToStringBuilder类

Apache Commons提供的类之一是ReflectionToStringBuilder类，它允许我们通过反射对象的字段和值来生成对象的字符串表示形式。

通过比较两个对象的字符串表示，我们可以断言它们相等而不需要使用equals()方法：

```java
Person expected = new Person(1L, "Jane", "Doe");
Person actual = new Person(1L, "Jane", "Doe");

assertThat(ReflectionToStringBuilder.toString(actual, ToStringStyle.SHORT_PREFIX_STYLE))
    .isEqualTo(ReflectionToStringBuilder.toString(expected, ToStringStyle.SHORT_PREFIX_STYLE));
```

但是，我们仍然需要在我们的类中重写toString()方法。

### 5.2 EqualsBuilder类

或者，我们可以使用EqualsBuilder类：

```java
Person expected = new Person(1L, "Jane", "Doe");
Person actual = new Person(1L, "Jane", "Doe");

assertTrue(EqualsBuilder.reflectionEquals(expected, actual));
```

它使用Java反射API来比较两个对象的字段。请务必注意，reflectionEquals()方法使用浅层相等性检查。

因此，当比较两个复杂对象时，我们需要忽略这些字段并分别进行比较：

```java
Person expected = new Person(1L, "Jane", "Doe");
Address address1 = new Address(1L, "New York", "Sesame Street", "United States");
expected.setAddress(address1);

Person actual = new Person(1L, "Jane", "Doe");
Address address2 = new Address(1L, "New York", "Sesame Street", "United States");
actual.setAddress(address2);

assertTrue(EqualsBuilder.reflectionEquals(expected, actual, "address"));
assertTrue(EqualsBuilder.reflectionEquals(expected.getAddress(), actual.getAddress()));
```

## 6. 使用Mockito

我们断言两个实例相等的另一种方法是使用[Mockito](https://www.baeldung.com/mockito-series)。

首先需要添加[mockito-core](https://mvnrepository.com/artifact/org.mockito/mockito-core)依赖项：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.4.0</version>
    <scope>test</scope>
</dependency>
```

现在，我们可以使用Mockito的ReflectionEquals类：

```java
Person expected = new Person(1L, "Jane", "Doe");
Person actual = new Person(1L, "Jane", "Doe");

assertTrue(new ReflectionEquals(expected).matches(actual));
```

此外，在检查相等性时将调用Apache Commons库中的EqualsBuilder。

再一次，我们需要使用与EqualsBuilder相同的补丁来比较复杂对象：

```java
Person expected = new Person(1L, "Jane", "Doe");
Address address1 = new Address(1L, "New York", "Sesame Street", "United States");
expected.setAddress(address1);

Person actual = new Person(1L, "Jane", "Doe");
Address address2 = new Address(1L, "New York", "Sesame Street", "United States");
actual.setAddress(address2);

assertTrue(new ReflectionEquals(expected, "address").matches(actual));
assertTrue(new ReflectionEquals(expected.getAddress()).matches(actual.getAddress()));
```

## 7. 总结

在本文中，我们学习了如何在不使用equals()方法的情况下断言两个实例相等。

综上所述，AssertJ的逐字段比较提供了比较复杂对象的最简单方法，而其他方法使用反射来比较字段，因此我们需要为复杂字段添加额外的断言。

通过利用本文中提到的工具，即使面对没有equals()方法的对象，我们也可以编写测试。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-assertions)上获得。
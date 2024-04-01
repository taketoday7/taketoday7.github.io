---
layout: post
title: 使用JUnit断言嵌套Map
category: assertion
copyright: assertion
excerpt: JUnit
---

## 1. 概述

在本教程中，我们将介绍一些断言外部Map内部存在嵌套Map的不同方法，我们主要讨论JUnit [Jupiter API](https://www.baeldung.com/junit-5) 和[Hamcrest API](https://www.baeldung.com/java-junit-hamcrest-guide)。

## 2. 使用Jupiter API进行断言

在本文中，我们将使用Junit 5，因此添加以下[Maven依赖项](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api)：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

我们以一个带有外部Map和内部Map的Map对象为例，外部Map有一个address键，内部Map作为值。它还具有一个name键，其值为John:

```json
{
    "name": "John",
    "address": {
        "city": "Chicago"
    }
}
```

通过示例，我们将断言内部Map中存在键值对。

**让我们从Jupiter API中的基本[assertTrue()](https://www.baeldung.com/junit-assertions#junit5-condition)方法开始**：

```java
@Test
void givenNestedMap_whenUseJupiterAssertTrueWithCasting_thenTest() {
    Map<String, Object> innerMap = Map.of("city", "Chicago");
    Map<String, Object> outerMap = Map.of("address", innerMap);

    assertTrue(outerMap.containsKey("address")
        && ((Map<String, Object>)outerMap.get("address")).get("city").equals("Chicago"));
}
```

我们使用布尔表达式来评估outerMap中是否存在innerMap，然后，我们检查内部Map是否有一个city键，其值为Chicago。

但是，由于上面使用类型转换来避免编译错误，因此失去了可读性。让我们尝试修复它：

```java
@Test
void givenNestedMap_whenUseJupiterAssertTrueWithoutCasting_thenTest() {
    Map<String, Object> innerMap = Map.of("city", "Chicago");
    Map<String, Map<String, Object>> outerMap = Map.of("address", innerMap);

    assertTrue(outerMap.containsKey("address") && outerMap.get("address").get("city").equals("Chicago"));
}
```

现在，我们改变了之前声明外部Map的方式，我们将其声明为Map<String，Map<String，Object\>\>而不是Map<String，Object\>。这样，我们就避免了类型转换并实现了更易读的代码。

但是，如果测试失败，我们将无法确切知道哪个断言失败了。为了解决这个问题，我们引入方法assertAll()：

```java
@Test
void givenNestedMap_whenUseJupiterAssertAllAndAssertTrue_thenTest() {
    Map<String, Object> innerMap = Map.of("city", "Chicago");
    Map<String, Map<String, Object>> outerMap = Map.of("address", innerMap);

    assertAll(
        () -> assertTrue(outerMap.containsKey("address")),
        () -> assertEquals(outerMap.get("address").get("city"), "Chicago")
    );
}
```

**我们将布尔表达式移至方法assertTrue()和assertEquals()。因此，现在我们可以知道确切的失败**。此外，它还提高了可读性。

## 3. 使用Hamcrest API进行断言

Hamcrest库提供了一个非常灵活的框架，可在Matchers的帮助下编写JUnit测试。我们将使用其开箱即用的Matcher，并使用其框架开发自定义Matcher来验证嵌套Map中是否存在键值对。

### 3.1 使用现有的匹配器

要使用Hamcrest库，我们必须更新pom.xml文件中的[Maven依赖项](https://mvnrepository.com/artifact/org.hamcrest/hamcrest)。

```xml
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
```

在开始示例之前，让我们先了解一下[Hamcrest库](https://www.baeldung.com/java-junit-hamcrest-guide)中对测试Map的支持。Hamcrest支持以下[匹配器](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html)，可与方法[assertThat()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/MatcherAssert.html)一起使用：

- **hasEntry()**：当检查的Map包含至少一个键等于指定键且值等于指定值的条目时，为Maps匹配创建匹配器
- **hasKey()**：当检查的Map包含至少一个满足指定匹配器的键时，为Maps匹配创建匹配器
- **hasValue()**：当检查的Map包含至少一个满足指定valueMatcher的值时，为Maps匹配创建一个匹配器

让我们从一个基本示例开始，就像前面的部分一样，但我们将使用方法assertThat()和hasEntry()：

```java
@Test
void givenNestedMap_whenUseHamcrestAssertThatWithCasting_thenTest() {
    Map<String, Object> innerMap = Map.of("city", "Chicago");
    Map<String, Object> outerMap = Map.of("address", innerMap);

    assertThat((Map<String, Object>)outerMap.get("address"), hasEntry("city", "Chicago"));
}
```

除了丑陋的类型转换之外，该测试更容易阅读和遵循。但是，我们在获取值之前错过了检查外部Map是否具有键address。

我们不应该尝试修复上述测试吗？让我们使用[hasKey()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#hasKey-K-)和[hasEntry()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#hasEntry-K-V-)来断言是否存在内部Map：

```java
@Test
void givenNestedMap_whenUseHamcrestAssertThat_thenTest() {
    Map<String, Object> innerMap = Map.of("city", "Chicago");
    Map<String, Map<String, Object>> outerMap = Map.of("address", innerMap);
    assertAll(
        () -> assertThat(outerMap, hasKey("address")),
        () -> assertThat(outerMap.get("address"), hasEntry("city", "Chicago"))
    );
}
```

有趣的是，**我们将Jupiter库中的assertAll()与Hamcrest库结合起来测试Map**。另外，为了删除类型转换，我们调整了变量outerMap的定义。

让我们看看用Hamcrest库做同样事情的另一种方法：

```java
@Test
void givenNestedMapOfStringAndObject_whenUseHamcrestAssertThat_thenTest() {
    Map<String, Object> innerMap = Map.of("city", "Chicago");
    Map<String, Map<String, Object>> outerMap = Map.of("address", innerMap);

    assertThat(outerMap, hasEntry(equalTo("address"), hasEntry("city", "Chicago")));
}
```

令人惊讶的是，我们可以嵌套hasEntry()方法，这可以在[equalTo()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#equalTo-T-)方法的帮助下实现。如果没有它，该方法将编译，但断言将失败。

### 3.2 使用自定义匹配器

到目前为止，我们尝试了现有方法来检查嵌套Map是否存在。让我们尝试通过扩展Hamcrest库中的[TypeSafeMatcher](https://www.baeldung.com/hamcrest-custom-matchers)类来创建自定义匹配器。

```java
public class NestedMapMatcher<K, V> extends TypeSafeMatcher<Map<K, Object>> {
    private K key;
    private V subMapValue;

    public NestedMapMatcher(K key, V subMapValue) {
        this.key = key;
        this.subMapValue = subMapValue;
    }

    @Override
    protected boolean matchesSafely(Map<K, Object> item) {
        if (item.containsKey(key)) {
            Object actualValue = item.get(key);
            return subMapValue.equals(actualValue);
        }
        return false;
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("a map containing key ").appendValue(key)
                .appendText(" with value ").appendValue(subMapValue);
    }

    public static <K, V> Matcher<V> hasNestedMapEntry(K key, V expectedValue) {
        return new NestedMapMatcher(key, expectedValue);
    }
}
```

我们必须重写matchSafely()方法来检查嵌套Map。

让我们看看如何使用它：

```java
@Test
void givenNestedMapOfStringAndObject_whenUseHamcrestAssertThatAndCustomMatcher_thenTest() {
    Map<String, Object> innerMap = Map.of
        (
            "city", "Chicago",
            "zip", "10005"
        );
    Map<String, Map<String, Object>> outerMap = Map.of("address", innerMap);

    assertThat(outerMap, hasNestedMapEntry("address", innerMap));
}
```

显然，方法assertThat()中检查嵌套Map的表达式要简化得多。我们只需调用hasNestedMapEntry()方法来检查innerMap。此外，它还会比较整个内部Map，这与之前仅检查一个条目不同。

**有趣的是，即使我们将外部Map定义为Map(String, Object)，自定义Matcher也能正常工作。我们也不需要进行任何类型转换**：

```java
@Test
void givenOuterMapOfStringAndObjectAndInnerMap_whenUseHamcrestAssertThatAndCustomMatcher_thenTest() {
    Map<String, Object> innerMap = Map.of
        (
            "city", "Chicago",
            "zip", "10005"
        );
    Map<String, Object> outerMap = Map.of("address", innerMap);

    assertThat(outerMap, hasNestedMapEntry("address", innerMap));
}
```

## 4. 总结

在本文中，我们讨论了测试内部嵌套Map中键值是否存在的不同方法，并探讨了Jupiter以及Hamcrest API。

Hamcrest提供了一些优秀的开箱即用方法来支持嵌套Map上的断言，这可以避免编写样板代码，因此有助于以更具声明性的方式编写测试。尽管如此，我们仍然必须编写一个自定义Matcher以使断言更加直观并支持在嵌套Map中断言多个条目。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-assertions)上找到。
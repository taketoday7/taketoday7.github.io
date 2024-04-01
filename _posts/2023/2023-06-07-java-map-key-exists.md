---
layout: post
title:  如何检查Map中是否存在键
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将研究检查Map中是否存在键的方法。

具体来说，我们将关注containsKey和get。

## 2. containsKey

如果我们看一下[Map#containsKey的JavaDoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#containsKey(java.lang.Object))：

>  如果此Map包含指定键的映射，则返回true

我们可以看到这种方法非常适合做我们想做的事情。

让我们创建一个非常简单的Map并使用containsKey验证其内容：

```java
@Test
public void whenKeyIsPresent_thenContainsKeyReturnsTrue() {
    Map<String, String> map = Collections.singletonMap("key", "value");
    
    assertTrue(map.containsKey("key"));
    assertFalse(map.containsKey("missing"));
}
```

**简而言之，containsKey告诉我们Map是否包含该键**。

## 3. get

现在，get有时也可以工作，但它会带来一些负担，具体取决于Map实现是否支持空值。

再次查看Map的JavaDoc，这次是[Map#put](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#put(K,V))，我们看到它只会抛出NullPointerException：

>   如果指定的键或值是null**并且此Map不允许null键或值**

由于Map的某些实现可以具有空值(如HashMap)，因此即使存在键，get也可能返回null值。

**因此，如果我们的目标是查看一个键是否有值，那么get将起作用**：

```java
@Test
public void whenKeyHasNullValue_thenGetStillWorks() {
    Map<String, String> map = Collections.singletonMap("nothing", null);

    assertTrue(map.containsKey("nothing"));
    assertNull(map.get("nothing"));
}
```

**但是，如果我们只是想检查键是否存在，那么我们应该坚持使用containsKey**。

## 4. 总结

在本文中，我们介绍了containsKey。我们还仔细研究了为什么使用get来验证键的存在性包含风险。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-1)上获得。
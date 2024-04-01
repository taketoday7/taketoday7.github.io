---
layout: post
title:  Java缺少return语句
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将研究Java开发过程中的一个常见错误。通常，初学者都会遇到这样的问题，即Java应用程序中缺少return语句的错误。

**缺少返回语句错误是编译时错误**。它在编译阶段抛出。现代IDE会即时检查此错误。因此，这种类型的错误往往很容易检测到。

主要原因是：

-   return语句只是被错误地省略了
-   该方法不返回任何值，但类型void未在方法签名中声明

## 2. 缺少Return语句

首先，我们将看几个例子。这些例子与错误省略的return语句有关。然后，我们将介绍方法签名中缺少void类型的示例。每个示例都展示了我们如何解决Java缺少返回语句错误。

### 2.1 省略Return声明

接下来，让我们定义一个简单的pow方法：

```java
public int pow(int number) {
    int pow = number * number;
}
```

编译前面的代码后，我们得到：

```java
java: missing return statement
```

为了解决这个问题，我们只需要在pow变量后面添加一个return语句：

```java
public int pow(int number) {
    int pow = number * number;
    return pow;
}
```

因此，如果我们调用方法pow，我们会得到预期的结果。

同样，但对于条件结构，会出现此错误：

```java
public static String checkNumber(int number) {
    if (number == 0) {
        return "It's equals to zero";
    }
    for (int i = 0; i < number; i++) {
        if (i > 100) {
            return "It's a big number";
        }
    }
}
```

上面的代码检查输入的数字。首先，将输入数字与0进行比较。如果条件为true，则返回一个字符串值。然后，如果数字大于0，我们会找到一个带有内部条件的[for循环](https://www.baeldung.com/java-for-loop)。如果“i”大于100，我们在for循环中的条件语句就满足了。但是，输入负数呢？是的你是对的。我们错过了默认的return语句。因此，如果我们编译代码，我们会再次得到java: missing return statement错误。

所以，为了修复它，我们只需要在方法的末尾放置一个默认的return语句：

```java
public static String checkNumber(int number) {
    if (number == 0) {
        return "It's equals to zero";
    }
    for (int i = 0; i < number; i++) {
        if (i > 100) {
            return "It's a big number";
        }
    }
    return "It's a negative number";
}
```

### 2.2 Lambda中缺少返回值

此外，当我们使用lambda时，可能会出现此错误。对于函数，检测此错误可能有点棘手。流中的[map](https://www.baeldung.com/java-8-streams-introduction#3-mapping)方法是发生此错误的常见位置。让我们检查一下我们的代码：

```java
public Map<String, Integer> createDictionary() {
    List<String> words = Arrays.asList("Hello", "World");
    Map<String, Integer> dictionary = new HashMap<>();
    words.stream().map(s -> {dictionary.put(s, 1);});
    return dictionary;
}
```

前面的代码看起来不错。有一个return语句。我们的返回数据类型等于方法签名。但是，流中map方法中的代码呢？map方法需要一个函数作为参数。在这种情况下，我们只将数据放入map方法内的字典中。结果，如果我们尝试编译这段代码，我们会再次遇到java: missing return statement错误。

接下来，要解决错误，我们只需将流中的map替换为[forEach](https://www.baeldung.com/foreach-java)方法即可：

```java
words.forEach(s -> {dictionary.put(s, 1);});
```

或者，直接从流中返回一个map：

```java
dictionary = words.stream().collect(Collectors.toMap(s -> s, s -> 1))
```

### 2.3 缺少方法签名

最后一种情况是我们错过了向方法签名添加返回类型。因此，当我们尝试编译我们的方法时，我们会得到一个错误。下面的代码示例向我们展示了这种行为：

```java
public pow(int number) {
    int pow = number * number;
    return pow;
}
```

我们忘记添加int作为返回类型。如果我们将它添加到我们的方法签名将修复此错误：

```java
public int pow(int number) {
    int pow = number * number;
    return pow;
}
```

## 3. 总结

在本文中，我们介绍了一些缺少return语句的示例。它如何出现在我们的代码中，以及我们如何修复它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-4)上获得。

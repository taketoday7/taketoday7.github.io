---
layout: post
title:  在Java中按字母顺序对List进行排序
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将探讨在Java中按字母顺序对列表进行[排序](https://www.baeldung.com/java-sorting)的各种方法。

首先，我们将从[Collections](https://www.baeldung.com/java-collections)类开始，然后使用[Comparator](https://www.baeldung.com/java-comparator-comparable)接口。我们还将使用List的API按字母顺序排序，然后是[Stream](https://www.baeldung.com/java-streams)，最后使用[TreeSet](https://www.baeldung.com/java-tree-set)。

此外，我们将扩展我们的示例以探索几种不同的场景，包括根据[特定区域设置](https://www.baeldung.com/java-8-localization)对列表进行排序、对[重音列表](https://www.baeldung.com/java-remove-accents-from-text)进行排序以及使用[RuleBasedCollator](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/text/RuleBasedCollator.html)来定义我们的自定义排序规则。

## 2. 使用Collections排序

首先，让我们看看如何使用Collections类对列表进行排序。

**Collections类提供了一个静态的重载方法sort**，它可以获取列表并按自然顺序对其进行排序，或者我们可以使用Comparator提供自定义排序逻辑。

### 2.1 按自然/词典顺序排序

首先，我们将定义输入列表：

```java
private static List<String> INPUT_NAMES = Arrays.asList("john", "mike", "usmon", "ken", "harry");
```

现在我们将首先使用Collections类按自然顺序(也称为[字典顺序](https://en.wikipedia.org/wiki/Lexicographic_order))对列表进行排序，：

```java
@Test
void givenListOfStrings_whenUsingCollections_thenListIsSorted() {
    Collections.sort(INPUT_NAMES);
    assertThat(INPUT_NAMES).isEqualTo(EXPECTED_NATURAL_ORDER);
}
```

其中EXPECTED_NATURAL_ORDER是：

```java
private static List<String> EXPECTED_NATURAL_ORDER = Arrays.asList("harry", "john", "ken", "mike", "usmon");
```

这里需要注意的一些要点是：

-   **该列表根据其元素的自然顺序按升序排序**
-   我们可以将任何Collection传递给其元素是Comparable的sort方法(实现Comparable接口)
-   在这里，**我们传递了一个String的List，它是一个Comparable类，因此排序有效**
-   **Collection类更改传递给sort的List对象的状态**。因此我们不需要返回列表

### 2.2 倒序排序

让我们看看如何按字母倒序对同一个列表进行排序。

让我们再次使用sort方法，但现在提供一个Comparator：

```java
Comparator<String> reverseComparator = (first, second) -> second.compareTo(first);
```

或者，我们可以简单地使用Comparator接口中的这个静态方法：

```java
Comparator<String> reverseComparator = Comparator.reverseOrder();
```

一旦我们有了反向比较器，我们就可以简单地将它传递给sort：

```java
@Test
void givenListOfStrings_whenUsingCollections_thenListIsSortedInReverse() {
    Comparator<String> reverseComparator = Comparator.reverseOrder();
    Collections.sort(INPUT_NAMES, reverseComparator); 
    assertThat(INPUT_NAMES).isEqualTo(EXPECTED_REVERSE_ORDER); 
}
```

其中EXPECTED_REVERSE_ORDER是：

```java
private static List<String> EXPECTED_REVERSE_ORDER = Arrays.asList("usmon", "mike", "ken", "john", "harry");
```

从中得出的一些相关总结是：

-   由于String类是final类，我们不能扩展重写Comparable接口的compareTo方法进行反向排序
-   我们可以使用Comparator接口来实现自定义的排序策略，即按字母降序排序
-   由于Comparator是一个函数式接口，我们可以使用Lambda表达式

## 3. 使用Comparator接口自定义排序

通常，我们必须对需要一些自定义排序逻辑的字符串列表进行排序。这是我们实现了Comparator接口并提供我们想要的排序标准的时候。

### 3.1 使用大写和小写字符串对列表进行排序的比较器

可以调用自定义排序的典型场景可以是字符串的混合列表，从大写和小写开始。

让我们设置并测试这个场景：

```java
@Test
void givenListOfStringsWithUpperAndLowerCaseMixed_whenCustomComparator_thenListIsSortedCorrectly() {
    List<String> movieNames = Arrays.asList("amazing SpiderMan", "Godzilla", "Sing", "Minions");
    List<String> naturalSortOrder = Arrays.asList("Godzilla", "Minions", "Sing", "amazing SpiderMan");
    List<String> comparatorSortOrder = Arrays.asList("amazing SpiderMan", "Godzilla", "Minions", "Sing");

    Collections.sort(movieNames);
    assertThat(movieNames).isEqualTo(naturalSortOrder);

    Collections.sort(movieNames, Comparator.comparing(s -> s.toLowerCase()));
    assertThat(movieNames).isEqualTo(comparatorSortOrder);
}
```

此处注意：

-   Comparator之前的排序结果将是不正确的：\["Godzilla", "Minions”, "Sing", "amazing SpiderMan"]
-   String::toLowerCase是键提取器，它从类型String中提取Comparable排序键并返回Comparator<String\>
-   使用自定义比较器排序后的正确结果将是：\["amazing SpiderMan", "Godzilla", "Minions", "Sing"]

### 3.2 用于对特殊字符进行排序的比较器

让我们考虑另一个列表示例，其中某些名称以特殊字符“@”开头。

我们希望它们排在列表的末尾，其余的应该按自然顺序排序：

```java
@Test
void givenListOfStringsIncludingSomeWithSpecialCharacter_whenCustomComparator_thenListIsSortedWithSpecialCharacterLast() {
    List<String> listWithSpecialCharacters = Arrays.asList("@laska", "blah", "jo", "@sk", "foo");

    List<String> sortedNaturalOrder = Arrays.asList("@laska", "@sk", "blah", "foo", "jo");
    List<String> sortedSpecialCharacterLast = Arrays.asList("blah", "foo", "jo", "@laska", "@sk");

    Collections.sort(listWithSpecialCharacters);
    assertThat(listWithSpecialCharacters).isEqualTo(sortedNaturalOrder);

    Comparator<String> specialSignComparator = Comparator.<String, Boolean>comparing(s -> s.startsWith("@"));
    Comparator<String> specialCharacterComparator = specialSignComparator.thenComparing(Comparator.naturalOrder());

    listWithSpecialCharacters.sort(specialCharacterComparator);
    assertThat(listWithSpecialCharacters).isEqualTo(sortedSpecialCharacterLast);
}
```

最后，一些关键点是：

-   不使用比较器的排序输出是：\["@laska", "blah", "jo", "@sk", "foo"]，这对我们的场景来说是不正确的
-   像以前一样，我们有一个键提取器，它从类型String中提取Comparable排序键并返回一个specialCharacterLastComparator
-   我们可以使用[thenComparing](https://www.baeldung.com/java-8-comparator-comparing)链接specialCharacterLastComparator进行二次排序，接收另一个比较器
-   Comparator.naturalOrder以自然顺序比较Comparable对象。
-   排序中比较器之后的最终输出是：\["blah", "foo", "jo", "@laska", "@sk"]

## 4. 使用Stream排序

现在，让我们使用[Java 8 Stream](https://www.baeldung.com/java-streams)对String列表进行排序。

### 4.1 按自然顺序排序

让我们先按自然顺序排序：

```java
@Test
void givenListOfStrings_whenSortWithStreams_thenListIsSortedInNaturalOrder() {
    List<String> sortedList = INPUT_NAMES.stream()
        .sorted()
        .collect(Collectors.toList());

    assertThat(sortedList).isEqualTo(EXPECTED_NATURAL_ORDER);
}
```

重要的是，这里的sorted方法返回一个按自然顺序排序的String流。

**此外，此Stream的元素是Comparable**。

### 4.2 倒序排序

接下来，让我们将一个Comparator传递给sorted，它定义了反向排序策略：

```java
@Test
void givenListOfStrings_whenSortWithStreamsUsingComparator_thenListIsSortedInReverseOrder() {
    List<String> sortedList = INPUT_NAMES.stream()
        .sorted((element1, element2) -> element2.compareTo(element1))
        .collect(Collectors.toList());
    assertThat(sortedList).isEqualTo(EXPECTED_REVERSE_ORDER);
}
```

注意，这里我们使用[Lambda](https://www.baeldung.com/java-8-sort-lambda)函数来定义Comparator。

或者，我们可以获取相同的Comparator：

```java
Comparator<String> reverseOrderComparator = Comparator.reverseOrder();
```

然后我们将简单地将这个reverseOrderComparator传递给sorted：

```java
List<String> sortedList = INPUT_NAMES.stream()
    .sorted(reverseOrder)
    .collect(Collectors.toList());
```

## 5. 使用TreeSet排序

**TreeSet使用Comparable接口按排序和升序存储对象**。

我们可以简单地将未排序的列表转换为TreeSet，然后将其作为列表收集回来：

```java
@Test
void givenNames_whenUsingTreeSet_thenListIsSorted() {
    SortedSet<String> sortedSet = new TreeSet<>(INPUT_NAMES);
    List<String> sortedList = new ArrayList<>(sortedSet);
    assertThat(sortedList).isEqualTo(EXPECTED_NATURAL_ORDER);
}
```

**这种方法的一个限制是我们的原始列表不应该有任何它想在排序列表中保留的重复值**。

## 6. 使用List的sort进行排序

我们还可以使用List的sort方法对列表进行排序：

```java
@Test
void givenListOfStrings_whenSortOnList_thenListIsSorted() {
    INPUT_NAMES.sort(Comparator.reverseOrder());
    assertThat(INPUT_NAMES).isEqualTo(EXPECTED_REVERSE_ORDER);
}
```

请注意，**sort方法根据指定的Comparator规定的顺序对该列表进行排序**。

## 7. 区域设置敏感列表排序

**根据语言环境，按字母顺序排序的结果可能会因语言规则而异**。

让我们以字符串列表为例：

```java
 List<String> accentedStrings = Arrays.asList("único", "árbol", "cosas", "fútbol");
```

让我们先对它们进行正常排序：

```java
Collections.sort(accentedStrings);
```

输出将是：\["cosas", "fútbol", "árbol", "único"]。

但是，我们希望使用特定的语言规则对它们进行排序。

**让我们创建一个具有特定语言环境的Collator实例**：

```java
Collator esCollator = Collator.getInstance(new Locale("es"));
```

注意，这里我们使用es语言环境创建了一个Collator实例。

**然后我们可以将此Collator作为Comparator传递，用于对List、Collection或使用Stream进行排序**：

```java
accentedStrings.sort((s1, s2) -> {
    return esCollator.compare(s1, s2);
});
```

排序的结果现在是：\[árbol, cosas, fútbol, único]。

最后，让我们一起展示一下：

```java
@Test
void givenListOfStringsWithAccent_whenSortWithTheCollator_thenListIsSorted() {
    List<String> accentedStrings = Arrays.asList("único", "árbol", "cosas", "fútbol");
    List<String> sortedNaturalOrder = Arrays.asList("cosas", "fútbol", "árbol", "único");
    List<String> sortedLocaleSensitive = Arrays.asList("árbol", "cosas", "fútbol", "único");

    Collections.sort(accentedStrings);
    assertThat(accentedStrings).isEqualTo(sortedNaturalOrder);

    Collator esCollator = Collator.getInstance(new Locale("es"));

    accentedStrings.sort((s1, s2) -> {
        return esCollator.compare(s1, s2);
    });

    assertThat(accentedStrings).isEqualTo(sortedLocaleSensitive);
}
```

## 8. 带重音字符的排序列表

带有重音符号或其他装饰的字符可以在Unicode中以几种不同的方式编码，从而以不同的方式排序。

我们可以在排序之前[规范化重音字符](https://www.baeldung.com/java-remove-accents-from-text)，也可以从字符中删除重音。

让我们研究一下这两种对重音列表进行排序的方法。

### 8.1 规范化重音列表并排序

为了准确地对这样的字符串列表进行排序，让我们使用java.text.Normalizer规范化重音字符

让我们从重音字符串列表开始：

```java
List<String> accentedStrings = Arrays.asList("único","árbol", "cosas", "fútbol");
```

接下来，让我们定义一个带有Normalize函数的Comparator并将其传递给我们的sort方法：

```java
Collections.sort(accentedStrings, (o1, o2) -> {
    o1 = Normalizer.normalize(o1, Normalizer.Form.NFD);
    o2 = Normalizer.normalize(o2, Normalizer.Form.NFD);
    return o1.compareTo(o2);
});
```

规范化后的排序列表将是：\[árbol, cosas, fútbol, único]

重要的是，我们在这里使用Normalizer.Form.NFD对重音字符使用Canonical分解的形式对数据进行规范化。还有一些其他形式也可用于规范化。

### 8.2 去除重音和排序

让我们在Comparator中使用StringUtils stripAccents方法并将其传递给sort方法：

```java
@Test
void givenListOfStrinsWithAccent_whenComparatorWithNormalizer_thenListIsNormalizedAndSorted() {
    List<String> accentedStrings = Arrays.asList("único", "árbol", "cosas", "fútbol");

    List<String> naturalOrderSorted = Arrays.asList("cosas", "fútbol", "árbol", "único");
    List<String> stripAccentSorted = Arrays.asList("árbol","cosas", "fútbol","único");

    Collections.sort(accentedStrings);
    assertThat(accentedStrings).isEqualTo(naturalOrderSorted);
    Collections.sort(accentedStrings, Comparator.comparing(input -> StringUtils.stripAccents(input)));
    assertThat(accentedStrings).isEqualTostripAccentSorted); 
}
```

## 9. 使用基于规则的Collator进行排序

我们还可以使用javatext.RuleBasedCollator定义自定义规则以按字母顺序排序。

在这里，让我们定义自定义排序规则并将其传递给[RuleBasedCollator](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/text/RuleBasedCollator.html)：

```java
@Test
void givenListofStrings_whenUsingRuleBasedCollator_then ListIsSortedUsingRuleBasedCollator() throws ParseException {
    List<String> movieNames = Arrays.asList("Godzilla","AmazingSpiderMan","Smurfs", "Minions");

    List<String> naturalOrderExpected = Arrays.asList("AmazingSpiderMan", "Godzilla", "Minions", "Smurfs");
    Collections.sort(movieNames);

    List<String> rulesBasedExpected = Arrays.asList("Smurfs", "Minions", "AmazingSpiderMan", "Godzilla");

    assertThat(movieNames).isEqualTo(naturalOrderExpected);

    String rule = "< s, S < m, M < a, A < g, G";

    RuleBasedCollator rulesCollator = new RuleBasedCollator(rule);
    movieNames.sort(rulesCollator);

    assertThat(movieNames).isEqualTo(rulesBasedExpected);
}
```

需要考虑的一些要点是：

-   **RuleBasedCollator将字符映射到排序键**
-   上面定义的规则是<relations\>形式，但规则中也有其他[形式](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/text/RuleBasedCollator.html)
-   **根据规则定义，其中字符s小于m，M小于a，A小于g**
-   如果规则的构建过程失败，将抛出格式异常
-   **RuleBasedCollator实现了Comparator接口**，因此可以传递给sort

## 10. 总结

在本文中，我们研究了在Java中按字母顺序对列表进行排序的各种技术。

首先，我们使用了Collections，然后我们用一些常见的例子解释了Comparator接口。

在Comparator之后，我们研究了List的sort方法，然后是TreeSet。

我们还探讨了如何在不同的语言环境中处理字符串列表的排序、重音列表的规范化和排序，以及使用带有自定义规则的RuleBasedCollator。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-4)上获得。
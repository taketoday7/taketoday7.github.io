---
layout: post
title:  使用Java Stream生成Map时处理重复键
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

使用Java [Stream](https://www.baeldung.com/java-8-streams)生成Map时，可能会遇到重复的键。这可能会在向Map添加值时导致问题，因为与键关联的先前值可能会被覆盖。

在本教程中，我们将讨论在使用Stream API生成Map时如何处理重复键。

## 2. 问题介绍

像往常一样，让我们通过示例来理解问题。假设我们有一个City类：

```java
class City {
    private String name;
    private String locatedIn;

    public City(String name, String locatedIn) {
        this.name = name;
        this.locatedIn = locatedIn;
    }

    // Omitted getter methods
    // Omitted the equals() and hashCode() methods
    // ...
}
```

如上面的类代码所示，City是一个具有两个字符串属性的[POJO](https://www.baeldung.com/java-pojo-class)类。一个是城市的名称，另一个提供了有关城市所在位置的更多信息。

此外，该类重写了[equals()和hashCode()](https://www.baeldung.com/java-equals-hashcode-contracts)方法。**这两种方法检查name和locatedIn属性**。为了简单起见，我们没有将方法的实现放在代码片段中。

接下来，让我们创建一个City实例列表：

```java
final List<City> CITY_INPUT = Arrays.asList(
    new City("New York City", "USA"),
    new City("Shanghai", "China"),
    new City("Hamburg", "Germany"),
    new City("Paris", "France"),
    new City("Paris", "Texas, USA"));
```

如上面的代码所示，我们[从数组中初始化一个List<City\>](https://www.baeldung.com/java-init-list-one-line#create-from-an-array)以与旧的Java版本兼容。CITY_INPUT列表包含五个城市。让我们关注一下我们添加到列表中的最后两个城市：

-   new City("Paris", "France") 
-   new City("Paris", "Texas", "USA") 

这两个城市具有相同的名称“Paris”。但是，它们不同的locatedIn值告诉我们这两个Paris实例是不同的城市。

现在，假设我们要从CITY_INPUT列表中使用城市名称作为键来生成一个Map。显然，这两个Paris城市将拥有相同的键。

接下来，让我们看看如何在使用Java Stream API生成Map时处理重复的键。

为简单起见，我们将使用单元测试断言来验证每个解决方案是否生成预期结果。

## 3. 使用groupingBy()方法生成Map<Key, List<Value\>>

处理Map中重复键的一种想法是**使键关联集合中的多个值**，例如Map<String, List<City\>>。一些流行的库提供了MultiMap类型，例如[Guava的Multimap](https://www.baeldung.com/guava-multimap)和[Apache Commons MultiValuedMap](https://www.baeldung.com/apache-commons-multi-valued-map)，以更轻松地处理多值Map。

在本教程中，我们将坚持使用标准Java API。因此，我们将使用[groupingBy()](https://www.baeldung.com/java-groupingby-collector)收集器来生成Map<String, List<City\>>结果，因为**groupingBy()方法可以按某些属性作为键对对象进行分组并将对象存储在Map实例中**：

```java
Map<String, List<City>> resultMap = CITY_INPUT.stream()
    .collect(groupingBy(City::getName));

assertEquals(4, resultMap.size());
assertEquals(Arrays.asList(new City("Paris", "France"), new City("Paris", "Texas, USA")),
    resultMap.get("Paris"));
```

正如我们在上面的测试中看到的，groupingBy()收集器生成的Map结果包含四个条目。此外，两个“Paris”城市实例被分组在键“Paris”下。

因此，使用多值Map的方法可以解决键重复问题。但是，此方法返回Map<String, List<City\>>。**如果我们需要Map<String, City\>作为返回类型，我们就不能再将具有重复键的对象组合在一个集合中**。

那么接下来，我们看看这种情况下如何处理重复的key。

## 4. 使用toMap()方法处理重复键

**Stream API提供了[toMap()](https://www.baeldung.com/java-collectors-tomap)收集器方法来将流收集到Map中**。

此外，**toMap()方法允许我们指定一个合并函数，该函数将用于组合与重复键关联的值**。

例如，我们可以使用一个简单的[lambda表达式](https://www.baeldung.com/java-8-lambda-expressions-tips)来忽略后面的City对象，如果它们的名称已经被收集的话：

```java
Map<String, City> resultMap1 = CITY_INPUT.stream()
    .collect(toMap(City::getName, Function.identity(), (first, second) -> first));

assertEquals(4, resultMap1.size());
assertEquals(new City("Paris", "France"), resultMap1.get("Paris"));
```

如上面的测试所示，由于输入列表中法国的Paris在美国得克萨斯州的Paris之前，因此生成的Map仅包含法国的城市巴黎。

或者，如果我们希望在出现重复键时始终覆盖Map中的现有条目，我们可以调整lambda表达式以返回第二个City对象：

```java
Map<String, City> resultMap2 = CITY_INPUT.stream()
    .collect(toMap(City::getName, Function.identity(), (first, second) -> second));

assertEquals(4, resultMap2.size());
assertEquals(new City("Paris", "Texas, USA"), resultMap2.get("Paris"));
```

如果我们运行测试，它就会通过。所以，这一次，键“Paris”分配为美国得克萨斯州的Paris。

当然，在实际项目中，除了简单的跳过或覆盖之外，我们可能还有更复杂的需求。我们始终可以在合并函数中实现所需的合并逻辑。

最后，让我们看另一个例子，将两个“Paris”城市的locatedIn属性合并为一个新的City实例，并将这个新合并的Paris实例放入结果Map中：

```java
Map<String, City> resultMap3 = CITY_INPUT.stream()
    .collect(toMap(City::getName, Function.identity(), (first, second) -> {
        String locations = first.getLocatedIn() + " and " + second.getLocatedIn();
        return new City(first.getName(), locations);
    }));

assertEquals(4, resultMap2.size());
assertEquals(new City("Paris", "France and Texas, USA"), resultMap3.get("Paris"));
```

## 5. 总结

在本文中，我们探讨了两种在使用Stream API生成Map结果时处理重复键的方法：

-   groupingBy()：以Map<Key, List<Value\>>类型创建一个Map结果
-   mapTo()：允许我们在合并函数中实现合并逻辑 

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-maps)上获得。

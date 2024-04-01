---
layout: post
title:  在Java中对HashMap进行排序
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将学习如何在Java中对HashMap进行排序。

更具体地说，我们将使用以下方法按键或值对HashMap条目进行排序：

-   TreeMap
-   ArrayList和Collections.sort()
-   TreeSet
-   使用Stream API
-   使用Guava库

## 2. 使用TreeMap

正如我们所知，**TreeMap中的键是使用它们的自然顺序排序的**。当我们想按键对键值对进行排序时，这是一个很好的解决方案。因此我们的想法是将我们的HashMap中的所有数据推送到[TreeMap](https://www.baeldung.com/java-treemap)中。

首先，让我们定义一个HashMap并使用一些数据初始化它：

```java
Map<String, Employee> map = new HashMap<>();

Employee employee1 = new Employee(1L, "Mher");
map.put(employee1.getName(), employee1);
Employee employee2 = new Employee(22L, "Annie");
map.put(employee2.getName(), employee2);
Employee employee3 = new Employee(8L, "John");
map.put(employee3.getName(), employee3);
Employee employee4 = new Employee(2L, "George");
map.put(employee4.getName(), employee4);
```

对于Employee类，**请注意我们实现了Comparable**：

```java
public class Employee implements Comparable<Employee> {

    private Long id;
    private String name;

    // constructor, getters, setters

    // override equals and hashCode
    @Override
    public int compareTo(Employee employee) {
        return (int)(this.id - employee.getId());
    }
}
```

接下来，我们使用其构造函数将条目存储在TreeMap中：

```java
TreeMap<String, Employee> sorted = new TreeMap<>(map);
```

我们也可以使用putAll方法来复制数据：

```java
TreeMap<String, Employee> sorted = new TreeMap<>();
sorted.putAll(map);
```

就是这样！为了确保我们的Map条目按键排序，让我们打印出来：

```text
Annie=Employee{id=22, name='Annie'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Mher=Employee{id=1, name='Mher'}
```

正如我们所见，键是按自然顺序排序的。

## 3. 使用ArrayList

当然，我们可以借助ArrayList对Map的条目进行排序。与前一种方法的主要区别在于**我们在这里不维护Map接口**。

### 3.1 按键排序

让我们将键集加载到ArrayList中：

```java
List<String> employeeByKey = new ArrayList<>(map.keySet());
Collections.sort(employeeByKey);
```

输出是：

```text
[Annie, George, John, Mher]
```

### 3.2 按值排序

现在，如果我们想按Employee对象的id字段对Map值进行排序怎么办？我们也可以为此使用ArrayList。

首先，让我们将值复制到列表中：

```java
List<Employee> employeeById = new ArrayList<>(map.values());
```

然后我们对其进行排序：

```java
Collections.sort(employeeById);
```

请记住，这是有效的，因为**Employee实现了Comparable接口**。否则，我们需要为Collections.sort调用定义一个手动比较器。

为了检查结果，我们打印employeeById：

```text
[Employee{id=1, name='Mher'}, 
Employee{id=2, name='George'}, 
Employee{id=8, name='John'}, 
Employee{id=22, name='Annie'}]
```

如我们所见，对象按其id字段排序。

## 4. 使用TreeSet

**如果我们不想在排序集合中接受重复值，TreeSet有一个很好的解决方案**。

首先，让我们在初始Map中添加一些重复的条目：

```java
Employee employee5 = new Employee(1L, "Mher");
map.put(employee5.getName(), employee5);
Employee employee6 = new Employee(22L, "Annie");
map.put(employee6.getName(), employee6);
```

### 4.1 按键排序

要按其键条目对Map进行排序：

```java
SortedSet<String> keySet = new TreeSet<>(map.keySet());
```

让我们打印keySet并查看输出：

```text
[Annie, George, John, Mher]
```

现在我们对Map键进行了排序，没有重复项。

### 4.2 按值排序

同样，对于Map值，转换代码如下所示：

```java
SortedSet<Employee> values = new TreeSet<>(map.values());
```

结果是：

```text
[Employee{id=1, name='Mher'}, 
Employee{id=2, name='George'}, 
Employee{id=8, name='John'}, 
Employee{id=22, name='Annie'}]
```

如我们所见，输出中没有重复项。**当我们覆盖equals和hashCode时，这适用于自定义对象**。

## 5. 使用Lambda和Stream

**从Java 8开始，我们可以使用Stream API和Lambda表达式对Map进行排序**。我们所需要做的就是通过Map的流管道调用sorted方法。

### 5.1 按键排序

要按键排序，我们使用comparingByKey比较器：

```java
map.entrySet()
    .stream()
    .sorted(Map.Entry.<String, Employee>comparingByKey())
    .forEach(System.out::println);
```

最后的forEach阶段打印出结果：

```text
Annie=Employee{id=22, name='Annie'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Mher=Employee{id=1, name='Mher'}
```

默认情况下，排序方式是升序。

### 5.2 按值排序

当然，我们也可以按Employee对象排序：

```java
map.entrySet()
    .stream()
    .sorted(Map.Entry.comparingByValue())
    .forEach(System.out::println);
```

如我们所见，上面的代码打印出一个按Employee对象的id字段排序的Map：

```text
Mher=Employee{id=1, name='Mher'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
```

此外，我们可以将结果收集到新Map中：

```java
Map<String, Employee> result = map.entrySet()
    .stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(
        Map.Entry::getKey, 
        Map.Entry::getValue, 
        (oldValue, newValue) -> oldValue, LinkedHashMap::new));
```

**请注意，我们将结果收集到LinkedHashMap中**。默认情况下，Collectors.toMap返回一个新的HashMap，但正如我们所知，**HashMap不保证迭代顺序**，而LinkedHashMap可以。

## 6. 使用Guava

最后，允许我们对HashMap进行排序的库是Guava。在开始之前，检查一下我们关于[Guava Map](https://www.baeldung.com/guava-maps)的文章会很有用。

首先，让我们声明一个[Ordering](https://www.baeldung.com/guava-ordering)，因为我们想按Employee的id字段对Map进行排序：

```java
Ordering naturalOrdering = Ordering.natural()
    .onResultOf(Functions.forMap(map, null));
```

现在我们只需要使用ImmutableSortedMap来说明结果：

```java
ImmutableSortedMap.copyOf(map, naturalOrdering);
```

再一次，输出是一个按id字段排序的Map：

```text
Mher=Employee{id=1, name='Mher'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
```

## 7. 总结

在本文中，我们回顾了多种按键或值对HashMap进行排序的方法。

当属性是自定义类时，我们还学习了如何通过实现Comparable来做到这一点。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。
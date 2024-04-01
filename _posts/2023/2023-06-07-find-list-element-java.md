---
layout: post
title:  如何使用Java在List中查找元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在列表中查找元素是我们作为开发人员遇到的一项非常常见的任务。

在这个快速教程中，我们将介绍使用Java执行此操作的不同方法。

## 2. 设置

首先让我们从定义Customer POJO开始：

```java
public class Customer {

    private int id;
    private String name;

    // getters/setters, custom hashcode/equals
}
```

然后是Customer的[ArrayList](https://www.baeldung.com/java-arraylist)：

```java
List<Customer> customers = new ArrayList<>();
customers.add(new Customer(1, "Jack"));
customers.add(new Customer(2, "James"));
customers.add(new Customer(3, "Kelly"));
```

请注意，我们已经在Customer类中重写了hashCode和equals。

根据我们当前的equals实现，具有相同id的两个Customer对象将被视为相等。

我们将在此过程中使用此Customer列表。

## 3. 使用Java API

Java本身提供了几种在列表中查找元素的方法：

-   **contains方法**
-   **index方法**
-   **for循环**
-   **Stream API**

### 3.1 contains()

List公开了一个名为contains的方法：

```java
boolean contains(Object element)
```

顾名思义，如果列表包含指定元素，则此方法返回true，否则返回false。

因此，当我们需要检查列表中是否存在特定元素时，我们可以：

```java
Customer james = new Customer(2, "James");
if (customers.contains(james)) {
    // ...
}
```

### 3.2 indexOf()

indexOf是另一种查找元素的有用方法：

```java
int indexOf(Object element)
```

此方法返回给定列表中指定元素第一次出现的索引，如果列表不包含该元素，则返回-1。

所以从逻辑上讲，如果此方法返回-1以外的任何值，我们就知道列表包含元素：

```java
if(customers.indexOf(james) != -1) {
    // ...
}
```

**使用此方法的主要优点是它可以告诉我们指定元素在给定列表中的位置**。

### 3.3 基本循环

现在如果我们想对元素进行基于字段的搜索怎么办？例如，假设我们要宣布彩票，我们需要将具有特定name的Customer声明为中奖者。

对于这种基于字段的搜索，我们可以转向迭代。

遍历列表的一种传统方法是使用[Java的循环](https://www.baeldung.com/java-loops)结构之一。在每次迭代中，我们将列表中的当前元素与我们要查找的元素进行比较，看它是否匹配：

```java
public Customer findUsingEnhancedForLoop(String name, List<Customer> customers) {
    for (Customer customer : customers) {
        if (customer.getName().equals(name)) {
            return customer;
        }
    }
    return null;
}
```

这里的name是指我们在给定的customers列表中搜索的名称。此方法返回列表中具有匹配名称的第一个Customer对象，如果不存在这样的Customer，则返回null。

### 3.4 使用Iterator循环

[Iterator](https://www.baeldung.com/java-iterator)是我们遍历列表元素的另一种方式。

我们可以简单地以前面的例子为例，对其进行一些调整：

```java
public Customer findUsingIterator(String name, List<Customer> customers) {
    Iterator<Customer> iterator = customers.iterator();
    while (iterator.hasNext()) {
        Customer customer = iterator.next();
        if (customer.getName().equals(name)) {
            return customer;
        }
    }
    return null;
}
```

因此，行为与以前相同。

### 3.5 Java 8 Stream API

从Java 8开始，我们还可以使用[Stream API](https://www.baeldung.com/java-8-streams)来查找列表中的元素。

要在给定列表中找到与特定条件匹配的元素，我们：

-   在列表上调用stream()
-    使用适当的Predicate调用filter()方法
-   调用findAny()构造，如果存在这样的元素，它将返回与包装在Optional中的filter谓词匹配的第一个元素
    

```java
Customer james = customers.stream()
    .filter(customer -> "James".equals(customer.getName()))
    .findAny()
    .orElse(null);
```

为了方便起见，如果Optional为空(empty)，我们默认为null，但这可能并不总是每个场景的最佳选择。

## 4. 第三方库

现在，虽然Stream API已经绰绰有余，**但如果我们停留在早期版本的Java上，我们应该怎么做？**

幸运的是，我们可以使用许多第三方库，例如Google Guava和Apache Commons。

### 4.1 Google Guava

Google Guava提供的功能类似于我们可以对流执行的操作：

```java
Customer james = Iterables.tryFind(customers, new Predicate<Customer>() {
    public boolean apply(Customer customer) {
        return "James".equals(customer.getName());
    }
}).orNull();
```

就像Stream API一样，我们可以选择返回默认值而不是null：

```java
Customer james = Iterables.tryFind(customers, new Predicate<Customer>() {
    public boolean apply(Customer customer) {
        return "James".equals(customer.getName());
    }
}).or(customers.get(0));
```

如果找不到匹配项，上面的代码将选择列表中的第一个元素。

**另外，不要忘记如果列表或谓词为null，Guava会抛出NullPointerException**。

### 4.2 Apache Commons

我们可以使用Apache Commons以几乎完全相同的方式找到一个元素：

```java
Customer james = IterableUtils.find(customers, new Predicate<Customer>() {
    public boolean evaluate(Customer customer) {
        return "James".equals(customer.getName());
    }
});
```

但是有几个重要的区别：

1.  如果我们传递一个null列表，Apache Commons只会返回null
2.  **它不像Guava的tryFind那样提供默认值功能**

## 5. 总结

在本文中，我们学习了在列表中查找元素的不同方法，从快速存在性检查开始，到基于字段的搜索。

我们还研究了第三方库Google Guava和Apache Commons作为Java 8 Stream API的替代品。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-1)上获得。
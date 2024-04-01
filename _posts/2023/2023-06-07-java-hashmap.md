---
layout: post
title:  Java HashMap指南
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

**在本文中，我们将了解如何在Java中使用HashMap，并了解它的内部工作原理**。

与HashMap非常相似的一个类是Hashtable。请参阅我们的其他几篇文章，以了解有关[java.util.Hashtable](https://www.baeldung.com/java-hash-table)类本身以及[HashMap和Hashtable之间的差异](https://www.baeldung.com/hashmap-hashtable-differences)的更多信息。

## 2. 基本用法

**让我们先看看HashMap是一个映射是什么意思。Map是键值映射，这意味着每个键都映射到一个值，我们可以使用键从Map中检索相应的值**。

有人可能会问为什么不简单地将值添加到列表中。为什么我们需要一个HashMap？原因很简单，就是性能。如果我们想在列表中查找特定元素，时间复杂度为O(n)，如果列表已排序，则为O(logn)，例如使用二分查找。

HashMap的优点是插入和检索值的时间复杂度平均为O(1)。稍后我们将研究如何实现这一目标，让我们先来看看如何使用HashMap。

### 2.1 设置

让我们创建一个我们将在整篇文章中使用的简单类：

```java
public class Product {

    private String name;
    private String description;
    private List<String> tags;

    // standard getters/setters/constructors

    public Product addTagsOfOtherProduct(Product product) {
        this.tags.addAll(product.getTags());
        return this;
    }
}
```

### 2.2 Put

我们现在可以使用String类型的键和Product类型的元素创建一个HashMap：

```java
Map<String, Product> productsByName = new HashMap<>();
```

并将产品添加到我们的HashMap中：

```java
Product eBike = new Product("E-Bike", "A bike with a battery");
Product roadBike = new Product("Road bike", "A bike for competition");
productsByName.put(eBike.getName(), eBike);
productsByName.put(roadBike.getName(), roadBike);
```

### 2.3 Get

我们可以通过其键从Map中检索一个值：

```java
Product nextPurchase = productsByName.get("E-Bike");
assertEquals("A bike with a battery", nextPurchase.getDescription());
```

如果我们尝试为Map中不存在的键查找值，我们将得到一个空值：

```java
Product nextPurchase = productsByName.get("Car");
assertNull(nextPurchase);
```

如果我们使用相同的键插入第二个值，我们只会得到该键的最后插入值：

```java
Product newEBike = new Product("E-Bike", "A bike with a better battery");
productsByName.put(newEBike.getName(), newEBike);
assertEquals("A bike with a better battery", productsByName.get("E-Bike").getDescription());
```

### 2.4 Null作为键

HashMap还允许我们将null作为键：

```java
Product defaultProduct = new Product("Chocolate", "At least buy chocolate");
productsByName.put(null, defaultProduct);

Product nextPurchase = productsByName.get(null);
assertEquals("At least buy chocolate", nextPurchase.getDescription());
```

### 2.5 具有相同键的值

此外，我们可以使用不同的键插入相同的对象两次：

```java
productsByName.put(defaultProduct.getName(), defaultProduct);
assertSame(productsByName.get(null), productsByName.get("Chocolate"));
```

### 2.6 删除值

我们可以从HashMap中删除键值映射：

```java
productsByName.remove("E-Bike");
assertNull(productsByName.get("E-Bike"));
```

### 2.7 检查Map中是否存在键或值

要检查Map中是否存在键，我们可以使用containsKey()方法：

```java
productsByName.containsKey("E-Bike");
```

或者，要检查Map中是否存在某个值，我们可以使用containsValue()方法：

```java
productsByName.containsValue(eBike);
```

在我们的示例中，这两个方法调用都将返回true。尽管它们看起来非常相似，但这两个方法调用在性能上存在重要差异。**检查键是否存在的复杂度为O(1)，而检查元素的复杂度为O(n)，因为有必要遍历Map中的所有元素**。

### 2.8 遍历HashMap

可以通过三种基本方法遍历HashMap中的所有键值对。

我们可以遍历所有键的集合：

```java
for(String key : productsByName.keySet()) {
    Product product = productsByName.get(key);
}
```

或者我们可以遍历所有条目的集合：

```java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

最后，我们可以遍历所有值：

```java
List<Product> products = new ArrayList<>(productsByName.values());
```

## 3. 键

**我们可以使用任何类作为HashMap中的键。但是，为了使Map正常工作，我们需要提供equals()和hashCode()的实现**。假设我们想要一个以产品为键，以价格为值的Map：

```java
HashMap<Product, Integer> priceByProduct = new HashMap<>();
priceByProduct.put(eBike, 900);
```

让我们实现equals()和hashCode()方法：

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    Product product = (Product) o;
    return Objects.equals(name, product.name) && Objects.equals(description, product.description);
}

@Override
public int hashCode() {
    return Objects.hash(name, description);
}
```

**请注意，hashCode()和equals()只需要为我们想用作Map键的类重写，而不需要重写仅用作Map中的值的类**。我们将在本文的第5节中了解为什么这是必要的。

## 4. Java 8的其他方法

Java 8向HashMap添加了几个函数式方法。在本节中，我们将研究其中的一些方法。

**对于每种方法，我们将查看两个示例**。第一个示例展示了如何使用新方法，第二个示例展示了如何在早期版本的Java中实现相同的功能。

由于这些方法非常简单，因此我们不会查看更详细的示例。

### 4.1 forEach()

forEach方法是迭代Map中所有元素的函数式方法：

```java
productsByName.forEach( (key, product) -> {
    System.out.println("Key: " + key + " Product:" + product.getDescription());
    //do something with the key and value
});
```

在Java 8之前：

```java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

我们的文章[Java 8 forEach指南](https://www.baeldung.com/foreach-java)更详细地介绍了forEach循环。

### 4.2 getOrDefault()

使用getOrDefault()方法，我们可以从Map中获取值，或者在给定键没有映射的情况下返回一个默认元素：

```java
Product chocolate = new Product("chocolate", "something sweet");
Product defaultProduct = productsByName.getOrDefault("horse carriage", chocolate); 
Product bike = productsByName.getOrDefault("E-Bike", chocolate);
```

在Java 8之前：

```java
Product bike2 = productsByName.containsKey("E-Bike") 
    ? productsByName.get("E-Bike") 
    : chocolate;
Product defaultProduct2 = productsByName.containsKey("horse carriage") 
    ? productsByName.get("horse carriage") 
    : chocolate;
```

### 4.3 putIfAbsent()

使用这种方法，我们可以添加一个新的Map，但前提是给定键还没有映射：

```java
productsByName.putIfAbsent("E-Bike", chocolate);
```

在Java 8之前：

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.put("E-Bike", chocolate);
}
```

我们的文章[使用Java 8合并两个Map](https://www.baeldung.com/java-merge-maps)深入介绍了这种方法。

### 4.4 merge()

使用[merge()](https://www.baeldung.com/java-merge-maps)，如果映射存在，我们可以修改给定键的值，否则添加新值：

```java
Product eBike2 = new Product("E-Bike", "A bike with a battery");
eBike2.getTags().add("sport");
productsByName.merge("E-Bike", eBike2, Product::addTagsOfOtherProduct);
```

在Java 8之前：

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.get("E-Bike").addTagsOfOtherProduct(eBike2);
} else {
    productsByName.put("E-Bike", eBike2);
}
```

### 4.5 compute()

使用compute()方法，我们可以计算给定键的值：

```java
productsByName.compute("E-Bike", (k,v) -> {
    if(v != null) {
        return v.addTagsOfOtherProduct(eBike2);
    } else {
        return eBike2;
    }
});
```

在Java 8之前：

```java
if(productsByName.containsKey("E-Bike")) {    
    productsByName.get("E-Bike").addTagsOfOtherProduct(eBike2); 
} else {
    productsByName.put("E-Bike", eBike2); 
}
```

值得注意的是方法merge()和compute()非常相似。compute()方法接收两个参数：键和用于重新映射的BiFunction。merge()接收三个参数：key，如果键不存在则添加到Map的默认值，以及用于重新映射的BiFunction。

## 5. HashMap内部结构

在本节中，我们将了解HashMap的内部工作原理以及使用HashMap而不是简单列表的好处。

正如我们所见，我们可以通过键从HashMap中检索元素。一种方法是使用列表，遍历所有元素，并在找到与键匹配的元素时返回。这种方法的时间和空间复杂度都是O(n)。

**使用HashMap，我们可以实现put和get操作的O(1)平均时间复杂度和O(n)的空间复杂度。让我们看看它是如何工作的**。

### 5.1 HashCode()和Equals()

**HashMap不是遍历其所有元素，而是尝试根据其键计算值的位置**。

天真的方法是拥有一个列表，该列表可以包含尽可能多的元素。例如，假设我们的键是一个小写字符。那么有一个大小为26的列表就足够了，如果我们想访问带有键'c'的元素，我们就会知道它是位置3的元素，我们可以直接检索它。

但是，如果我们有更大的键空间，这种方法就不会很有效。例如，假设我们的键是一个整数。在这种情况下，列表的大小必须为2147483647。在大多数情况下，我们的元素也会少得多，因此分配的内存的很大一部分将保持未使用状态。

**HashMap将元素存储在所谓的桶中，桶的数量称为容量**。

当我们在Map中放入一个值时，键的hashCode()方法用于确定存储该值的桶。

为了检索值，HashMap以相同的方式计算桶-使用hashCode()。然后它遍历在该桶中找到的对象并使用键的equals()方法找到精确匹配项。

### 5.2 键的不变性

**在大多数情况下，我们应该使用不可变键。或者至少，我们必须意识到使用可变键的后果**。

让我们看看当我们使用它在Map中存储值后我们的键发生变化时会发生什么。

对于这个例子，我们将创建MutableKey：

```java
public class MutableKey {
    private String name;

    // standard constructor, getter and setter

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        MutableKey that = (MutableKey) o;
        return Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

测试如下：

```java
MutableKey key = new MutableKey("initial");

Map<MutableKey, String> items = new HashMap<>();
items.put(key, "success");

key.setName("changed");

assertNull(items.get(key));
```

正如我们所看到的，一旦键发生变化，我们就无法再获取相应的值，而是返回null。**这是因为HashMap在错误的桶中搜索**。

如果我们没有很好地理解HashMap的内部工作原理，那么上面的测试用例可能会令人惊讶。

### 5.3 碰撞

**为了使其正常工作，相等的键必须具有相同的哈希，但是，不同的键可以具有相同的哈希**。如果两个不同的键有相同的哈希值，则属于它们的两个值将存储在同一个桶中。在桶内，值存储在列表中，并通过遍历所有元素进行检索。这样做的成本是O(n)。

从Java 8开始(参见[JEP 180](https://openjdk.java.net/jeps/180))，如果一个桶包含8个或更多值，则存储一个桶内的值的数据结构从列表更改为平衡树，如果在某个时候，桶中只剩下6个值。这将性能提高到O(logn)。

### 5.4 容量和负载系数

为避免拥有多个具有多个值的桶，如果75%(负载因子)的桶变为非空，则容量将翻倍。负载因子默认值为75%，默认初始容量为16，两者都可以在构造函数中设置。

### 5.5 put和get操作总结

让我们总结一下put和get操作的工作原理。

**当我们向Map中添加一个元素时**，HashMap会计算桶。如果桶中已经包含一个值，则将该值添加到属于该桶的列表(或树)中。如果负载因子变得大于Map的最大负载因子，则容量加倍。

**当我们想从Map中获取一个值时**，HashMap计算桶并从列表(或树)中获取具有相同键的值。

## 6.  总结

在本文中，我们了解了如何使用HashMap以及它在内部是如何工作的。与ArrayList一起，HashMap是Java中最常用的数据结构之一，因此很好地了解如何使用它以及它在幕后如何工作是非常方便的。我们的文章[Java HashMap底层](https://www.baeldung.com/java-hashmap-advanced)更详细地介绍了HashMap的内部结构。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。
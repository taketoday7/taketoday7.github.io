---
layout: post
title:  重写记录的hashCode()和equals()
category: java-new
copyright: java-new
excerpt: Java 17
---

## 1. 简介

Java 14引入了[记录](https://www.baeldung.com/java-record-keyword)的概念，作为传递不可变数据对象的一种简单且更好的方法。记录只有类所具有的最基本的方法，构造函数和getter、setter，因此是类的一种受限形式，类似于Java中的枚举。记录是一种纯数据载体，是一种用于传递未更改数据的类的形式。 

在本教程中，我们将讨论如何覆盖记录的默认hashCode()和equals()实现。

## 2. hashCode()和equals()方法

Java Object类定义了equals()和hashCode()方法。由于Java中的所有类都继承自Object类，因此它们也具有方法的默认[实现](https://www.baeldung.com/java-equals-hashcode-contracts)。

equals()方法旨在断言两个对象相等，默认实现意味着如果两个对象具有相同的标识，则它们是相等的。hashCode()方法返回一个基于当前类实例的整数值，它与相等性定义一起实现。

记录是Java中类的一种受限形式，带有其默认实现equals()、hashCode()和toString()。我们可以使用new关键字实例化记录。我们还可以比较一个记录的两个实例是否相等，就像我们对一个类所做的那样。

## 3. 记录的hashCode()和equals()的默认实现

R类型的任何记录都直接继承自java.lang.Record。默认情况下，Java提供了这两种方法的默认实现以供使用。 

默认的equals()实现遵循Object的equals()方法的约定。**此外，当我们通过复制另一个记录实例的所有属性来创建新记录实例时，这两个记录实例必须相等。这一点很重要，因为记录的概念是成为不被更改的数据的数据载体**。 

假设我们有一个Person类型的记录，我们创建了该记录的两个实例：

```java
public record Person(String firstName, String lastName, String SSN, String dateOfBirth) {}

Person rob = new Person("Robert", "Frost", "HDHDB223", "2000-01-02");
Person mike = new Person("Mike", "Adams", "ABJDJ2883", "2001-01-02");
```

**请注意，我们可以这样做，因为记录提供默认构造函数，考虑到开箱即用的所有记录字段**。此外，我们还有一个开箱即用的equals()可供使用，使我们能够比较Person的实例： 

```java
@Test
public void givenTwoRecords_whenDefaultEquals_thenCompareEquality() {
    Person robert = new Person("Robert", "Frost", "HDHDB223", "2000-01-02");
    Person mike = new Person("Mike", "Adams", "ABJDJ2883", "2001-01-02");
    assertNotEquals(robert, mike);
}
```

默认的hashCode()实现返回一个哈希码(整数)值，该哈希码值是通过组合记录属性的所有哈希值并遵循Object的哈希码协定得出的。从相同组件创建的两个记录也必须具有相同的哈希码值：

```java
@Test
public void givenTwoRecords_hashCodesShouldBeSame() {
    Person robert = new Person("Robert", "Frost", "HDHDB223", "2000-01-02");
    Person robertCopy = new Person("Robert", "Frost", "HDHDB223", "2000-01-02");
    assertEquals(robert.hashCode(), robertCopy.hashCode());
}
```

## 4. 自定义实现hashCode()和equals()

Java确实提供了[覆盖](https://www.baeldung.com/java-method-overload-override)equals()和hashCode()方法的默认实现的能力。例如，假设我们决定如果标题和发行年份相同，就足以断言两个Movie记录(具有多个属性)相等。

在这种情况下，我们可以选择覆盖默认实现，它认为每个属性都断言与我们自己的相等：

```java
record Movie(String name, Integer yearOfRelease, String distributor) {

    @Override
    public boolean equals(Object other) {
        if (this == other) {
            return true;
        }
        if (other == null) {
            return false;
        }
        Movie movie = (Movie) other;
        if (movie.name.equals(this.name) && movie.yearOfRelease.equals(this.yearOfRelease)) {
            return true;
        }
        return false;
    }
}
```

此实现有点类似于Java Class的equals()的任何其他自定义实现： 

-   如果两个实例相同，则它们必须相等
-   如果另一个实例为null，则相等性失败
-   将另一个对象转换为记录类型后，如果name和yearOfRelease属性与当前对象的相同，则它们必须相等

请注意，我们没有添加条件来检查另一个对象是否属于Movie类型，因为记录本质上是最终的。

但是，在检查两个Movie记录是否相等时，如果发生冲突，编译器将转向hashCode()来确定是否相等。因此，重写hashCode()方法的实现也很重要：

```java
@Override
public int hashCode() {
    return Objects.hash(name, yearOfRelease);
}
```

我们现在可以正确地测试两个Movie记录的相等性：

```java
@Test
public void givenTwoRecords_whenCustomImplementation_thenCompareEquality() {
    Movie movie1 = new Movie("The Batman", 2022, "WB");
    Movie movie2 = new Movie("The Batman", 2022, "Dreamworks");
    assertEquals(movie1, movie2);
    assertEquals(movie1.hashCode(), movie2.hashCode());
}
```

## 5. 何时覆盖默认实现

Java规范期望记录的equals()的任何自定义实现都满足记录的副本必须等于原始记录的规则。 但是，由于Java记录旨在传递不可变数据记录，因此使用Java提供的equals()和hashCode()的默认实现通常是安全的。

**但是，记录是[浅层](https://www.baeldung.com/cs/deep-vs-shallow-copy)不可变的**。这意味着如果记录具有可变属性(例如List)，它们不会自动不可变。在默认实现中，只有当两个实例的所有属性都相等时，两个记录才相等，而不管属性是原始类型还是引用类型。 

这对记录实例服务于它们的目的提出了挑战，最好覆盖默认的equals()和hashCode()实现。它也非常适合用作Map中的键。这意味着我们应该小心处理带有可能可变数据元素的记录。 

## 6. 总结

在本文中，我们了解了记录 如何为我们提供equals()和hashCode()方法的默认实现。我们还研究了如何使用自定义实现覆盖默认实现。

我们还研究了何时应该考虑覆盖默认行为。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-17)上获得。
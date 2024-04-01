---
layout: post
title:  Java类型系统面试题
category: interview
copyright: interview
excerpt: Java
---

## 1. 概述

Java类型系统是Java开发人员技术面试中经常提到的一个话题，本文回顾一些最常被问到但可能很难正确回答的重要问题。

## 2. 问题

### Q1. 描述Object类在类型层次结构中的位置。哪些类继承自Object，哪些不继承？数组是否继承自Object？可以将Lambda表达式分配给Object变量吗？

java.lang.Object位于Java类层次结构的顶部。所有类都从它继承-显式、隐式(当类定义中省略extends关键字时)或通过继承链传递。

但是，有8种原始类型不继承自Object，即boolean、byte、short、char、int、float、long和double。

根据Java语言规范，数组也是对象。它们可以分配给Object引用，并且可以对它们调用所有Object方法。

Lambda表达式不能直接分配给Object变量，因为Object不是函数接口。但是你可以将Lambda分配给函数接口变量，然后将其分配给Object变量(或者通过同时将其转换为函数接口来简单地将其分配给Object变量)。

### Q2. 解释原始类型和引用类型之间的区别

引用类型继承自顶级java.lang.Object类并且它们本身是可继承的(final类除外)。基本类型不继承，也不能被子类化。

原始类型的参数值总是通过堆栈传递，这意味着它们是按值传递的，而不是按引用传递的。这具有以下含义：对方法内原始参数值所做的更改不会传播到实际参数值。

原始类型通常使用底层硬件值类型进行存储。

例如，要存储一个int值，可以使用32位存储单元。引用类型引入了对象头的开销，它存在于引用类型的每个实例中。

对象头的大小相对于简单的数值大小可能非常重要。这就是为什么首先引入基本类型的原因-为了节省对象开销的空间。不利之处在于，并非Java中的所有东西在技术上都是对象-原始值不继承自Object类。

### Q3. 描述不同的原始类型和它们占用的内存量

Java有8种原始类型：

-   boolean：逻辑true/false值。布尔值的大小不是由JVM规范定义的，并且在不同的实现中可能会有所不同
-   byte：有符号的8位值
-   short：有符号的16位值
-   char：无符号16位值
-   int：有符号的32位值
-   long：有符号的64位值
-   float：符合IEEE 754标准的32位单精度浮点值
-   double：符合IEEE 754标准的64位双精度浮点值

### Q4. 抽象类和接口有什么区别？它们的用例是什么？

抽象类是在其定义中带有abstract修饰符的类，它不能被实例化，但它可以被子类化。接口是一种用interface关键字描述的类型。它也不能实例化，但可以被实现。

抽象类和接口之间的主要区别是一个类可以实现多个接口，但只能扩展一个抽象类。

抽象类通常用作某些类层次结构中的基类型，它表示从它继承的所有类的主要意图。

抽象类还可以实现所有子类所需的一些基本方法。例如，JDK中的大多数Map集合都继承自AbstractMap类，该类实现了子类使用的许多方法(例如equals方法)。

接口指定类同意的一些契约。一个实现的接口可能不仅表示该类的主要意图，而且还表示一些额外的契约。

例如，如果一个类实现了Comparable接口，这意味着可以比较该类的实例，无论该类的主要目的是什么。

### Q5. 接口类型的成员(字段和方法)有哪些限制？

接口可以声明字段，但它们被隐式声明为public、static和final，即使你没有指定这些修饰符也是如此。因此，你不能将接口字段显式定义为private。本质上，接口可能只有常量字段，没有实例字段。

接口的所有方法也是隐式公开的，它们也可以是(隐式)abstract或default。

### Q6. 内部类和静态嵌套类有什么区别？

简单地说-嵌套类基本上是在另一个类中定义的一个类。

嵌套类分为两类，具有非常不同的属性。内部类是在不先实例化封闭类的情况下无法实例化的类，即内部类的任何实例都隐式绑定到封闭类的某个实例。

下面是一个内部类的示例-你可以看到它能够以OuterClass1.this构造的形式访问对外部类实例的引用：

```java
public class OuterClass1 {

    public class InnerClass {

        public OuterClass1 getOuterInstance() {
            return OuterClass1.this;
        }
    }
}
```

要实例化这样的内部类，你需要有一个外部类的实例：

```java
OuterClass1 outerClass1 = new OuterClass1();
OuterClass1.InnerClass innerClass = outerClass1.new InnerClass();
```

静态嵌套类则完全不同。从语法上讲，它只是一个嵌套类，在其定义中带有static修饰符。

实际上，这意味着此类可以像任何其他类一样被实例化，而无需将其绑定到封闭类的任何实例：

```java
public class OuterClass2 {

    public static class StaticNestedClass {
    }
}
```

要实例化此类，你不需要外部类的实例：

```java
OuterClass2.StaticNestedClass staticNestedClass = new OuterClass2.StaticNestedClass();
```

### Q7. Java有多重继承吗？

Java不支持类的多重继承，这意味着一个类只能继承自一个超类。

但是你可以用一个类实现多个接口，并且这些接口的某些方法可以被定义为default并且有一个实现。这允许你以更安全的方式在单个类中混合不同的功能。

### Q8. 什么是包装类？什么是自动装箱？

对于Java中的8种原始类型中的每一种，都有一个包装类，可用于包装原始值并像对象一样使用它。相应地，这些类是Boolean、Byte、Short、Character、Integer、Float、Long和Double。这些包装器很有用，例如，当你需要将原始值放入仅接受引用对象的泛型集合中时。

```java
List<Integer> list = new ArrayList<>();
list.add(new Integer(5));
```

为了省去手动来回转换原始类型的麻烦，Java编译器提供了一种称为自动装箱/自动拆箱的自动转换。

```java
List<Integer> list = new ArrayList<>();
list.add(5);
int value = list.get(0);
```

### Q9. 描述equals()和==之间的区别

==运算符允许你比较两个对象的“相等性”(即两个变量都引用内存中的同一个对象)。重要的是要记住new关键字总是创建一个新对象，它不会传递与任何其他对象的==相等性，即使它们看起来具有相同的值：

```java
String string1 = new String("Hello");
String string2 = new String("Hello");

assertFalse(string1 == string2);
```

此外，==运算符允许比较原始值：

```java
int i1 = 5;
int i2 = 5;

assertTrue(i1 == i2);
```

equals()方法在java.lang.Object类中定义，因此可用于任何引用类型。默认情况下，它只是通过==运算符检查对象是否相同。但它通常在子类中被重写，以提供类的特定比较语义。

例如，对于String类，此方法检查字符串是否包含相同的字符：

```java
String string1 = new String("Hello");
String string2 = new String("Hello");

assertTrue(string1.equals(string2));
```

### Q10. 假设你有一个引用类类型实例的变量，如何检查一个对象是这个类的一个实例？

在这种情况下，你不能使用instanceof关键字，因为它仅在你以文字形式提供实际类名时才有效。

值得庆幸的是，Class类有一个方法isInstance允许检查一个对象是否是某个类的实例：

```java
Class<?> integerClass = new Integer(5).getClass();
assertTrue(integerClass.isInstance(new Integer(4)));
```

### Q11. 什么是匿名类？描述它的用例

匿名类是在需要其实例的同一位置定义的一次性类。此类在同一位置定义和实例化，因此不需要名称。

在Java 8之前，你通常会使用匿名类来定义单个方法接口的实现，例如Runnable。在Java 8中，使用Lambda代替单个抽象方法接口。但是匿名类仍然有些用例，例如，当你需要具有多个方法的接口实例或具有某些附加功能的类实例时。

以下是创建和填充Map的方法：

```java
Map<String, Integer> ages = new HashMap<String, Integer>(){{
    put("David", 30);
    put("John", 25);
    put("Mary", 29);
    put("Sophie", 22);
}};
```
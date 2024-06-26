---
layout: post
title:  使用默认方法的接口与抽象类
category: java-new
copyright: java-new
excerpt: Java 8
---

## 1. 简介

在Java接口中引入默认方法之后，接口和抽象类之间似乎不再有任何区别。但是，事实并非如此，它们之间还是存在一些根本差异。

在本教程中，我们将仔细研究接口和抽象类，看看它们有何不同。

## 2. 为什么使用默认方法？

[默认方法](https://www.baeldung.com/java-static-default-methods#why-default-methods-in-interfaces-are-needed)**的目的是在不破坏现有实现的情况下提供额外功能**。引入默认方法的最初动机是通过新的lambda函数为集合框架提供向后兼容性。

## 3. 接口默认方法与抽象类

让我们来看看主要的根本区别。

### 3.1 状态

**抽象类可以具有状态，它的方法可以访问实现的状态**。尽管接口中允许使用默认方法，但它们无法访问实现的状态。

**我们在默认方法中编写的任何逻辑都应该与接口的其他方法相关，这些方法将独立于对象的状态**。

假设我们创建了一个抽象类CircleClass，其中包含一个字符串color来表示CircleClass对象的状态：

```java
public abstract class CircleClass {

    private String color;
    private List<String> allowedColors = Arrays.asList("RED", "GREEN", "BLUE");

    public boolean isValid() {
        if (allowedColors.contains(getColor())) {
            return true;
        } else {
            return false;
        }
    }

    // standard getters and setters
}
```

在上面的抽象类中，我们有一个名为isValid()的非抽象方法，用于根据其状态验证CircleClass对象。**isValid()方法可以访问CircleClass对象的状态**，并根据allowedColors验证CircleClass的实例。由于这种行为，**我们可以根据对象的状态在抽象类方法中编写任何逻辑**。

让我们创建一个CircleClass的简单实现类：

```java
public class ChildCircleClass extends CircleClass {
}
```

现在，让我们创建一个实例并验证颜色：

```java
CircleClass redCircle = new ChildCircleClass();
redCircle.setColor("RED");
assertTrue(redCircle.isValid());
```

在这里，我们可以看到，当我们在CircleClass对象上设置一个有效颜色并调用isValid()方法时，在内部，isValid()方法可以访问CircleClass对象的状态，并检查该实例是否包含有效颜色。

让我们尝试使用带有默认方法的接口来做类似的事情：

```java
public interface CircleInterface {
    List<String> allowedColors = Arrays.asList("RED", "GREEN", "BLUE");

    String getColor();

    public default boolean isValid() {
        if (allowedColors.contains(getColor())) {
            return true;
        } else {
            return false;
        }
    }
}
```

众所周知，接口不能有状态，因此默认方法不能访问状态。

在这里，我们定义了getColor()方法来提供状态信息。子类将重写getColor()方法，以在运行时提供实例的状态：

```java
public class ChildCircleInterfaceImpl implements CircleInterface {
    private String color;

    @Override
    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}
```

让我们创建一个实例并验证颜色：

```java
ChidlCircleInterfaceImpl redCircleWithoutState = new ChidlCircleInterfaceImpl();
redCircleWithoutState.setColor("RED");
assertTrue(redCircleWithoutState.isValid());
```

正如我们在此处看到的，我们在子类中重写了getColor()方法，以便默认方法在运行时验证状态。

### 3.2 构造函数

**抽象类可以有构造函数，允许我们在创建时初始化状态**。当然，接口没有构造函数。

### 3.3 语法差异

此外，在语法方面几乎没有什么不同。**抽象类可以重写Object类的方法，但接口不能**。

**抽象类可以使用所有可能的访问修饰符声明实例变量**，并且可以在子类中访问它们。而接口只能有public、static和final变量，不能有任何实例变量。

此外，**抽象类可以声明实例和静态块**，而接口中不能有这些。

最后，**抽象类不能引用lambda表达式**，而接口可以有一个可以引用lambda表达式的抽象方法。

## 4. 总结

本文展示了抽象类和具有默认方法的接口之间的区别。我们还根据我们的方案了解了哪一个最适合。

只要有可能，**我们应该始终选择具有默认方法的接口，因为它允许我们继承一个类并实现一个接口**。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-8-2)上获得。
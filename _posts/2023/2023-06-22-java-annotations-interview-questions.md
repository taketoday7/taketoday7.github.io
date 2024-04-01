---
layout: post
title:  Java注解面试题(+答案)
category: interview
copyright: interview
excerpt: Java
---

## 1.概述

注解自Java 5以来就已经存在，如今，它们是无处不在的编程结构，可以丰富代码。

在这篇文章中，我们将回顾一些在技术面试中经常被问到的关于注解的问题。

## 2. 问题

### Q1. 什么是注解？他们的典型用例是什么？

注解是绑定到程序源代码元素的元数据，对它们操作的代码的运行没有影响。

它们的典型用例是：

-   **编译器信息**：通过注解，编译器可以检测错误或抑制警告
-   **编译时和部署时处理**：软件工具可以处理注解并生成代码、配置文件等
-   **运行时处理**：可以在运行时检查注解以自定义程序的行为

### Q2. 描述标准库中的一些有用注解

java.lang和java.lang.annotation包中有几个注解，比较常见的包括：

-   @Override：标记方法用于重写在超类中声明的元素，如果未能正确重写该方法，编译器将发出错误
-   @Deprecated：表示该元素已弃用，不应使用。如果程序使用标有此注解的方法、类或字段，编译器将发出警告
-   @SuppressWarnings：告诉编译器抑制特定的警告，在与泛型出现之前编写的遗留代码交互时最常使用
-   @FunctionalInterface：在Java 8中引入，表示类型声明是一个函数接口，其实现可以使用Lambda表达式提供

### Q3. 如何创建注解？

注解是接口的一种形式，其中关键字interface前面有@，其主体包含看起来与方法非常相似的注解类型元素声明：

```java
public @interface SimpleAnnotation {
    String value();

    int[] types();
}
```

定义注解后，你就可以开始在你的代码中使用它了：

```java
@SimpleAnnotation(value = "an element", types = 1)
public class Element {
    @SimpleAnnotation(value = "an attribute", types = { 1, 2 })
    public Element nextElement;
}
```

请注意，为数组元素提供多个值时，必须将它们括在方括号中。

可选地，可以提供默认值，只要它们是编译器的常量表达式：

```java
public @interface SimpleAnnotation {
    String value() default "This is an element";

    int[] types() default { 1, 2, 3 };
}
```

现在，你可以在不指明这些元素的情况下使用注解：

```java
@SimpleAnnotation
public class Element {
    // ...
}
```

或者只是指明其中的一些：

```java
@SimpleAnnotation(value = "an attribute")
public Element nextElement;
```

### Q4. 注解方法声明可以返回哪些对象类型？

返回类型必须是原始类型、String、Class、Enum或上述类型之一的数组。否则，编译器将抛出错误。

下面是成功遵循此原则的示例代码：

```java
enum Complexity {
    LOW, HIGH
}

public @interface ComplexAnnotation {
    Class<? extends Object> value();

    int[] types();

    Complexity complexity();
}
```

以下示例将无法编译，因为Object不是有效的返回类型：

```java
public @interface FailingAnnotation {
    Object complexity();
}
```

### Q5. 可以标注哪些程序元素？

注解可以应用于整个源代码中的多个位置。它们可以应用于类、构造函数和字段的声明：

```java
@SimpleAnnotation
public class Apply {
    @SimpleAnnotation
    private String aField;

    @SimpleAnnotation
    public Apply() {
        // ...
    }
}
```

方法及其参数：

```java
@SimpleAnnotation
public void aMethod(@SimpleAnnotation String param) {
    // ...
}
```

局部变量，包括循环和资源变量：

```java
@SimpleAnnotation
int i = 10;

for (@SimpleAnnotation int j = 0; j < i; j++) {
    // ...
}

try (@SimpleAnnotation FileWriter writer = getWriter()) {
    // ...
} catch (Exception ex) {
    // ...
}
```

其他注解类型：

```java
@SimpleAnnotation
public @interface ComplexAnnotation {
    // ...
}
```

甚至包，通过package-info.java文件：

```java
@PackageAnnotation
package cn.tuyucheng.taketoday.interview.annotations;
```

从Java 8开始，它们也可以应用于类型的使用。为此，注解必须指定值为ElementType.USE的@Target注解：

```java
@Target(ElementType.TYPE_USE)
public @interface SimpleAnnotation {
    // ...
}
```

现在，注解可以应用于类实例创建：

```java
new @SimpleAnnotation Apply();
```

类型转换：

```java
aString = (@SimpleAnnotation String) something;
```

implements子句：

```java
public class SimpleList<T> implements @SimpleAnnotation List<@SimpleAnnotation T> {
    // ...
}
```

和throws子句：

```java
void aMethod() throws @SimpleAnnotation Exception {
    // ...
}
```

### Q6. 有没有办法限制可以应用注解的元素？

是的，@Target注解可用于此目的。如果我们尝试在不适用的上下文中使用注解，编译器将发出错误。

下面是一个将@SimpleAnnotation注解的使用限制为仅用于字段声明的示例：

```java
@Target(ElementType.FIELD)
public @interface SimpleAnnotation {
    // ...
}
```

如果我们想让它适用于更多上下文，我们可以传递多个常量：

```java
@Target({ ElementType.FIELD, ElementType.METHOD, ElementType.PACKAGE })
```

我们甚至可以让一个注解不能用来标注任何东西。当声明的类型仅用作复杂注解中的成员类型时，这可能会派上用场：

```java
@Target({})
public @interface NoTargetAnnotation {
    // ...
}
```

### Q7. 什么是元注解？

元注解是应用于其他注解的注解。

所有未标有@Target或标有它但包含ANNOTATION_TYPE常量的注解也是元注解：

```java
@Target(ElementType.ANNOTATION_TYPE)
public @interface SimpleAnnotation {
    // ...
}
```

### Q8. 什么是重复注解？

重复注解可以多次应用于同一元素声明。

出于兼容性原因，由于Java 8引入了这个特性，所以重复的注解存储在一个容器注解中，由Java编译器自动生成。编译器要做到这一点，有两个步骤来声明它们。

首先，我们需要声明一个可重复的注解：

```java
@Repeatable(Schedules.class)
public @interface Schedule {
    String time() default "morning";
}
```

然后，我们使用强制value元素定义包含注解，其类型必须是可重复注解类型的数组：

```java
public @interface Schedules {
    Schedule[] value();
}
```

现在，我们可以多次使用@Schedule：

```java
@Schedule
@Schedule(time = "afternoon")
@Schedule(time = "night")
void scheduledMethod() {
    // ...
}
```

### Q9. 如何检索注解？这与其保留策略有何关系？

你可以使用反射API或注解处理器来检索注解。

@Retention注解及其RetentionPolicy参数会影响你检索它们的方式。RetentionPolicy枚举中包含三个常量：

-   RetentionPolicy.SOURCE：使编译器丢弃注解，但注解处理器可以读取它们
-   RetentionPolicy.CLASS：表示注解被添加到class文件中但不能通过反射访问
-   RetentionPolicy.RUNTIME：注解由编译器记录在class文件中，并在运行时由JVM保留，以便反射式读取

下面是创建可在运行时读取的注解的示例代码：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Description {
    String value();
}
```

现在，可以通过反射检索注解：

```java
Description description = AnnotatedClass.class.getAnnotation(Description.class);
System.out.println(description.value());
```

注解处理器可以与RetentionPolicy.SOURCE一起工作，这在[Java注解处理和创建构建器](https://www.baeldung.com/java-annotation-processing-builder)一文中进行了描述。

RetentionPolicy.CLASS在你编写Java字节码解析器时可用。

### Q10. 以下代码可以编译吗？

```java
@Target({ ElementType.FIELD, ElementType.TYPE, ElementType.FIELD })
public @interface TestAnnotation {
    int[] value() default {};
}
```

不能，如果相同的枚举常量在@Target注解中出现多次，则会出现编译时错误。

删除重复常量将使代码成功编译：

```java
@Target({ ElementType.FIELD, ElementType.TYPE})
```

### Q11. 是否可以扩展注解？

否。如[Java语言规范](http://docs.oracle.com/javase/specs/jls/se7/html/jls-9.html#jls-9.6)中所述，注解始终扩展java.lang.annotation.Annotation。

如果我们尝试在注解声明中使用extends子句，我们会得到一个编译错误：

```java
public @interface AnAnnotation extends OtherAnnotation {
    // Compilation error
}
```

## 3. 总结

在本文中，我们介绍了Java开发人员技术面试中出现的一些与注解有关的常见问题。这绝不是一个详尽的列表，只能被视为进一步研究的开始。
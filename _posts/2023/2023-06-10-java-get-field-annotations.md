---
layout: post
title:  使用反射获取字段的注解
category: java
copyright: java
excerpt: Java注解
---

## 1. 概述

在本教程中，我们将学习如何获取字段的注解。此外，我们将解释保留元注解的工作原理。之后，我们将展示返回字段注解的两种方法之间的区别。

## 2. 注解保留政策

首先，让我们看一下[Retention](https://www.baeldung.com/java-default-annotations#2-retention)注解。它定义了注解的生命周期。此元注解采用RetentionPolicy属性。也就是说，该属性定义了注解可见的生命周期：

-   RetentionPolicy.SOURCE：仅在源代码中可见
-   RetentionPolicy.CLASS：在编译时对编译器可见
-   RetentionPolicy.RUNTIME：对编译器和运行时可见

因此，只有RUNTIME保留策略允许我们以编程方式读取注解。

## 3. 使用反射获取字段的注解

现在，让我们创建一个带有注解字段的示例类。我们将定义三个注解，其中只有两个在运行时可见。

第一个注解在运行时可见：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface FirstAnnotation {
}
```

第二个具有相同的保留：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface SecondAnnotation {
}
```

最后，让我们创建第三个仅在源代码中可见的注解：

```java
@Retention(RetentionPolicy.SOURCE)
public @interface ThirdAnnotation {
}
```

现在，让我们定义一个带有字段classMember的类，并用我们的所有三个注解进行注解：

```java
public class ClassWithAnnotations {

    @FirstAnnotation
    @SecondAnnotation
    @ThirdAnnotation
    private String classMember;
}
```

之后，让我们在运行时检索所有可见的注解。我们将使用[Java反射](https://www.baeldung.com/java-reflection)，它允许我们检查字段的属性：

```java
@Test
public void whenCallingGetDeclaredAnnotations_thenOnlyRuntimeAnnotationsAreAvailable() throws NoSuchFieldException {
    Field classMemberField = ClassWithAnnotations.class.getDeclaredField("classMember");
    Annotation[] annotations = classMemberField.getDeclaredAnnotations();
    assertThat(annotations).hasSize(2);
}
```

结果，我们只检索到两个在运行时可用的注解。getDeclaredAnnotations方法返回一个长度为零的数组，以防字段上没有注解。

我们可以用相同的方式读取超类字段的注解：[检索超类的字段](https://www.baeldung.com/java-reflection-class-fields#inherited)并调用相同的getDeclaredAnnotations方法。

## 4. 检查字段是否标注了特定类型

现在让我们看看如何检查字段上是否存在特定注解。Field类有一个方法isAnnotationPresent，当元素上存在指定类型的注解时，[该方法](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Field.html)返回true。让我们在我们的classMember字段上测试它：

```java
@Test
public void whenCallingIsAnnotationPresent_thenOnlyRuntimeAnnotationsAreAvailable() throws NoSuchFieldException {
    Field classMemberField = ClassWithAnnotations.class.getDeclaredField("classMember");
    assertThat(classMemberField.isAnnotationPresent(FirstAnnotation.class)).isTrue();
    assertThat(classMemberField.isAnnotationPresent(SecondAnnotation.class)).isTrue();
    assertThat(classMemberField.isAnnotationPresent(ThirdAnnotation.class)).isFalse();
}
```

正如预期的那样，ThirdAnnotation不存在，因为它具有为Retention元注解指定的SOURCE保留策略。

## 5. 字段方法getAnnotations和getDeclaredAnnotations

现在让我们看一下Field类公开的两个方法，getAnnotations和getDeclaredAnnotations。根据Javadoc，[getDeclaredAnnotations](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AccessibleObject.html#getDeclaredAnnotations())方法返回直接出现在元素上的注解。另一方面，Javadoc对[getAnnotations](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AccessibleObject.html#getAnnotations())说它返回元素上存在的所有注解。

类中的字段在其定义之上包含注解。结果，根本不涉及注解的继承。所有注解必须与字段定义一起定义。因此，方法getAnnotations和getDeclaredAnnotations总是返回相同的结果。

让我们用一个简单的测试来展示它：

```java
@Test
public void whenCallingGetDeclaredAnnotationsOrGetAnnotations_thenSameAnnotationsAreReturned() throws NoSuchFieldException {
    Field classMemberField = ClassWithAnnotations.class.getDeclaredField("classMember");
    Annotation[] declaredAnnotations = classMemberField.getDeclaredAnnotations();
    Annotation[] annotations = classMemberField.getAnnotations();
    assertThat(declaredAnnotations).containsExactly(annotations);
}
```

而且，在Field类中，我们可以发现getAnnotations方法调用了getDeclaredAnnotations方法：

```java
@Override
public Annotation[] getAnnotations() {
    return getDeclaredAnnotations();
}
```

## 6. 总结

在这篇简短的文章中，我们解释了保留策略元注解在检索注解中的作用。然后我们展示了如何读取字段的注解。最后，我们证明了字段不存在注解继承。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-annotations)上获得。
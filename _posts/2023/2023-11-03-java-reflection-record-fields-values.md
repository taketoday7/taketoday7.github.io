---
layout: post
title: 通过反射获取所有记录字段及其值
category: java-new
copyright: java-new
excerpt: Java 17
---

## 1. 概述

自Java 14以来，[记录](https://www.baeldung.com/java-record-keyword)被引入来表示不可变数据。记录包含具有各种值的字段，有时，我们需要以编程方式提取所有这些字段及其相应的值。

在本教程中，我们将探讨如何使用Java的[反射API](https://www.baeldung.com/java-reflection)检索记录类中的所有字段及其值。

## 2. 问题介绍

一个例子可以快速说明问题，假设我们有一个Player记录：

```java
record Player(String name, int age, Long score) {}
```

如上面的代码所示，Player记录具有3个不同类型的字段：String、原始类型int和Long。此外，我们还创建了一个Player实例：

```java
Player ERIC = new Player("Eric", 28, 4242L);
```

现在，我们将找到Player记录中声明的所有字段，并获取此ERIC播放器实例，以编程方式提取其相应的值。

为简单起见，我们将利用单元测试断言来验证每种方法是否产生预期结果。

接下来，让我们深入了解一下。

## 3. 使用RecordComponent

我们已经提到过，记录类是在Java 14中引入的。与记录一起，java.lang.reflect包中出现了一个新成员：[RecordComponent](https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/lang/reflect/RecordComponent.html)，这是Java 14和15中的预览功能。但是，它在Java 16中“升级”为永久功能。**RecordComponent类提供有关记录类组件的信息**。

此外，**如果[Class](https://www.baeldung.com/java-getclass-vs-class)对象是Record实例，Class类还提供getRecordComponents()方法来返回记录类的所有组件。值得注意的是，返回数组中的组件与记录中声明的顺序相同**。 

接下来，我们看看如何使用RecordComponent和反射API来获取Player记录类中的所有字段以及ERIC实例的相应值：

```java
var fields = new ArrayList<Field>();
RecordComponent[] components = Player.class.getRecordComponents();
for (var comp : components) {
    try {
        Field field = ERIC.getClass()
            .getDeclaredField(comp.getName());
        field.setAccessible(true);
        fields.add(field);
    } catch (NoSuchFieldException e) {
        // for simplicity, error handling is skipped
    }
}
```

首先，我们创建了一个空字段列表来保存稍后提取的字段。我们知道**可以使用class.getDeclaredField(fieldName)方法检索Java类中[声明的字段](https://www.baeldung.com/java-reflection-class-fields#class)**。因此，fieldName就成为解决问题的关键。

RecordComponent携带了记录字段的各种信息，包括类型、名称等。也就是说，如果我们有Player的RecordComponent对象，我们就可以拥有它的Field对象。Player.class.getRecordComponents()以数组形式返回Player记录类中的所有组件。因此，我们可以从组件数组中按名称获取所有Field对象。

**由于我们想要稍后提取这些字段的值，因此在将每个字段添加到结果字段列表之前，需要在每个字段上设置setAccessible(true)**。

接下来，让我们验证一下从上述循环中获得的字段是否符合预期：

```java
assertEquals(3, fields.size());

var nameField = fields.get(0); 
var ageField = fields.get(1);
var scoreField = fields.get(2);
try {
    assertEquals("name", nameField.getName());
    assertEquals(String.class, nameField.getType());
    assertEquals("Eric", nameField.get(ERIC));

    assertEquals("age", ageField.getName());
    assertEquals(int.class, ageField.getType());
    assertEquals(28, ageField.get(ERIC));

    assertEquals("score", scoreField.getName());
    assertEquals(Long.class, scoreField.getType());
    assertEquals(4242L, scoreField.get(ERIC));
} catch (IllegalAccessException exception) {
    // for simplicity, error handling is skipped
}
```

正如断言代码所示，我们可以通过调用field.get(ERIC)来获取ERIC实例的值。另外，**调用此方法时，我们必须捕获IllegalAccessException[受检异常](https://www.baeldung.com/java-checked-unchecked-exceptions)**。

## 4. 使用Class.getDeclaredFields()

新的RecordComponent使我们能够轻松获取记录组件的属性。不过，不使用新的RecordComponent类就可以解决该问题。

**Java反射API提供了Class.getDeclaredFields()方法来获取类中所有声明的字段**。因此，我们可以使用该方法获取记录类的字段。

值得注意的是，我们不应该使用Class.getFields()方法来获取记录类的字段，这是因为getFields()只返回类中声明的公共字段。但是，**记录类中的所有字段都是私有的**。因此，如果我们在记录类上调用Class.getFields()，我们将不会得到任何字段：

```java
// record has no public fields
assertEquals(0, Player.class.getFields().length);
```

同样，在将每个字段添加到结果列表之前，我们对每个字段应用setAccessible(true)：

```java
var fields = new ArrayList<Field>();
for (var field : Player.class.getDeclaredFields()) {
    field.setAccessible(true);
    fields.add(field);
}
```

接下来，我们检查结果列表中的字段是否与Player类匹配，以及是否可以通过这些字段获取ERIC对象的期望值：

```java
assertEquals(3, fields.size());
var nameField = fields.get(0);
var ageField = fields.get(1);
var scoreField = fields.get(2);

try {
    assertEquals("name", nameField.getName());
    assertEquals(String.class, nameField.getType());
    assertEquals("Eric", nameField.get(ERIC));

    assertEquals("age", ageField.getName());
    assertEquals(int.class, ageField.getType());
    assertEquals(28, ageField.get(ERIC));
 
    assertEquals("score", scoreField.getName());
    assertEquals(Long.class, scoreField.getType());
    assertEquals(4242L, scoreField.get(ERIC));
} catch (IllegalAccessException ex) {
    // for simplicity, error handling is skipped
}
```

当我们运行测试时，它通过了。因此，这种方法也解决了这个问题。

## 5. 总结

在本文中，我们探讨了两种使用反射从记录类中提取字段的方法。

在第一个解决方案中，我们从新的RecordComponent类获取字段名称。然后，我们可以通过调用Class.getDeclaredField(Field_Name)来获取字段对象。

记录类也是一个Java类。因此，我们也可以通过Class.getDeclaredFields()方法获取记录类的所有字段。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-17)上获得。
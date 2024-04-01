---
layout: post
title:  Java序列化：readObject()与readResolve()
category: java
copyright: java
excerpt: Java序列化
---

## 1. 概述

在本教程中，我们将介绍如何在Java反序列化API中使用readObject()和readResolve()方法。此外，我们将研究这两种方法之间的区别。

## 2. 序列化

[Java序列化](https://www.baeldung.com/java-serialization)更深入地介绍了序列化和反序列化的工作原理。在本文中，我们将重点关注readResolve()和readObject()方法，这些方法在使用反序列化时经常会引发问题。

## 3. readObject()的使用

Java对象在序列化过程中被转换为字节流，以便保存在文件中或通过互联网传输。在反序列化期间，使用ObjectInputStream的readObject()方法将序列化的字节流转换回原始对象，该方法在内部调用defaultReadObject()进行默认反序列化。

**如果我们的类中存在readObject()方法，则ObjectInputStream的readObject()方法将使用我们类的readObject()方法从流中读取对象**。

例如，在某些情况下，我们可以在类中实现readObject()以特定方式反序列化任何字段。

在介绍用例之前，让我们检查一下在类中实现readObject()方法的语法：

```java
private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException;
```

现在，假设我们有一个包含两个字段的User类：

```java
public class User implements Serializable {

    private static final long serialVersionUID = 3659932210257138726L;
    private String userName;
    private String password;

    // standard setters, getters, constructor(s) and toString()
}
```

此外，我们不想以明文形式序列化密码，那么我们该怎么办？让我们看看Java的readObject()如何帮助我们。

### 3.1 添加writeObject()以在序列化期间进行自定义更改

首先，我们可以在序列化期间对对象的字段进行特定更改，例如在writeObject()方法中对密码进行编码。

因此，对于我们的User类，让我们实现writeObject()方法，并在序列化期间向password字段添加额外的字符串前缀：

```java
private void writeObject(ObjectOutputStream oos) throws IOException {
    this.password = "xyz" + password;
    oos.defaultWriteObject();
}
```

### 3.2 在没有readObject()实现的情况下进行测试

现在，让我们测试我们的User类，但不实现readObject()。在这种情况下，将调用ObjectInputStream类的readObject()：

```java
@Test
public void testDeserializeObj_withDefaultReadObject() throws ClassNotFoundException, IOException {
    // Serialization
    FileOutputStream fos = new FileOutputStream("user.ser");
    ObjectOutputStream oos = new ObjectOutputStream(fos);
    User acutalObject = new User("Sachin", "Kumar");
    oos.writeObject(acutalObject);

    // Deserialization
    User deserializedUser = null;
    FileInputStream fis = new FileInputStream("user.ser");
    ObjectInputStream ois = new ObjectInputStream(fis);
    deserializedUser = (User) ois.readObject();
    assertNotEquals(deserializedUser.hashCode(), acutalObject.hashCode());
    assertEquals(deserializedUser.getUserName(), "Sachin");
    assertEquals(deserializedUser.getPassword(), "xyzKumar");
}
```

在这里，我们可以看到密码是xyzKumar，因为我们的类中还没有任何可以检索原始字段并进行自定义更改的readObject()。

### 3.3 添加readObject()以在反序列化过程中进行自定义更改

接下来，我们可以在反序列化期间对对象的字段进行特定更改，例如在readObject()方法中解码密码。

让我们在User类中实现readObject()方法，并删除序列化期间添加到password字段的额外字符串前缀：

```java
private void readObject(ObjectInputStream ois) throws ClassNotFoundException, IOException {
    ois.defaultReadObject();
    this.password = password.substring(3);
}
```

### 3.4 使用readObject()实现进行测试

让我们再次测试我们的User类，只是这一次，我们有一个自定义的readObject()方法，该方法将在反序列化期间被调用：

```java
@Test
public void testDeserializeObj_withOverriddenReadObject() throws ClassNotFoundException, IOException {
    // Serialization
    FileOutputStream fos = new FileOutputStream("user.ser");
    ObjectOutputStream oos = new ObjectOutputStream(fos);
    User acutalObject = new User("Sachin", "Kumar");
    oos.writeObject(acutalObject);

    // Deserialization
    User deserializedUser = null;
    FileInputStream fis = new FileInputStream("user.ser");
    ObjectInputStream ois = new ObjectInputStream(fis);
    deserializedUser = (User) ois.readObject();
    assertNotEquals(deserializedUser.hashCode(), acutalObject.hashCode());
    assertEquals(deserializedUser.getUserName(), "Sachin");
    assertEquals(deserializedUser.getPassword(), "Kumar");
}
```

在这里，我们可以注意到一些事情。首先，对象不同，其次，调用了我们自定义的readObject()，并且password字段被正确转换。

## 4. readResolve()的使用

在Java反序列化中，**readResolve()方法用于将反序列化期间创建的对象替换为不同的对象**。当我们需要确保应用程序中仅存在特定类的单个实例时，或者当我们想要用内存中可能已存在的不同实例替换对象时，这非常有用。

让我们回顾一下在我们的类中添加readResolve()的语法：

```java
ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
```

**在readObject()示例中需要注意的一件事是对象的hashCode是不同的。这是因为，在反序列化期间，新对象是从流对象创建的**。

我们可能想要使用readResolve()的常见场景是创建单例实例时，我们可以使用readResolve()来确保反序列化的对象与单例实例的现有实例相同。

我们以创建单例对象为例：

```java
public class Singleton implements Serializable {

    private static final long serialVersionUID = 1L;
    private static Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### 4.1 在没有readResolve()实现的情况下进行测试

此时，我们还没有添加任何readResolve()方法。让我们测试一下我们的Singleton类：

```java
@Test
public void testSingletonObj_withNoReadResolve() throws ClassNotFoundException, IOException {
    // Serialization
    FileOutputStream fos = new FileOutputStream("singleton.ser");
    ObjectOutputStream oos = new ObjectOutputStream(fos);
    Singleton actualSingletonObject = Singleton.getInstance();
    oos.writeObject(actualSingletonObject);

    // Deserialization
    Singleton deserializedSingletonObject = null;
    FileInputStream fis = new FileInputStream("singleton.ser");
    ObjectInputStream ois = new ObjectInputStream(fis);
    deserializedSingletonObject = (Singleton) ois.readObject();
    assertNotEquals(actualSingletonObject.hashCode(), deserializedSingletonObject.hashCode());
}
```

在这里，我们可以看到两个对象是不同的，这违背了我们Singleton类的目标。

### 4.2 使用readResolve()实现进行测试

为了解决这个问题，我们在Singleton类中添加readResolve()方法：

```java
private Object readResolve() throws ObjectStreamException {
    return INSTANCE;
}
```

现在，让我们再次进行测试：

```java
@Test
public void testSingletonObj_withCustomReadResolve() throws ClassNotFoundException, IOException {
    // Serialization
    FileOutputStream fos = new FileOutputStream("singleton.ser");
    ObjectOutputStream oos = new ObjectOutputStream(fos);
    Singleton actualSingletonObject = Singleton.getInstance();
    oos.writeObject(actualSingletonObject);

    // Deserialization
    Singleton deserializedSingletonObject = null;
    FileInputStream fis = new FileInputStream("singleton.ser");
    ObjectInputStream ois = new ObjectInputStream(fis);
    deserializedSingletonObject = (Singleton) ois.readObject();
    assertEquals(actualSingletonObject.hashCode(), deserializedSingletonObject.hashCode());
}
```

在这里，我们可以看到两个对象具有相同的hashCode。

## 5. readObject()与readResolve()

让我们快速总结一下两者之间的差异：

| readResolve()                                       | readObject()                                         |
| ------------------------------------------------------ | -------------------------------------------------------- |
| 方法返回类型为Object                                 | 方法返回类型为void                                     |
| 无方法参数                                             | ObjectInputStream作为参数                              |
| 通常用于实现单例模式，其中反序列化后需要返回相同的对象 | 用于设置对象的未序列化的非transient字段的值，例如从其他字段派生的字段或动态初始化的字段|
| 抛出ClassNotFoundException、ObjectStreamException  | 抛出ClassNotFoundException、IOException              |
| 比readObject()更快，因为它不读取整个对象图          | 比readResolve()慢，因为它读取整个对象图                |

## 6. 总结

在本文中，我们介绍了Java Serialization API的readObject()和readResolve()方法。此外，我们还看到了两者之间的差异。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-serialization)上获得。
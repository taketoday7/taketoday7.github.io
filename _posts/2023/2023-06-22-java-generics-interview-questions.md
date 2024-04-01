---
layout: post
title:  Java泛型面试题(+答案)
category: interview
copyright: interview
excerpt: Java泛型
---

## 1. 简介

在本文中，我们将讨论一些示例Java泛型面试问题和答案。

泛型是Java中的一个核心概念，最早是在Java 5中引入的。正因为如此，几乎所有的Java代码库都会使用它们，几乎可以保证开发人员在某个时候会遇到它们。这就是为什么正确理解它们很重要，也是为什么它们很可能在面试过程中被问及的原因。

## 2. 问题

### Q1. 什么是泛型类型参数？

类型是类或接口的名称。顾名思义，**泛型类型参数是指类型可以用作类、方法或接口声明中的参数**。

让我们从一个简单的例子开始，一个不使用泛型的例子来演示这一点：

```java
public interface Consumer {
    void consume(String parameter);
}
```

本例中consume()方法的方法参数类型为String，它未参数化且不可配置。

现在让我们将String类型替换为我们将称为T的泛型类型。按照惯例，它的命名方式如下：

```java
public interface Consumer<T> {
    void consume(T parameter);
}
```

当我们实现Consumer时，我们可以提供我们希望它作为参数使用的类型。这是一个泛型类型参数：

```java
public class IntegerConsumer implements Consumer<Integer> {
    public void consume(Integer parameter);
}
```

在这种情况下，现在我们可以使用整数。我们可以将这种类型换成我们需要的任何类型。

### Q2. 使用泛型类型有哪些优势？

使用泛型的一个优点是避免强制转换并提供类型安全，这在处理集合时特别有用。让我们演示一下：

```java
List list = new ArrayList();
list.add("foo");
Object o = list.get(0);
String foo = (String) o;
```

在我们的示例中，编译器不知道列表中的元素类型。这意味着唯一可以保证的是它是一个对象。因此，当我们检索元素时，我们得到的是一个Object。作为代码的作者，我们知道它是一个String，但我们必须将我们的对象强制转换为一个String以显式解决问题，这会产生很多噪音和样板代码。

接下来，如果我们开始考虑手动错误的余地，转换问题就会变得更糟。如果我们不小心在列表中保存一个整数怎么办？

```java
list.add(1)
Object o = list.get(0);
String foo = (String) o;
```

在这种情况下，我们会在运行时得到[ClassCastException](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ClassCastException.html)，因为Integer不能转换为String。

现在，让我们尝试重复一遍，这次使用泛型：

```java
List<String> list = new ArrayList<>();
list.add("foo");
String o = list.get(0);    // No cast
Integer foo = list.get(0); // Compilation error
```

如我们所见，**通过使用泛型，我们进行了编译类型检查，可以防止ClassCastExceptions并消除强制转换的需要**。

**另一个优点是避免代码重复**。如果没有泛型，我们必须要为不同的类型复制并粘贴相同的代码。使用泛型，我们不必这样做。我们甚至可以实现适用于泛型类型的算法。

### Q3. 什么是类型擦除？

重要的是要认识到泛型类型信息只对编译器可用，对JVM不可用。换句话说，**类型擦除意味着泛型类型信息在运行时对JVM不可用，只能在编译时使用**。

主要实现选择背后的原因很简单-保持与旧版本Java的向后兼容性。当泛型代码被编译成字节码时，就好像泛型类型从未存在过一样。这意味着编译将：

1.  用Object替换泛型类型
2.  用第一个绑定类替换有界类型(在后面的问题中详细介绍)
3.  检索泛型对象时插入等效的强制转换

了解类型擦除非常重要，否则，开发人员可能会感到困惑，并认为他们能够在运行时获取类型：

```java
public foo(Consumer<T> consumer) {
   Type type = consumer.getGenericTypeParameter();
}
```

上面的例子是一个伪代码，相当于没有类型擦除的情况，但不幸的是，这是不可能的。同样，**泛型类型信息在运行时不可用**。

### Q4. 如果实例化对象时省略了泛型，代码还能编译吗？

由于泛型在Java 5之前不存在，所以根本不使用它们是可能的。例如，泛型被改造为大多数标准Java类，例如集合。如果我们从问题一中查看我们的列表，那么我们将看到我们已经有一个省略泛型类型的示例：

```java
List list = new ArrayList();
```

尽管能够编译，但编译器仍有可能发出警告。这是因为我们失去了使用泛型获得的额外编译时检查。

要记住的一点是，**虽然向后兼容性和类型擦除使得省略泛型成为可能，但这是一种不好的做法**。

### Q5. 泛型方法与泛型类型有何不同？

**泛型方法是将类型参数引入到方法中，位于该方法的作用域内**。让我们来看一个例子：

```java
public static <T> T returnType(T argument) { 
    return argument; 
}
```

我们使用了静态方法，但如果愿意，也可以使用非静态方法。通过利用类型推断(在下一个问题中介绍)，我们可以像调用任何普通方法一样调用它，而无需在调用时指定任何类型参数。

### Q6. 什么是类型推断？

类型推断是指编译器可以查看方法参数的类型来推断泛型类型。例如，如果我们将T传递给返回T的方法，那么编译器可以计算出返回类型。让我们通过调用上一个问题中的泛型方法来尝试一下：

```java
Integer inferredInteger = returnType(1);
String inferredString = returnType("String");
```

正如我们所见，不需要强制转换，也不需要传入任何泛型类型参数。参数类型仅推断返回类型。

### Q7. 什么是有界类型参数？

到目前为止，我们所有的问题都涵盖了无界的泛型类型参数。这意味着我们的泛型类型参数可以是我们想要的任何类型。

**当我们使用有界参数时，我们限制了可以用作泛型类型参数的类型**。

例如，假设我们要强制泛型类型始终是Animal的子类：

```java
public abstract class Cage<T extends Animal> {
    abstract void addAnimal(T animal);
}
```

通过使用extends，我们强制T成为Animal的子类。然后我们可以有一笼猫：

```java
Cage<Cat> catCage;
```

但是我们不能使用Object，因为Object不是Animal的子类：

```java
Cage<Object> objectCage; // Compilation error
```

这样做的一个好处是编译器可以使用Animal的所有方法。我们知道我们的类型扩展了它，所以我们可以编写一个对任何Animal进行操作的通用算法。这意味着我们不必为不同的Animal子类复制我们的方法：

```java
public void firstAnimalJump() {
    T animal = animals.get(0);
    animal.jump();
}
```

### Q8. 是否可以声明多个有界类型参数？

可以为泛型类型声明多个边界。在前面的示例中，我们指定了一个边界，但如果愿意，我们也可以指定更多：

```java
public abstract class Cage<T extends Animal & Comparable>
```

在我们的示例中，Animal是一个类，Comparable是一个接口。现在，我们的类型必须遵守这两个上限。如果我们的类是Animal的子类但没有实现Comparable，那么代码将无法编译。**还值得记住的是，如果其中一个上限是类，则它必须是第一个参数**。

### Q9. 什么是通配符类型？

**通配符类型表示未知类型**，用问号表示：

```java
public static void consumeListOfWildcardType(List<?> list)
```

在这里，我们指定了一个可以是任何类型的列表。我们可以将任何内容的列表传递给此方法。

### Q10. 什么是上界通配符？

**上界通配符是指通配符类型继承自具体类型**，这在处理集合和继承时特别有用。

让我们尝试使用一个将存储Animal的Farm类来演示这一点，首先不使用通配符类型：

```java
public class Farm {
    private List<Animal> animals;

    public void addAnimals(Collection<Animal> newAnimals) {
        animals.addAll(newAnimals);
    }
}
```

如果我们有Animal的多个子类，例如Cat和Dog，我们可能会错误地假设我们可以将它们全部添加到Farm中：

```java
farm.addAnimals(cats); // Compilation error
farm.addAnimals(dogs); // Compilation error
```

这是因为编译器需要一个具体类型Animal的集合，而不是它的子类。

现在，让我们在addAnimals方法中引入一个上界通配符：

```java
public void addAnimals(Collection<? extends Animal> newAnimals)
```

现在，如果我们再试一次，我们的代码就会编译通过。这是因为我们现在告诉编译器接受任何Animal子类型的集合。

### Q11. 什么是无界通配符？

**无界通配符是没有上限或下限的通配符，可以表示任何类型**。

同样重要的是要知道通配符类型不是对象的同义词。这是因为通配符可以是任何类型，而对象类型具体是一个对象(并且不能是对象的子类)。让我们用一个例子来演示这一点：

```java
List<?> wildcardList = new ArrayList<String>(); 
List<Object> objectList = new ArrayList<String>(); // Compilation error
```

同样，第二行无法编译的原因是需要一个Object列表，而不是String列表。第一行编译成功是因为任何未知类型的列表都是可接受的。

### Q12. 什么是下界通配符？

下界通配符是指我们不提供上限，而是使用super关键字提供下界。换句话说，**下界通配符意味着我们强制该类型成为有界类型的超类**。让我们用一个例子来尝试一下：

```java
public static void addDogs(List<? super Animal> list) {
   list.add(new Dog("tom"))
}
```

通过使用super，我们可以在对象列表上调用addDogs：

```java
ArrayList<Object> objects = new ArrayList<>();
addDogs(objects);
```

这是有道理的，因为Object是Animal的超类。如果我们不使用下界通配符，代码将无法编译，因为Object列表不是Animal列表。

如果我们考虑一下，我们将无法将Dog添加到任何Animal子类的列表中，例如Cat，甚至Dog。只有Animal超类可以。例如，这不会编译：

```java
ArrayList<Cat> objects = new ArrayList<>();
addDogs(objects);
```

### Q13. 什么时候会选择使用上界类型与下界类型？

处理集合时，在上界或下界通配符之间进行选择的通用规则是PECS。PECS代表**生产者extends，消费者super**。

这可以通过使用一些标准的Java接口和类来轻松演示。

Producer extends只是意味着如果你正在创建一个泛型类型的生产者，那么使用extends关键字。让我们尝试将此原则应用于集合，看看它为什么有意义：

```java
public static void makeLotsOfNoise(List<? extends Animal> animals) {
    animals.forEach(Animal::makeNoise);   
}
```

在这里，我们想对我们集合中的每个Animal调用makeNoise()，这意味着我们的集合是一个生产者，因为我们所做的只是让它返回Animal供我们进行操作。如果我们去掉extends，我们将无法传入Cat、Dog或任何其他Animal子类的列表。通过应用生产者扩展原则，我们拥有最大的灵活性。

Consumer super与Producer extends相反。这意味着如果我们处理的是消耗元素的东西，那么我们应该使用super关键字。我们可以通过重复之前的示例来证明这一点：

```java
public static void addCats(List<? super Animal> animals) {
    animals.add(new Cat());   
}
```

我们只是添加到我们的animals列表中，所以我们的animals列表是消费者，这就是我们使用super关键字的原因。这意味着我们可以传入Animal的任何超类的列表，但不能传入子类。例如，如果我们尝试传入Dog或Cat的列表，那么代码将无法编译。

最后要考虑的是，如果集合既是消费者又是生产者，该怎么办。这方面的一个例子可能是添加和删除元素的集合。在这种情况下，应使用无界通配符。

### Q14. 在运行时是否存在泛型类型信息可用的情况？

在一种情况下，泛型类型在运行时可用。这是当泛型类型是类签名的一部分时，如下所示：

```java
public class CatCage implements Cage<Cat>
```

通过使用反射，我们可以得到这个类型参数：

```java
(Class<T>) ((ParameterizedType) getClass()
    .getGenericSuperclass()).getActualTypeArguments()[0];
```

这段代码有些脆弱。例如，它取决于在直接超类上定义的类型参数。但是，它表明JVM确实具有此类型信息。
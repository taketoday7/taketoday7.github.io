---
layout: post
title:  在Java中从字符串编译和执行代码
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

##  1. 概述

在本教程中，我们将学习如何将包含Java源代码的String转换为已编译的类并执行它。在运行时编译代码有许多潜在的应用：

-   生成的代码：来自运行前不可用或经常更改的信息的动态代码
-   热插拔：无需循环我们的应用程序即可替换代码
-   代码存储/注入：将应用程序逻辑存储在数据库中，以便临时检索和执行。请注意，自定义类可以在不使用时卸载。

尽管编译类的方法有多种，但这里，我们将重点关注JavaCompiler API。

## 2. 工具与策略

javax.tools包包含编译String所需的大部分抽象。让我们看看其中的一些，以及我们将遵循的一般流程：

![](/assets/images/2023/javajvm/javastringcompileexecutecode01.png)

1.  首先，我们将代码传递给JavaCompiler API。
2.  接下来，FileManager提取JavaCompiler的源代码。
3.  然后，JavaCompiler编译它并返回字节码。
4.  最后，自定义的ClassLoader将类加载到内存中。

我们如何以String格式生成源代码的确切方式不是本教程的重点。目前，我们将使用一个简单的硬编码文本值：

```java
final static String sourceCode = "package cn.tuyucheng.taketoday.inmemorycompilation;\n" 
    + "public class TestClass {\n" 
    + "@Override\n" 
    + "    public void runCode() {\n" 
    + "        System.out.println(\"code is running...\");\n" 
    + "    }\n" 
    + "}\n";
```

## 3. 表示我们的代码(源和编译)

我们清单上的第一项是**以FileManager熟悉的格式表示我们的代码**。

Java源文件和class文件的顶级抽象是FileObject。虽然没有提供适合我们需求的完整实现，但我们可以利用部分实现SimpleJavaFileObject并仅覆盖我们关心的方法。

### 3.1 源代码

对于我们的源代码，**我们必须定义FileManager应该如何读取它**，这意味着覆盖getCharContent()。此方法需要一个CharSequence，由于我们的代码已经包含在String中，我们可以简单地按原样返回它：

```java
public class JavaSourceFromString extends SimpleJavaFileObject {

    private String sourceCode;

    public JavaSourceFromString(String name, String sourceCode) {
        super(URI.create("string:///" + name.replace('.', '/') + Kind.SOURCE.extension),
                Kind.SOURCE);
        this.sourceCode = requireNonNull(sourceCode, "sourceCode must not be null");
    }

    @Override
    public CharSequence getCharContent(boolean ignoreEncodingErrors) {
        return sourceCode;
    }
}
```

### 3.2 编译代码

对于我们的编译代码，我们做完全相反的事情。**我们需要定义FileManager应该如何写入我们的对象**，这仅仅意味着覆盖openOutputStream()并提供[OutputStream](https://www.baeldung.com/java-outputstream)的实现。

我们将我们的代码存储在ByteArrayOutputStream中，并创建一个方便的方法用于稍后在类加载期间提取字节：

```java
public class JavaClassAsBytes extends SimpleJavaFileObject {

    protected ByteArrayOutputStream bos = new ByteArrayOutputStream();

    public JavaClassAsBytes(String name, Kind kind) {
        super(URI.create("string:///" + name.replace('.', '/') + kind.extension), kind);
    }

    public byte[] getBytes() {
        return bos.toByteArray();
    }

    @Override
    public OutputStream openOutputStream() {
        return bos;
    }
}
```

### 3.3 顶层接口

虽然不是绝对必要的，但在使用内存编译为我们编译的类创建顶级接口时，它会很有帮助。这个额外步骤有两个主要好处：

1.  我们知道从ClassLoader期待什么类型的对象，因此我们可以更安全/更容易地进行强制转换。
2.  我们可以在类加载器之间保持对象相等。**如果从不同类加载器加载的类中创建，则完全相同的对象可能会出现相等性问题**。由同一个ClassLoader加载的共享接口弥补了这一差距。

许多预定义的[函数接口](https://www.baeldung.com/java-8-functional-interfaces)都适合这种编码模式，例如Function、Runnable和Callable。但是，对于本指南，我们将创建自己的：

```java
public interface InMemoryClass {
    void runCode();
}
```

现在，我们只需要返回并稍微调整我们的源代码来实现我们的新接口：

```java
static String sourceCode = "package cn.tuyucheng.taketoday.inmemorycompilation;\n" 
    + "public class TestClass implements InMemoryClass {\n" 
    + "@Override\n" 
    + "    public void runCode() {\n" 
    + "        System.out.println(\"code is running...\");\n" 
    + "    }\n" 
    + "}\n";
```

## 4. 管理我们的内存代码

现在我们已经为JavaCompiler API获得了正确格式的代码，我们需要一个可以对其进行操作的FileManager。标准的FileManager不能满足我们的目的，并且与JavaCompiler API中的大多数其他抽象一样，没有可供我们使用的默认实现。

幸运的是，tools包确实包含ForwardingJavaFileManager，它只是将所有方法调用转发给底层的FileManager。**我们可以通过扩展ForwardingJavaFileManager并仅覆盖我们想要自己处理的行为来利用此行为，类似于我们对SimpleJavaFileObject所做的**。

首先，我们需要覆盖getJavaFileForOutput()。JavaCompiler将在我们的FileManager上调用此方法以获得已编译字节码的JavaFileObject，我们需要为它提供新的自定义类JavaClassAsBytes的实例：

```java
public class InMemoryFileManager extends ForwardingJavaFileManager<JavaFileManager> {

    // standard constructor

    @Override
    public JavaFileObject getJavaFileForOutput(Location location, String className, Kind kind, FileObject sibling) {
        return new JavaClassAsBytes(className, kind);
    }
}
```

我们还需要在某个地方存储编译后的类，以便稍后可以通过我们的自定义ClassLoader检索它们。我们将把这些类插入到一个Map中，并提供一个方便的方法来访问它：

```java
public class InMemoryFileManager extends ForwardingJavaFileManager<JavaFileManager> {

    private Map<String, JavaClassAsBytes> compiledClasses;

    public InMemoryFileManager(StandardJavaFileManager standardManager) {
        super(standardManager);
        this.compiledClasses = new Hashtable<>();
    }

    @Override
    public JavaFileObject getJavaFileForOutput(Location location, String className, Kind kind, FileObject sibling) {
        JavaClassAsBytes classAsBytes = new JavaClassAsBytes(className, kind);
        compiledClasses.put(className, classAsBytes);

        return classAsBytes;
    }

    public Map<String, JavaClassAsBytes> getBytesMap() {
        return compiledClasses;
    }
}
```

## 5. 加载我们的内存代码

最后一步是创建一些东西，以便在编译后加载我们的类。我们将为我们的InMemoryFileManager构建一个补充的ClassLoader。

[类加载](https://www.baeldung.com/java-classloaders)本身就是一个相当深入的主题，它超出了本文的范围。简而言之，**我们要将自定义ClassLoader挂接到现有委托层次结构的底部，并使用它直接从FileManager加载类**：

![](/assets/images/2023/javajvm/javastringcompileexecutecode02.png)

首先，我们需要创建一个扩展ClassLoader的自定义类。我们将稍微修改构造函数以接收我们的InMemoryFileManager作为参数，这将允许我们的ClassLoader稍后在管理器中执行查找：

```java
public class InMemoryClassLoader extends ClassLoader {

    private InMemoryFileManager manager;

    public InMemoryClassLoader(ClassLoader parent, InMemoryFileManager manager) {
        super(parent);
        this.manager = requireNonNull(manager, "manager must not be null");
    }
}
```

接下来，我们需要覆盖ClassLoader的findClass()方法来定义在哪里查找已编译的类。对我们来说幸运的是，这只是检查存储在InMemoryFileManager中的Map：

```java
@Override
protected Class<?> findClass(String name) throws ClassNotFoundException {
    Map<String, JavaClassAsBytes> compiledClasses = manager.getBytesMap();

    if (compiledClasses.containsKey(name)) {
        byte[] bytes = compiledClasses.get(name).getBytes();
        return defineClass(name, bytes, 0, bytes.length);
    } else {
        throw new ClassNotFoundException();
    }
}
```

我们应该注意，**如果找不到该类，我们将抛出ClassNotFoundException**。由于我们处于层次结构的底部，如果在这里还没有找到它，也就不会在任何地方找到它。

现在我们已经完成了InMemoryClassLoader，我们需要返回并对我们的InMemoryFileManager进行一些小修改以完成它们的双向关系。首先，我们将创建一个ClassLoader成员变量并修改构造函数以接收我们的新InMemoryClassLoader：

```java
private ClassLoader loader; 

public InMemoryFileManager(StandardJavaFileManager standardManager) {
    super(standardManager);
    this.compiledClasses = new Hashtable<>();
    this.loader = new InMemoryClassLoader(this.getClass().getClassLoader(), this);
}
```

接下来，我们需要覆盖getClassLoader()以返回我们新的InMemoryClassLoader实例：

```java
@Override
public ClassLoader getClassLoader(Location location) {
    return loader;
}
```

**现在，如果我们愿意，我们可以将相同的FileManager和ClassLoader一起重新用于多个内存中编译**。

## 6. 整合一切

剩下唯一要做的就是把我们所有不同的部分放在一起，让我们看看如何通过一个简单的单元测试来做到这一点：

```java
@Test
public void whenStringIsCompiled_ThenCodeShouldExecute() throws ClassNotFoundException, InstantiationException, IllegalAccessException {
    JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
    DiagnosticCollector<JavaFileObject> diagnostics = new DiagnosticCollector<>();
    InMemoryFileManager manager = new InMemoryFileManager(compiler.getStandardFileManager(null, null, null));

    List<JavaFileObject> sourceFiles = Collections.singletonList(new JavaSourceFromString(qualifiedClassName, sourceCode));

    JavaCompiler.CompilationTask task = compiler.getTask(null, manager, diagnostics, null, null, sourceFiles);

    boolean result = task.call();

    if (result) {
        diagnostics.getDiagnostics().forEach(d -> LOGGER.error(String.valueOf(d)));
    } else {
        ClassLoader classLoader = manager.getClassLoader(null);
        Class<?> clazz = classLoader.loadClass(qualifiedClassName);
        InMemoryClass instanceOfClass = (InMemoryClass) clazz.newInstance();

        Assertions.assertInstanceOf(InMemoryClass.class, instanceOfClass);

        instanceOfClass.runCode();
    }
}
```

当我们执行测试时，观察控制台输出：

```text
code is running...
```

**可以看到我们String源码中的方法已经执行成功了**。

## 7. 总结

在本文中，我们学习了如何将包含Java源代码的String转换为已编译的类，然后执行它。

作为一般警告，我们应该注意在使用类加载器时要格外小心。Class和ClassLoader之间的双向关系使得自定义类加载容易出现[内存泄漏](https://www.baeldung.com/java-memory-leaks)。在使用第三方库时尤其如此，它们可能会在幕后保留类引用。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-3)上获得。
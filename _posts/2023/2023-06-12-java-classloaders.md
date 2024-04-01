---
layout: post
title:  Java中的类加载器
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 类加载器简介

**类加载器负责在运行时将Java类动态加载到JVM(Java虚拟机)中，它们也是JRE(Java运行时环境)的一部分**。因此，由于类加载器的存在，JVM无需了解底层文件或文件系统即可运行Java程序。

此外，这些Java类不会一次性全部加载到内存中，而是在应用程序需要它们时才加载。这就是类加载器发挥作用的地方，他们负责将类加载到内存中。

在本教程中，我们将讨论不同类型的内置类加载器及其工作原理。然后我们将介绍我们自己的自定义实现。

## 2. 内置类加载器的类型

让我们从学习如何使用各种类加载器加载不同的类开始：

```java
public void printClassLoaders() throws ClassNotFoundException {
    System.out.println("Classloader of this class:" + PrintClassLoader.class.getClassLoader());

    System.out.println("Classloader of DriverManager:" m+ DriverManager.class.getClassLoader());

    System.out.println("Classloader of ArrayList:" + ArrayList.class.getClassLoader());
}
```

执行时，上面的方法打印：

```plaintext
Classloader of this class:jdk.internal.loader.ClassLoaders$AppClassLoader@73d16e93
Classloader of DriverManager:jdk.internal.loader.ClassLoaders$PlatformClassLoader@1fb700ee
Classloader of ArrayList:null
```

正如我们所看到的，这里有三个不同的类加载器：application、extension和bootstrap(显示为null)。

应用程序类加载器加载包含示例方法的类，**应用程序或系统类加载器在类路径中加载我们自己的文件**。

接下来，扩展类加载器加载DriverManager类。**扩展类加载器加载的类是标准核心Java类的扩展**。

最后，引导类加载器加载ArrayList类。**引导或原始类加载器是所有其他类加载器的父级**。

但是，我们可以看到对于ArrayList，它在输出中显示为null。**这是因为引导类加载器是用本机代码而不是Java编写的，所以它不会显示为Java类**。因此，引导类加载器的行为在JVM之间会有所不同。

现在，让我们更详细地讨论这些类加载器中的每一个。

### 2.1 引导类加载器

Java类由java.lang.ClassLoader的实例加载。但是，类加载器本身就是类。所以问题是，谁来加载java.lang.ClassLoader本身？

这就是引导或原始类加载器发挥作用的地方。

它主要负责加载JDK内部类，通常是rt.jar和位于$JAVA_HOME/jre/lib目录下的其他核心库。此外，**引导类加载器充当所有其他ClassLoader实例的父级**。

**这个引导类加载器是核心JVM的一部分，并且是用本机代码编写的**，如上例中所指出的。不同的平台可能对这个特定的类加载器有不同的实现。

### 2.2 扩展类加载器

**扩展类加载器是引导类加载器的子类，负责加载标准核心Java类的扩展，以便平台上运行的所有应用程序都可以使用它们**。

扩展类加载器从JDK扩展目录(通常是$JAVA_HOME/lib/ext目录)加载，或java.ext.dirs系统属性中提到的任何其他目录。

### 2.3 系统类加载器

另一方面，系统或应用程序类加载器负责将所有应用程序级类加载到JVM中。**它加载在类路径环境变量、-classpath或-cp命令行选项中找到的文件**。它也是扩展类加载器的子类。

## 3. 类加载器如何工作？

类加载器是Java运行时环境的一部分。当JVM请求一个类时，类加载器尝试定位该类并使用完全限定的类名将类定义加载到运行时中。

**java.lang.ClassLoader.loadClass()方法负责将类定义加载到运行时，它尝试根据完全限定名称加载类**。

如果该类尚未加载，它将请求委托给父类加载器。此过程以递归方式发生。

最终，如果父类加载器找不到该类，则子类将调用java.net.URLClassLoader.findClass()方法在文件系统本身中查找类。

如果最后一个子类加载器也无法加载该类，则会抛出[java.lang.NoClassDefFoundError或java.lang.ClassNotFoundException](https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror)。

让我们看一个抛出ClassNotFoundException时的输出示例：

```text
java.lang.ClassNotFoundException: cn.tuyucheng.taketoday.classloader.SampleClassLoader    
    at java.net.URLClassLoader.findClass(URLClassLoader.java:381)    
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)    
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)    
    at java.lang.Class.forName0(Native Method)    
    at java.lang.Class.forName(Class.java:348)
```

如果我们从调用java.lang.Class.forName()开始查看事件序列，我们可以看到它首先尝试通过父类加载器加载类，然后java.net.URLClassLoader.findClass()来查找类本身。

当它仍然找不到该类时，它会抛出ClassNotFoundException。

现在让我们研究类加载器的三个重要特性。

### 3.1 委托模型

类加载器遵循委托模型，**在请求查找类或资源时，ClassLoader实例会将类或资源的搜索委托给父类加载器**。

假设我们有一个将应用程序类加载到JVM中的请求，系统类加载器首先将该类的加载委托给它的父扩展类加载器，而后者又将其委托给引导类加载器。

只有引导和扩展类加载器加载类不成功时，系统类加载器才会尝试加载类本身。

### 3.2 唯一类

**作为委托模型的结果，很容易确保唯一类**，因为我们总是尝试向上委托。

如果父类加载器无法找到该类，则当前实例才会尝试自己查找该类。

### 3.3 可见性

此外，**子类加载器对其父类加载器加载的类可见**。

例如，系统类加载器加载的类对扩展和引导类加载器加载的类具有可见性，但反之则不然。

为了说明这一点，如果类A由应用程序类加载器加载，而类B由扩展类加载器加载，那么对于应用程序类加载器加载的其他类而言，A类和B类都是可见的。

但是，B类是扩展类加载器加载的其他类唯一可见的类。

## 4. 自定义类加载器

对于大多数文件已经在文件系统中的情况，内置类加载器就足够了。

但是，在我们需要从本地硬盘驱动器或网络加载类的场景中，我们可能需要使用自定义类加载器。

在本节中，我们将介绍自定义类加载器的一些其他用例并演示如何创建一个。

### 4.1 自定义类加载器用例

自定义类加载器不仅有助于在运行时加载类，一些用例可能包括：

1.  帮助修改现有的字节码，例如织入代理。
2.  动态创建适合用户需求的类，例如在JDBC中，不同驱动程序实现之间的切换是通过动态类加载完成的。
3.  实现类版本控制机制，同时为具有相同名称和包的类加载不同的字节码。这可以通过URL类加载器(通过URL加载jar)或自定义类加载器来完成。

下面是自定义类装入器可能派上用场的更具体示例。

**例如，浏览器使用自定义类加载器从网站加载可执行内容**。浏览器可以使用不同的类加载器从不同的网页加载小程序。用于运行小程序的小程序查看器包含一个类加载器，它访问远程服务器上的网站而不是在本地文件系统中查找。

然后它通过HTTP加载原始字节码文件，并将它们转换为JVM中的类。**即使这些小程序具有相同的名称，如果由不同的类加载器加载，它们也会被视为不同的组件**。

现在我们理解了为什么自定义类加载器是相关的，让我们实现一个ClassLoader的子类来扩展和总结JVM如何加载类的功能。

### 4.2 创建自定义类加载器

为了便于说明，假设我们需要使用自定义类加载器从文件中加载类。

**我们需要扩展ClassLoader类并覆盖findClass()方法**：

```java
public class CustomClassLoader extends ClassLoader {

    @Override
    public Class findClass(String name) throws ClassNotFoundException {
        byte[] b = loadClassFromFile(name);
        return defineClass(name, b, 0, b.length);
    }

    private byte[] loadClassFromFile(String fileName)  {
        InputStream inputStream = getClass().getClassLoader().getResourceAsStream(fileName.replace('.', File.separatorChar) + ".class");
        byte[] buffer;
        ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
        int nextValue = 0;
        try {
            while ( (nextValue = inputStream.read()) != -1 ) {
                byteStream.write(nextValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        buffer = byteStream.toByteArray();
        return buffer;
    }
}
```

在上面的例子中，我们定义了一个自定义的类加载器，它扩展了默认的类加载器，并从指定的文件中加载一个字节数组。

## 5. 理解java.lang.ClassLoader

让我们讨论java.lang.ClassLoader类中的一些基本方法，以更清楚地了解它的工作原理。

### 5.1 loadClass()方法

```java
public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
```

此方法负责加载给定name参数的类，name参数指的是完全限定的类名。

Java虚拟机调用loadClass()方法来解析类引用，将resolve设置为true。但是，并不总是需要解析一个类。**如果我们只需要判断类是否存在，那么resolve参数设置为false**。

此方法用作类加载器的入口点。

我们可以尝试从java.lang.ClassLoader的源码中了解loadClass()方法的内部工作原理：

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

该方法的默认实现按以下顺序搜索类：

1.  调用findLoadedClass(String)方法以查看该类是否已加载。
2.  在父类加载器上调用loadClass(String)方法。
3.  调用findClass(String)方法来查找类。

### 5.2 defineClass()方法

```java
protected final Class<?> defineClass(String name, byte[] b, int off, int len) throws ClassFormatError
```

此方法负责将字节数组转换为类的实例。在我们使用这个类之前，我们需要解析它。

如果数据不包含有效类，则会抛出ClassFormatError。

此外，我们不能覆盖此方法，因为它被标记为final。

### 5.3 findClass()方法

```java
protected Class<?> findClass(String name) throws ClassNotFoundException
```

此方法查找具有完全限定名称作为参数的类。我们需要在遵循用于加载类的委托模型的自定义类加载器实现中重写此方法。

此外，如果父类加载器找不到请求的类，loadClass()将调用此方法。

如果类加载器的父类找不到该类，默认实现将抛出ClassNotFoundException。

### 5.4 getParent()方法

```java
public final ClassLoader getParent()
```

该方法返回委托的父类加载器。

一些实现(如前面在第2节中看到的)使用null来表示引导类加载器。

### 5.5 getResource()方法

```java
public URL getResource(String name)
```

此方法尝试查找具有给定名称的资源。

它将首先委托给资源的父类加载器。如果父级为null，则搜索虚拟机内置的类加载器的路径。

如果失败，则该方法将调用findResource(String)来查找资源。指定为输入的资源名称可以是相对于类路径的，也可以是绝对的。

它返回一个用于读取资源的URL对象，如果找不到资源或调用者没有足够的权限返回资源，则返回null。

重要的是要注意Java从类路径加载资源。

最后，**Java中的资源加载被认为是与位置无关的**，因为只要环境设置为查找资源，代码在何处运行并不重要。

## 6. 上下文类加载器

通常，上下文类加载器为J2SE中引入的类加载委托方案提供了一种替代方法。

正如我们之前了解到的，**JVM中的类加载器遵循分层模型，因此每个类加载器都有一个父级，引导类加载器除外**。

但是，有时当JVM核心类需要动态加载应用程序开发人员提供的类或资源时，我们可能会遇到问题。

例如，在JNDI中，核心功能由rt.jar中的引导类实现。但是这些JNDI类可能会加载由独立供应商实现的JNDI提供程序(部署在应用程序类路径中)。此场景要求引导类加载器(父类加载器)加载一个对应用程序加载器(子类加载器)可见的类。

**J2SE委托在这里不起作用，为了解决这个问题，我们需要找到类加载的替代方法**。这可以使用线程上下文加载器来实现。

**java.lang.Thread类有一个方法getContextClassLoader()，它返回特定线程的ContextClassLoader**。ContextClassLoader由线程的创建者在加载资源和类时提供。

如果未设置该值，则默认为父线程的类加载器上下文。

## 7. 总结

类加载器对于执行Java程序是必不可少的。在本文中，我们对它们进行了很好的介绍。

我们讨论了不同类型的类加载器，即引导、扩展和系统类加载器。引导类加载器充当所有这些类的父级，并负责加载JDK内部类。另一方面，扩展和系统类加载器分别从Java扩展目录和类路径加载类。

我们还了解了类加载器的工作原理并研究了一些特性，例如委托、可见性和唯一性。然后我们简要说明了如何创建自定义类加载器。最后，我们介绍了上下文类加载器。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-1)上获得。
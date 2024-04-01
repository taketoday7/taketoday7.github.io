---
layout: post
title:  java.security.egd JVM选项
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

启动Java虚拟机(JVM)时，我们可以定义各种属性来改变JVM的行为方式，其中一个属性是java.security.egd。

在本教程中，我们将研究它是什么、如何使用它以及它有什么作用。

## 2. java.security.egd是什么？

作为JVM属性，我们可以使用java.security.egd来影响[SecureRandom](https://www.baeldung.com/java-secure-random)类的初始化方式。

与所有JVM属性一样，我们在启动JVM时在命令行中使用-D参数声明它：

```shell
java -Djava.security.egd=file:/dev/urandom -cp . cn.tuyucheng.taketoday.java.security.JavaSecurityEgdTester
```

通常，如果我们在Linux上运行Java 8或更高版本，那么我们的JVM将默认使用file:/dev/urandom。

## 3. java.security.egd有什么作用？

当我们第一次调用从SecureRandom读取字节时，我们会使其初始化并读取JVM的java.security配置文件。**此文件包含一个securerandom.source属性**：

```properties
securerandom.source=file:/dev/random
```

**安全提供程序(比如默认的sun.security.provider.Sun)在初始化的时候会读取这个属性**。

当我们设置java.security.egd JVM属性时，安全提供程序可能会使用它来覆盖在securerandom.source中配置的那个。

当我们使用SecureRandom生成随机数时，java.security.egd和securerandom.source控制哪个熵收集设备(EGD)将被用作种子数据的主要来源。

在Java 8之前，我们可以在$JAVA_HOME/jre/lib/security中找到java.security，但在以后的实现中，它在$JAVA_HOME/conf/security中。

**egd选项是否有效取决于安全提供程序的实现**。

## 4. java.security.egd可以取什么值？

我们可以用URL格式指定java.security.egd，其值如下：

-   file:/dev/random
-   file:/dev/urandom
-   file:/dev/./urandom

**此设置是否有任何效果，或任何其他值是否有所不同，取决于我们使用的平台和Java版本，以及我们的JVM安全性是如何配置的**。

在基于Unix的操作系统(OS)上，/dev/random是一个特殊的文件路径，它作为普通文件出现在文件系统中，但从中读取实际上与操作系统的设备驱动程序交互以生成随机数。一些设备实现还提供通过/dev/urandom甚至/dev/arandom URI的访问。

## 5. file:/dev/./urandom有什么特别之处？

首先，让我们了解文件/dev/random和/dev/urandom之间的区别：

-   /dev/random从各种来源收集熵；/dev/random将阻塞，直到它有足够的熵来满足我们对不可预测数据的读取请求
-   /dev/urandom将从可用的任何内容中获取伪随机性，而不会阻塞

当我们第一次使用SecureRandom时，我们的默认Sun SeedGenerator会初始化。

当我们使用特殊值file:/dev/random或file:/dev/urandom时，我们会导致Sun SeedGenerator使用本机(平台)实现。

Unix上的提供程序实现可能会因仍然从/dev/random读取而阻塞。在Java 1.4中，一些实现被发现有这个问题，该错误随后在Java 8的[JDK增强提案(JEP123)](https://openjdk.java.net/jeps/123)下得到修复。

**使用file:/dev/./urandom等URL或任何其他值，会使SeedGenerator将其视为我们要使用的种子源的URL**。

在类Unix系统上，我们的file:/dev/./urandom URL解析为相同的非阻塞/dev/urandom文件。

但是，**我们并不总是希望使用这个值**。在Windows上，我们没有此文件，因此我们的URL无法解析。这会触发生成随机性的最终机制，并可能将我们的初始化延迟约5秒。

## 6. SecureRandom的演变

java.security.egd的作用在不同的Java版本中发生了变化。

那么，让我们看看影响SecureRandom行为的一些更重要的事件：

-   Java 1.4

    -   JDK-4705093下出现的/dev/random阻塞问题：[使用/dev/urandom而不是/dev/random(如果存在)](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=4705093)

-   Java 5

    -   [JDK-4705093](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=4705093)的修复

        -   添加NativePRNG算法以遵守java.security.egd设置，但我们需要手动配置它
        -   如果使用SHA1PRNG，那么如果我们使用file:/dev/urandom以外的任何东西，它可能会阻塞。换句话说，**如果我们使用file:/dev/./urandom它可能会阻塞**

-   Java 8

    -   [JEP123：可配置的安全随机数生成器](http://openjdk.java.net/jeps/123)
        -   添加新的SecureRandom实现，该实现遵循安全属性
        -   添加一个新的getInstanceStrong()方法，用于平台原生的强随机数。非常适合生成高值和长期存在的机密，例如RSA私钥/公钥对
        -   [我们不再需要file:/dev/./urandom解决方法](https://docs.oracle.com/javase/8/docs/technotes/guides/security/enhancements-8.html)

-   Java 9

    -   [JEP273：基于DRBG的SecureRandom实现](http://openjdk.java.net/jeps/273)
        -   实现三种确定性随机位生成器(DRBG)机制，如[使用确定性随机位生成器生成随机数的建议](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-90Ar1.pdf)中所述

了解SecureRandom的变化方式让我们深入了解java.security.egd属性的可能影响。

## 7. 测试java.security.egd的效果

确定JVM属性效果的最佳方法是尝试它。因此，让我们通过运行一些代码来创建新的SecureRandom并计算获取一些随机字节所需的时间来查看java.security.egd的效果。

首先，让我们创建一个带有main()方法的JavaSecurityEgdTester类，我们将使用System.nanoTime()对secureRandom.nextBytes()的调用计时并显示结果：

```java
public class JavaSecurityEgdTester {
    public static final double NANOSECS = 1000000000.0;

    public static void main(String[] args) {
        SecureRandom secureRandom = new SecureRandom();
        long start = System.nanoTime();
        byte[] randomBytes = new byte[256];
        secureRandom.nextBytes(randomBytes);
        double duration = (System.nanoTime() - start) / NANOSECS;

        System.out.println("java.security.egd = " + System.getProperty("java.security.egd") + " took " + duration + " seconds and used the " + secureRandom.getAlgorithm() + " algorithm");
    }
}
```

现在，让我们通过启动一个新的Java实例并为java.security.egd属性指定一个值来运行JavaSecurityEgdTester测试：

```shell
java -Djava.security.egd=file:/dev/random -cp . cn.tuyucheng.taketoday.java.security.JavaSecurityEgdTester
```

让我们检查输出，看看我们的测试花了多长时间以及使用了哪种算法：

```text
java.security.egd=file:/dev/random took 0.692 seconds and used the SHA1PRNG algorithm
```

由于我们的系统属性只在初始化时读取，因此让我们在新的JVM中为java.security.egd的每个不同值启动我们的类：

```shell
java -Djava.security.egd=file:/dev/urandom -cp . cn.tuyucheng.taketoday.java.security.JavaSecurityEgdTester
java -Djava.security.egd=file:/dev/./urandom -cp . cn.tuyucheng.taketoday.java.security.JavaSecurityEgdTester
java -Djava.security.egd=tuyucheng -cp . cn.tuyucheng.taketoday.java.security.JavaSecurityEgdTester
```

**在使用Java 8或Java 11的Windows上，使用值file:/dev/random或file:/dev/urandom进行的测试给出亚秒级时间。使用其他任何东西，例如file:/dev/./urandom，甚至tuyucheng，都会使我们的测试花费超过5秒**！

有关为什么会发生这种情况的解释，请参阅我们[之前的部分](https://www.baeldung.com/java-security-egd#initializationDelay)。在Linux上，我们可能会得到不同的结果。

## 8. SecureRandom.getInstanceStrong()怎么样？

Java 8引入了[SecureRandom.getInstanceStrong()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html#getInstanceStrong())方法。让我们看看这如何影响我们的结果。

首先，让我们将new SecureRandom()替换为SecureRandom.getInstanceStrong()：

```java
SecureRandom secureRandom = SecureRandom.getInstanceStrong();
```

现在，让我们再次运行测试：

```shell
java -Djava.security.egd=file:/dev/random -cp . cn.tuyucheng.taketoday.java.security.JavaSecurityEgdTester
```

**在Windows上运行时，java.security.egd属性的值在使用SecureRandom.getInstanceStrong()时没有明显的影响**，即使是无法识别的值也会给我们快速响应。

让我们再次检查我们的输出并注意不到0.01秒的时间，我们还可以观察到该算法现在是Windows-PRNG：

```text
java.security.egd=tuyucheng took 0.003 seconds and used the Windows-PRNG algorithm
```

请注意，算法名称中的PRNG代表伪随机数生成器。

## 9. 播种算法

由于随机数在密码学中大量用于安全密钥，因此它们必须是不可预测的。

因此，我们如何播种算法直接影响它们产生的随机数的可预测性。

**为了产生不可预测性，SecureRandom实现使用从累积输入中收集的熵来播种他们的算法**。这来自于鼠标和键盘等IO设备。

**在类Unix系统上，我们的熵累积在文件/dev/random中**。

**Windows上没有/dev/random文件，将-Djava.security.egd设置为file:/dev/random或file:/dev/urandom会导致默认算法(SHA1PRNG)使用本机Microsoft Crypto API进行播种**。

## 10. 虚拟机呢？

有时，我们的应用程序可能在/dev/random中很少或没有熵收集的虚拟机中运行。

**虚拟机没有物理鼠标或键盘来生成数据**，因此/dev/random中的熵积累要慢得多。**这可能会导致我们的默认SecureRandom调用阻塞**，直到有足够的熵来生成不可预测的数字。

我们可以采取一些措施来缓解这种情况。例如，在RedHat Linux中运行VM时，系统管理员可以[配置虚拟IO随机数生成器virtio-rng](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_device_configuration-random_number_generator_device)，这会从托管它的物理机器中读取熵。

## 11. 故障排除技巧

如果我们的应用程序在它或其依赖项生成SecureRandom数字时挂起，请考虑java.security.egd-特别是当我们在Linux上运行时，以及如果我们在Java 8之前的版本上运行。

**我们的Spring Boot应用程序经常使用嵌入式[Tomcat](https://www.baeldung.com/tomcat)**，这使用SecureRandom生成会话密钥。**当我们看到Tomcat的“Creation of SecureRandom instance”操作需要5秒或更长时间时，我们应该为java.security.egd尝试不同的值**。

## 12. 总结

在本教程中，我们了解了JVM属性java.security.egd是什么、如何使用它以及它有什么作用。我们还发现它的效果会根据我们运行的平台和我们使用的Java版本而有所不同。

最后一点，我们可以在[JCA参考指南](https://docs.oracle.com/en/java/javase/11/security/java-cryptography-architecture-jca-reference-guide.html#GUID-AEB77CD8-D28F-4BBE-B9E5-160B5DC35D36)和[SecureRandom API规范](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html)的SecureRandom部分阅读更多关于SecureRandom及其工作原理的信息，并了解一些[关于urandom的神话](https://www.2uo.de/myths-about-urandom/)。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-2)上获得。
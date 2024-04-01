---
layout: post
title:  在Java中使用Argon2进行哈希处理
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在构建涉及用户身份验证的Web应用程序时，确保用户免受黑客攻击非常重要。**大多数Web应用程序设计为不存储纯文本密码，而是存储密码的哈希值**。[哈希](https://www.baeldung.com/java-password-hashing)和加盐是帮助保护密码免受任何可能的攻击的技术。

在本教程中，我们将学习哈希和加盐技术，以及如何在Java中使用Argon2进行哈希。

## 2. 密码哈希和加盐

密码[哈希](https://www.baeldung.com/sha-256-hashing-java)和加盐是两种可以增强数据库中存储的密码安全性的技术，**哈希算法涉及将密码更改或转换为随机字符串的数学运算**。

但是，黑客可以尝试通过比较常见密码的哈希值来猜测密码。为了防止这种情况发生，密码加盐就发挥了作用。

**密码加盐是在应用哈希算法之前将随机数据(称为盐)附加到密码的方法**。盐确保哈希是不同的，并且具有相同密码的两个用户将具有不同的哈希。

此外，**哈希算法是单向的，这意味着哈希不能转换回纯文本，这与[加密](https://www.baeldung.com/java-aes-encryption-decryption)不同**。这增加了另一层安全和保护。

## 3. 什么是Argon2？

**Argon2是一个基于密码的密钥导出函数**。它是一种安全的密码哈希函数，设计有许多可以调整的参数。而且，**Argon2是一个内存困难函数，这意味着它需要大量内存来计算，并且很难在内存有限的硬件上实现**。

此外，它允许应用程序根据其安全需求自定义算法，这对于具有不同安全要求的应用程序至关重要。

**此外，由于Argon2提供高安全性，因此建议将其用于需要强密码保护的应用程序**。它可以抵御来自GPU和其他专用硬件的攻击。

## 4. 使用Argon2进行哈希处理

**Argon2的优点之一是我们可以根据不同的需求对其进行配置**。我们可以设置迭代次数，这是密码被哈希的次数。**迭代次数越多，对密码进行哈希处理所需的时间就越长，但会使密码更加安全**。

此外，我们可以设置内存成本，这是Argon2将使用的内存量。**较高的内存成本将使密码更安全，但会消耗更多的系统内存**。

此外，我们还可以设置并行成本，这是Argon2算法将使用的线程数。**较高的并行成本将加快密码哈希过程，但会降低密码安全性**。

在以下小节中，我们将使用Spring Security Crypto库和[Bouncy Castle](https://www.baeldung.com/java-bouncy-castle)库通过Argon2实现哈希。

### 4.1 使用Spring Security Crypto实现Argon2哈希

**Spring Security Crypto库有一个类使用Argon2来哈希密码，它内部依赖于Bouncy Castle库**。

让我们使用Spring Security Crypto库来对密码进行哈希处理。首先，我们需要将其[依赖项](https://mvnrepository.com/artifact/org.springframework.security/spring-security-crypto)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
    <version>6.0.3</version>
</dependency>
```

接下来，让我们看一个基于Argon2对密码进行哈希处理的单元测试：

```java
@Test
public void givenRawPassword_whenEncodedWithArgon2_thenMatchesEncodedPassword() {
    String rawPassword = "Tuyucheng";
    Argon2PasswordEncoder arg2SpringSecurity = new Argon2PasswordEncoder(16, 32, 1, 60000, 10);
    String springBouncyHash = arg2SpringSecurity.encode(rawPassword);
        
    assertTrue(arg2SpringSecurity.matches(rawPassword, springBouncyHash));
}
```

在上面的示例中，我们声明一个变量来存储原始密码“Tuyucheng”。接下来，我们创建一个带有5个参数的Argon2PasswordEncoder实例。我们将要使用的迭代次数设置为10，并将哈希长度设置为32字节。默认哈希长度为64字节。此外，**我们将内存成本设置为60000KB，并行度设置为1个线程，时间成本设置为16次迭代**。

最后，我们验证原始密码是否与哈希密码匹配。

### 4.2 使用Bouncy Castle实现Argon2哈希

**与Spring Security Crypto库相比，Bouncy Castle库的实现更加低级**。要使用Bouncy Castle库，我们需要将其[依赖项](https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.70</version>
</dependency>
```

让我们看一个使用Bouncy Castle库实现哈希的示例。

首先，让我们创建一个方法来生成随机盐：

```java
private byte[] generateSalt16Byte() {
    SecureRandom secureRandom = new SecureRandom();
    byte[] salt = new byte[16];
    secureRandom.nextBytes(salt);
        
    return salt;
}
```

在上面的示例代码中，我们创建了一个[SecureRandom](https://www.baeldung.com/java-secure-random)对象，它是一个提供加密强随机数生成器的类。接下来，我们创建一个大小为16的字节数组来存储16字节的数据。然后，我们调用secureRandom上的nextBytes()方法来生成盐。

最后，让我们对密码“Tuyucheng”进行哈希处理：

```java
@Test
public void givenRawPasswordAndSalt_whenArgon2AlgorithmIsUsed_thenHashIsCorrect() {
    byte[] salt = generateSalt16Byte();
    String password = "Tuyucheng";
        
    int iterations = 2;
    int memLimit = 66536;
    int hashLength = 32;
    int parallelism = 1;
        
    Argon2Parameters.Builder builder = new Argon2Parameters.Builder(Argon2Parameters.ARGON2_id)
        .withVersion(Argon2Parameters.ARGON2_VERSION_13)
        .withIterations(iterations)
        .withMemoryAsKB(memLimit)
        .withParallelism(parallelism)
        .withSalt(salt);
        
    Argon2BytesGenerator generate = new Argon2BytesGenerator();
    generate.init(builder.build());
    byte[] result = new byte[hashLength];
    generate.generateBytes(password.getBytes(StandardCharsets.UTF_8), result, 0, result.length);
        
    Argon2BytesGenerator verifier = new Argon2BytesGenerator();
    verifier.init(builder.build());
    byte[] testHash = new byte[hashLength];
    verifier.generateBytes(password.getBytes(StandardCharsets.UTF_8), testHash, 0, testHash.length);
        
    assertTrue(Arrays.equals(result, testHash));
}
```

在上面的示例中，我们使用generateSalt16Byte()方法创建一个随机的16字节盐。**接下来，我们定义算法的基本参数，例如迭代次数、内存限制、哈希长度、并行因子和盐**。

然后，我们创建一个Argon2BytesGenerator对象，该对象有助于生成密码哈希。此外，我们定义了一个字节数组来存储生成的哈希结果。

最后，我们创建Argon2BytesGenerator的另一个实例，将结果与测试哈希进行比较。这断言密码哈希是正确的并且可以通过Argon2算法进行验证。

## 5. 总结

在本文中，我们学习了密码哈希和加盐的基础知识。此外，我们深入研究了Argon2算法，并看到了使用Spring Security Crypto和Bouncy Castle的实现。Spring Security Crypto看起来很简单，因为它抽象了一些过程。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-3)上获得。
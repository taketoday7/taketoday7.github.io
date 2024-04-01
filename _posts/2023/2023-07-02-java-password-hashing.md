---
layout: post
title:  在Java中哈希密码
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在本教程中，我们将讨论密码哈希的重要性。

我们将快速了解它是什么、它为什么重要，以及在Java中执行此操作的一些安全和不安全的方法。

## 2. 什么是哈希？

[哈希](https://www.baeldung.com/cs/hashing)是使用称为加密哈希函数的数学函数从给定消息生成字符串或哈希的过程。

虽然有很多哈希函数，但那些专为哈希密码而设计的哈希函数需要具有四个主要属性才能保证安全：

1.  它应该是确定性的：由同一个哈希函数处理的同一条消息应该始终产生相同的哈希
2.  它是不可逆的：从其哈希生成消息是不切实际的 
3.  它具有高[熵](https://www.baeldung.com/cs/cs-entropy-definition)：对消息的微小更改应该产生截然不同的哈希
4.  并且它抵抗冲突：两个不同的消息不应该产生相同的哈希

具有所有四个属性的哈希函数是密码哈希的有力候选者，因为它们一起大大增加了从哈希对密码进行逆向工程的难度。

此外，尽管如此，**密码哈希函数应该很慢**。快速的算法将有助于[暴力攻击](https://www.baeldung.com/cs/brute-force-cybersecurity-string-search)，在这种攻击中，黑客将尝试通过每秒哈希和比较数十亿([或数万亿](https://www.wired.com/2014/10/snowdens-first-emails-to-poitras/))个潜在密码来猜测密码。

满足所有这些标准的一些很棒的哈希函数是PBKDF2、BCrypt和SCrypt。但首先，让我们看一下一些较旧的算法以及不再推荐它们的原因。

## 3. 不推荐：MD5

我们的第一个哈希函数是MD5消息摘要算法，早在1992年就已开发出来。

Java的MessageDigest使它易于计算并且在其他情况下仍然有用。

然而，在过去几年中，**人们发现[MD5不符合第四个密码哈希属性](https://blog.avira.com/md5-the-broken-algorithm/)，因为它在计算上很容易产生冲突**。最重要的是，MD5是一种快速算法，因此无法抵抗暴力攻击。

由于这些，不推荐使用MD5。

## 4. 不推荐：SHA-512

接下来，我们将了解SHA-512，它是安全哈希算法系列的一部分，该系列始于1993年的SHA-0。

### 4.1 为什么选择SHA-512？

随着计算机功能的增强，以及我们发现新的漏洞，研究人员会推导出新版本的SHA。新版本的长度逐渐变长，或者有时研究人员会发布底层算法的新版本。

SHA-512代表第三代算法中最长的密钥。

虽然**现在有更安全的SHA版本**，但[SHA-512是Java中实现的最强大的版本](https://docs.oracle.com/en/java/javase/12/docs/specs/security/standard-names.html)。

### 4.2 Java中的实现

现在，让我们看一下在Java中实现SHA-512哈希算法。

首先，我们必须了解盐的概念。简单地说，**这是为每个新哈希生成的随机序列**。

通过引入这种随机性，我们增加了哈希的熵，并保护我们的数据库免受称为彩虹表的预编译哈希列表的影响。

然后，我们的新哈希函数大致变为：

```text
salt <- generate-salt;
hash <- salt + ':' + sha512(salt + password)
```

### 4.3 生成盐

为了引入盐，我们将使用java.security中的[SecureRandom](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html)类：

```java
SecureRandom random = new SecureRandom();
byte[] salt = new byte[16];
random.nextBytes(salt);
```

然后，我们将使用[MessageDigest](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/MessageDigest.html)类来配置带有盐的SHA-512哈希函数：

```java
MessageDigest md = MessageDigest.getInstance("SHA-512");
md.update(salt);
```

添加后，我们现在可以使用digest方法生成哈希密码：

```java
byte[] hashedPassword = md.digest(passwordToHash.getBytes(StandardCharsets.UTF_8));
```

### 4.4 为什么不推荐？

当使用盐时，[SHA-512仍然是一个不错的选择](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms)，**但还有更强大和更慢的选择**。

此外，我们将介绍的其余选项有一个重要特征：可配置的强度。

## 5. PBKDF2、BCrypt和SCrypt

PBKDF2、BCrypt和SCrypt是三种推荐的算法。

### 5.1 为什么推荐这些？

这些中的每一个都很慢，并且每个都具有可配置强度的出色特性。

这意味着随着计算机强度的增加，**我们可以通过改变输入来减慢算法的速度**。

### 5.2 在Java中实现PBKDF2

现在，**盐是密码哈希的基本原则**，所以我们也需要一个用于PBKDF2的盐：

```java
SecureRandom random = new SecureRandom();
byte[] salt = new byte[16];
random.nextBytes(salt);
```

接下来，我们将创建一个[PBEKeySpec](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/spec/PBEKeySpec.html)和一个[SecretKeyFactory](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/SecretKeyFactory.html)，我们将使用PBKDF2WithHmacSHA1算法对其进行实例化：

```java
KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 128);
SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
```

第三个参数(65536)实际上是强度参数。它指示此算法运行的迭代次数，从而增加生成哈希所需的时间。

最后，我们可以使用我们的SecretKeyFactory来生成哈希：

```java
byte[] hash = factory.generateSecret(spec).getEncoded();
```

### 5.3 在Java中实现BCrypt和SCrypt

因此，事实证明**BCrypt和SCrypt支持尚未随Java一起提供**，尽管一些Java库支持它们。

其中一个库是Spring Security。

## 6. 使用Spring Security进行密码哈希

尽管Java原生支持PBKDF2和SHA哈希算法，但它不支持BCrypt和SCrypt算法。

对我们来说幸运的是，Spring Security通过[PasswordEncoder](https://docs.spring.io/spring-security/site/docs/4.2.4.RELEASE/apidocs/org/springframework/security/crypto/password/PasswordEncoder.html)接口支持所有这些推荐的算法：

-   Pbkdf2PasswordEncoder为我们提供PBKDF2
-   BCryptPasswordEncoder为我们提供了BCrypt，并且
-   SCryptPasswordEncoder可用于SCrypt

**PBKDF2、BCrypt和SCrypt的密码编码器都支持配置所需的密码哈希强度**。

我们可以直接使用这些编码器，即使没有基于Spring Security的应用程序。或者，如果我们使用Spring Security保护我们的站点，那么我们可以通过它的DSL或通过[依赖注入](https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)来配置我们想要的密码编码器。

而且，与我们上面的示例不同，**这些加密算法将在内部为我们生成盐**。该算法将盐存储在输出哈希中，供以后验证密码时使用。

## 7. 总结

因此，我们深入研究了密码哈希；探索概念及其用途。

在用Java编写代码之前，我们已经了解了一些历史哈希函数以及一些当前实现的哈希函数。

最后，我们看到Spring Security附带了它的密码加密类，实现了一系列不同的哈希函数。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-2)上获得。
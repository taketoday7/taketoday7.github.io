---
layout: post
title:  Java中的SHA-256和SHA3-256哈希
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

SHA(安全哈希算法)是流行的加密哈希函数之一，加密哈希可用于为文本或数据文件制作签名。

在本教程中，让我们看看如何使用各种Java库执行SHA-256和SHA3-256哈希操作。

[SHA-256](https://en.wikipedia.org/wiki/SHA-2)算法生成几乎唯一的、固定大小的256位(32字节)哈希。这是一个单向函数，因此无法将结果解密回原始值。

目前，SHA-2哈希被广泛使用，因为它被认为是密码学领域中最安全的哈希算法。

[SHA-3](https://en.wikipedia.org/wiki/SHA-3)是继SHA-2之后最新的安全哈希标准。与SHA-2相比，SHA-3提供了一种不同的方法来生成唯一的单向哈希，并且在某些硬件实现上速度更快。SHA3-256与SHA-256类似，是SHA-3中的256位定长算法。

[NIST](https://csrc.nist.gov/projects/hash-functions)于2015年发布了SHA-3，因此目前SHA-3库的数量不如SHA-2。直到JDK 9，SHA-3算法才在内置的默认提供程序中可用。

现在让我们从SHA-256开始。

## 2. Java中的MessageDigest类

Java为SHA-256哈希提供了内置的MessageDigest类：

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] encodedhash = digest.digest(originalString.getBytes(StandardCharsets.UTF_8));
```

但是，这里我们必须使用自定义字节到十六进制转换器来获取十六进制的哈希值：

```java
private static String bytesToHex(byte[] hash) {
    StringBuilder hexString = new StringBuilder(2 * hash.length);
    for (int i = 0; i < hash.length; i++) {
        String hex = Integer.toHexString(0xff & hash[i]);
        if(hex.length() == 1) {
            hexString.append('0');
        }
        hexString.append(hex);
    }
    return hexString.toString();
}
```

我们需要知道**MessageDigest不是线程安全的**。因此，我们应该为每个线程使用一个新实例。

## 3. Guava库

Google Guava库还提供了一个用于哈希的实用程序类。

首先，让我们定义依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

接下来，我们可以使用Guava对字符串进行哈希处理：

```java
String sha256hex = Hashing.sha256()
    .hashString(originalString, StandardCharsets.UTF_8)
    .toString();
```

## 4. Apache Commons Codecs

同样，我们也可以使用Apache Commons Codecs：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.11</version>
</dependency>
```

这是支持SHA-256哈希的实用程序类-称为DigestUtils：

```java
String sha256hex = DigestUtils.sha256Hex(originalString);
```

## 5. Bouncy Castle库

### 5.1 Maven依赖

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.60</version>
</dependency>
```

### 5.2 使用Bouncy Castle库进行哈希

Bouncy Castle API提供了一个实用程序类，用于将十六进制数据转换为字节，然后再转换回来。

但是，我们需要先使用内置的Java API填充摘要：

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(originalString.getBytes(StandardCharsets.UTF_8));
String sha256hex = new String(Hex.encode(hash));
```

## 6. SHA3-256

现在让我们继续使用SHA3-256，Java中的SHA3-256哈希与SHA-256没有什么不同。

### 6.1 Java中的MessageDigest类

[从JDK 9开始](https://docs.oracle.com/javase/9/security/oracleproviders.htm#JSSEC-GUID-3A80CC46-91E1-4E47-AC51-CB7B782CEA7D)，我们可以简单地使用内置的SHA3-256算法：

```java
final MessageDigest digest = MessageDigest.getInstance("SHA3-256");
final byte[] hashbytes = digest.digest(originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = bytesToHex(hashbytes);
```

### 6.2 Apache Commons Codecs

Apache Commons Codecs为MessageDigest类提供了一个方便的DigestUtils包装器。

这个库从[1.11](https://search.maven.org/artifact/commons-codec/commons-codec/1.11/jar)版本开始支持SHA3-256，并且它也[需要JDK 9+](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/digest/MessageDigestAlgorithms.html#SHA3_256)：

```java
String sha3Hex = new DigestUtils("SHA3-256").digestAsHex(originalString);
```

### 6.3 Keccak-256

Keccak-256是另一种流行的SHA3-256哈希算法。目前，它作为标准SHA3-256的替代品。Keccak-256提供与标准SHA3-256相同的安全级别，它与SHA3-256的区别仅在于填充规则。它已被用于多个区块链项目，例如[Monero](https://monerodocs.org/cryptography/keccak-256/)。

同样，我们需要导入Bouncy Castle库以使用Keccak-256哈希：

```java
Security.addProvider(new BouncyCastleProvider());
final MessageDigest digest = MessageDigest.getInstance("Keccak-256");
final byte[] encodedhash = digest.digest(originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = bytesToHex(encodedhash);
```

我们还可以使用Bouncy Castle API来进行哈希：

```java
Keccak.Digest256 digest256 = new Keccak.Digest256();
byte[] hashbytes = digest256.digest(originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = new String(Hex.encode(hashbytes));
```

## 7. 总结

在这篇简短的文章中，我们介绍了使用内置库和第三方库在Java中实现SHA-256和SHA3-256哈希的几种方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-2)上获得。
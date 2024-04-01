---
layout: post
title:  Java KeyStore API
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在本教程中，我们将学习如何使用KeyStore API在Java中管理加密密钥和证书。

## 2. 密钥库

如果我们需要在Java中管理密钥和证书，我们需要一个keystore，**它只是密钥和证书的别名条目的安全集合**。

我们通常将密钥库保存到文件系统中，并且可以使用密码保护它。

默认情况下，Java有一个位于JAVA_HOME/jre/lib/security/cacerts的密钥库文件。我们可以使用默认密钥库密码changeit访问此密钥库。

## 3. 创建密钥库

### 3.1 构造

我们可以使用[keytool](https://docs.oracle.com/en/java/javase/11/tools/keytool.html)轻松创建一个密钥库，或者我们可以使用KeyStore API以编程方式进行创建：

```java
KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
```

这里我们使用了默认类型，尽管有一些可用的[密钥库类型](https://docs.oracle.com/en/java/javase/11/docs/specs/security/standard-names.html#keystore-types)，如jceks或pkcs12。

我们可以使用-Dkeystore.type参数覆盖默认的“JKS”(Oracle专有密钥库协议)类型：

```shell
-Dkeystore.type=pkcs12
```

或者我们可以在getInstance中列出一种支持的格式：

```java
KeyStore ks = KeyStore.getInstance("pkcs12");
```

### 3.2 初始化

最初，我们需要加载密钥库：

```java
char[] pwdArray = "password".toCharArray();
ks.load(null, pwdArray);
```

无论是创建新密钥库还是打开现有密钥库，我们都会使用load。我们将通过传递null作为第一个参数来告诉KeyStore创建一个新的。

我们还提供了一个密码，用于将来访问密钥库。我们也可以将其设置为null，但是这会泄露我们的密钥。

### 3.3 存储

最后，我们将新的密钥库保存到文件系统：

```java
try (FileOutputStream fos = new FileOutputStream("newKeyStoreFileName.jks")) {
    ks.store(fos, pwdArray);
}
```

请注意，上面未显示的是getInstance、load和store每次抛出的几个受检异常。

## 4. 加载密钥库

要加载密钥库，我们首先需要像以前一样创建一个KeyStore实例。

不过这一次，我们将指定格式，因为我们正在加载一个现有的格式：

```java
KeyStore ks = KeyStore.getInstance("JKS");
ks.load(new FileInputStream("newKeyStoreFileName.jks"), pwdArray);
```

如果我们的JVM不支持我们传递的密钥库类型，或者如果它与我们打开的文件系统上的密钥库类型不匹配，我们将得到一个KeyStoreException：

```text
java.security.KeyStoreException: KEYSTORE_TYPE not found
```

此外，如果密码错误，我们将得到一个UnrecoverableKeyException：

```text
java.security.UnrecoverableKeyException: Password verification failed
```

## 5. 存储条目

在密钥库中，我们可以存储三种不同的条目，每一种都在其别名下：

-   对称密钥(在JCE中称为密钥)
-   非对称密钥(在JCE中称为公钥和私钥)
-   受信任的证书

### 5.1 保存对称密钥

我们可以在密钥库中存储的最简单的东西是对称密钥。

要保存对称密钥，我们需要三样东西：

1.  **别名**：这只是我们将来用来指代条目的名称
2.  **密钥**：包装在KeyStore.SecretKeyEntry中
3.  **密码**：包装在所谓的ProtectionParam中

```java
KeyStore.SecretKeyEntry secret = new KeyStore.SecretKeyEntry(secretKey);
KeyStore.ProtectionParameter password = new KeyStore.PasswordProtection(pwdArray);
ks.setEntry("db-encryption-secret", secret, password);
```

**请记住，密码不能为null；但是，它可以是空字符串**。如果我们将条目的密码保留为null，我们将得到KeyStoreException：

```text
java.security.KeyStoreException: non-null password required to create SecretKeyEntry
```

我们需要将密钥和密码包装在包装类中，这似乎有点奇怪。

我们包装密钥是因为setEntry是一个泛型方法，也可用于其他条目类型。条目的类型允许KeyStore API以不同的方式处理它。

我们包装密码是因为KeyStore API支持回调到GUI和CLI以从最终用户那里收集密码。我们可以查看[KeyStore.CallbackHandlerProtection Javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/KeyStore.CallbackHandlerProtection.html)以了解更多详细信息。

我们还可以使用此方法更新现有密钥；我们只需要使用相同的别名和密码以及我们的新密钥再次调用它。

### 5.2 保存私钥

存储非对称密钥有点复杂，因为我们需要处理证书链。

KeyStore API为我们提供了一个名为setKeyEntry的专用方法，它比通用的setEntry方法更方便。

因此，要保存非对称密钥，我们需要四样东西：

1.  **别名**：像以前一样
2.  **私钥**：因为我们没有使用泛型方法，所以密钥不会被包装。此外，在我们的例子中，它应该是PrivateKey的一个实例
3.  **密码**：用于访问条目。这次，密码是强制性的
4.  **证书链**：这证明相应的公钥

```java
X509Certificate[] certificateChain = new X509Certificate[2];
chain[0] = clientCert;
chain[1] = caCert;
ks.setKeyEntry("sso-signing-key", privateKey, pwdArray, certificateChain);
```

当然，这里可能会出现很多错误，比如如果pwdArray为空：

```text
java.security.KeyStoreException: password can't be null
```

但是有一个非常奇怪的异常需要注意，如果pwdArray是一个空数组就会发生：

```text
java.security.UnrecoverableKeyException: Given final block not properly padded
```

要解决这个问题，我们只需使用相同的别名和新的privateKey和certificateChain再次调用该方法。

### 5.3 保存受信任证书

**存储受信任的证书非常简单，它只需要别名和证书本身，其类型为Certificate**：

```java
ks.setCertificateEntry("google.com", trustedCertificate);
```

通常，证书不是我们生成的，而是来自第三方的。

因此，请务必注意，KeyStore实际上并不验证此证书。我们应该在存储之前自行验证它。

要更新，我们只需使用相同的别名和新的trustedCertificate再次调用该方法。

## 6. 读取条目

### 6.1 读取单个条目

首先，我们可以通过别名提取密钥和证书：

```java
Key ssoSigningKey = ks.getKey("sso-signing-key", pwdArray);
Certificate google = ks.getCertificate("google.com");
```

如果没有该名称的条目，或者它属于不同的类型，则getKey仅返回null：

```java
public void whenEntryIsMissingOrOfIncorrectType_thenReturnsNull() {
    // ... initialize keystore
    // ... add an entry called "widget-api-secret"

    Assert.assertNull(ks.getKey("some-other-api-secret"));
    Assert.assertNotNull(ks.getKey("widget-api-secret"));
    Assert.assertNull(ks.getCertificate("widget-api-secret")); 
}
```

但是**如果密钥的密码是错误的，我们会得到我们之前谈到的同样奇怪的错误**：

```text
java.security.UnrecoverableKeyException: Given final block not properly padded
```

### 6.2 检查密钥库是否包含别名

由于KeyStore仅使用Map存储条目，因此它可以在不检索条目的情况下检查是否存在：

```java
public void whenAddingAlias_thenCanQueryWithoutSaving() {
    // ... initialize keystore
    // ... add an entry called "widget-api-secret"
    assertTrue(ks.containsAlias("widget-api-secret"));
    assertFalse(ks.containsAlias("some-other-api-secret"));
}
```

### 6.3 检查条目类型

KeyStore#entryInstanceOf更强大一些。

它类似于containsAlias，并且还会检查条目类型：

```java
public void whenAddingAlias_thenCanQueryByType() {
    // ... initialize keystore
    // ... add a secret entry called "widget-api-secret"
    assertTrue(ks.containsAlias("widget-api-secret"));
    assertFalse(ks.entryInstanceOf(
        "widget-api-secret",
        KeyType.PrivateKeyEntry.class));
}
```

## 7. 删除条目

当然，KeyStore也支持删除我们添加的条目：

```java
public void whenDeletingAnAlias_thenIdempotent() {
    // ... initialize a keystore
    // ... add an entry called "widget-api-secret"
    assertEquals(ks.size(), 1);
    ks.deleteEntry("widget-api-secret");
    ks.deleteEntry("some-other-api-secret");
    assertFalse(ks.size(), 0);
}
```

幸运的是，deleteEntry是幂等的，因此无论条目是否存在，该方法的反应都是一样的。

## 8. 删除密钥库

如果我们想删除我们的密钥库，API对我们没有帮助，但我们仍然可以使用Java来做到这一点：

```java
Files.delete(Paths.get(keystorePath));
```

或者，作为替代方案，我们可以保留密钥库并只删除条目：

```java
Enumeration<String> aliases = keyStore.aliases();
while (aliases.hasMoreElements()) {
    String alias = aliases.nextElement();
    keyStore.deleteEntry(alias);
}
```

## 9. 总结

在本文中，我们学习了如何使用KeyStore API管理证书和密钥。我们讨论了什么是密钥库，并探讨了如何创建、加载和删除密钥库。我们还演示了如何在密钥库中存储密钥或证书，以及如何使用新值加载和更新现有条目。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-1)上获得。
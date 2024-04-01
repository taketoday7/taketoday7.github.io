---
layout: post
title:  使用Java计算X509证书的指纹
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

**证书的指纹是证书的唯一标识符，它不是证书的一部分**，但它是从中计算出来的。

在这个简短的教程中，我们将了解如何使用Java计算X509证书的指纹。

## 2. 使用纯Java

首先，让我们从我们的证书文件中获取一个X509Certificate对象：

```java
public static X509Certificate getCertObject(String filePath) throws IOException, CertificateException {
    try (FileInputStream is = new FileInputStream(filePath)) {
        CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
        return (X509Certificate) certificateFactory.generateCertificate(is);
    }
}
```

接下来，让我们从这个对象中获取指纹：

```java
private static String getThumbprint(X509Certificate cert) throws NoSuchAlgorithmException, CertificateEncodingException {
    MessageDigest md = MessageDigest.getInstance("SHA-1");
    md.update(cert.getEncoded());
    return DatatypeConverter.printHexBinary(md.digest()).toLowerCase();
}
```

例如，如果我们有一个名为tuyucheng.pem的X509证书文件，我们可以使用上述方法轻松打印其指纹：

```java
X509Certificate certObject = getCertObject("tuyucheng.pem");
System.out.println(getThumbprint(certObject));
```

结果将类似于：

```plaintext
c9fa9f008655c8401ad27e213b985804854d928c
```

## 3. 使用Apache Commons Codec

我们还可以使用[Apache Commons Codec](https://search.maven.org/search?q=g:commons-codec)库中的DigestUtils类来实现相同的目标。

让我们在pom.xml文件中添加一个依赖项：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

现在，我们只需使用sha1Hex()方法从X509Certificate对象中获取指纹：

```java
DigestUtils.sha1Hex(certObject.getEncoded());
```

## 4. 总结

在这个快速教程中，我们学习了两种在Java中计算X509证书指纹的方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-3)上获得。
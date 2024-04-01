---
layout: post
title:  在Java中下载电子邮件附件
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在本教程中，我们将了解如何使用Java下载电子邮件附件。**为此，我们需要[Java Mail API](https://www.baeldung.com/java-email)**。Java Mail API可以作为Maven依赖项或单独的jar提供。

## 2. Jakarta Mail API概述

[Angus Mail](https://eclipse-ee4j.github.io/angus-mail/)库提供了[Jakarta Mail API](https://github.com/jakartaee/mail-api)规范的实现，该规范是Java Mail API的继承者。

此API用于撰写、发送和接收来自Gmail等电子邮件服务器的电子邮件。它为使用抽象类和接口的电子邮件系统提供了一个框架。API支持大多数RFC822和MIME互联网消息传递协议，如SMTP、POP、IMAP、MIME和NNTP。

## 3. Jakarta Mail API设置

我们需要在我们的Java项目中添加[angus-mail](https://mvnrepository.com/artifact/org.eclipse.angus/angus-mail) Maven依赖项：

```xml
<dependency>
    <groupId>org.eclipse.angus</groupId>
    <artifactId>angus-mail</artifactId>
    <version>2.0.1</version>
</dependency>
```

## 4. 下载电子邮件附件

为了在Java中处理电子邮件，我们使用jakarta.mail包中的Message类。消息实现jakarta.mail.Part接口。

**Part接口有BodyPart和属性。带有附件的内容是一个名为MultiPart的BodyPart**。如果[电子邮件有任何附件](https://www.baeldung.com/java-send-emails-attachments)，它的disposition等于“ Part.ATTACHMENT”。如果没有附件，则disposition为null。来自Part接口的getDisposition方法为我们提供了disposition。

我们通过一个简单的基于Maven的项目来了解下载电子邮件附件的工作原理。我们将专注于获取要下载的电子邮件并将附件保存到磁盘。

我们的项目有一个实用程序可以处理下载电子邮件并将它们保存到我们的磁盘。我们还显示附件列表。

要下载附件，我们首先检查内容类型是否包含多部分内容。如果是，我们可以进一步处理，检查该零件是否有附件。要检查内容类型，我们写：

```java
if (contentType.contains("multipart")) {
    //send to the download utility...
}
```

如果我们有一个多部分，我们首先检查它是否属于Part.ATTACHMENT类型，如果是，我们使用saveFile方法将文件保存到我们的目标文件夹。因此，在下载实用程序中，我们将检查：

```java
if (Part.ATTACHMENT.equalsIgnoreCase(part.getDisposition())) {
    String file = part.getFileName();
    part.saveFile(downloadDirectory + File.separator + part.getFileName());
    downloadedAttachments.add(file);
}
```

**由于我们使用的Java Mail API版本高于1.4，因此我们可以使用Part接口中的saveFile方法。saveFile方法适用于File对象或String。我们在示例中使用了一个字符串**。此步骤将附件保存到我们指定的文件夹中。我们还维护一个显示附件列表。

在JavaMail API 1.4版本之前，我们必须使用FileStream和InputStream逐字节写入整个文件。在我们的示例中，我们为Gmail帐户使用了Pop3服务器。因此，要调用示例中的方法，我们需要一个有效的Gmail用户名和密码以及一个用于下载附件的文件夹。

让我们看看下载附件并将它们保存到磁盘的示例代码：

```java
public List<String> downloadAttachments(Message message) throws IOException, MessagingException {
    List<String> downloadedAttachments = new ArrayList<String>();
    Multipart multiPart = (Multipart) message.getContent();
    int numberOfParts = multiPart.getCount();
    for (int partCount = 0; partCount < numberOfParts; partCount++) {
        MimeBodyPart part = (MimeBodyPart) multiPart.getBodyPart(partCount);
        if (Part.ATTACHMENT.equalsIgnoreCase(part.getDisposition())) {
            String file = part.getFileName();
            part.saveFile(downloadDirectory + File.separator + part.getFileName());
            downloadedAttachments.add(file);
        }
    }
    return downloadedAttachments;
}  
```

## 5. 总结

本文介绍了如何使用Jakarta Mail API在Java中下载电子邮件附件。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-3)上获得。

---
layout: post
title:  通知用户从新设备或位置登录
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 简介

在本教程中，我们将演示如何**验证我们的用户是否从新设备/位置登录**。

我们将向他们发送登录通知，让他们知道我们在他们的帐户上检测到不熟悉的活动。

## 2. 用户位置和设备详细信息

我们需要两件事：用户的位置以及他们用于登录的设备信息。

考虑到我们使用HTTP与用户交换消息，我们将不得不完全依赖传入的HTTP请求及其元数据来检索此信息。

对我们来说幸运的是，HTTP标头的唯一目的就是携带此类信息。

### 2.1 设备位置

在我们可以估计用户的位置之前，我们需要获取他们的原始IP地址。

我们可以通过以下方式做到这一点：

-   [X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)：事实上的标准标头，用于识别通过HTTP代理或负载平衡器连接到Web服务器的客户端的原始IP地址
-   [ServletRequest.getRemoteAddr()](https://docs.oracle.com/javaee/6/api/javax/servlet/ServletRequest.html#getRemoteAddr())：一种工具方法，返回客户端的原始IP或发送请求的最后一个代理

从HTTP请求中提取用户的IP地址并不十分可靠，因为它们可能会被篡改。但是，让我们在教程中对此进行简化，并假设情况并非如此。

检索到IP地址后，我们可以通过[geolocation](https://en.wikipedia.org/wiki/Geolocation)将其转换为真实世界的位置。

### 2.2 设备详情

与原始IP地址类似，还有一个HTTP标头，其中包含有关用于发送称为[User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)的请求的设备的信息。

简而言之，它携带的信息使我们**能够识别发出请求的用户代理的应用程序类型、操作系统和软件供应商/版本**。

这是它可能看起来像的示例：

```text
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 
  (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
```

在上面的示例中，设备在MacOS X10.14上运行并使用Chrome 71.0发送请求。

我们不会从头开始实现用户代理解析器，而是求助于已经过测试且更可靠的现有解决方案。

## 3. 检测新设备或位置

现在我们已经介绍了我们需要的信息，让我们修改我们的[AuthenticationSuccessHandler](https://www.baeldung.com/spring_redirect_after_login)以在用户登录后执行验证：

```java
public class MySimpleUrlAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    // ...
    @Override
    public void onAuthenticationSuccess(
          final HttpServletRequest request,
          final HttpServletResponse response,
          final Authentication authentication)
          throws IOException {
        handle(request, response, authentication);
        // ...
        loginNotification(authentication, request);
    }

    private void loginNotification(Authentication authentication, HttpServletRequest request) {
        try {
            if (authentication.getPrincipal() instanceof User) {
                deviceService.verifyDevice(((User)authentication.getPrincipal()), request);
            }
        } catch(Exception e) {
            logger.error("An error occurred verifying device or location");
            throw new RuntimeException(e);
        }
    }
    // ...
}
```

我们只是添加了对新组件的调用：DeviceService。该组件将封装我们识别新设备/位置并通知用户所需的一切。

但是，在进入我们的DeviceService之前，让我们创建DeviceMetadata实体来随着时间的推移持久保存我们的用户数据：

```java
@Entity
public class DeviceMetadata {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private Long userId;
    private String deviceDetails;
    private String location;
    private Date lastLoggedIn;
    // ...
}
```

及其Repository：

```java
public interface DeviceMetadataRepository extends JpaRepository<DeviceMetadata, Long> {
    List<DeviceMetadata> findByUserId(Long userId);
}
```

有了的实体和Repository，我们就可以开始收集我们需要的信息来记录我们的用户设备及其位置。

## 4. 提取用户的位置

在估计用户的地理位置之前，我们需要提取他们的IP地址：

```java
private String extractIp(HttpServletRequest request) {
    String clientIp;
    String clientXForwardedForIp = request.getHeader("x-forwarded-for");
    if (nonNull(clientXForwardedForIp)) {
        clientIp = parseXForwardedHeader(clientXForwardedForIp);
    } else {
        clientIp = request.getRemoteAddr();
    }
    return clientIp;
}
```

如果请求中有X-Forwarded-For标头，我们将使用它来提取他们的IP地址；否则，我们将使用getRemoteAddr()方法。

一旦我们有了他们的IP地址，我们就可以在[Maxmind](https://www.baeldung.com/geolocation-by-ip-with-maxmind)的帮助下估计他们的位置：

```java
private String getIpLocation(String ip) {
    String location = UNKNOWN;
    InetAddress ipAddress = InetAddress.getByName(ip);
    CityResponse cityResponse = databaseReader.city(ipAddress);
        
    if (Objects.nonNull(cityResponse) && Objects.nonNull(cityResponse.getCity()) && !Strings.isNullOrEmpty(cityResponse.getCity().getName())) {
        location = cityResponse.getCity().getName();
    }    
    return location;
}
```

## 5. 用户设备详情

由于User-Agent标头包含我们需要的所有信息，因此只需提取它即可。正如我们之前提到的，在User-Agent解析器(本例中为[uap-java](https://central.sonatype.com/artifact/com.github.ua-parser/uap-java/1.5.4))的帮助下，获取此信息变得非常简单：

```java
private String getDeviceDetails(String userAgent) {
    String deviceDetails = UNKNOWN;
    
    Client client = parser.parse(userAgent);
    if (Objects.nonNull(client)) {
        deviceDetails = client.userAgent.family
            + " " + client.userAgent.major + "." 
            + client.userAgent.minor + " - "
            + client.os.family + " " + client.os.major
            + "." + client.os.minor; 
    }
    return deviceDetails;
}
```

## 6. 发送登录通知

要向我们的用户发送登录通知，我们需要将提取的信息与过去的数据进行比较，以检查我们过去是否已经在该位置看到过该设备。

让我们来看看DeviceService.verifyDevice()方法：

```java
public void verifyDevice(User user, HttpServletRequest request) {
    String ip = extractIp(request);
    String location = getIpLocation(ip);

    String deviceDetails = getDeviceDetails(request.getHeader("user-agent"));
        
    DeviceMetadata existingDevice = findExistingDevice(user.getId(), deviceDetails, location);
        
    if (Objects.isNull(existingDevice)) {
        unknownDeviceNotification(deviceDetails, location, ip, user.getEmail(), request.getLocale());

        DeviceMetadata deviceMetadata = new DeviceMetadata();
        deviceMetadata.setUserId(user.getId());
        deviceMetadata.setLocation(location);
        deviceMetadata.setDeviceDetails(deviceDetails);
        deviceMetadata.setLastLoggedIn(new Date());
        deviceMetadataRepository.save(deviceMetadata);
    } else {
        existingDevice.setLastLoggedIn(new Date());
        deviceMetadataRepository.save(existingDevice);
    }
}
```

提取信息后，我们将其与现有的DeviceMetadata条目进行比较，以检查是否存在包含相同信息的条目：

```java
private DeviceMetadata findExistingDevice(Long userId, String deviceDetails, String location) {
    List<DeviceMetadata> knownDevices = deviceMetadataRepository.findByUserId(userId);
    
    for (DeviceMetadata existingDevice : knownDevices) {
        if (existingDevice.getDeviceDetails().equals(deviceDetails) && existingDevice.getLocation().equals(location)) {
            return existingDevice;
        }
    }
    return null;
}
```

如果没有，我们需要向用户发送通知，让他们知道我们在他们的帐户中检测到不熟悉的活动。然后，我们持久化信息。

否则，我们只需更新熟悉设备的lastLoggedIn属性。

## 7. 总结

在本文中，我们演示了如何在检测到用户帐户中有不熟悉的活动时发送登录通知。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)上获得。
---
layout: post
title: 更新你的密码
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在这篇简短的文章中，我们将实现一个简单的“更改我自己的密码”功能，该功能在用户注册和登录后可供用户使用。

## 2. 客户端-更改我的密码页面

让我们看一下非常简单的客户端页面：

```html

<html>
<body>
<div id="errormsg" style="display:none"></div>
<div>
    <input id="oldpass" name="oldpassword" type="password"/>
    <input id="pass" name="password" type="password"/>
    <input id="passConfirm" type="password"/>
    <span id="error" style="display:none">Password mismatch</span>

    <button type="submit" onclick="savePass()">Change Password</button>
</div>

<script src="jquery.min.js"></script>
<script type="text/javascript">

    var serverContext = [[@{/}]];
        function savePass() {
            var pass = $("#pass").val();
            var valid = pass == $("#passConfirm").val();
            if (!valid) {
                $("#error").show();
                return;
            }
            $.post(serverContext + "user/updatePassword",
                    {password: pass, oldpassword: $("#oldpass").val()}, function (data) {
                        window.location.href = serverContext + "/home.html?message=" + data.message;
                    })
                    .fail(function (data) {
                        $("#errormsg").show().html(data.responseJSON.message);
                    });
        }
</script>
</body>
</html>
```

## 3. 更新用户密码

现在让我们也实现服务器端操作：

```java
@PostMapping("/user/updatePassword")
@PreAuthorize("hasRole('READ_PRIVILEGE')")
public GenericResponse changeUserPassword(Locale locale,@RequestParam("password") String password,@RequestParam("oldpassword") String oldPassword){
        User user=userService.findUserByEmail(SecurityContextHolder.getContext().getAuthentication().getName());
        if(!userService.checkIfValidOldPassword(user,oldPassword)){
        throw new InvalidOldPasswordException();
        }
        userService.changeUserPassword(user,password);
        return new GenericResponse(messages.getMessage("message.updatePasswordSuc",null,locale));
        }
```

请注意该方法是如何通过@PreAuthorize注解来保护的，因为**它应该只对登录用户可用**。

## 4. API测试

最后，让我们通过一些API测试来使用API，以确保一切正常；我们将从测试的简单配置和数据初始化开始：

```java

@ExtendWith(SpringExtension.class)
@ContextConfiguration(
        classes = {ConfigTest.class, PersistenceJPAConfig.class},
        loader = AnnotationConfigContextLoader.class)
public class ChangePasswordApiTest {
    private final String URL_PREFIX = "http://localhost:8080/";
    private final String URL = URL_PREFIX + "/user/updatePassword";

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    FormAuthConfig formConfig = new FormAuthConfig(URL_PREFIX + "/login", "username", "password");

    @BeforeEach
    public void init() {
        User user = userRepository.findByEmail("test@test.com");
        if (user == null) {
            user = new User();
            user.setFirstName("Test");
            user.setLastName("Test");
            user.setPassword(passwordEncoder.encode("test"));
            user.setEmail("test@test.com");
            user.setEnabled(true);
            userRepository.save(user);
        } else {
            user.setPassword(passwordEncoder.encode("test"));
            userRepository.save(user);
        }
    }
}
```

现在-让我们尝试**更改已登录用户的密码**：

```java
@Test
public void givenLoggedInUser_whenChangingPassword_thenCorrect(){
        RequestSpecification request=RestAssured.given().auth()
        .form("test@test.com","test",formConfig);

        Map<String, String> params=new HashMap<String, String>();
        params.put("oldpassword","test");
        params.put("password","newtest");

        Response response=request.with().params(params).post(URL);

        assertEquals(200,response.statusCode());
        assertTrue(response.body().asString().contains("Password updated successfully"));
        }
```

接下来-让我们尝试在**错误的旧密码**下更改密码：

```java
@Test
public void givenWrongOldPassword_whenChangingPassword_thenBadRequest(){
        RequestSpecification request=RestAssured.given().auth()
        .form("test@test.com","test",formConfig);

        Map<String, String> params=new HashMap<String, String>();
        params.put("oldpassword","abc");
        params.put("password","newtest");

        Response response=request.with().params(params).post(URL);

        assertEquals(400,response.statusCode());
        assertTrue(response.body().asString().contains("Invalid Old Password"));
        }
```

最后-让我们尝试在**没有身份验证**的情况下更改密码：

```java
@Test
public void givenNotAuthenticatedUser_whenChangingPassword_thenRedirect(){
        Map<String, String> params=new HashMap<String, String>();
        params.put("oldpassword","abc");
        params.put("password","xyz");

        Response response=RestAssured.with().params(params).post(URL);

        assertEquals(302,response.statusCode());
        assertFalse(response.body().asString().contains("Password updated successfully"));
        }
```

请注意，对于每个测试，我们如何提供一个FormAuthConfig来处理身份验证。

我们还通过init()重置密码，以确保我们在测试前使用正确的密码。

## 5. 总结

这是一个包装-一种允许用户在注册并登录到应用程序后更改自己的密码的直接方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)
上获得。
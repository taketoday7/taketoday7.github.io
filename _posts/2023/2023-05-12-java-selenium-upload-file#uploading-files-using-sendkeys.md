---
layout: post
title:  在Java中使用Selenium WebDriver上传文件
category: selenium
copyright: selenium
excerpt: Selenium WebDriver
---

## 1. 概述

[Selenium WebDriver](https://www.baeldung.com/java-selenium-with-junit-and-testng)是一个工具，可以自动执行用户与Web浏览器的交互以测试Web应用程序。它自动执行文件上传、[获取输入值](https://www.baeldung.com/java-selenium-html-input-value)、抓取HTML内容等过程。

在本教程中，我们将探讨如何使用Selenium中的sendKeys()方法上传文件。

## 2. 使用sendKeys()上传文件

简而言之，文件上传是许多Web应用程序中的常见功能。但是，使用Selenium WebDriver测试文件上传可能很棘手，因为它涉及与操作系统的本机文件系统交互。为了克服这个挑战，我们可以使用sendKeys()方法。

sendKeys()方法有助于模拟键盘操作。此方法可以将数据作为输入发送到HTML中的表单元素。

sendKeys()接收字符串作为参数并将其插入到选定的HTML元素中，这是自动化测试中的一种重要方法。常见用例包括填写Web表单和在网页上搜索特定元素。

在本教程中，我们将使用sendKeys()来填写Web表单，重点是将文件上传到网页。让我们看一个使用sendKeys()上传图像文件的例子：

```java
class FileUploadWebDriverLiveTest {

    private WebDriver driver;

    private static final String URL = "http://www.csm-testcenter.org/test?do=show&subdo=common&test=file_upload";
    private static final String INPUT_NAME = "file_upload";

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        driver = new ChromeDriver(options);
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }

    @Test
    void givenFileUploadPage_whenInputFilePath_thenFileUploadEndsWithFilename() {
        driver.get(URL);

        String filePath = System.getProperty("user.dir") + "/1688web.png";
        WebElement inputElement = driver.findElement(By.name(INPUT_NAME));
        WebElement submitButton = driver.findElement(By.name("http_submit"));

        inputElement.sendKeys(filePath);

        String actualFilePath = inputElement.getAttribute("value");
        String fileName = filePath.substring(filePath.lastIndexOf("/") + 1);

        submitButton.click();

        assertTrue(actualFilePath.endsWith(fileName));
    }
}
```

首先，我们将[WebDriver](https://www.baeldung.com/java-selenium-html-input-value#setting-up-a-selenium-webdriver-project)配置为使用Google Chrome并编写一个teardown()方法以在测试完成后关闭浏览器。接下来，我们声明一个名为URL的字段，其中包含我们可以上传文件的URL [http://www.csm-testcenter.org/test?do=show&subdo=common&test=file_upload](http://www.csm-testcenter.org/test?do=show&subdo=common&test=file_upload)。然后我们找到接收文件的HTML输入元素的名称属性。该图像位于项目的根目录中。

此外，我们创建WebElement实例来访问输入字段的name属性和提交按钮。此外，我们指定文件路径并调用inputElement上的sendKeys()方法以在输入字段中输入图像路径。

最后，我们通过在submitButton上执行鼠标单击来启动上传操作，并验证上传的文件是否与原始文件具有相同的名称和扩展名。

## 3. 总结

在本文中，我们学习了如何使用Selenium WebDriver上传文件。此外，我们使用sendKeys()方法将命令发送到HTML输入元素。此技能对于自动化Web测试和与不同类型的Web元素交互非常有用。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/selenium-webdriver)上找到。
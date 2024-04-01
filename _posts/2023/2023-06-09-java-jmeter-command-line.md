---
layout: post
title:  从命令行运行JMeter.jmx文件并配置报告文件
category: load
copyright: load
excerpt: JMeter
---

## 1. 概述

[Apache JMeter](https://jmeter.apache.org/)是一个基于Java的开源应用程序，旨在分析和测量Web应用程序的性能。它基本上是一个应用程序，我们可以使用它来测试和分析服务器在不同负载条件下的整体性能。

**[JMeter](https://www.baeldung.com/jmeter)提供了一个易于使用的GUI，我们可以使用它来定义、执行和查看各种负载测试的报告。它还支持非GUI模式，我们可以通过命令行接口运行脚本**。

在本教程中，我们将学习如何在配置报告文件时从命令行运行JMeter JMX文件。

## 2. 设置

在开始之前，让我们设置一个JMeter脚本，我们将在整个演示过程中使用该脚本。为了模拟某些API，我们将使用https://postman-echo.com提供的示例REST端点。

首先，我们将在JMeter中创建一个带有[线程组](https://www.baeldung.com/jmeter-run-multiple-thread-groups)的测试计划。其次，我们将向我们的线程组添加一个HTTP请求采样器，该采样器使用Postman回显服务提供的模拟端点。最后，我们将向线程组添加一个摘要报告，用于为测试运行生成摘要。

让我们详细看看这些步骤中的每一个。

### 2.1 创建JMeter测试计划和线程组

我们将使用JMeter GUI来设置我们的Test Plan。首先，让我们在Test Plan中添加一个Thread Group：

![](/assets/images/2023/load/javajmetercommandline01.png)

此外，我们将线程组配置为使用5个线程，这些线程的启动周期为1秒，循环计数为10：

![](/assets/images/2023/load/javajmetercommandline02.png)

### 2.2 配置HTTP请求采样器

现在，让我们在这个Thread Group中创建一个HTTP Request Sampler：

![](/assets/images/2023/load/javajmetercommandline03.png)

让我们配置HTTP Sampler以使用Postman回显服务提供的GET端点：

![](/assets/images/2023/load/javajmetercommandline04.png)

### 2.3 配置摘要报告监听器

最后，让我们向Thread Group添加一个Summary Report监听器，用于汇总测试计划的结果：

![](/assets/images/2023/load/javajmetercommandline05.png)

这样，我们就准备好了基本的JMeter脚本。让我们在GUI中运行它并查看生成的摘要报告：

![](/assets/images/2023/load/javajmetercommandline06.png)

显然，在运行JMeter脚本时，我们能够成功地为我们的测试配置生成一个摘要报告。我们将其保存为“Summary-Report.jmx”并使用它从命令行运行测试。

## 3. 从命令行运行JMeter.jmx文件

到目前为止，我们有一个带有测试计划的示例JMX文件，该文件配置为运行具有示例模拟API的测试。

如前所述，JMeter JMX文件可以使用命令行接口在非GUI模式下运行。**我们使用带有选项的jmeter命令从CLI运行JMeter脚本文件。可以通过将目录更改为JMeter安装路径中的bin目录来运行此命令**。

让我们看一下运行“Summary-Report.jmx”文件并生成测试报告的CLI命令：

```shell
$ jmeter -n -t Summary-Report.jmx -l Summary-Report.jtl
```

运行上述命令会在CLI上生成以下输出：

```shell
Creating summariser <summary>
Created the tree successfully using /Users/tuyucheng/Summary-Report.jmx
Starting standalone test @ 2023 Jun 3 13:32:11 IST (1685779331706)
Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
summary =     50 in 00:00:08 =    6.6/s Avg:   499 Min:   193 Max:  3601 Err:     0 (0.00%)
Tidying up ...    @ 2023 Jun 3 13:32:19 IST (1685779339273)
... end of run
```

该命令基本上运行我们的测试文件“Summary-Report.jmx”并在“Summary-Report.jtl”文件中生成一个总结报告。

JMeter CLI提供了多个选项来配置脚本的运行方式，我们使用这些参数来覆盖JMX文件中设置的参数。让我们看一下我们在脚本中使用的一些选项：

-   -n：指定JMeter以非GUI模式运行
-   -t：指定包含测试计划的JMX文件的位置
-   -l：指定JTL(JMeter测试日志)文件的位置

此外，让我们看一下“Summary-Report.jtl”文件内容的前几行：

```shell
timeStamp,elapsed,label,responseCode,responseMessage,threadName,dataType,success,failureMessage,bytes,sentBytes,grpThreads,allThreads,URL,Latency,IdleTime,Connect
1685779331808,978,HTTP Request,200,OK,Thread Group 1-1,text,true,,681,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,974,0,780
1685779331959,827,HTTP Request,200,OK,Thread Group 1-2,text,true,,679,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,823,0,625
1685779332163,786,HTTP Request,200,OK,Thread Group 1-3,text,true,,681,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,785,0,589
1685779332787,194,HTTP Request,200,OK,Thread Group 1-2,text,true,,681,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,194,0,0
```

我们可以看到生成的JTL文件以CSV格式包含每个API请求的详细信息。

## 4. 配置JMeter摘要报告

我们可以自定义摘要报告中显示的参数及其格式选项，有两种方法可以自定义摘要报告：

-   更新JMeter属性
-   将参数作为命令行选项传递

### 4.1 配置JMeter属性

JMeter属性文件“jmeter.properties”位于JMeter安装目录的bin目录中。属性文件中与摘要报告相关的属性以“jmeter.save.saveservice”前缀开头：

```properties
#jmeter.save.saveservice.output_format=csv
#jmeter.save.saveservice.response_code=true
#jmeter.save.saveservice.latency=true
```

默认情况下，这些属性是注释的，我们可以取消注释来设置它们的值。让我们取消注释一些属性并更新它们的值：

```properties
jmeter.save.saveservice.response_code=false
jmeter.save.saveservice.latency=false
jmeter.save.saveservice.output_format=xml
```

现在重新运行JMeter脚本会生成以下结果：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testResults version="1.2">
    <httpSample t="801" it="0" ct="593" ts="1685791090172" s="true" lb="HTTP Request" rm="OK" tn="Thread Group 1-3"
                dt="text" by="679" sby="143" ng="5" na="5">
        <java.net.URL>https://postman-echo.com/get?foo1=bar1&foo2=bar2</java.net.URL>
    </httpSample>
    <httpSample t="1004" it="0" ct="782" ts="1685791089969" s="true" lb="HTTP Request" rm="OK" tn="Thread Group 1-2"
                dt="text" by="679" sby="143" ng="5" na="5">
        <java.net.URL>https://postman-echo.com/get?foo1=bar1&foo2=bar2</java.net.URL>
    </httpSample>
</testResults>
```

从结果中可以看出，输出格式现在是XML形式。此外，我们观察到响应代码和延迟字段现在不会在输出中生成。

### 4.2 JMeter属性作为CLI选项

**我们可以通过命令行将它们作为选项传递来覆盖“jmeter.properties”文件中设置的属性**：

```shell
$ jmeter -Jjmeter.save.saveservice.output_format=csv -n -t Summary-Report.jmx -l Summary-Report.jtl
```

在这里，**我们使用-J选项在运行测试脚本时传递和覆盖任何现有的JMeter属性**。

该命令现在生成以下结果：

```shell
timeStamp,elapsed,label,responseMessage,threadName,dataType,success,failureMessage,bytes,sentBytes,grpThreads,allThreads,URL,IdleTime,Connect
1685792003144,961,HTTP Request,OK,Thread Group 1-1,text,true,,685,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,0,741
1685792003306,799,HTTP Request,OK,Thread Group 1-2,text,true,,681,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,0,599
1685792004106,200,HTTP Request,OK,Thread Group 1-2,text,true,,681,143,5,5,https://postman-echo.com/get?foo1=bar1&foo2=bar2,0,0
```

因此，我们通过将参数作为CLI选项传递，成功地将输出格式从XML覆盖为CSV。

## 5. 总结

在本文中，我们学习了如何从命令行运行JMeter JMX文件并配置摘要报告文件。

首先，我们研究了如何使用jmeter命令及其一些选项来运行JMX文件。然后我们看到了如何通过配置jmeter.properties文件中的属性来配置摘要报告。最后，我们讨论了如何通过命令行将它们作为选项传递来覆盖这些属性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/jmeter)上获得。
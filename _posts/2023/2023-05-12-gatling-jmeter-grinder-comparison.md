---
layout: post
title:  负载测试工具的对比
category: load
copyright: load
excerpt: JMeter
---

## 一、简介

为工作选择正确的工具可能会令人生畏。在本教程中，我们将通过比较三个 Web 应用程序负载测试工具（Apache JMeter、Gatling 和 The Grinder）与一个简单的 REST API 来简化这一点。

## 2.负载测试工具

首先，让我们快速回顾一下每个方面的一些背景。

### 2.1。加特林

[Gatling](https://gatling.io/)是一个负载测试工具，可以在 Scala 中创建测试脚本。**Gatling 的记录器生成 Scala 测试脚本，这是 Gatling 的一个关键特性。** 查看我们[的 Gatling 简介](https://www.baeldung.com/introduction-to-gatling) 教程以获取更多信息。

### 2.2. JMeter

[JMeter](https://jmeter.apache.org/)是 Apache 的负载测试工具。它提供了一个不错的 GUI，我们可以使用它来进行配置。**称为逻辑控制器的独特功能为在 GUI 中设置测试提供了极大的灵活性。**

请访问我们 [的 JMeter 简介](https://www.baeldung.com/jmeter) 教程以获取屏幕截图和更多说明。

### 2.3. 磨床

我们的最后一个工具[The Grinder](http://grinder.sourceforge.net/)提供了比其他两个更基于编程的脚本引擎，并使用了 Jython。但是，The Grinder 3 确实具有录制脚本的功能。

Grinder 还与其他两个工具不同，它允许控制台和代理进程。此功能为代理进程提供了能力，以便负载测试可以跨多个服务器扩展。 **它专门宣传为为开发人员构建的负载测试工具，用于发现死锁和减速。**

## 3. 测试用例设置

接下来，对于我们的测试，我们需要一个 API。我们的 API 功能包括：

-   添加/更新奖励记录
-   查看一个/所有奖励记录
-   将交易链接到客户奖励记录
-   查看客户奖励记录的交易

**我们的场景：**

一家商店正在全国范围内进行销售，新客户和回头客需要客户奖励账户来节省开支。奖励 API 通过客户 ID 检查客户奖励帐户*。*如果不存在奖励帐户，请添加它，然后链接到交易。

之后，我们查询交易。

### **3.1。我们的 REST API**

让我们通过查看一些方法存根来快速了解 API：

```java
@PostMapping(path="/rewards/add")
public @ResponseBody RewardsAccount addRewardsAcount(@RequestBody RewardsAccount body)

@GetMapping(path="/rewards/find/{customerId}")
public @ResponseBody Optional<RewardsAccount> findCustomer(@PathVariable Integer customerId)

@PostMapping(path="/transactions/add")
public @ResponseBody Transaction addTransaction(@RequestBody Transaction transaction)

@GetMapping(path="/transactions/findAll/{rewardId}")
public @ResponseBody Iterable<Transaction> findTransactions(@PathVariable Integer rewardId)
复制
```

请注意一些关系，例如通过奖励 id 查询交易和通过客户 id 获取奖励帐户*。* 这些关系为我们的测试场景创建强制执行一些逻辑和一些响应解析。

被测应用程序还使用 H2 内存数据库进行持久化。

**幸运的是，我们的工具都能很好地处理它，有些比其他的要好。**

### 3.2. 我们的测试计划

接下来，我们需要测试脚本。

为了公平比较，我们将为每个工具执行相同的自动化步骤：

1.  生成随机客户帐户 ID
2.  发布交易
3.  解析随机客户 ID 和交易 ID 的响应
4.  使用客户 ID 查询客户奖励帐户 ID
5.  解析奖励帐户 ID 的响应
6.  如果不存在奖励帐户 ID，则添加一个带有帖子的帐户
7.  使用交易 ID 发布相同的初始交易并更新奖励 ID
8.  通过奖励账户id查询所有交易

让我们仔细看看每个工具的第 4 步。并且，请务必查看[所有三个已完成脚本的示例](https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts)。

### 3.3. 加特林

**对于 Gatling 而言，熟悉 Scala 为开发人员带来了福音，因为 Gatling API 很健壮并且包含很多特性。**

Gatling 的 API 采用了构建器 DSL 方法，正如我们在其第 4 步中所见：

```scala
.exec(http("get_reward")
  .get("/rewards/find/${custId}")
  .check(jsonPath("$.id").saveAs("rwdId")))
复制
```

特别值得注意的是，当我们需要读取和验证 HTTP 响应时，Gatling 对 JSON 路径的支持。在这里，我们将获取奖励 id 并将其保存到 Gatling 的内部状态。

此外，Gatling 的表达式语言使动态请求正文*字符串更容易：*

```javascript
.body(StringBody(
  """{ 
    "customerRewardsId":"${rwdId}",
    "customerId":"${custId}",
    "transactionDate":"${txtDate}" 
  }""")).asJson)
复制
```

最后是我们用于此比较的配置。1000 次运行设置为整个场景的重复，*atOnceUsers* 方法设置线程/用户：

```scala
val scn = scenario("RewardsScenario")
  .repeat(1000) {
  ...
  }
  setUp(
    scn.inject(atOnceUsers(100))
  ).protocols(httpProtocol)复制
```

**整个[Scala 脚本](https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts) 可在我们的 Github 存储库中查看。**

### 3.4. JMeter

**JMeter 在 GUI 配置后生成一个 XML 文件。**该文件包含具有设置属性及其值的 JMeter 特定对象，例如：

```xml
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Add Transaction" enabled="true">复制
<JSONPostProcessor guiclass="JSONPostProcessorGui" testclass="JSONPostProcessor" testname="Transaction Id Extractor" enabled="true">复制
```

查看*testname*属性，当我们识别出它们与上述逻辑步骤匹配时，它们可以被标记。添加子项、变量和依赖项步骤的能力为 JMeter 提供了脚本所提供的灵活性。此外，我们甚至设置了变量的范围！

我们在 JMeter 中运行和用户的配置使用*ThreadGroups*：

```xml
<stringProp name="ThreadGroup.num_threads">100</stringProp>复制
```

[查看整个 *jmx*文件作为参考](https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts)。虽然可能，但在 XML 中将测试编写为*.jmx*文件对于功能齐全的 GUI 没有意义。

### 3.5. 磨床

如果没有 Scala 和 GUI 的函数式编程，我们的 The Grinder 的 Jython 脚本看起来非常基础。添加一些系统 Java 类，我们的代码行数就会少很多。

```python
customerId = str(random.nextInt());
result = request1.POST("http://localhost:8080/transactions/add",
  "{"'"customerRewardsId"'":null,"'"customerId"'":"+ customerId + ","'"transactionDate"'":null}")
txnId = parseJsonString(result.getText(), "id")复制
```

但是，更少的测试设置代码行需要更多的字符串维护代码（例如解析 JSON 字符串）来平衡。此外，[HTTPRequest](http://grinder.sourceforge.net/g3/script-javadoc/net/grinder/plugin/http/HTTPRequest.html) API 的功能也很精简。

使用 The Grinder，我们在外部属性文件中定义线程、进程和运行值：

```shell
grinder.threads = 100
grinder.processes = 1
grinder.runs = 1000复制
```

**我们为 The Grinder 编写的完整 Jython 脚本将如下[所示](https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts)。**

## 4. 试运行

### 4.1。测试执行

**这三个工具都建议使用命令行进行大型负载测试。**

为了运行测试，我们将使用 Gatling[开源](https://gatling.io/open-source/)版本 3.4.0 作为独立工具、[JMeter 5.3](https://jmeter.apache.org/download_jmeter.cgi)和 The Grinder[版本 3](https://sourceforge.net/projects/grinder/)。

Gatling 只需要我们 *设置 JAVA_HOME*和*GATLING_HOME*。要执行 Gatling，我们使用：

```bash
./gatling.sh复制
```

在 GATLING_HOME/bin 目录中。

JMeter在启动GUI进行配置时需要一个参数来禁用测试的GUI：

```bash
./jmeter.sh -n -t TestPlan.jmx -l log.jtl复制
```

与 Gatling 一样，The Grinder 要求我们设置*JAVA_HOME*和*GRINDERPATH*。但是，它还需要更多属性：

```bash
export CLASSPATH=/home/lore/Documents/grinder-3/lib/grinder.jar:$CLASSPATH
export GRINDERPROPERTIES=/home/lore/Documents/grinder-3/examples/grinder.properties复制
```

如上所述，我们提供了一个*grinder.properties*文件用于附加配置，例如线程、运行、进程和控制台主机。

最后，我们使用以下命令引导控制台和代理：

```bash
java -classpath $CLASSPATH net.grinder.Console复制
java -classpath $CLASSPATH net.grinder.Grinder $GRINDERPROPERTIES复制
```

### 4.2. 试验结果

每个测试使用 100 个用户/线程运行 1000 次。让我们解开一些亮点：

|            | **成功的请求** | **错误** | **总测试时间（秒）** | **平均响应时间（毫秒）** | **平均吞吐量** |
| ---------- | -------------- | -------- | -------------------- | ------------------------ | -------------- |
| **加特林** | 500000 个请求  | 0        | 218s                 | 42                       | 2283 个请求/秒 |
| **JMeter** | 499997 个请求  | 0        | 237s                 | 46                       | 2101 个请求/秒 |
| **磨床**   | 499997 个请求  | 0        | 221s                 | 43                       | 2280 个请求/秒 |

**结果显示这 3 个工具具有相似的速度，基于平均吞吐量，Gatling 略微超过其他 2 个。** 

每个工具还在更友好的用户界面中提供附加信息。

**Gatling 将在运行结束时生成一个 HTML 报告，其中包含多个图表和统计信息，用于总运行以及每个请求。**以下是测试结果报告的片段：

[![加特林结果新 1](https://www.baeldung.com/wp-content/uploads/2018/11/gatling-result-new-1.png)](https://www.baeldung.com/wp-content/uploads/2018/11/gatling-result-new-1.png)

**使用 JMeter 时，我们可以在测试运行后打开 GUI，并根据**我们保存结果的日志文件生成 HTML 报告：

[![2020/10/jmeter-result-new.png](https://www.baeldung.com/wp-content/uploads/2020/10/jmeter-result-new.png)](https://www.baeldung.com/wp-content/uploads/2020/10/jmeter-result-new.png)

JMeter HTML 报告还包含每个请求的统计信息细分。

**最后，Grinder 控制台记录每个代理的统计信息并运行：**

[![研磨机结果新 1](https://www.baeldung.com/wp-content/uploads/2020/10/grinder-result-new-1.png)](https://www.baeldung.com/wp-content/uploads/2020/10/grinder-result-new-1.png)

虽然 The Grinder 是高速的，但它的代价是额外的开发时间和更少的输出数据多样性。

## 五、总结

现在是时候全面了解每个负载测试工具了。

|              | **加特林** | **JMeter** | **磨床** |
| ------------ | ---------- | ---------- | -------- |
| 项目和社区   | 9          | 9          | 6        |
| 表现         | 9          | 8          | 9        |
| 脚本能力/API | 7          | 9          | 8        |
| 用户界面     | 9          | 8          | 6        |
| 报告         | 9          | 7          | 6        |
| 一体化       | 7          | 9          | 7        |
| **概括**     | **8.3**    | **8.3**    | **7**    |

**加特林：**

-   可靠、完善的负载测试工具，可使用 Scala 脚本输出漂亮的报告
-   产品的开源和企业支持级别

**JMeter：**

-   用于测试脚本开发的强大 API（通过 GUI），无需编码
-   Apache Foundation 支持以及与 Maven 的出色集成

**研磨机：**

-   为使用 Jython 的开发人员提供的快速性能负载测试工具
-   跨服务器可扩展性为大型测试提供了更多潜力

简而言之，如果需要速度和可扩展性，请使用 The Grinder。

如果漂亮的交互式图表有助于显示性能提升以支持更改，那么请使用 Gatling。

JMeter 是用于复杂业务逻辑的工具或具有多种消息类型的集成层。作为 Apache 软件基金会的一部分，JMeter 提供了成熟的产品和大型社区。

## 六，结论

总之，我们看到这些工具在某些领域具有类似的功能，而在其他领域则大放异彩。正确工作的正确工具是适用于软件开发的口语智慧。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/load-testing-comparison)上获得。
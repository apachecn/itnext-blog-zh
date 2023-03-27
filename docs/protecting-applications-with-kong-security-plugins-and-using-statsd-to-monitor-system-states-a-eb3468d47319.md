# 使用 Kong 安全插件保护应用程序并使用 StatsD 监控系统状态——一个健康的相机故事

> 原文：<https://itnext.io/protecting-applications-with-kong-security-plugins-and-using-statsd-to-monitor-system-states-a-eb3468d47319?source=collection_archive---------0----------------------->

![](img/c508f5170097469f70e8ba5635f16010.png)

在当前开发软件时，我们都面临的一个问题是监控和检查应用程序的需要和意愿。除了使用度量来研究和调查我们的应用程序，我们还可以将它们用于其他方面，比如通知和警报。为了实现这个目标，**孔门户**提供了许多我们可以探索的免费选项。在本文中，我将尝试展示免费的 **Kong Gateway** 插件的几个重要方面，这些插件可以帮助我们检测我们的 **Java/Kotlin** 实现中的问题。我将尝试使用最先进和最复杂的方法来实现这两种语言的算法，以及孔可以如何帮助我们。对于本文，我们的真实生活环境是保护珍贵物品的摄像机。特别是，我们将保护一种植物。如果你不知道那是什么，这是我自己种的这种植物的照片:

![](img/9d264dc0a58741eeadb84b3f7533177d.png)

钝叶胡椒

那么，为什么是孔，又为什么要努力寻找它能做什么呢？Kong 框架虽然是一个付费的框架，但如果我们真的想很好地升级我们的产品(就像任何其他框架一样)，它确实允许我们在应用程序中实现很多东西，并为它提供安全性。

在这个例子的应用程序中，我们还将看看如何用 Kong 轻松地保护运行中不安全的应用程序。我们将看看没有网关的情况下如何工作的例子，什么时候使用它是个好主意，什么时候它可能是一件过犹不及的事情。

但现在，我只是想漫谈一下安全性，以及为什么安全性很重要，尤其是在当前这个时代，对于几乎每个人来说，除非你根本不使用互联网，没有电子邮件，没有电话，也不用电。

请注意，在你继续阅读之前，这篇文章在未来会有多次更新。支持这个的回购位于 [GitHub](https://github.com/jesperancinha/healthy-cameras) 上。

# **私人用户眼中的安全性**

如果我们看看今天的时代，我们生活在一个几乎不可能不谈网络安全的时代。每个人都在某种程度上受此影响。讨论最多的例子是欺诈。它可以采取许多形状或形式。我们可能会收到电子邮件，告诉我们一些关于我们的故事，承诺奖励和奖品，或者告诉我们我们有身体问题，我们需要点击一些链接以避免罚款、惩罚或其他任何事情。欺诈者无所不用其极，即使他们意识到我们注意到了，他们仍会继续使用规避技术发送电子邮件，如创建多个子域、创建带有哈希的电子邮件、发送带有 6 位数前缀组合的子域、使用陌生字符伪装成真实交易等。最糟糕的例子是当一个人收到一封来自真实域名的电子邮件，但它仍然只是一个骗局。也许它来自于那个组织的一个渗透者，或者也许它是一封伪装的电子邮件。不幸的是，当我们收到这种疯狂的电子邮件泛滥时，我们无法一下子就摆脱它们，因为欺诈者能够尽可能频繁地创建匿名电子邮件。我们应该继续将它们报告为网络钓鱼企图，当它们来自 Gmail 帐户时报告给谷歌，并阻止它们。虽然有时候，你似乎不能阻止他们，但你真的总是可以的。我收到了以上所有邮件，但也收到了没有发件人的邮件或域名前带点的邮件。然而，**所有的邮件都有标题，在标题中**，你可以找到**大量关于邮件来源的信息**。您的电子邮件提供商应该为您提供查看源邮件头的选项。在那里你会发现邮件来自哪里。清除这些邮件的一个简单方法是让你的邮件提供商使用你的**垃圾邮件文件夹**，或者**创建重定向规则**。然而，问题是，我们仍然会收到这些电子邮件，所以犯罪实体没有理由相信你永远不会对特定的电子邮件做出反应。如果你自动删除了那封邮件，你仍然没有给出任何迹象表明你明白什么是假邮件。如果你**屏蔽了邮件**或者整个**域名**，那就另当别论了。如果您的电子邮件提供商允许您阻止电子邮件，以至于它们甚至不会进入您的电子邮件收件箱，那么这也意味着犯罪实体将会收到电子邮件未送达的回复。不会有任何迹象表明你的电子邮件帐户存在，是阻止电子邮件，或其他任何事情。他们将得到的唯一数据只是他们的黑客企图没有进入你的电子邮件。你应该**永远不要**做的事情，是与发送者接触，责备发送者，或者把这些事情掌握在你自己手中。**千万不要这样！每次你把你的挫败感发送进来，你只是在给犯罪实体提供额外的信息。务必阅读邮件头，在邮件头的发件人字段中阻止发件人发送电子邮件，并至少使用网络钓鱼选项来报告您的电子邮件。一些提供商举报犯罪邮件的级别更高。有时候，向本国或国际上负责网络安全的第三方报告也是一个好主意。我已经建立了一个回购协议，在那里我不断开发开源代码来帮助抵御这种现象。可以在 [GitHub](https://github.com/jesperancinha/jeorg-security-defense-test-drives) 上找到。**

他们会放弃吗？不幸的是，让他们放弃可能相当困难。不幸的是，网络犯罪确实在增加，为了对此做些什么，我们必须明白这是我们新生活的一部分，接受它并共同努力阻止它。

这和这篇文章有什么关系？潜在的信息是，为了保护我们的应用程序，我们需要考虑到在最糟糕的情况下，威胁参与者可能会非常顽固，他们会进入兔子洞，用尽他们所知道的一切来达到他们的目标。

对欺诈邮件的几点建议？为了确保您阻止了正确的邮件，请查看您的邮件标题。您的提供商应该可以选择这样做，并在您查看时获得的很长的文本中查找这些标题:

```
**From**: "possibly an actual email. this could be masked one so it's not always a good idea to block whatever is put here"
**To**: undisclosed-recipients:;
**Subject**: "whatever"
**Reply-To**: "this is the email you should block"
**Mail-Reply-To**: "this is the other email you should block"
**In-Reply-To**: "block this one too"
**References**: "whatever"
**Message-ID**: "whatever"
```

如果您需要知道这封电子邮件是从哪个国家和地点发送到您这里的，请查找这些邮件头:

```
**Authentication-Results**: spf=pass (sender IP is "this is the IP you are looking for")
```

或者你也可以在这个标题上查找:

```
**X-Sender-IP**: "this is the IP you are looking for"
```

然后，您可以选择此 IP 并在以下位置查找:

 [## IP WHOIS 查找

### IP WHOIS 查找确定任何 IP 地址的所有权信息。使用 IP 搜索 IP WHOIS 信息…

www.whatismyip.com](https://www.whatismyip.com/ip-whois-lookup/) 

# 公共服务眼中的安全

今天，有几种不同的算法可以让我们创建非常安全的应用程序访问。我们可以拥有完全基于用户名/密码组合、共享机密、令牌、最新完整性检查和第三方授权码的安全系统。此外，我们还可以使用可用于不同监控应用的标准来监控我们的应用。这意味着，一方面，我们必须负责保护我们的应用程序和网关。另一方面，我们必须确保无论何时出现安全漏洞，我们都能检测到。在这篇文章中，我们将看看这个等式的两边。

# 1 —简介

我们会去看一看，就像之前提到的监测植物的情况。为此，我们有 6 个摄像头，或者至少 6 个模拟摄像头，每个摄像头都有自己的安全算法。摄像机 1 运行的是一种称为 StatsD 的公制系统。摄像机编号从 1 到 6，它们各自的安全实现的安全算法是:基本、HMAC、JWT、ApiKey、LDAP 和 OAuth2。其工作方式是在 Kong 网关后面运行一个完全不受保护的服务。出于本文的目的，我们不打算深入研究入口控制器或其他将我们的域与外界隔离的方法。目前，我们只是使用最简单形式的孔网关。最后，我们将快速浏览一下如何使用 StatsD 处理数据。

# **2 —通用概述**

为了启动我们的系统，我们将简单地创建一个 Spring Boot 服务，在那里我们将提供一个欢迎消息和一个端点，该端点将简单地给出一个图像。我们展示的形象和受欢迎的终点没有受到任何保护。在这篇文章中，我们将简单地看看如何通过孔来保护它们。以下是我们为我们的案例实现的可视化表示:

![](img/3f636378defecd101024074c6dffcab6.png)

系统简图

在我们的系统中，我们将有 6 台摄像机。它们都以完全相同的方式实现。因为理解正在发生的事情很重要，所以让我们首先深入服务的后端代码。为此，我们将涵盖几个主要部分:StatsD 定制指标、Rest 服务实现、反应式 WebSockets 实现，以及最后与 OAuth2 相关的安全标记。

# 3.— StatsD

# 3.1.—自定义 StatsD 标记

为了开始使用 StatsD，我们需要了解它的基本工作原理。 **StatsD** 最初由 [**Etsy**](https://www.etsy.com/codeascraft/measure-anything-measure-everything) 开发，用于许多需要监控应用的应用开发。许多框架都提供了标准的 StatsD 度量标准，通常包括发出请求所需的时间、请求负载、响应时间和请求类型。我们通常关心我们的应用程序真正的响应速度，它的容量，它的弹性，以及它的反应速度。 [**StatsD**](https://github.com/statsd/statsd) 的设计正是为了让我们能够使用标准来执行这些类型的测量。网上有关于它开始于 2011 年**左右**的记录，但很可能在那之前几年就已经在制作了。在那之后，像**孔**和 **Datadog** 这样的主要竞争对手创建了插件和适配器，以便读取这些数据并以图表的形式呈现出来。这也是因为， **StatsD** 作为标准创建，也允许我们以标准方式创建**新的** **指标**。在讨论安全性之前，这也是本文的重点。**孔**为了消化 **StatsD** 信息，提供了两个插件。它有一个 [**StatsD**](https://docs.konghq.com/hub/kong-inc/statsd/) 常规插件和一个 [**StatsD 高级**](https://docs.konghq.com/hub/kong-inc/statsd-advanced/) 插件。后者用于更高级的企业应用程序。第一个是我们要看的。但在此之前，让我们看看纯粹通过**弹簧**和使用**石墨**作为服务来监听这些数据，从而使 **StatsD** 成为现实的必要部分:

![](img/585c4269a56b5ea91d48ff7ed26d6036.png)

自定义 StatsD 流向石墨

Graphite 为其 web 接口打开端口 8085。它不是有史以来最迷人的网页，但该服务的启动时间很长，并且不会消耗大量资源。因此它非常适合用于演示目的。流程从每个单独的摄像机开始，我们使用一个**千分尺**库以一种 **StatsD 识别的**格式发送我们的**普罗米修斯**数据。我们为我们的项目选择的格式是 **Etsy** ，它被 [**Graphite**](https://graphiteapp.org/) 所接受。那么，我们该怎么做呢？首先，[我们添加必要的库](https://github.com/jesperancinha/healthy-cameras):

![](img/ff4c76630f13f1d2fcb5e4d3ffa05c80.png)

StatsD 必要的依赖关系

这使我们能够配置 SringBoot 服务:

![](img/b15b728b81b18799034e8a461737bd58.png)

**跳趾的 StatsD 配置**

在这种配置中，我们说 SpringBoot 打开 **Prometheus** 和 **health** 端点。这两个非常重要。普罗米修斯端点是我们将要公开自定义指标的地方。显示**健康细节**可能很重要，因为通用**健康** **致动器**端点通常是不同健康端点的组合。在我们的例子中，我们也有数据库端点，因此，如果我们的服务没有变得健康，对正在发生的事情有一个完整的描述总是好的。此外，我们启用了 **StatsD** ，这将立即允许我们的 SpringBoot 服务将 **StatsD** 数据发送到目的地。我们在接下来的 2 个步骤中定义目的地，我们说**主机**是**本地主机**，而**端口**是 **8125** 。这个端口不是我们之前看到的 **8085 的 **GUI** 端口。**字段**味道**只是一种调用协议的有趣方式，我们的度量数据就是用它来发送的。有其他口味可以替代 Etsy。最后，我们定义了推送到 **Graphite** 的频率以及对我们已经实现的任何计量器的轮询周期。如果任何一个仪表显示的数值与之前的读数不同，StatsD 输出器将再次将所有数据发送回**石墨**。

现在我们已经配置好了一切，我们可以看看[代码](https://github.com/jesperancinha/healthy-cameras):

```
@Configuration
class MetricsConfiguration(
    val cameraService: CameraService,
    @Value("\${hc.camera.number}")
    val cameraNumber: Long,
) {
    private val last10FileDeltaNSReading: MutableList<Double> = mutableListOf()

    /**
     * You should see all fails happening every 10 seconds if your polling rate is every 10 seconds
     * For every 10 seconds you'll get all fails in that period
     */
    @Bean
    @Qualifier("counterMetric")
    fun counterMetric(meterRegistry: MeterRegistry) = Counter.builder("camera.fail.count")
        .tag("type", "fail_count")
        .description("The number of anomalies detected by the camera")
        .register(meterRegistry)

    /**
     * You should see all fails happening every 10 seconds if your polling rate is every 10 seconds
     * For every 10 seconds you'll get all heartbeats in the period
     */
    @Bean
    @Qualifier("heartBeats")
    fun heartBeatCounter(meterRegistry: MeterRegistry) = Counter.builder("camera.heartbeats")
        .tag("type", "heart_beats")
        .description("The number of heartbeats measured per time and counts 1 per second")
        .baseUnit("beats")
        .register(meterRegistry)

    /**
     * Measures the time it takes to read an image.
     * This is purely to illustrate how can we inject our own custom metrics in StatsD.
     */
    @Bean
    fun gaugeMetric(meterRegistry: MeterRegistry) =
        Gauge.builder("camera.image.read.time", last10FileDeltaNSReading)
        { last10Records ->
            logger.debug("$last10FileDeltaNSReading")
            measureNanoTime { runBlocking { cameraService.getImageByteArrayByCameraNumber(cameraNumber) } }.toDouble()
                .let { record ->
                    last10Records.add(record)
                    if (last10Records.size == 11) {
                        last10Records.removeFirst()
                    }
                    logger.info("Refreshed ${last10Records.size} metrics. Last value read is ${last10Records.last()} ns")
                    record
                }
        }
            .tag("type", "image_read_ns")
            .description("Time to read one image from camera in ns")
            .baseUnit("ns")
            .register(meterRegistry)

}
```

对于这种情况，我们正在创建一个名为`**counterMetric**` 的类型为`**Counter**`的 bean。这种类型的指标的功能是简单地统计某个事件的发生次数。我们可以通过编程来实现这一点，稍后我们将看到这是如何发生的。现在，我们只需要记住，bean `**counterMetric**` 和 bean `**heartBeatCounter**` 将分别计算访问具有错误范围的 **OAuth** 端点的失败尝试次数，另一个将计算每**秒**的 1 次**心跳**。最后，纯粹因为我们想以自己的方式制作一个奇怪的度量，我们创建了一个`**gaugeMetric**`类型`**Gauge**` **。这个计量器只会读取从相机读取图像所需的时间。`**Gauge**`是一种度量标准，与计数器不同，它测量在某个时间点读取的实际值。在这一点上，重要的是要意识到一个计量器**在我们定义的特定点处理**信息。我们将通过编程来实现这一点。因此，为了更好地说明**计数器**和**仪表**之间的区别，我们可以说计数器测量读数之间事件发生的次数。因此，在我们的具体例子中，我们将记录每秒 10 次心跳。我们的**计数器**只知道每秒钟计数加 1。然而，当将该信息发送到 **Graphite，**时，它将记录从最后一点开始计数的次数，或者如果没有读数，则记录到该点为止的次数。因此，如果在 graphite 中每 10 秒处理一次数据，我们将每 10 秒得到 10 个读数。每两分钟发送一次数据并不能改变这一点。**仪表，**则记录该场合的特定值。这是通过将数据保存在列表**中来实现的。然而，这个列表在向 **StatsD** 发送数据的过程中并没有真正发挥作用。相反，如果我们想使用一个公共对象来共享读数之间的数据，它就在那里。它的工作方式更像一个**豆**，我们可以插入任何我们想插入的东西，或者用它做任何我们想做的事情。我们传入的函数的结果对我们的**量规**度量很重要。让我们放大一下:****

![](img/db4eb9e76f3ca836abdb1b44b9fd9d30.png)

负责**的函数测量**读取一个相机文件需要多长时间

因此，在我们函数的接收器中，我们得到了一个 **last10Records** 对象，我们已经将它制作成了一个列表。然后我们测量读取一台相机需要多长时间。在这种情况下，我们必须以一种阻塞的方式来完成，纯粹是因为我们不在协程上下文中，而且我们想知道读取图像需要多长时间。如果我们通过类似于 **GlobalScope.launch** 的其他方式来做，我们将不会对读取图像所花费的时间有一个真实的想法，因为这样一来，协程将在线程之间展开，并且不可能对读取图像时会有多糟糕有一个真实的想法。我们将**记录**测量添加到 10 个记录的列表中，并确保我们总是保留最后 10 个记录。

现在我们已经有了度量标准，我们需要看看计数器是如何工作的，以及我们在哪里触发度量。我们先来看看类**CameraServiceApplication**:

![](img/4072ab1f77a1736b1fc2fe4ec437db2f.png)

**心跳发送**

这里需要理解的重要一点是，我们使用一个 Spring 调度程序来调用心跳计数器的递增函数。

所以现在我们知道**心跳**计数器和**度量**发生了什么。那么失败计数器呢？这位于服务级别。在我们的端点，我们需要保护我们的入口点，我们这样做:

![](img/64886c84eca8f0cb23e7eddc4f728181.png)

**受保护的 OAuth 端点**

这里我们使用**预授权**注释来表示一个**表达式语言(EL)** 条件，以验证我们可以进行 rest 调用。这些注释在应用程序运行类中被激活:

![](img/fbc66b6988a18dbd02e69cd04c37e4b1.png)

**激活预授权注释**

所以回到我们的方法，我们看到它们都使用了一个叫做`**securityService**`的组件。本质上，这是一个组件，也是一个**服务**，我们只是简单地调用一个名为**check<scope>Header**的方法。在这个方法中，我们检查标题是否与我们的**范围**预期相匹配，然后在内部让我们的计数器增加 1 个单位:

![](img/cc931d9b930d10d1487db077c032a959.png)

**在范围验证失败时触发计数器计数**

我们将进一步了解**仪表**是累积的，但目前这是我们需要了解的关于可能的定制 StatsD 配置的所有基本信息。

# 3.2.— StatsD via Kong

C 定制标签是向 **Graphite** 发送指标的好方法，使用`**micrometer**`作为触发机制非常容易。然而，在大多数情况下，正如我之前提到的，我们并不真的需要手动或者定制的指标。这就是 **StatsD** 变得如此重要的原因。**孔**用标准 **StatsD** 插件提供了许多功能。让我们来看看它是如何工作的。我们将使用 **deck** 容器使 **Kong** 更新成为可能。在这篇文章的后面，我们将研究 **deck** 容器的细节。现在，让我们记住 **deck** 容器允许重新加载和更新我们的 **konh.yml** 文件。我们将只配置 **StatsD** 与**摄像机 1** 一起工作。我们首先需要配置一条路由:

![](img/b5d630a694c9f38c98b3c46e6442b61a.png)

**摄像机-1-服务的 YAML 片段**

所以这仅仅意味着我们将能够访问在主机 **camera-1-service** 上本地运行的服务，在端口 **8080** 上，路径为`**/api/v1/hc**`。然后通过路径`**/camera-1-service/api/v1/hc**.`上的端口 **8000 (** 孔网关端口)访问该位置，这确保了当我们试图访问我们的**摄像机-1-服务**时，我们将总是必须通过**孔**网关。这种机制与简单的代理没有太大区别。然而，因为**孔**是一个网关平台，它是高度可管理的，我们可以实时做出改变。我们可以做的事情之一是直接添加 **StatsD** 插件:

![](img/58f160d0a96dc38b6e021c943da3fce4.png)

**配置 StatsD 插件向 Graphite 发送数据的 YAML 代码片段**

在这种情况下，我们使用现成的指标。**请求 _ 计数**、请求 _ 大小**、**以及最后的**响应 _ 大小**。数据将每秒发送到**石墨**。正如我们之前看到的，这相当于轮询间隔，其工作方式完全相同。

# **4。—常见相机功能**

正如我之前提到的，所有摄像机都将从同一个部署包开始。它们具有共同的功能和特定于某些协议的其他功能。现在，让我们来看看我们所有相机可以使用的功能。

# **4.1。—欢迎信息**

欢迎信息只是一个文本，对于测试我们将要看的内容是有用的。它就像一个**得到**端点一样简单:

![](img/604b1cb35325ed3e3ce20f13fbb35f04.png)

通用欢迎信息

旁边是负责检索给定时间的摄像机图像的端点:

![](img/35a8cefd572574d77465e8446f16da1d.png)

相机端点

及其在这里的实现:

![](img/40d9af7f5e90a90fb765534b4df36b31.png)

**摄像机图像检索实现**

一般来说，一旦我们被认证，我们就可以得到一个时间点的图像。由于这个练习只是一个模拟，我们将根据**10**秒的匹配分数得到一个新的图像。这样我们在刷新前端页面的时候就有一种运动的感觉。

无论用户是否通过身份验证，前端都将连接到后端。在这里，我们将向前端实时发送状态数据。这将通过 **Websockets** 完成:

![](img/75d7e1edb6152bdcd52db5f34102f0c1.png)

**汇聚流向前端发送实时数据，显示摄像机状态**

# 5.—身份验证协议

# 5.1.—基本身份验证

在我们将要讨论的所有 6 种访问摄像机的协议中，这可能是最不安全的一种。下图显示了消息流的工作方式。

![](img/457f3fbf64f603a88970dec581ff5573.png)

**摄像机 1 序列图**

在任何情况下，我们都知道我们的相机最初没有受到保护，并且出于某种奇怪的原因，我们希望通过简单的用户名和密码实现最简单，当然也是最不安全的访问方式。如果您的命令行包含 **base64** 命令，我们已经可以尝试做一个简单的测试了。假设我是用户**露西**，我的密码是`**winkelallemaal**`。我们还假设这个凭证已经被配置为在系统中工作。有了基本的 auth，我们所要做的就是将`**Lucy:winkelallemaal**`和你的请求一起发送到服务器。但是，我们不会空白发送。我们使用的是它的 **base64** 编码版本。这将通过命令行以下列方式完成:

```
**echo ‘Lucy:winkelallemaal’ | base64**
```

结果将是这样的:

```
**THVjeTp3aW5rZWxhbGxlbWFhbAo=**
```

所以现在这看起来像一些加密的东西，我们通过**授权**头中的线路发送它:

```
Authorization: basic **THVjeTp3aW5rZWxhbGxlbWFhbAo=**
```

当出于这样或那样的原因，有人获取令牌时，这个安全协议就会出现问题。在这种情况下，一个简单的解码操作就可以检索用户名和密码:

```
echo **THVjeTp3aW5rZWxhbGxlbWFhbAo=** | base64 — decode
```

结果将会是:

```
**Lucy:winkelallemaal**
```

但是当然，我们想知道这是如何与**孔**一起工作的，这实际上是学习至少这 6 个安全协议的存在的一个很好的开始。

在我们的`**Kong.yaml**`文件中，我们需要将路线添加到我们的 **camera-1-service** 中，并创建一个与之相关的插件。对于路径，我们可以使用以下内容:

![](img/07a993f29908c6708a803bcb610c29ff.png)

**摄像头 1 服务孔路由**

对于摄像机 1，我们定义了 3 条路线。第一种是**相机特有的**1**1**。另外两条是我们将为所有摄像机复制的路线。我们将保护路线 1，因为在那里我们将获得相机信息和当前图像。另外两条路线是**致动器**和**套筒**路线。重要的是，它们保持开放和不受保护，以便我们可以实时检查摄像机的状态。

为了保护第一条路由，我们为基本身份验证创建了一个插件配置:

![](img/4e658b6711b88619598da5611e5cc789.png)

**摄像机 1 对**的基本认证

当我们创建这个时，我们已经保护了我们的路由，但是我们仍然没有任何用户。为了进行配置，我们可以简单地运行几个命令，这些命令可以通过 GUI 运行，或者在我们的例子中，通过`**cUrl**`命令运行:

```
curl -d "username=camera1&custom_id=CC1" http://127.0.0.1:8001/consumers/
curl -X POST http://127.0.0.1:8001/consumers/camera1/basic-auth \
  --data "username=cameraUser1" \
  --data "password=administrator"
```

在第一个命令中，我们创建了一个消费者。在第二个示例中，我们为同一个用户定义了用户名和密码。

这是用这个插件配置基本身份验证的简单方法。

# 5.2.— HMAC

**HMAC** 是一种基于位置、时间戳、秘密和协议选择的非凡算法。通过 HMAC 发送数据非常简单，如下图所示:

![](img/103d1c652801acbdbf0e00f388a5d182.png)

**摄像机 2 序列图**

我们可以看到，它遵循与基本 Auth 协议完全相同的流程。但是，它的令牌不可能被解密。它是以一种方式创建的，此外，它还取决于生成它的位置以及为其专门生成的路径。这意味着如果黑客获得了 **HMAC** 令牌，他们将无法在任何地方使用它，即使它是一个有效的令牌。在两端使用相同的秘密或私有密钥是一个被称为**对称密钥加密的概念。**这种算法常用于****(机器对机器)**连接和文件传输。其思想是以一种安全的方式认证和授权各方之间的数据传输。越复杂的**秘密**就越难猜到我们用的是哪一个。正如我们之前看到的，这就是为什么在某些情况下，我们使用 **PEM** 私钥。不过，对于我们的例子，为了尽量不使用 **HMAC** ，我们将只使用`**dragon**` 作为我们的秘密。**

**为了使这个配置在**孔**我们首先像往常一样需要创建路线。在这种情况下，让我们看看**摄像机 2 的路线**:**

**![](img/fd1ef7c9ef4283775164b7710bd77104.png)**

****摄像机 2 路线****

**现在让我们用 HMAC 来保护它:**

**![](img/227e86d1b01f27a55bd2c5a6cd9a3c49.png)**

****摄像头 2 HMAC 插件配置****

**使用这种配置，我们可以保护摄像机 2，但目前无法访问它。为了使其可访问，我们需要发出几个命令:**

```
curl -d "username=camera2&custom_id=CC2" http://127.0.0.1:8001/consumers/
curl -X POST http://127.0.0.1:8001/consumers/camera2/hmac-auth \
  --data "username=cameraUser2" \
  --data "secret=dragon"
```

**对于这种情况，我们还需要一个消费者。在这种情况下，我们还为它创建了一个**用户名**，但是我们没有创建**密码**，而是创建了一个**密码****

**我们还没有看到算法到底是如何工作的，但这也不是本文的目的。这篇文章的真正目的是向你展示我们需要什么样的变量来创建一个最低可接受的 HMAC 令牌。为此，最好是使用 **crypto** 创建算法:**

**![](img/94780a0b82cebe101d9e637739c9cf87.png)**

****HMAC** 头文件创建函数的类型脚本实现**

**让我们大致了解一下我们在这里的意图。当我们试图连接到摄像机 2 时，我们需要创建一个 **HMAC** 令牌。为此，我们传入我们试图访问的路径和我们希望用来执行请求的方法。这些只是这个项目中的 **GET** 请求，但是它们可以应用于任何方法。我们首先初始化对我们的算法很重要的变量:**用户名、密码、算法和当前日期**。在我们的 **HMAC** 头中，我们需要包含一个**摘要头。通过提供由“ **SHA=256=base64Hash** ”组成的文本来创建该**头**。现在，我们只需要知道这是算法工作所需要的。****

**![](img/1d3a34f3081aaa291fe50b484cde7068.png)**

****关于 HMAC 如何工作的摘要****

**因此，对于 HMAC 算法的这个实现，我们需要 4 个基本元素在我们的头中。**摘要**，它包含关于我们正在使用的算法的加密信息，**授权**，它是一个包含不同元素的加密字符串，**当前日期，**，最后是一个**内容类型**头，它决定了我们将要通过网络发送的内容的类型。所有四个标题都非常重要，但最广泛的一个是**授权**，其形式如下:**

```
**Authorization** : **hmac** **username**="**cameraUser2**",**algorithm**="**hmac-sha256****"**,**headers**="**x-date request-line digest**",**signature**="**<signature>**"
```

**以这种方式通过网络发送请求意味着每个请求都可以用一个新的 **HMAC** 头发送。关键是，既然我们能做到，为什么不呢？ **HMAC** 可以是一个重算法过程。我们牺牲性能来获得最佳的安全性。**

**对于这款相机，我们仅仅触及了 HMAC 工作原理的表面。我们只发出 get 请求，它们都没有要加密的有效负载。尽管 **HMAC** 是专门为保护出站数据而设计的，但我们在本文中的目标只是对其进行配置并了解一点它是如何工作的。**

# **5.3.— JWT**

**JWT 令牌通常被创建为用户可以添加到他们的请求中的令牌，以便对某些方法的使用进行身份验证和授权。jwt 可以按照我们想要的方式创建。它依赖于非对称密钥加密的原理。加密发生在签名级别。通常在 JWT 我们只需要一个代币。令牌可能有到期日期，它包含有关登录用户和用户权限的各种信息。一个 **JWT** 令牌，很像基本的认证，并没有隐藏太多。如果请求带有无效的令牌，它就会失效。一个 **JWT** 令牌由一个报头、一个有效载荷和一个签名组成。报头和有效载荷采用 **base64** 编码，而**签名**是**报头**和**有效载荷**的函数。在这个函数中，我们使用一个**私有**密钥来签署我们的令牌。如果签名不正确，那么我们的应用程序将拒绝它。令牌的验证需要一个**公共**密钥。该过程不需要密码，尽管可以使用密码来请求新的 **JWT** 令牌。本质上，它是这样工作的:**

**![](img/1e0cc131ab7e7d76535b6734fffedc73.png)**

****JWT 令牌的非对称签名加密****

**使用**孔**插件，我们可以实现这个工作流程:**

**![](img/6b1d48a40476d48e02b0cb50cfbffb53.png)**

****摄像机 3 序列图****

**这是一个非常简单的工作流程，我们向 **Kong** **Gateway** 发出请求，只需添加一个这样的令牌:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuU0Q3WlNxU3dTeDIwN3ZSNVh4aW9jVFVaaHNCMlF4ViIsImV4cGlyZXNJbiI6IjEyaCIsImFsZ29yaXRobSI6IkhTMjU2In0.R4S8yV_CPJRvJ2dAX_56HfNUZrbKr7Jt4AxEmsqqx8w
```

**稍后，我们将在我们的应用程序中看到**孔**如何以及何时可以将这些代币返还给我们。现在，让我们看看 [**JWT.io**](https://jwt.io/) ，看看破译这个令牌能告诉我们什么:**

**![](img/1e9de1144e6e6e36352979d2bca8fd8e.png)**

****一孔 JWT 令牌已破译但未验证****

**如果你是这些对红色信息非常敏感的人之一，当检查这个令牌时，你可能首先意识到的是 [**JWT。IO**](https://jwt.io/) ，是在告诉我们令牌无效。我们没有将签名插入到**验证签名**框中。因为我们没有给 JWT.io 一个能够验证的秘密，它只是说这个令牌是无效的。但是让我们看看**割台**和**有效载荷**。我们能够破译这两个，原因是它们只是被加密了。让我们来破译这个。至此，您已经看到了它是如何工作的。我们只发出几个命令:**

```
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 --decode
```

**结果是:**

```
{“alg”:”HS256",”typ”:”JWT”}
```

**这说明这是一个基本的 **JWT** 令牌，用**算法 HS256** 实现，用于验证我们正在发送的签名。**

**现在让我们检查一下尸体:**

```
echo “eyJpc3MiOiJuU0Q3WlNxU3dTeDIwN3ZSNVh4aW9jVFVaaHNCMlF4ViIsImV4cGlyZXNJbiI6IjEyaCIsImFsZ29yaXRobSI6IkhTMjU2In0” | base64 --decode
```

**结果是:**

```
{"iss":"nSD7ZSqSwSx207vR5XxiocTUZhsB2QxV","expiresIn":"12h","algorithm":"HS256"}
```

****Iss** 是发行人。这个问题可以指定，如果我们不指定，**孔**会随机给我们一个。虽然不受保护，但是**发布者**是签名的重要组成部分。每当我们要求一个 JWT，一个新的秘密就产生了。这次跑步的秘密是:`xwRs1oR22OhzBeq2hWH4NnIxdF5Jr6jv`。如果我们把它输入 JWT。IO 我们将能够得到验证，我们在有效载荷和请求中提供的任何操作都将被允许通过安全检查。如果当时授权允许或进一步操作无关紧要。我们已经能够到达系统，但是只能在我们在有效载荷中指定的条件下:**

**![](img/81a0c2a6412bf884881654aea7bc7653.png)**

****一孔 JWT 令牌破译验证****

**这里的秘密是**密钥**，它永远不会被共享。这就是我们通过 **JWT 验证请求的方式。IO** 。但是当然，这里我们拥有私钥。在**孔**端的算法，仅用其**公共**密钥验证请求是否有效。**

**为了在**孔**中进行配置，我们需要首先配置我们的路线:**

**![](img/a4c118b3f525026ebaf7b964612904fa.png)**

****3 号摄像机路线****

**然后只需配置插件:**

**![](img/19fdbd7b2f6e13cd4e0a9e7e89b13082.png)**

****启用摄像机 3 服务的 JWT****

**一旦在摄像机 3 运行的情况下启用服务，我们就可以请求我们的 **JWT** 令牌参数:**

```
curl -d "username=camera3&custom_id=CC3" http://127.0.0.1:8001/consumers/
curl -X POST http://127.0.0.1:8001/consumers/camera3/jwt -H "Content-Type: application/x-www-form-urlencoded"
curl -X GET http://127.0.0.1:8001/consumers/camera3/jwt > ../e2e/cypress/fixtures/CC3KongToken.json
```

**这些命令创建了一个名为 **camera3** 的消费者。然后，我们用 **POST** 请求生成令牌。为了能够在网站中使用令牌，我创建了一个名为 **CC3KongToken.json** 的文件，在这里我们可以找到**发布者**和**秘密。**后者也是我们所说的**私有**密钥。它被称为 **private** ，因为它不是共享的。如果另一台摄像机配置了相同的**公共**密钥，它将接受具有相同令牌的请求。**

**如果我们执行最后一个 **GET** 请求，我们将得到与这个类似的回答:**

```
{
  "data": [
    {
      "tags": null,
      "rsa_public_key": null,
      "consumer": {
        "id": "137558e2-bcc0–4fc4-a423–33a37e2d9f6e"
      },
      "key": "nSD7ZSqSwSx207vR5XxiocTUZhsB2QxV",
      "created_at": 1665153305,
      "algorithm": "HS256",
      "id": "9d6febc6–2602–42ae-a2b2–4ec3924e7bce",
      "secret": "xwRs1oR22OhzBeq2hWH4NnIxdF5Jr6jv"
    }
  ],
  "next": null
}
```

**为了获得 **JWT** 令牌，我们手动完成，我们在**类型脚本**代码中有一个这样的例子。我们首先对令牌进行编码:**

**![](img/1064fff318d8da59e16bacc0f8e830f1.png)**

****打印头和有效载荷创建****

**然后我们签署完整的令牌:**

**![](img/7891f4ff14e50161cc8689f3a8288150.png)**

****签署令牌****

**我们可以从几个层面来看待 JWT。在某种程度上，我们可以认为这是纯 HMAC 的某种进化。也许 JWT**和**之间不太好的地方在于我们确实共享一些数据。当然这也不是一个 **TLS** 隧道解决不了的。但同样，向潜在黑客提供一些信息总是比什么都不提供更糟糕。JWT 允许作为前两个安全协议的额外功能的是赋予权限和角色的能力。有效授权。也许我们需要理解的另一个区别是，这种情况下的令牌是一个**不记名**令牌:**

```
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuU0Q3WlNxU3dTeDIwN3ZSNVh4aW9jVFVaaHNCMlF4ViIsImV4cGlyZXNJbiI6IjEyaCIsImFsZ29yaXRobSI6IkhTMjU2In0.R4S8yV_CPJRvJ2dAX_56HfNUZrbKr7Jt4AxEmsqqx8w"
}
```

****持有人**令牌实际上是我们将在 **OAuth** 中看到的同一问题的一部分。如果令牌有效，那么官方 [RFC6750](https://oauth.net/2/bearer-tokens/) 将其描述为:**

> **一种安全令牌，具有这样的属性:拥有该令牌的任何一方(“持有者”)都可以像拥有该令牌的任何其他方一样使用该令牌**

**这意味着，只要令牌有效，任何拥有它的人都可以访问受保护的应用程序。**

# **5.4.— API 密钥**

**对于这种类型或形式的身份验证，没有什么可说的。API 密钥是一种身份验证形式，本质上允许用户完全基于拥有 API 密钥来访问应用程序。 **apiKey** 不是通过**授权**头发送的，而是通过同名头发送的:**

**![](img/b38587386391bc73d28f22d89cce9941.png)**

****摄像机 4 序列图****

**摄像机 4 受到该安全协议的保护。要配置它，我们首先配置路由:**

**![](img/fd47a02cb8b6d73739e042bef5a6a637.png)**

****摄像机 4 路线****

**然后我们激活 **apiKey** 插件:**

**![](img/8e32482e795bc1ad0bfdd35753e44432.png)**

****4 号摄像头 apiKey****

**在这种情况下，我们也需要一个消费者。要创建它，我们可以发出以下命令:**

```
curl -d "username=camera4&custom_id=CC4" http://127.0.0.1:8001/consumers/
curl -X POST http://127.0.0.1:8001/consumers/camera4/key-auth
curl -X GET http://127.0.0.1:8001/key-auths > ../e2e/cypress/fixtures/CC4KongKeys.json
```

**简而言之，我们首先创建一个名为 **camera4，**的用户名，然后请求一个 **apiKey** 。使用 **key-auths** 端点，我们能够发出请求，作为响应，我们将得到一个这种形式的请求，然后我们能够在 **CC4KongKeys.json** 文件中找到它:**

```
{
  "data": [
    {
      "ttl": null,
      "consumer": {
        "id": "99647244-bc39-4981-a187-00e2402be24b"
      },
      "created_at": 1665157122,
      "id": "7a5a39f5-011d-4557-b021-d35e6fb60030",
      "tags": null,
      "key": "87FNNInZXturFdEsal3InGMu7Me3CX5C"
    }
  ],
  "next": null
}
```

**在这种情况下，**键**就是我们的 **apiKey** 。让我们看看带有 apiKey 的请求在**类型脚本**代码中是什么样子的:**

**![](img/43b53a5815e60c3d1415c94729a0de4b.png)**

**ApiKey 类型脚本发送代码**

**因此，重要的是要看到我们正在通过线路明文发送 **apiKey** :**

```
{"apiKey":"87FNNInZXturFdEsal3InGMu7Me3CX5C"}
```

**如果我们想自动化应用程序之间的连接，那么`**apiKey**`可能会很有用。它是非常轻量级的，只需要少量的处理。然而，如果我们选择使用这个安全协议，我们必须确保我们是通过 **TLS/HTTPS** 连接或者加密连接来使用它。**

# **5.5.— LDAP**

**这个安全协议让我们想起了 windows，但它并不一定与 windows 有内在联系。在这篇文章中，我的目的不是解释它是如何工作的，而是我们如何通过**孔**网关来配置它？我们将从 LDAP 实现自身的方式中抽象出来，并关注它的实际实现。 **LDAP** 流的最简单形式如下:**

**![](img/0db455a1fd6532219c3f90b0e64b719e.png)**

****摄像机 5 序列图****

**首先我们实现路线:**

**![](img/3dd225357829399519dced02bca57f35.png)**

****摄像机 5 路线****

**然后我们可以配置插件:**

**![](img/e2cd4066b33312489e8a5be7a4f997be.png)**

**对于 LDAP ，我们有一些与其他协议稍有不同的东西。 **LDAP** 需要一个外部 **LDAP** 服务才能工作。所以，本质上，通过**孔**使用 **LDAP** ，我们不仅将我们的请求代理到摄像机，还代理到认证 **LDAP** 服务。 **LDAP** 服务就是我们在 docker-compose 中发现的`**openldap**`服务。**

**一旦所有这些都准备就绪，那么我们只需要将我们的请求发送给服务，并在旁边附上身份验证头。在这种情况下，我们将发送一个与我们在 **basic-auth** 算法中用于摄像机 1 的头非常相似的头，但是在这种情况下，我们称它为 **ldap** :**

```
{ Authorization: ldap YWRtaW46cGFzc3dvcmQ= }
```

**这是相同的原理，并且以与基本授权请求相同的方式实现:**

**![](img/1379ea9794309799acf5257ba88b69b9.png)**

****带有 LDAP 报头的 LDAP 请求****

# **5.6.— OAuth2**

**最后，我们到达了 OAuth2。这个安全算法有点复杂，因为它涉及到我们应用程序的外部方。当我们试图登录一些应用程序，并且这些应用程序要求我们通过谷歌、脸书、iCloud、GitHub 或任何其他认证提供商登录时，我们通常会看到这个外部方。当我们第一次通过身份验证提供商登录时，它可能会询问我们是否要向他们提供我们的数据，如我们的姓名和电子邮件，当我们说是并登录时，这仅意味着身份验证成功，此外，我们的应用程序已经用身份验证提供商提供的数据注册了我们。接下来，我们可以成功登录。对于本文的这一部分，我们将跳过注册过程，只使用一个身份验证提供者，就好像我们已经注册了一样。身份验证提供者可以通过多种方式实现，这已经超出了本文的范围。然而，我想创建一个例子，这样我们就可以看到 OAuth2 在本地运行。所以我创建了一个简单的 SpringBoot 服务，用**基本认证**保护。当然，这只是用作真实事物的模拟。其思想是，在我们认证之后，该服务将在数据库中寻找用户，获取用户的范围并使用它来向 camera-6 服务请求载体认证报头。然后，该不记名令牌可用于通过 **HTTPS** 访问摄像机 6 服务。以下数据流说明了整个过程:**

**![](img/cf9f1223f5d8107cbc02b1ea3c315a89.png)**

****摄像机 6 序列图****

**为了进行配置，我们首先需要定义摄像机 6 的路线:**

**![](img/485373c801728d89c23390d71323fe9b.png)**

****摄像机 6 路线****

**最后，我们可以配置 **OAuth2** 插件。对于 **OAuth2** 我决定只通过 curl 命令保护路线:**

```
curl -X POST http://127.0.0.1:8001/consumers/ \
  --data "username=camera6" \
  --data "custom_id=CC6"
curl -X POST \
  --url http://127.0.0.1:8001/services/camera-6-service/plugins/ \
  --data "name=oauth2" \
  --data "config.enable_password_grant=true" \
  --data "config.enable_client_credentials=true" \
  --data "config.scopes=admin" \
  --data "config.scopes=observer" \
  --data "config.scopes=visitor" \
  --data "config.scopes=researcher" \
  --data "config.mandatory_scope=true" \
  --data "config.enable_authorization_code=true" > ../e2e/cypress/fixtures/CC6KongProvOauth2.json
cp ../e2e/cypress/fixtures/CC6KongProvOauth2.json ../cameras-auth-service/target/CC6KongProvOauth2.json
curl -X POST http://127.0.0.1:8001/consumers/camera6/oauth2 \
  --data "name=Camera%20Application" \
  --data "client_id=CAMERA06CLIENTID" \
  --data "client_secret=CAMERA06CLIENTSECRET" \
  --data "redirect_uris=https://127.0.0.1:8443/camera-6-service/api/v1/hc" \
  --data "hash_secret=true" > ../e2e/cypress/fixtures/CC6KongOauth2.json
curl -X GET http://127.0.0.1:8001/oauth2_tokens/
```

**因此，我们首先创建一个消费者，然后我们用范围 **admin** 、 **observer** 、 **visitor** 、**research、**配置 OAuth2，最后我们用消费者自己的 **client_id、client_secret** 和**redirect _ uri**配置消费者。创建文件 **CC6KongProvOauth2.json** 和 **CC6KongOauth2.json** 是因为我们需要这些命令生成的数据来运行我们的测试。**

**现在我们可以看看我们的身份认证提供者。在项目中，它被称为**camera-auth-service**。此服务连接到包含以下数据的 SQL 数据库:**

**![](img/80aa2aaafca928fcfb5696d7341102ca.png)**

****授权数据库中的用户****

**字段**角色**实际上是范围。所有用户都有相同的密码，密码是 **admin。**不断重申 **admin** 可以用作演示目的的密码**只是**是远远不够的。千万不要在自己的帐户中使用这种密码。**

**要检索`**authorization**` `**bearer**` `**token**`，有两种方法。我们将首先检索一个**访问代码**，然后使用它来获得**访问令牌。**这在以下 Kotlin 代码中实现:**

**![](img/f27834b4036bbe293820843514da6c84.png)**

****检索 OAuth 承载接入码****

**如果我们放大第一个请求的方式，我们可以看到我们需要创建一个特殊的令牌，其中我们需要`**clientId**`、用户的`**scope**`、`**provisionKey**`、(这是由孔提供的)、`**authenticatedUserId**`、**、`**responseType**`、**、**:****

**![](img/6c02eb43017109960349e120e58fa1c4.png)**

****createAuthFormRequestBody****

**这个第一个请求将为我们提供**访问代码**，然后我们可以使用它来进行请求，这将为我们提供授权持有人代码**

**![](img/dc41d34614e57658238c8462465fa90a.png)**

****createTokenFormRequestBody****

**在[http://localhost:8000/camera-auth-service/API/v1/camera/auth/log in](http://localhost:8000/cameras-auth-service/api/v1/cameras/auth/login)上登录 auth 应用程序的结果将是:**

```
{
  "refresh_token": "zy2hwv8ov90e5XWRXmbNxZ9NiITMkRYG",
  "access_token": "se4rpVnIofug611XgoqYwAmV7FSTwoMy",
  "expires_in": "7200",
  "token_type": "bearer",
  "redirect_uri": "https://localhost:8443/camera-6-service/api/v1/hc?code=HGsfiB63XY2MyoX7YZk42ryLKNchucyP"
}
```

**这是对我们的摄像机执行第一个 OAuth2 请求所必需的一组数据。**

**发送带有**载体**令牌的数据将允许我们访问摄像机:**

```
{
  "Content-Type": "application/text",
  "Authorization": "Bearer kY0FMKviJgHQTuSjEE9ago1CSz5of8WA"
}
```

**除了我们的相机，任何人都无法读取不记名令牌中的内容。将这个传递到后端将很容易。当我们访问我们的 OAuth2 应用程序时，我们就可以访问 header**x-authenticated-scope。**我们知道，当我们通过 Kong 网关时，我们已经在畅通无阻地访问我们的应用程序。**孔**给我们提供的不仅仅是这个表头，还有`**x-authenticated-userid**` **、**、`**x-consumer-username**`、**、**、**。**这就是我们上面接近本文开头的所有内容。**

# **6.—运行演示**

**要运行演示，请使用以下命令:**

```
make dcup-full-action
```

**运行的时候直接去 [http://localhost:8000](http://localhost:8000)**

**![](img/2556e2551180707cb686d69a9fff44eb.png)**

****测试所有摄像机的应用概述****

**![](img/e57a8aa8676c2f7716181ebc37181ee4.png)**

****StatsD 捕获****

# **7.结论**

**我们在本文中看到的是，通过使用应用程序网关，我们几乎可以控制应用程序面临的任何方面。在本文中，我们只讨论了解释和阅读度量的可能性，但是还有很多其他的可能性。**

**在任何情况下，我们也看到了基本 Auth 如何能够很好地用于简单的演示应用程序。例如，这就是我们在 OAuth2 案例中为身份验证提供者所做的事情。我们现在知道 Basic-Auth 可能非常不安全。我们实际上是通过线路发送`username`和`passwords`，而没有任何活动协议来保护这一对。我们可以说 TLS 隧道最终会解决这个问题。但是在 TLS 隧道中窃听已经被证明是可能的，这是我们不想冒的风险。所以 Basic-Auth 绝对不能在生产中使用。**

**当我们看着 HMAC 时，我们也看到了一些基本的好处。它是高度加密的，即使有人得到了 HMAC 令牌乱码，它也几乎是无法破解的。我们知道 HMAC 也是基于起源的，从令牌中我们不能得到起源，所以令牌的许多方面对潜在的黑客来说是不可及的。HMAC 算法的缺点是它的秘密是固定的，而且它被用于永久的自动连接，如 M2M。我们仅通过提供用户名和密码来使用它。我们不通过网络发送纯文本，加密依赖于时间戳和位置，当然还有秘密。但是不管我们考虑到什么程度，后端都会识别这个请求，检查它的完整性是否正常，以及用户是否完全基于这个秘密被授权。从令牌中获取秘密几乎是不可能的，但通过社会工程，这就变得不那么清楚了。这是人们不使用像`**dragon**` 这样的东西作为秘密，而是使用他们可以配置但不记得的东西，例如 PEM 私钥的部分原因。但是另一个可能阻碍 HMAC 的事情是概要文件、角色的定义，以及对某些方法的访问限制。**

**对于 JWT，我们也看到了我们所需要的是一个秘密和一个发布者。这种算法在保护我们的应用程序方面也非常强大。也许失败的地方在于报头和有效载荷总是在请求时是透明的。这意味着黑客可能无法通过请求，但因为标头和正文始终只是 Base64 编码，所以解码操作始终是可能的。在任何情况下，主体都是非常方便的，因为在那里我们可以定义，例如，用户可能拥有的角色，但是同样，如果角色被暴露，那么黑客已经拥有了这条信息。对于一个黑客来说，这足以造成大破坏吗？单独发生这种情况并不是因为它不一定危及应用程序，而是因为它仍然是一条信息，有助于增加可能的黑客攻击的表面积**

**使用 APIKey 是一个有趣的概念，本质上并不能很好地转化为赋予我们的应用程序密钥更多的安全性。本质上，拥有 APIKey 的消费者只是另一个拥有用户名和密码的用户。只是在这种情况下，用户不能确定密码。API 给用户一个 APIKey 来访问应用程序。它可用于预认证目的或创建另一层安全防护形式。**

**LDAP 是另一种安全形式，它引入了基本身份验证提供的几乎相同类型的范例、问题和情况。不同之处在于 LDAP 在网络级别管理访问。在我们的例子中，我们没有探究它是如何真正工作的，但重要的是要记住它更复杂，并且它提供认证和授权功能。只是对于本文的目的来说，它在使应用程序安全方面并没有真正的区别，因为它仍然基于用户以明文形式发送其密码的原则。**

**最后，我们已经了解了 OAuth2 的工作原理。这是目前为止我们见过的六种算法中最先进的。如果我们有一个允许 OTP(一次性密码)、MFA、SMS 身份验证或任何其他形式的安全登录的授权提供者，那么我们可以保证对我们的应用程序的访问也是安全的。一旦我们从身份验证提供者那里获得了令牌，我们就知道可以用它来访问我们的应用程序。通常通过 OAuth，我们会得到一个请求许可的屏幕。这是示波器意义的一部分。每个范围意味着用户可以访问应用程序的某些部分。另一方面，这也意味着 **camera-6-service** 将能够访问所提供的与该范围相关联的用户信息。**

**StatsD 也是我们如何发送标准统计数据的完美例子。**

***正如我在引言中提到的，鉴于本文的实验性质，它将受到更频繁的审查。***

**我已经把这个应用程序的所有源代码放在了 [GitHub](https://github.com/jesperancinha/healthy-cameras) 上**

**我希望你能像我喜欢写这篇文章一样喜欢它。**

**我很想听听你的想法，所以请在下面留下你的评论。**

**感谢您的阅读！**

**[](https://www.etsy.com/codeascraft/measure-anything-measure-everything) [## Etsy 工程|测量一切，测量一切

### 如果 Etsy 的工程学有一种信仰，那就是图形教堂。如果它动了，我们就追踪它。有时我们会画一个…

www.etsy.com](https://www.etsy.com/codeascraft/measure-anything-measure-everything) [](https://github.com/statsd/statsd) [## GitHub - statsd/statsd:用于简单而强大的统计数据聚合的守护进程

### 一个运行在 Node.js 平台上的网络守护进程，它监听通过 UDP 发送的统计信息，如计数器和计时器…

github.com](https://github.com/statsd/statsd) [](https://www.datadoghq.com/blog/statsd/) [## StatsD，它是什么以及它如何帮助您

### 在不到 3 年的时间里，StatsD 已经成为……

www.datadoghq.com](https://www.datadoghq.com/blog/statsd/) [](https://github.com/danielkocot/kong-blogposts) [## GitHub-danielkocot/Kong-blog posts:以代码为中心的 Kong blogposts

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/danielkocot/kong-blogposts) [](https://blog.codecentric.de/en/2019/12/kong-api-gateway-observability-with-prometheus-grafana-and-opsgenie/) [## kong API-Gateway-Prometheus、Grafana 和 OpsGenie 的可观测性

### 在我的博客上，我看到了最好的演示设置。修女应该死去…

blog.codecentric.de](https://blog.codecentric.de/en/2019/12/kong-api-gateway-observability-with-prometheus-grafana-and-opsgenie/) [](https://docs.konghq.com/hub/) [## Kong Hub |插件和集成| Kong 文档

### 面向 API 和微服务的云连接公司 Kong 的文档。

docs.konghq.com](https://docs.konghq.com/hub/)**
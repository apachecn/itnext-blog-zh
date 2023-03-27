# 如何最小化 AWS Lambda 冷启动

> 原文：<https://itnext.io/how-to-minimize-aws-lambda-cold-starts-98eebc610f9d?source=collection_archive---------0----------------------->

![](img/f1038031d1ad488f39fa3fac878afdaf.png)

无服务器架构是一个新生事物，根据无服务器公司最近的调查，绝大多数开发人员将在今年年底开始使用它。无服务器模式包括在云中运行代码而无需管理任何服务器，允许您构建业务逻辑并创造价值，而无需考虑基础架构或底层软件。本质上，它让你专注于你的代码。

无服务器不仅涵盖了 T2 AWS Lambda 和其他 FaaS 提供商，而且基本上涵盖了你可以用来运行代码、托管文件、存储图像和数据的一切。这意味着，作为一名工程师，您不需要管理、扩展或操作任何服务器。锦上添花的是:您只需为代码运行的时间付费！

尽管无服务器提供了许多好处，但仍有一些缺陷，如延迟。在本文中，我们将讨论如何最小化 AWS Lambda 中的延迟。这种可怕的现象是由冷启动引起的，根据定义，冷启动是来自无服务器 API 的较慢的初始响应。

在我们开始之前，让我们更深入地了解一下 FaaS 是什么以及它是如何工作的。

# 什么是 FaaS？

AWS Lambda 是 AWS 上的计算服务，让您无需关心服务器就可以运行代码，它是许多 FaaS(功能即服务)提供商之一。但是什么是 FaaS 呢？这是一种全新的云模式，为工程师提供了一个开发、运行和管理应用的平台，而无需维护和构建基础架构。按照这种模型创建应用程序是实现无服务器架构的一种方式。使用这种架构最适合构建微服务。

您可以利用 FaaS 提供者的各种用例。其中最强大的是运行短期运行脚本和进程的能力，您可以触发和忘记。除此之外，您还可以设置定期流程，每隔一段时间执行一次，或者定期执行(例如，每天早上 9 点)。看到了吗？在这种情况下，不关心服务器的好处是非常方便的。

尽管我们知道 FaaS 是因为 AWS Lambda、谷歌云功能和微软 Azure 功能，但它是在 2014 年底由 hook.io 首次推出并向世界提供的。拥有这样的服务意味着我们可以将模块化的代码上传到云中，并完全独立地执行它们。

# AWS Lambda 是如何工作的？

让我们在表面下挖掘一下，特别是因为 AWS 已经在 re:Invent 2018 中描述了 Lambda 如何在引擎盖下工作。这里我们将解释当 AWS Lambda 函数被触发时实际发生了什么。

第一步是配置一个事件触发器来调用 lambda 函数。这可以是任何东西，从 HTTP 请求到图像上传，到 S3 或 SQS 队列。lambda 函数将首先告诉 AWS Lambda 创建该函数的一个实例。这个实例基本上是一个类似于 Docker 容器的容器，但是 AWS 有自己专有的容器化软件来管理 AWS Lambda 函数实例。

一旦实例启动，代码将被部署到容器中。这是代码运行、执行并返回值的时候。在此期间，函数被认为是活动的和正在运行的，而空闲期间是它已经完成执行代码的时候。

由于初始化过程的原因，第一个请求总是会慢一点(以前是容器，现在是[鞭炮](https://aws.amazon.com/blogs/opensource/firecracker-open-source-secure-fast-microvm-serverless/))，但是后续的请求会遇到相同的实例，这意味着它们会快如闪电。不幸的是，有一个陷阱。并发请求将触发新的 AWS Lambda 实例的创建。例如，如果对同一个 lambda 函数有 10 个并发请求，AWS 将创建 10 个容器来服务这些请求。崔琰在他的文章[中对此做了惊人的解释，“恐怕你对 AWS Lambda 冷启动的想法完全错误。”如果你对 Lambda 是如何实现的感兴趣，你应该看看 Lambda 内部的博客系列。](https://hackernoon.com/im-afraid-you-re-thinking-about-aws-lambda-cold-starts-all-wrong-7d907f278a4f)

# 什么是冷启动？

现在有意义吗？冷启动是由创建运行代码的容器的初始过程引起的。根据您使用的语言，延迟通常会超过几秒钟。但这真的是一个大问题吗？根据 AWS 无服务器的主要开发者倡导者 Chris Munns 的说法，不到 0.2%的函数调用是冷启动的。它只会成为同步调用的一个问题。

现在你可能在想，这有什么大惊小怪的？好吧，说实话。你有没有离开过一个网站，因为几秒钟后它仍然没有加载？这就是工程师发挥重要作用的地方，确保用户永远不必面对这个问题。以下是避免问题的方法。

# 优化冷启动

在对抗冷启动时，有两个主要策略可以使用:最小化冷启动的持续时间(意味着减少冷启动本身的延迟)和最小化冷启动发生的次数。前者是通过使用常见的编程模式和常识来完成的，而后者是通过一种称为函数预热的技术来实现的。让我们更详细地解释一下。

# 最小化冷启动的持续时间

您可以通过使用解释语言编写函数来减少冷启动的时间影响。Node.js 或 Python 的冷启动延迟远低于一秒。像 Go 这样的编译语言是低冷启动延迟的另一个例子。另一个简单的步骤是为你的函数选择更高的内存设置。这也给了你的函数更多的 CPU 能力。然而，最重要的考虑是避免 VPC。VPC 必须创建 Eni，这需要 10 秒以上的时间来初始化。请不要把你的功能放在 VPC 里。就是不要！

# 最小化冷启动的频率

通过功能预热，您可以减少冷启动的次数。这是向您的函数发送预定的 ping 事件的行为，以使它们保持活动和空闲，准备好为请求服务。有了 [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) ，定期触发函数以始终保持固定数量的 AWS Lambda 实例是很简单的。只需设置一个[周期性 cron 作业](https://read.acloud.guru/how-to-keep-your-lambda-functions-warm-9d7e1aa6e2f0)每 5–15 分钟触发一次您的函数，请放心，它将一直处于空闲状态。

这又提出了一个问题:如何处理并发冷启动？幸运的是，有一些模块和插件可以使用。如果你是 Node.js 的开发者，那么 [Lambda Warmer](https://www.npmjs.com/package/lambda-warmer) 将会击中要害。AlertMe.news 的首席技术官 Jeremy Daly 在我们的[无服务器和可观测性网络研讨会](https://blog.epsagon.com/serverless-observability-webinar)上讨论了 Lambda Warmer。它允许您预热并发函数，甚至选择您想要的并发级别。而且，它可以与无服务器框架和 AWS SAM 一起工作。无服务器框架有另一个你可以使用的插件，叫做[无服务器预热插件](https://github.com/FidelLimited/serverless-plugin-warmup)。但可悲的是，它不支持并发功能升温。

为了正确预热您的功能，您应该遵循几个简单的步骤:

*   不要每五分钟调用一次函数。
*   用 Amazon CloudWatch 事件直接调用函数。
*   运行预热时，传入一个测试有效负载。
*   创建处理程序逻辑，它在运行预热时不运行所有功能逻辑。

下面是包含更热逻辑的处理函数的样子:

```
const warmer = require('lambda-warmer')
exports.handler = async (event) => {
    // if a warming event
    if (await warmer(event)) return 'warmed'
        // else proceed with handler logic

    return 'Hello from Lambda'
}
```

无服务器框架的事件触发器配置只需要您添加几行 YAML:

```
myFunction:
name: myFunction
handler: myFunction.handler
events:
 - schedule:
     name: warmer-schedule-name
     rate: rate(5 minutes)
     enabled: true
     input:
       warmer: true
       concurrency: 1
```

通过使用这个预热器，您可以安全地保持尽可能多的 AWS Lambda 实例运行，包括相同函数的副本(如果您期望并发请求的话)。所以，跟冷启动说再见吧。

# 包扎

最后，这一切都归结为延迟，所有工程师都希望在他们的软件中减少这种可怕的现象。正如我们所讨论的，您可以通过最小化冷启动的持续时间和次数来优化冷启动。您还可以通过增加该功能的内存设置并将其置于 VPC 之外来缩短持续时间，同时功能预热可以防止冷启动的总量。

尽管只有不到 0.2%的函数调用是冷启动，但工程师有责任确保他们的所有用户在任何时候都能体验到最佳性能。说起来容易，做起来难，但是遵循上面的指导方针，这绝对是可以实现的。

*原载于 2018 年 12 月 4 日*[*epsagon.com*](https://epsagon.com/blog/how-to-minimize-aws-lambda-cold-starts/)*。*
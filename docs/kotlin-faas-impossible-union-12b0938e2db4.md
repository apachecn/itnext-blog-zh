# 科特林和 FaaS，不可能的结合？

> 原文：<https://itnext.io/kotlin-faas-impossible-union-12b0938e2db4?source=collection_archive---------9----------------------->

![](img/552530d384857a028983fea01be9315c.png)

前段时间，我看了一个帖子，描述了如何在 [OpenFaaS](https://www.openfaas.com/) 上运行一个无服务器的 Kotlin 函数。虽然内容在技术上是正确的，但我认为概念本身是非常错误的。这样的帖子可能会导致人们做出不明智的决定:“因为我们能”很难成为一个成功的策略。在这篇文章中，我想首先解释为什么 JVM 平台对 FaaS 来说是个坏主意。然后，我将继续提出使用 Kotlin 的替代方案。

我故意选择不链接到最初的帖子，以避免给它任何可信度。如果你想看，谷歌是你的朋友。

# 语义学

在做任何事情之前，我们应该首先解决 FaaS 和无服务器的语义。虽然这些术语有时会互换使用，但我将在本文中使用以下定义。在所有情况下，看看人们在维基百科上达成的共识总是值得的:

> *无服务器计算是一种云计算执行模式，云提供商运行服务器，动态管理机器资源的分配。定价基于应用程序消耗的实际资源量，而不是预先购买的容量单位。*
> 
> 【https://en.wikipedia.org/wiki/Serverless_computing】—[](https://en.wikipedia.org/wiki/Serverless_computing)
> 
> **功能即服务(FaaS)是云计算服务的一个类别，它提供了一个平台，允许客户开发、运行和管理应用功能，而无需构建和维护通常与开发和启动应用相关的基础设施。**
> 
> **——*[*https://en.wikipedia.org/wiki/Function_as_a_service*](https://en.wikipedia.org/wiki/Function_as_a_service)*

*根据以上定义:*

*   *无服务器是关于资源的管理:
    你需要动态地处理它，属性是**弹性**。*
*   *FaaS 是关于代码的大小:
    大小的演变从一个成熟的应用程序到微服务再到**一个功能**。原帖提到了。*

*因此，FaaS 意味着没有服务器。*

# *JVM 和 FaaS*

*JVM 平台是一项优秀的技术。特别是，抽象层允许 JVM 将*字节码*编译成适应工作负载的本机代码。这就是为什么即使 C/C++编译的应用程序更接近裸机，JVM 也能够在性能方面与其竞争——甚至获胜。*

*然而，这种优化是有代价的:JVM 需要时间预热，*，例如，*，将类加载到内存中。这就是 JVM 上的性能测试需要几轮预热的原因。因此，JVM 非常适合长时间运行的流程，与整个生命周期流程相比，启动时间可以忽略不计。应用服务器是这种用例的典型代表:一旦启动，就应该运行很长时间，比如说几天，非常保守。在这方面，一分钟的启动时间没有任何意义。*

*现在是 FaaS:在 JVM 上下文中，FaaS 意味着 JVM 启动，函数运行，然后一切都被丢弃。在这种情况下，启动时间对总的执行时间有很大的影响。此外，JVM 没有时间将*字节码*编译成本地代码:它保持“冷”状态。*

*有些人建议保持 JVM 运行，以便以后重用，从而避免支付启动时间的成本。实现这一点非常简单:只需比 FaaS 提供者丢弃底层 JVM 的速度更快地向函数发送请求。然而，它只是挫败了无服务器的目的，因为弹性属性现在已经不存在了。*

*它适用于 Kotlin、Java、Scala、Groovy、Clojure 和其他所有运行在 JVM 之上的语言。尽管我喜欢科特林，但这是不可行的:任何人都不需要 FaaS 方法提供的弹性——或者更糟，没有线索。在大多数情况下，这类似于构建微服务架构:YAGNI。*

# *两全其美*

*然而，并非所有的希望都破灭了。有很多方法既能使用科特林，又能从 FaaS 中获益。既然 FaaS 的问题是 JVM，为什么不删除它呢？有两种简单的方法可以实现这一点:*

*   *使用 Graal 的本机映像:
    GraalVM 是不同技术的组合。其中有 [SubstrateVM](https://github.com/oracle/graal/blob/master/docs/reference-manual/native-image/README.md) :它允许将一个 JAR(或一个类)转换成本地可执行文件。然后，您可以将生成的应用程序包装到 Docker 容器中，该容器必须与您选择的 FaaS 框架兼容。这是这种方法的一个例子。*
*   *Transpile to JavaScript:
    另一种方法是将 Kotlin 转换成 JavaScript。JavaScript 不需要 JVM 平台来运行。然后，代码可以如上所述包装到一个容器中，或者直接打包到一个 ZIP 存档中。后一种选择可以在专有基础设施上运行，比如 AWS 功能。*

*这两种方法都需要一个坚实的构建管道，从 Kotlin 开始，到 Docker 容器(或 ZIP)结束。与最初的设计一样，他们既需要单元测试来测试代码是否正确，也需要集成测试来测试最终的工件是否如预期的那样工作。*

# *结论*

*人们需要了解大多数技术的背景。你(可能)不需要微服务，FaaS，或任何炒作曲线现在的趋势。然而，通过理解它们的利弊，你将能够在适当的时候利用它们。*

*就目前而言，对 FaaS 使用 JVM 是不明智的。这并不意味着你不能使用 Kotlin。*

*   *[OpenFaaS 原生 Kotlin 函数](https://thenatureofsoftware.se/posts/openfaas_graalvm_native/)*
*   *[二元函数](https://medium.com/@napperley/binary-as-a-function-a0d6490efb27)*

**原载于* [*一个 Java 极客*](https://blog.frankel.ch/kotlin-faas-impossible-union/)*2021 年 10 月 16 日**
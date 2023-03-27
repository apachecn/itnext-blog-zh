# 可持续代码和反应式编程——架构(下)

> 原文：<https://itnext.io/sustainable-code-and-reactive-programming-architecture-part-2-3f786faafa3c?source=collection_archive---------0----------------------->

![](img/08d7fc3850343fd29398a7b3f3875b2b.png)

## 介绍与反应式编程相关的架构概念

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fsustainable-code-and-reactive-programming-architecture-part-2–3f786faafa3c%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

## 体系结构

有许多反应式编程实现，我们将在本系列的后续文章中讨论它们，但在本章中，我将解释反应式架构的主要原则和模式。实际的反应式实现有各种形状和大小。例如，如果你将 ReactiveX 与 Elm 编程语言进行比较，它看起来和感觉上完全不同。表面上看，Elm 和 ReactiveX 也与反应式微服务世界中的实现有很大不同，但它们都实现了相同的推送型信令模型来定义代码中的关系。这种模型通常被称为消息或信号驱动架构。

在我们开始讨论反应系统之前，认识到它们具有分形类型是很重要的。反应性原则适用于所有规模的软件开发。你可以应用它来制作通过物理网络进行通信的微服务。也可以选择通过 RxJS 或者 Elm 语言在自己的前端代码中实现。您还可以自由选择连接和使用用不同语言编写、在不同操作系统上运行的反应式组件，等等。分形类型学允许您将组件的反应性集群组成封装这些集群的更高阶反应性组件。这提供了粒度的灵活性，这是一个非常强大的概念。在我们的系统中，从最小到最高层的抽象使得这个想法高度可扩展。查看来自[戴夫·法利](http://www.davefarley.net/)的演讲，从实用的角度了解更多关于反应式系统的知识。

反应式系统:21 世纪系统的 21 世纪架构

> 我们需要考虑构建系统的新方法。旧的方法是建立在妥协的基础上的，这种妥协是由我们都习惯的硬件的某些性能特征强加给我们的。这些假设不再成立。
> 戴夫·法利

为了查看更多关于如何使用微服务将单片系统拆分为反应式应用的实际例子，请查看来自 [James Roper](https://twitter.com/jroper?lang=en) 的视频。

从整体到被动——都是关于建筑——詹姆斯·罗珀

你可能已经听说过[事件驱动架构](https://en.wikipedia.org/wiki/Event-driven_architecture)，它详细描述了**事件** **发射器****消费者**和**通道**。在这种架构中，具有异步**消息驱动**逻辑系统的反应式编程占用通道的空间，并作为通道的子集存在。这个逻辑系统由[消息中间件](https://en.wikipedia.org/wiki/Message-oriented_middleware)描述，它负责过滤、转换和转发发出的初始事件有效载荷。转换后的有效负载可以通过信号发送到其他松散耦合的接收器类型组件，这些组件可以进一步应用转换或最终的有状态副作用。

虽然有许多事件驱动模式的实现，如 [eventsourcing](https://martinfowler.com/eaaDev/EventSourcing.html) 或 [CQRS](https://martinfowler.com/bliki/CQRS.html) 并且理解它们可能会令人不知所措，但你可以通过阅读[马丁·福勒的](https://martinfowler.com/)博客帖子并观看他在下面的 [GOTO](https://gotoams.nl/) 会议上的 2017 年演讲来获得很好的介绍。

[](https://martinfowler.com/articles/201701-event-driven.html) [## 你说的“事件驱动”是什么意思？

### 去年年底，我和我在 ThoughtWorks 的同事参加了一个研讨会，讨论…

martinfowler.com](https://martinfowler.com/articles/201701-event-driven.html) 

因为事件驱动架构确保了系统的水平可伸缩性，所以当形成反应式应用的主干时，它是首选选项。在这个架构之上，更容易定义清晰的消息传递模型，该模型通过形成松散耦合的反应组件来进一步确保水平和垂直可伸缩性。当流过这些组件时，事件流可以分支或加入到主干中。这种通信配置文件中的组件不能访问其他组件的状态。他们只是订阅事件并采取行动。理想情况下，这些反应组件是无状态的，并且执行纯逻辑。在反应式编程中，所有事件都被同等对待，我们通常不关心这些事件是如何发出的，也不关心这些事件在所有有效负载转换结束时将产生的具体副作用，我们关心的是订阅其他组件的事件的每个组件的明确边界，并定义自己的事件表面供其他组件订阅。我们的应用程序中“发生”的事件被作为可观察的流进行管理，不管这些事件是我们听到的点击事件、我们的 Redux 存储中的状态变化、计划的任务、API 响应还是其他系统发送给我们的推送通知。

## 代理和独立性

事件驱动架构与反应式消息传递相结合，使得我们的应用程序的构建块很好地分离、分布和松散耦合。这是由于事件并不真正关心它们的后果。按键事件可能有许多不同的解释或副作用，但键盘不需要关心这些。它只需要知道它的按钮状态是否被按下。另一方面，对事件及其解释的反应流或反应不应该是键盘所关心的，而是对这种关键状态变化感兴趣的各方所关心的。这与 Andre Staltz 在他的视频中解释的原则是一致的，我在本系列的第 1 部分中介绍了该原则，该原则指出

> 反应式编程处理模块，这些模块定义了如果受到外部事件的影响，它们如何改变自己

因此，如果通过按键我们将改变监视器的亮度，它将只需要订阅键盘事件，并在自身中实现该改变的效果。这种思维方式很好地将键盘和显示器的关注点分开，但是当我们开始构建包含数千个模块的应用程序时，这种思维方式的威力变得更加明显，这些模块之间相互通信，并通过这种关系模型发出能够影响其他模块的事件。

这个原理也可以用这个简单的等式来表达

**a = b + c**

命令式编程说 a 被赋予 b 和 c 之和的值。如果在应用程序生命周期的后期 b 或 c 发生变化，a 保持不变。像 Elm 这样的语言以不同的方式处理这种关系，并且会在 b 或 c 发生变化时更新 a。在这种情况下，等式运算符不指定当前的值，而是指定关系本身。所以 a 有它的函数定义。当 b 被更新时，系统会发送消息来更新 a。您可以在 Excel 中找到类似的应用原则，其中每个单元格都定义了它与其他单元格的关系模型。这呼应了本系列第一部分中[提出的](/sustainable-code-and-reactive-programming-the-footprint-part-1-4b5278d8d78c) [Bret Victor](https://twitter.com/worrydream?lang=en) 描述的即时性原则。这里的即时性由创建实时关系结构的编程模型来保证。

## 现实

这种代理式的交流感觉非常自然，在现实生活中也是如此。活生物体中的细胞不需要开发复杂的心理模型，也不需要知道它们周围有数十亿个其他细胞，但只有知道当几个相邻细胞的状态发生变化时该做什么，这样的系统作为一个整体才能表现出极其复杂的行为，即使对我们最先进的超级计算机来说，模拟这些行为也可能具有挑战性。他们也不需要担心将发生的事件通知整个网络，或者定义其他细胞应该如何对这些事件做出反应。这种系统中的每个代理都定义了自己的代理。

反应模型甚至与社会科学有一些很好的相似之处，在社会科学中，我们可以通过像[行动者网络理论](https://en.wikipedia.org/wiki/Actor%E2%80%93network_theory)或[网络理论](https://en.wikipedia.org/wiki/Network_theory)这样的概念找到描述经济、社会和自然系统的想法。诸如此类的想法试图通过复杂网络的透镜来解释世界，这些复杂网络是由行动者通过各种反应性的互动关系连接起来的。你可以在这些话题上找到很多来自布鲁诺·拉图尔和米歇尔·卡隆的有趣论文。这当然只是另一个例子，让我们看到为什么反应式的世界观如此有效——因为我们发现这是非常自然的——我们习惯于在生活中体验能动性。从这些角度考虑软件有助于我们将系统构建成看起来自然的架构。

> 如今，我们由软件运行的高度互联的全球系统形成了复杂的网络

当思考如此复杂的系统如何工作时，我们的大脑考虑明确定义的代理及其行为也比考虑整个系统简单得多。通过理解行为主体的行为，我们也可以更容易地理解整体的复杂的新兴性质，这些性质大于其部分的总和。

Duncan DeVore 也做了一个关于反应式建筑的很好的采访，在采访中他解释了这种由相对愚蠢的部分组成的简单系统是如何表现出非常复杂的行为的。对他来说，这是反应式编程作为一种技术和反应式架构作为一个整体的主要区别。

我希望这是对反应式架构的一个很好的介绍，请务必查看 comming 文章，在那里我们将通过查看更具体的示例来讨论该架构的各种实现。
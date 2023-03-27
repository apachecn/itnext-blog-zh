# 面向模块的体系结构

> 原文：<https://itnext.io/module-oriented-architecture-4b54c8976415?source=collection_archive---------2----------------------->

***软件开发不是一门艺术*** 除了简单的结构和计算逻辑序列之外，它留给我们的创造力、想象力和智慧的空间非常小。有人可能会说，当通过团队和组织来观察时，所有这些都发生了巨大的变化，我当然会同意这一点，但这是我不会在这个简单的评估中进行的。

> **从将代码输入我的 IDE 的核心开始，我将软件开发视为一场持续的斗争，为不断变化的用户世界(包括人和机器)提供数字解决方案。**

我的行业又大又强。它为我提供了一系列工具、计算机语言和设备，但也许最重要的是，它为我提供了同一个软件开发社区的数百万成员，他们不断分享他们关于如何使用和应用工具、计算机语言和设备来实现用户期望的想法和想法。

> 就我个人记忆所及，我们的奋斗是一样的:努力解决同样的老问题和挑战，或者更好地说，不断兑现承诺和期望。

在这些年的经验中，我们学到了许多最佳实践和概念，作为应对现有挑战的答案。在某种程度上，这是我对软件架构的看法。

这是一系列博客，讲述了创建客户端解决方案的有用方法。这不是一个突破性的概念。它是在整个开发栈中，经过一段时间构建的学习实践的集合。尽管在演示项目中它是作为一个配方来呈现和实现的，但它并不是作为一种药物来开处方的。每个人都可以找到自己的变化，想法，做出改变。有些部分可能看起来有点挑战性，但是花一些时间在演示项目上，你应该能够了解每一个步骤的底部并完全理解它。毕竟反正代码也不多…

这里是增量开发的故事/想法的链接，因此我建议以同样的顺序阅读。

*   [**第 1 部分:耦合和解耦**](https://medium.com/@poksi/module-oriented-architecture-part-1-coupling-and-decoupling-4443dd7f598a)
*   [**第二部分:路由和模块**](https://medium.com/@poksi/module-oriented-architecture-part-2-routing-and-modules-2437e6a292e7)
*   [**第三部分:模块和路由**](https://medium.com/@poksi/module-oriented-architecture-part-3-modules-and-routing-241b06439a9f)
*   [**第 4 部分:不合格模块**](https://medium.com/@poksi/module-oriented-architecture-part-4-non-conforming-modules-9c18ec2d2180)
*   [**第五部分:隐式路由**](https://medium.com/@poksi/module-oriented-architecture-part-5-implicit-routing-655b468ca1b4)
*   [**第六部分:智胜 MVC**](https://medium.com/@poksi/module-oriented-architecture-part-6-outsmarting-the-mvc-26ef66111057)
*   [**第 7 部分:总结**](https://medium.com/@poksi/module-oriented-architecture-part-7-wrap-up-742aaf696de6)

尽管这是一个非常笼统的概念，可以从手机、桌面到网络，但我还是用 Swift 创建了演示项目，因为它已经成为我的主要语言有一段时间了。我没有花太多的心思在代码的改进上，也没有使用所有最好的语义方法，我将把这留给每一个人，特别是每一个人，不管你想使用什么语言。我对这个概念很感兴趣，这个博客的故事关注的是概念，而不是最好的和最新的 Swift 语义。
# 高级 NestJS 技术—第 1 部分—定制装饰器

> 原文：<https://itnext.io/advanced-nestjs-techniques-part-1-custom-decorators-aa6d7f85c5b6?source=collection_archive---------1----------------------->

![](img/e6dd9e4aabd09f3d4d454f2d5d5b582f.png)

“和猫在一起的时间从来不会浪费。”——西格蒙德·弗洛伊德。

好的，首先，一个小小的免责声明:我称这些技术为“高级”,不是因为我认为它们真的很复杂或很聪明，而是因为它们没有在官方的 NestJS 文档中提出，并且这种需求可能出现在中型或大型 NestJS 项目中。

免责声明 2:我本来打算把这篇文章命名为“高级 NestJS 技术——第 1 部分——控制反转”，但是 IoC 是一个广泛的话题，对不同的人来说，它的含义略有不同。尽管如此，下面建议的技术对我来说感觉像 IoC。不同意也没关系。

这里是这样的:假设我们有一个 drinks 模块和一个 DrinkService，它们的工作是返回饮料列表。由于某种原因，饮料可能来自不同的供应商，每个供应商都有自己的模块。OOP 原则告诉我们，解决这个问题的一个方法是:

*   声明一个 DrinkProvider 接口，
*   在 DrinkService 上声明一个 DrinkProviders 数组，并为任何人提供一个向列表中添加新提供者的方法，
*   遍历注册的提供者列表，从每个提供者获取饮料，生成最终的连接列表。

# 首次实施

下面是 NestJS 中第一个“幼稚”的实现:

类似于 SpiritModule，我们想象我们也有一个 BeerModule，一个 SoftModule，等等。为简洁起见省略。

有几件事应该已经让人觉得尴尬了:

*   我们必须将 DrinkService 注入到每个 DrinkProvider 中，这样他们就可以注册自己。
*   我们必须向应用程序的其余部分公开(“导出”)DrinkService，即使我们的应用程序可能只使用 DrinkModule 中的 DrinkService。

# 丰富

为了解决这个问题，我们将更好地利用 NestJS 提供的**依赖注入**机制:

*   我们将创建一个**定制的类型脚本装饰器**，用一个特殊的元数据来注释我们的每个饮料提供者
*   在模块初始化期间，在所有的提供者被实例化之后，我们将尝试**在我们的 NestJS 应用程序中现有的所有服务中检索带注释的提供者**，并在一个地方进行注册
*   然后，我们将思考代码如何变得像以前一样更加解耦，以及上面提到的笨拙的部分是如何消失的。

因此，有趣的部分发生在 DrinkModule 的 onModuleInit 方法中:**我们使用来自 NestJS 核心**本身的 DiscoveryService 来浏览我们的应用程序的所有声明的提供者，并且在 Typescript 的反射特性的帮助下，我们找到我们的带注释的提供者。

注意现在 SpiritModule 不再需要导入 DrinkModule，因此 DrinkService 不再需要从 DrinkModule 导出。

# 结论

总结一些警告:

像我们一样浏览提供者*可能*并不是在所有情况下都有效(cf 模块使用 registerAsync，提供者提供字符串注入令牌而不是类注入令牌，等等。).幸运的是，这个 NPM 模块[https://github . com/golevelup/nestjs/tree/master/packages/discovery](https://github.com/golevelup/nestjs/tree/master/packages/discovery)公开了一个具有更加用户友好的 API 的替代 DiscoveryService(例如参见“providersWithMetaAtKey”方法),并处理了所有的边缘情况。

在某些时候，我们不得不大胆假设我们声明的 decorator 只用于修饰实现 DrinkProvider 接口的类。否则，东西会坏掉。不是在注册期间，而是在检索饮料清单时，这可能发生在很久以后。当然，所有这些都可以通过广泛的测试套件和系统的代码审查来处理；-).

说到测试，我把饮料服务的测试留给你。这并不是一个可以独立于 NestJS 框架执行的单元测试，因为我们需要 NestJS 来完成它的工作，以便注册提供者，等等。这也不是 E2E 测试。但无论如何这是个测试。

你可以在这里找到完整的代码:【https://github.com/paztek/nestjs-ioc-example[。](https://github.com/paztek/nestjs-ioc-example)
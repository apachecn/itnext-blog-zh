# APP_INITIALIZER 是如何工作的？那么关于 Angular 中的动态配置需要了解什么呢？

> 原文：<https://itnext.io/how-does-app-initializer-work-so-what-do-you-need-to-know-about-dynamic-configuration-in-angular-718e7c345971?source=collection_archive---------1----------------------->

当我们开发一个设计为在多种环境下运行的应用程序时，我们必须决定如何根据这些环境提供适当的变量配置

# **TLDR；📕**

*   *environment.ts* 不是提供环境配置的最佳方式
*   不要在构建时引入环境配置
*   名为 *APP_INITIALIZER* 的特殊注入令牌用于提供方便有效的外部配置
*   我已经为你准备了一个最小的例子，通过使用 *APP_INITIALIZER* 来解决这个问题。你可以在[***stack blitz***](https://stackblitz.com/edit/angular-configuration-with-app-initializer)上找到

# 第一轮❓

环境变量最简单的例子是 API 配置。通常，我们根据目标环境使用不同的 URL。例如:

*   戴夫:https://yourdomain.dev.org/api/resources
*   测试:https://yourdomain.test.org/api/resources
*   制片人:https://yourdomain.prod.org/api/resources

在我看到的很多 Angular 项目中，都是利用 Angular 框架提供的类比文件来解决这个问题: *environment.dev.ts* 、 *environment.test.ts* 和 *environment.prod.ts* 。

我们示例中的这些文件如下所示

不能说这种方法很糟糕或者不符合所做的假设。但是，看看如果您选择这种方式来保留您的变量并希望完成应用程序交付到生产的整个周期会发生什么。

1.  如果本地一切正常，应用程序将使用 environment.dev.ts 文件在开发环境中构建
2.  如果在开发环境中一切正常，我们可以将其提升到测试环境中。为了让应用程序在测试环境中运行，这次必须使用 environment.test.ts 文件重新构建应用程序
3.  如果应用程序已经在测试环境中测试过，并且已经准备好生产，我们必须采取相同的步骤。因此，我们需要使用 environment.prod.ts 重新构建应用程序

请注意一个事实。应用程序必须构建三次(**！！！**)在它从开发者手中到生产环境之前。

## 首先得出的结论是:👈

*   如果您想要交付一个小的变更到产品中，您必须重复相同的构建过程几次，这将显著地延长交付时间。
*   您永远不能确定来自开发环境或测试的测试工件将在生产环境中正确构建

基于这些结论和我自己的经验，我可以安全而诚实地告诉您，environment.ts 文件并不适合保存大多数类型的配置。更准确地说:

某些配置类型不应该在构建工件时发布。在 Angular 的 environment.ts 中存储环境相关变量是反模式。不幸的是，常见的反模式。

因此，如果您选择不在 environment.ts 中保存您的配置，您能做什么呢？

# 第 2 轮— APP_INITIALIZER 方法💪

Angular 的注入令牌 APP_INITIALIZER 提供了很大的帮助，有了它，您可以向应用程序引入额外的初始化函数。Angular 框架的这一功能确保了当应用程序启动时所提供的功能被执行，而应用程序仍在被初始化。这些函数返回一个 Promise 对象，Angular 的应用在执行之前不会被初始化。

这是传递配置的正确方式吗？

## 当然啦！即使在官方的 Angular 框架文档中，您也可以看到:📕

> 例如，您可以创建一个加载语言数据或外部配置的工厂函数，并将该函数提供给 [APP_INITIALIZER](https://angular.io/api/core/APP_INITIALIZER) 令牌。这样，该函数在应用程序引导过程中执行，所需的数据在启动时可用。

> [如果你想了解新的伟大的棱角和前端的东西，请在 Twitter 上关注我](https://twitter.com/marcin_milewicz)！*😄*

## 实际运行时配置

首先，我们需要一个文件来保存配置细节，它不会是执行 *ng build* 时构建的 JavaScript 工件的一部分。这个配置文件应该在应用程序初始化时按需下载。

对于以下假设的配置模型。

我们还需要一个负责下载配置并在应用程序运行时使其可用的服务。

最后一步是使用带有 *APP_INITIALIZER* 标记的上述方法。它用于特定 *NgModule* 的 providers 部分。

值得考虑创建一个专门用于获取配置的独立模块，而不是将以下代码添加到主 app.module.ts 中

*加载配置*功能带有*使用工厂*参数。函数本身看起来像这样

通过遵循上述步骤，我们可以确保我们的应用程序具有以下行为

*   当应用程序启动时，来自 *ConfigurationLoader* 实例的 *loadConfiguration* 运行，并开始从*/assets/config/configuration . JSON*下载配置
*   应用程序会一直等待，直到从*/assets/config/configuration . JSON*加载配置
*   最后，通过 *ConfigurationLoader* 实例的 *getConfiguration* 方法，应用被初始化并且配置可用

神奇的是，放置在 *assets* 目录中的配置在构建时没有被考虑。您可以随时修改*资产*目录的内容，而无需重新构建应用程序。因此，将应用程序交付到生产环境的方案现在看起来是这样的

1.  如果本地一切正常，应用程序将构建在开发环境上，并且*/assets/config/configuration . JSON*文件将从外部交付
2.  如果在开发环境中一切正常，应用程序就被提升到测试环境。为了在测试环境中运行应用程序，我们使用已经构建好的工件并替换掉*/assets/config/configuration . JSON*文件，这次它包含了特定于测试环境的配置
3.  如果应用程序通过了测试环境中的测试，并准备进入生产环境，我们再次获取相同的工件，将其推送到生产环境，并为生产环境替换类似的配置文件*/assets/config/configuration . JSON*。

哇！通过使用 *APP_INITIALIZER* 和运行时配置方法，您能够在整个交付周期中将构建应用程序的过程限制为仅一次！坦率地说，根据最佳实践，这一次是最大次数。您还确保了您在初始环境中构建的工件与在最终环境中放置的工件是一个字节。

如果你想看看这个解决方案是如何工作的，我为你准备了一个关于[***stack blitz***](https://stackblitz.com/edit/angular-configuration-with-app-initializer)的实例。

![](img/11bb06e170536b76359bff8fb8d6bf71.png)

# 摘要📕

*   *环境. ts* 是**不是**在角生态系统中提供环境配置的最佳方式。我们可以放心地将其命名为反模式。
*   不要引入依赖于运行时的环境配置
*   注入令牌——APP _ INITIALIZER 用于**方便有效的**外部配置在 Angular 生态系统中的交付。
*   我已经为你准备了一个使用 **APP_INITIALIZER** 解决问题的最小示例。你会在[***stack blitz***](https://stackblitz.com/edit/angular-configuration-with-app-initializer)上找到他

## 反馈很重要，在 medium.com 最好的方法是多次点击拍手图标👏在你的左边。如果你点击它，我会知道我的文章对你有价值，❤️，我会被鼓励写更多📙。谢谢！

> 如果你想了解新的伟大的角度和前沿的东西，请在 Twitter 上关注我。*😄*
# 一个更干净的开始。NET 6 的最小方法

> 原文：<https://itnext.io/a-cleaner-startup-for-net-6s-minimal-approach-c1c05a672c6a?source=collection_archive---------1----------------------->

## 中更简洁的应用程序启动和配置方法。在微软杀死并保留了旧的启动文件之后。

![](img/f6e0a3188777249327438702a129b93a.png)

克里斯蒂安·保罗·斯托比在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

看完[。在过去的一周里，我制定了一个微软推荐方法的替代方案。老实说，我去年创造了这种方法，最近又对它进行了改进，使它可以与他人分享。](https://docs.microsoft.com/en-us/aspnet/core/migration/50-to-60?view=aspnetcore-6.0&tabs=visual-studio)

起点是找到一些关于在新版本中使用`Startup.cs`文件的帖子。NET 6 最小主机模型。我知道不止我一个人觉得微软的格式比什么都讨厌。由于微软取消了默认的`Startup.cs`文件，这让我认为他们认为 Program.cs 会是一个又大又长的烂摊子。在阅读帖子和微软的文档时，我意识到他们有一个建议的实现。问题是我认为实现有点混乱。

# 微软的解决方案

当我通读他们的文档时，我发现…

> *迁移到 6.0 的应用不需要使用新的最小托管模型。*

这是一个惊喜，因为我以为他们已经杀死了`Startup.cs`文件，这是一个问题，因为我发现这侵蚀了他们的最小模型。这种方法导致在`Program.cs`中插入一些额外的行来处理所有的服务和中间件配置。它创建了一个新格式的`Startup.cs`文件，由两个方法和一组注入的配置组成，很容易就完成了工作。

微软记录的应用程序启动方法。网络 6

结合以下 Startup.cs 类

微软记录的应用程序启动方法。网络 6

# 可选择的解决方案

上面的代码工作得很好，它可能看起来有点笨拙，但它确实工作。我提出的另一种方法只是扩展方法的简单使用。没有什么改变世界，但我发现它是一个更精简和干净的外观。扩展方法传入了`WebApplicationBuilder`和`WebApplication`对象。`WebApplication`可以访问`Environment`配置，双方也可以访问`IConfiguraiton`。这保持了旧`Startup.cs`的精神，就像微软的方法一样，但我发现它也保持了`Program.cs`的干净和清晰。

中应用程序启动的替代方法。网络 6

结合下面的`ProgramExtentions.cs`类。

中应用程序启动的替代方法。网络 6

这种方法不会改变你的生活，但我发现了它的价值。可能有点迂腐，但是我喜欢这些扩展方法提供的简单性。如果它们的大小很大，可以针对每种方法将它们分成单独的文件。如果任何配置变得更大更重要，就有可能产生`ConfigureApplication`和`ConfigureExceptionHandling`方法。这种模式的可能性是无限的。

```
**Connect or Support?**If you like this, or want to checkout my other work, please connect with me on [LinkedIn](https://www.linkedin.com/in/stphnwlsh), [Twitter](https://twitter.com/stphnwlsh) or [GitHub](https://github.com/stphnwlsh), and consider supporting me at [Buy Me a Coffee](https://www.buymeacoffee.com/stphnwlsh).
```
# 。NET Core: Windows 兼容包

> 原文：<https://itnext.io/net-core-windows-compatibility-pack-1c8924c2434?source=collection_archive---------8----------------------->

Windows 兼容包

![](img/f1157a16bc9d550aec1db62888ea13c0.png)

一年多以前，微软发布了 Windows 兼容包。NET Core 填补了。NET 核心 API，如果你的应用不需要跨平台运行的话。虽然兼容性包中包含的一些 API 是在支持。NET 标准 2 发布了，比如 System。SQL 客户端。有些 API 永远不会进入。NET 标准，因为它们是仅用于 Windows 的构造，如注册表访问。

由于兼容性包已经发布了很长时间，我想这篇文章会是一个很好的提醒。在这篇文章中，我们将创建一个新的。NET 核心应用程序并使用 Windows 兼容包来访问注册表。

# 应用程序创建

在命令提示符下使用以下命令创建新的。NET 核心控制台应用程序。

```
dotnet new console
```

使用以下命令添加对兼容性包 NuGet 包的引用。

```
dotnet add package Microsoft.Windows.Compatibility
```

# 使用注册表

以下是该应用程序的完整代码。这段代码可能不是最佳实践，但是它演示了注册表的用法。

```
class Program { static void Main(string[] args) { var thing = "World"; if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows)) { using (var regKey = Registry .CurrentUser .CreateSubKey(@"Software\Testing\Test")) { thing = regKey.GetValue("ThingText")?.ToString() ?? thing; regKey.SetValue("ThingText", "Registry"); } } Console.WriteLine($"Hello {thing}!"); } }
```

请特别注意与注册表访问相关的 if 语句，因为它允许您检查您在什么平台上运行并改变您的实现，如果您的应用程序实际上是跨平台运行的，这将非常有帮助，但是当您在特定的操作系统上运行时，您需要一些细微的差异。

应用程序应该输出“Hellow World！”第一次运行和“你好注册表！”第二轮。

# 包扎

如果您正在创建一个新的应用程序，最好尽可能远离特定于平台的 API，坚持使用。净标准。如果你试图转换一个现有的应用程序，我可以看到兼容性包加速你的转换，而不必重写应用程序的一部分，发生在我们基于 Windows 的 API 上。

*原载于 2019 年 3 月 17 日*[*【elanderson.net*](https://elanderson.net/2019/03/net-core-windows-compatibility-pack/)*。*
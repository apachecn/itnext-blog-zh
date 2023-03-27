# 在中创建桌面应用程序。网络核心 3

> 原文：<https://itnext.io/creating-desktop-applications-in-net-core-3-f5fe3d20d2a0?source=collection_archive---------6----------------------->

在我的工作中，我仍然做相当多的与桌面相关的工作，这意味着当我听到这个消息时。NET Core 3 将带来对 Winforms 和 WPF 的支持，我非常兴奋。这篇文章将向你展示如何安装的预览版。NET Core 3，然后使用。NET CLI 到新的 Winforms 或 WPF 应用程序。

![](img/f1157a16bc9d550aec1db62888ea13c0.png)

此信息包含在[中。NET Core 3 Preview 1 公告](https://blogs.msdn.microsoft.com/dotnet/2018/12/04/announcing-net-core-3-preview-1-and-open-sourcing-windows-desktop-frameworks/)来自 12 月，但是我想我最好在这里记录它，因为我正在试用它。

# 下载并安装

前往[号。NET Core 3 下载](https://dotnet.microsoft.com/download/dotnet-core/3.0)页面，选择适合自己操作系统的 SDK。请记住，Winforms 和 WPF 只能在 Windows 机器上运行，而不像。NET 是跨平台的。下载完成后，运行安装程序并按照提示进行操作。

# 项目创建

使用。您可以使用以下命令来创建 Winform 应用程序。

```
dotnet new winforms
```

创建一个 WPF 应用程序。

```
dotnet new wpf
```

为任一类型创建项目后，您可以使用以下命令运行应用程序。

```
dotnet run
```

下面是一个新的 WPF 应用程序的例子。

# 限制

如果你打算玩基于桌面的框架，我建议你抓住 Visual Studio 2019 的[预览。即使有了预览版，在撰写本文时，Winforms 或 WPF 也没有可用的设计器。](https://docs.microsoft.com/en-us/visualstudio/releases/2019/release-notes-preview)

# 包扎

我知道微软已经展示了一些简洁的演示，但是在第一个预览版中并没有太多的工具，所以如果你要像我上面做的那样做更多的新应用，你将会有大量的手工工作。请不要把这当成挖苦，正如我上面所说的，我对这个功能感到非常兴奋。

如果你想知道什么版本的 Windows。网络核心 3 将被支持，信息可以在这里找到。

*原载于 2019 年 2 月 24 日*[*elanderson.net*](https://elanderson.net/2019/02/creating-desktop-applications-in-net-core-3/)*。*
# 每个人都在谈论的布拉索是什么？

> 原文：<https://itnext.io/what-is-this-blazor-everyones-talking-about-34529a3e1419?source=collection_archive---------4----------------------->

> JavaScript 已经被贴上了 web 应用程序开发的主干标签，它允许你创建漂亮、实用的 web 应用程序。但是对于像我这样的. NET 开发人员来说，使用 JavaScript 就像在蛮荒的西部一样。别再烦恼了！Blazor 通过允许我们在浏览器中运行 C#代码来帮助减轻这种感觉！

请继续阅读，了解 Blazor 是什么，以及使用 Blazor 的利弊。

# 布拉索是什么？

Blazor 是一个新的单页面 web 应用程序(SPA)框架，构建于。NET 框架。它允许你。NET 开发人员来构建与使用 JavaScript 相同的优秀用户界面，除了使用 C#。这也意味着我们可以重用现有的。NET 库和我们创建的逻辑，比如字段验证。

现在，有一些有经验的开发人员，包括你真正的开发人员，还记得上一个提供这个特性的框架，[微软 Silverlight](https://en.wikipedia.org/wiki/Microsoft_Silverlight) ，以及它的命运。不过，使用微软 Silverlight 有一个警告，那就是它需要在你的浏览器中安装一个兼容的插件。这对于很多网络用户来说是一个障碍，现在依然如此，尤其是随着移动浏览器的流行。今天对我们来说，好消息是 Blazor 没有附带这个警告。

Blazor 可以在两种不同的模式下运行，服务器端和客户端。每种模式都有不同的优点和缺点，使它们适用于不同的场景。

# 服务器端 Blazor

在服务器端模式下运行 Blazor 意味着我们的 C#代码的所有执行都发生在服务器上。代码执行完成后，对用户界面(UI)的更新会立即发送回客户端，并合并到文档对象模型(DOM)中。Blazor 使用 SignalR，这是一个实时通信框架，允许服务器代码将内容“在可用时立即推送到连接的客户端，而不是让服务器等待客户端请求新数据。”[【1】](https://docs.microsoft.com/en-us/aspnet/signalr/overview/getting-started/introduction-to-signalr#what-is-signalr)

# 客户端 Blazor

当使用客户端模式时，所有代码都在客户端执行，因为一切都在客户端进行，所以应用程序的所有资源都作为静态文件部署。这意味着您可以在任何可以提供静态文件的服务上托管文件，如内容交付网络(CDN)。为了执行 C#代码，Blazor 使用 WebAssembly (Wasm)，这是一种允许在大多数现代 web 浏览器中执行二进制代码的新技术。

要了解 Wasm 的更多信息，请查看 Eric Elliot 的文章。

# JavaScript 没有死

如果你是一名前端开发人员，正在阅读这篇文章，试图跟上最近出现的所有新框架，那么你可能会认为 Blazor 或 Wasm 正在试图取代 JavaScript。让我来消除你的忧虑，因为这与事实相差甚远。Wasm 是对 JavaScript 的补充，而不是替代。

Blazor 背后的团队意识到已经有一个丰富的 JavaScript 库生态系统可供开发者使用。同样，许多公司和企业在多年前就已经对客户端库进行了大量投资，这也是他们提供 [JavaScript 互操作性](https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-3.1#javascript-interop)的原因。

# 好的，坏的和丑陋的

好了，我们已经讨论了足够多关于布拉索是什么和不是什么。让我们来看看利弊。

# 赞成的意见

**1。一个框架开发者已经知道**

该团队围绕已经使用的流行 web 开发框架构建了 Blazor。因此，如果你已经有开发 Razor 页面、MVC 或 Web APIs 的经验，Blazor 的学习曲线会大大缩短！

**2。更快的页面渲染**

因为客户端 Blazor 使用 WebAssembly 来执行代码和呈现页面，所以我们能够以接近本机的速度运行，Blazor 团队正在继续寻找提高这些速度的方法。

**3。客户端模式可以在离线模式下运行**

在客户端模式下运行应用程序时，应用程序需要运行的所有资源和库都会下载到客户端。这允许你的应用脱离互联网运行，或者在一个[离线模式](https://docs.microsoft.com/en-us/aspnet/core/blazor/hosting-models?view=aspnetcore-3.1#blazor-webassembly)下运行。还需要记住的是，这是假设您的站点不需要访问外部服务来运行，如果需要，您已经考虑了数据的预加载或本地持久化，直到可以连接到外部服务。

**4。服务器端通信高效**

使用服务器端 Blazor，没有任何代码或库被发送到客户端，只有少量的 JavaScript 被发送来引导应用程序，使其非常高效地通过网络传输。服务器端 Blazor 也使用 SignalR 在服务器和客户端之间进行通信。SignalR 使用 web 套接字来实现实时的来回通信，而不必等待客户端请求新数据。

**5。代码重用**

布拉佐实现了。NET 标准 2.0，它是。通用的. NET APIs。NET 实现。这意味着我们可以在浏览器、服务器或任何其他应用程序实现中使用相同的代码和库。NET Standard 2.0，并利用 [NuGet](https://www.nuget.org/) 中丰富且不断增加的库。

**6。客户端在浏览器中运行沙盒安全**

长期以来，由于担心恶意意图或数据窥探，用户一直担心运行 web 应用程序是否安全。Blazor 运行在 Wasm 中，因此不需要任何额外的插件或使用确认。它在浏览器沙盒安全上下文中运行。

**7。顶级组件供应商的 UI 组件生态系统**

许多顶级组件供应商，如 [Telerik](https://www.telerik.com/blazor-ui) 、 [DevExpress](https://www.devexpress.com/blazor/) 、 [ComponentOne](https://www.grapecity.com/componentone/blazor) 和 [SyncFusion](https://www.syncfusion.com/blazor-components) 等，一直在关注 Blazor 的开发，并已经开始创建可重用的 UI 组件。这些组件进一步减少了应用程序运行所需的时间。

# 骗局

**1。。NET 框架需要下载到客户端**

客户端模式在浏览器的上下文中执行 C#二进制文件，由于这种客户端执行，所以。需要. NET Framework 库。这导致下载量更大，初始应用加载时间更长。一旦库被加载，它们就可以被缓存。

**2。客户端模式仍在预览中**

与服务器端模式不同，服务器端模式是在正式发布时发布的。NET Core 3.0，客户端模式还在预览版。预计 2020 年 5 月上映。

**3。仅适用于现代浏览器**

由于对 Wasm 的依赖，旧的和非标准的浏览器，如 Internet Explorer 和“瘦客户端”，不能运行客户端模式版本的 Blazor。

**4。调试可能很棘手**

因为客户端模式下的执行是在客户端以二进制形式进行的，所以调试需要定制工具。这些工具目前在功能上受到限制，并且设置起来很麻烦。参见此处的说明[。](https://docs.microsoft.com/en-us/aspnet/core/blazor/debug?view=aspnetcore-3.0)

**5。服务器端模式下性能优势降低**

与客户端模式不同，服务器端模式需要客户端和服务器之间的网络连接，因此，网络中的任何缓慢都会对应用程序的性能产生负面影响。

# 结论

当决定你的团队的下一个项目的技术时，这从来都不是一个容易的决定，我希望我已经为你提供了一些有益的信息来评估你的下一个 web 技术。虽然我希望看到 Blazor 像大炸弹一样起飞，但总是要为正确的工作使用正确的工具。🙂

要了解关于 Blazor 的更多信息，请前往官方文档，或者如果您渴望获得一些代码并安装最新版本的 Visual Studio，只需创建一个新项目并选择 Blazor app。你也可以在这里查看官方入门指南[。](https://docs.microsoft.com/en-us/aspnet/core/blazor/get-started)

## 该系列的其他文章:

1.  每个人都在谈论的布拉索是什么？
2.  如何用 Blazor 建立一个互动 SPA
3.  用空间功能扩展交互式 Blazor 站点
4.  将 Blazor 应用程序部署到 Azure
5.  将 Blazor 应用程序的构建和部署自动化到 Azure

*原载于 2019 年 12 月 12 日*[*https://dev . to*](https://dev.to/onyxprime/what-is-this-blazor-everyone-s-talking-about-25mf/)*。*
# 风格:Dart 的后端框架项目。

> 原文：<https://itnext.io/style-backend-framework-d544bdb78a36?source=collection_archive---------1----------------------->

![](img/7081e7759ba4e1c536b0f6fe09e03815.png)

我需要一个新的开源后端框架，我写的反馈。语法是否适合后端，是否可以理解？我能补充什么？优点和缺点可能是什么？如果你能回答这样的问题，并提出新的问题来支持我这个非常令人兴奋的项目，我会很高兴。

这个项目是开源的。我知道文档中(这篇文章-github 之间)有冲突。我希望这些会得到好评，因为我还在开发中。

[GitHub 资源库](https://github.com/Mehmetyaz/style)

[酒馆套餐](https://pub.dev/packages/style_dart)

> 加入我们的不和谐社区[https://discord.gg/bPcscvBM](https://discord.gg/hdCdYfwS4C)
> 
> 您可以通过电子邮件订阅 medium 帐户，以跟踪事态发展。除了这个系列和风格之外，我不会在其他主题上添加任何文章，所有文章都是免费阅读的。系列的第一个故事:[异常处理风格](/exception-handling-with-style-6020f01af7d8)

## 为什么我需要？

随着 Flutter 最近变得越来越流行，开发人员对 Dart 语言越来越熟悉。所以像我这种不是很有经验或者很多语言都不是高级水平的程序员，考虑把飞镖作为首选来满足自己的需求。

仅使用 dart 语言，就可以一起开发应用程序的前端和后端。但是在后端没有像 flutter 一样容易理解，适合社区开发，并且利用了类结构的框架。

## 重火力点

Flutter 开发人员将 Firebase 用于许多后端服务。

尽管 Firebase 由于其简单性而具有许多优点，但它也带来了缺点，例如服务的不灵活性。

我也对 Firestore 有很多抱怨。我相信很多开发者也是这样。此外，应用程序与 Firebase 和谷歌服务的集成也存在许多问题。

简而言之，尽管 Firebase 有很多优点，但是它的不灵活性和定价政策并不适合更专业的后端。

# 我从哪里开始？

为了满足这一需求，我需要开发一个对社区开发开放的框架(带有软件包版本)并且易于集成。

我之前用一个项目做了这个框架的介绍。

该项目只有控制器，这些控制器为我自己的需求提供了以下服务:

*   MongoDB 集成
*   查询标准化
*   权限处理程序
*   作家（author 的简写）
*   数据库触发器
*   Web 套接字
*   密码系统
*   数据库流

我将这些服务设计成互连的模块。最终，这些依赖性导致后端架构难以管理和测试。此外，由于它完全是为 web socket 设计的，添加新模块变得很困难。

当在我的项目中添加一个标准的错误捕获和日志记录机制时，我更加感受到了这些缺点。

最后，我从一开始就有了足够的动力去开发一个简单明了的新框架，我开始了。

# 类似颤动的语法

首先，我需要设置一个语法。在我看来，一个使用这个框架的服务器应该可以访问多个数据库，连接微服务，轻松管理复杂的路由。这也是一个非常灵活的结构。开发人员应该能够为这个框架编写一个服务，并发布给其他开发人员使用。

我在试图创建一个合适的架构时回顾了 Flutter。我对 Flutter 架构印象深刻，并想尝试一下类似的后端架构会是什么样子。

为什么我需要颤振架构？

首先，在 flutter 中开发接口时，我们可以根据需要将代码分成独立的部分，所有这些部分都作为一个大型互联结构的一部分工作。所以每个小部件都有自己的依赖项。但是在运行时，这些依赖关系可以与父部件和子部件的依赖关系一起正常工作。我们使用相同的类，需要容器)用于不同的视图。

但是我所看到的后端例子并非如此。我们为每个路由和服务编写单独的函数，这些函数彼此之间没有任何依赖关系。如果一个服务有 10 个不同的端点，我们需要对其中的 10 个进行相同的控制，我们要么在其中的 10 个中编写一个单独的函数，要么最多在不同的点调用相同的函数 10 次(或者添加中间件)。

此外，如果您检查 Flutter 源代码，我们会看到一个名为 ComponentElement(无状态和有状态小部件的元素)的漂亮结构。虽然这些小部件实际上对渲染没有影响，但是它们是模块化编码的一个很好的工具。

我还尝试用同样的架构写了一个后端服务。我为路由子路由使用父子结构，为包装器(像中间件一样)构建上下文逻辑。

## 现在我想分享我通过各种例子从代码中得到的结果。

我在示例中使用了图像，因为这种风格还没有发布，现在尝试还为时过早。我觉得不需要抄袭。此外，彩色代码更好地表达了后端编码的风格。

颤振当量:

*   组件:小部件
*   绑定:元素
*   调用:RenderObject
*   BuildContext : BuildContext

在示例中，所有不以“My”开头的类都从框架中提供。

定义服务器:

![](img/0e132f9ee474b5b4bbac574b66ffcbae.png)

手柄`example.com/foo`:

![](img/6325e1083d5479a98ee15c098fdf331c.png)

或者:

![](img/f89e58b301153baefcc73376f3d27d9c.png)

使用网关:

![](img/032a3a4db567105ca28ccf319b482698.png)

使用 route 来用 route 和 argument 路径段进行子路由:

![](img/7f8721deae14f976b755c0ac6bb047d7.png)

# 有状态-无状态

使用、有状态组件和有状态端点存储数据。

你可以在`context`上用`Key`或`Type`找到你的州

![](img/19a4c14bcd50e44a816f7805ba90b549.png)

# 服务

设置默认服务

![](img/bae6b9b7a26980cc3678b8d0e163b128.png)

为服务器的一部分指定服务

![](img/df8efefce997b2dec3af168cd415fe71.png)

到达当前上下文的服务

![](img/f8d41f0477952fed8d1eac706da92b95.png)

# 盖茨

像中间件一样的门

![](img/b1380d92a707cf2a35e62735ba394fdc.png)

门影响它下面的每条路线和端点。

有许多像`MethodFilterGate`、`SchemaGate`、`ResponseSchemaGate`、`PermissionGate`或自定义`Gate`的门。

# 扳机

当请求发送到子节点或响应发送到父节点时触发。

您可以确保已发送或已回复。

![](img/406a2da414d749d68f29eef8fe30c09a.png)

## 操作

![](img/c7f9ac95375eab6907093a2b215973e9.png)

# 再直接的

重定向内部或外部请求

![](img/dc7735bbaed7fdd9442ec6ad2d0b5b31.png)

或者生成重定向

![](img/d4921feb932fbdf4176ca9405b97ea9d.png)

# 异常处理

在任何地方引发异常:

![](img/f532e3728a5e58525ca07baa8ec56a05.png)

设置默认例外端点

![](img/c32305d6b9361338ff0504509b378b25.png)

按路由类型指定例外

![](img/48ac8cd4838a06f2fadc9f1ed472f9c3.png)

## 存取点

![](img/8d382fccf45dea1968b1e4bf247aebb1.png)

# 简易端点

![](img/99fb9714ef0fee41fb8c7e33e1043f00.png)

# 在未来:

首先，许多新的门、重定向器、简单端点等。让我们考虑一下在主标题下要添加的特性。

## CLI 应用程序

将启动和运行服务的命令行应用程序。在这里可以访问和更改许多关于运行服务的信息。

您还可以更改服务和组件的基本属性。例如，您可以停用或激活重定向的网关或地址的子级，或者附加或分离服务。

## 监视

一个具有命令行应用程序所有功能的界面，您可以在其中分析所有端点、服务及其负载。

## 基于角色的身份验证

服务器管理员的授权。使用管理员登录监控应用程序或 CLI，并通过角色所有者管理服务。

## 建设者

构建器从 JSON 构建服务，构建服务器集成 angular-dart 和 docker。

从 JSON 构建服务，如:

```
{
  "unknown": "BerberUnknownRequest",
  "root": "../home",
  "host": "localhost",
  "port": 9090,
  "rootName" : "berber_server",
  "gateway": [
    {
      "segment": "home",
      "handleUnknownAsRoot": true,
      "root": "MyHomePage",
      "child": {
        "gate": {
          "allowed_agents": [
            "webSocket"
          ]
        },
        "gate_child": {
          "gate": {
            "auth_required": true
          },
          "gate_child": "MySecondEndpoint"
        }
      }
    },
```

感谢阅读！

如果你给出反馈，你将影响项目的发展。

祝你今天开心！玩的开心！

[https://github.com/Mehmetyaz/style/tree/main/packages/style](https://github.com/Mehmetyaz/style/tree/main/packages/style)

mehmedyaz@gmail.com
# 用 Apollo 客户端和 React 钩子管理本地状态

> 原文：<https://itnext.io/managing-local-state-with-apollo-client-and-react-hooks-9ad357e6d649?source=collection_archive---------0----------------------->

在本帖中，我们将学习如何在 Apollo 客户端处理本地数据

![](img/fe2b959921270b27075670cd741ad743.png)

塞尔索·奥利维拉在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

# 对地方政府的需求

现代网络和移动应用已经变得非常数据驱动。构建大型单页应用程序是复杂的，因为数据会随着时间不断变化。每次检测这些数据变化并重新呈现 UI 是很痛苦的。

我们希望以最佳方式访问应用程序不同组件中存储的所有属性。

**进入阿波罗客户端**

> Apollo 客户端具有内置的本地状态处理功能，允许您将本地数据与远程数据一起存储在 Apollo 缓存中。要访问本地数据，只需用 GraphQL 查询即可。您甚至可以在同一个查询中请求本地和服务器数据！— [阿波罗文档](https://www.apollographql.com/docs/react/essentials/local-state/#handling-client-fields-with-the-cache)

让我们开始设置 apollo 客户机和本地状态:

# **设置 Apollo 客户端🚀**

Apollo 客户端配置

现在，阿波罗客户端由几个关键部分组成。

*   [**Apollo link**](https://www.apollographql.com/docs/react/advanced/network-layer/)**—**这是用来配置 Apollo 客户端的`network layer`也就是说，Apollo link 可以让你配置如何通过 HTTP 发送查询。
*   [**缓存**](https://www.apollographql.com/docs/react/advanced/caching/) —阿波罗客户端使用阿波罗缓存实例来处理其缓存策略。
*   [**解析器**](https://www.apollographql.com/docs/react/essentials/local-state/)——将本地状态更新实现为 GraphQL 变异
*   [**typedef s**](https://www.apollographql.com/docs/react/essentials/local-state/)**—**到可选设置一个客户端模式，用于 Apollo 客户端。这个模式在 [Apollo 客户端开发工具](https://github.com/apollographql/apollo-client-devtools)中用于自省，在那里你可以在 GraphiQL 中探索你的模式。

让我们把上面的每一部分连接起来。我们将为名为`isDarkModeEnabled`的属性设置一个本地状态。

首先，我们创建一个新的 Apollo 实例`InMemoryCache`。这就是我们当地州的所在地。然后我们定义我们的网络层，以便 apollo 客户机连接到 GraphQL 服务器。

`getState`和`writeState`是读写缓存**中数据的两个辅助函数。**助手函数实现了`cache.readData`和`cache.writeData`，它们在缓存上充当 getter 和 setter 方法。接下来，我们[用默认值初始化](https://www.apollographql.com/docs/react/essentials/local-state/#initializing-the-cache)我们的阿波罗状态。将初始状态写入缓存很重要，这样在触发突变(更新本地状态)之前查询数据的任何组件都不会出错。最后，我们初始化我们的阿波罗客户端。

让我们继续定义类型定义和解析器。我们有一个定义应用程序状态的简单模式。

# 管理缓存📝

当您使用 Apollo 客户端管理应用程序状态时，Apollo 缓存是唯一的事实来源，或者用 redux 术语来说是“存储”。

在`readState`和`writeState`辅助函数的帮助下，我们查询初始/当前状态，然后用新值更新它。为了查询本地状态，我们在 state 上使用了`@client`指令，表示应该从 Apollo 客户端缓存或本地解析器函数中解析它。

现在，让我们用一个简单的例子来看看如何更新应用程序状态

就是这样！所有监听本地状态的组件都将获得`appState`的更新值，并在更新后重新呈现。
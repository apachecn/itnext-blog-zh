# 在 TypeScript 中从头开始使用 React 路由器

> 原文：<https://itnext.io/a-react-router-from-scratch-in-typescript-f0eec6ccb293?source=collection_archive---------1----------------------->

![](img/90e2009ebe9caa6be8ca611b5b33550d.png)

由[埃尔顿·容](https://unsplash.com/@elton_yung?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

路由是每个单页面应用程序的基本部分。在 React 生态系统[中，react-router](https://github.com/ReactTraining/react-router) 是实现客户端路由的最流行的包。有了这个库，路由简单地以优雅和声明的方式完成。但在幕后，有许多先进的概念正在进行，使这一切工作。在本文中，我将探讨其中的一些，并给出 React 路由库的一个示例实现。

在我们开始之前，我有一点要坦白:虽然标题是“从头开始”,但我今天将引用来自 NPM 的两个包，因为重写它们本身至少是一篇文章。但是我仍然认为这个适合于 TypeScript 中的'[一个从零开始的 Web 服务器和节点](/a-web-server-from-scratch-in-typescript-854642a85402)'以及 TypeScript 中的'[一个从零开始的 Workerpool 和节点](/a-workerpool-from-scratch-in-typescript-and-node-c4352106ffde)'(实际上我将只使用一个)。

React 中的声明性路由

# 挑战

现代单页应用程序的路由器库有一些主要的挑战需要克服。首先，它必须连接到浏览器并劫持路由。它还必须实现一些高级设计模式，以创建我们喜欢的 react-router 的声明式风格。我们必须确保用正确的 html 呈现客户端链接(意味着仍然使用`<a>`标签和呈现`href`)。

# 历史 API

JavaScript 中的[历史](https://developer.mozilla.org/en-US/docs/Web/API/History) API 允许开发者监听事件并操纵路由器状态。在这个项目中，我将使用来自 NPM 的 [history](https://www.npmjs.com/package/history) 包，它提供了 HTML5 history API 的跨浏览器实现。它与 react-router 使用的相同。在我们的库中，我将创建一个单例实例，可以在应用程序的任何地方轻松使用。

# 路由器组件

`Router`组件保存路由器的状态。它必须监听来自`history`的事件以更新其状态。每次导航发生时，`listen`回调将被调用并更新`Router`的状态。请注意，我们没有清理我们的侦听器，并且带着有点天真的假设生活，即`Router`组件将总是被呈现。为了保持简单，我将让它保持原样。

# 路由组件

路由是用位于`Router`组件内部的`Route`组件声明的。`Router`将确保只呈现路径与当前 url 匹配的路线。这与 react-router 略有不同，因为我们没有`Switch`组件。这是因为 react-router 使用 react 的上下文将路由器的状态传递给交换机，因此它可以支持嵌套。我们不会走这条路，所以我们不需要`Switch`组件。

`Route`组件与普通组件略有不同。如果你看看它的实现，如下图所示，你会注意到它什么也不做，但仍然是作为一个类来实现的。这是因为`Route`组件的工作仅仅是作为一个指示器存在于`Router`中，它应该被区别对待。

为了使`Route`工作，我们必须更新`Router`组件，以查找其子组件中的任何路由。这里我们将遍历`Router`的子节点，隐藏任何路径与当前 url 不匹配的路由。

这里需要执行多项检查。第一个是检查我们的孩子是不是一个数组。如果是，我们映射它并检查每个孩子是否是一个`Route`。为此，我们在 react 中使用了一个不太为人所知的技巧:如果您想检查某个元素是否是某个组件的实例，您可以将该元素的`type`属性与组件类进行比较。如果是一个`Route`，我们将该路径与状态中的 url 进行比较。除此之外，我们还对孩子的`null`和`undefined`进行了检查，以防止孩子不是`ReactElement`(如`string`或`null`)时发生崩溃。

# 链接组件

`Link`组件使用正确的`href`呈现一个`<a>`标签，但是实际上使用`onClick`将新的状态推送到`history`。我们确实需要这样做，因为我们不希望当用户点击一个链接时页面重新加载，但仍然希望我们的 html 中的链接具有可访问性和 SEO。

当我们把所有这些放在一起时，我们可以编写一个路由应用程序，如下所示。我们还不支持的一件重要事情是通配符和其他有用特性的路径模式。为了实现这些，我建议看一下 [path-to-regexp](https://www.npmjs.com/package/path-to-regexp) ，react-router 也使用它。

# 结论

在本文中，我们已经了解了 React 应用程序中的路由。我希望您了解了所有这些组件如何一起工作来创建 React 社区已经学会喜欢的漂亮的声明式路由器接口。源代码可以在 [Github](https://github.com/WimJongeneel/ts-react-router) 上找到。
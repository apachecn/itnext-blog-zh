# 如何使用 Socket 的单个实例？React 应用程序中的 IO

> 原文：<https://itnext.io/how-to-use-a-single-instance-of-socket-io-in-your-react-app-6a4465dcb398?source=collection_archive---------3----------------------->

在我目前正在开发的聊天室应用程序中，我需要一个单一的套接字。可以由几个 React 组件共享的 IO 实例。在下面的文章中，我将谈谈我是如何做到这一点的。

React 发布了新的上下文 API，并在 v16.4.0 中正式发布。上下文 API 旨在共享可以被视为 React 组件树的“全局”数据。如你所见，它是我们的最佳候选。

因为有些组件需要使用单个插座。IO 实例深深地嵌套在 React 组件树中，我们将创建一个上下文实例并将其存储在一个单独的文件中，这样当组件需要它时，它就可以导入它。

然后，在您的根组件中，您可以创建一个套接字。IO 实例，并使用提供程序来传递套接字。组件树的 IO 实例。这样做了以后，不管一个组件有多深，都可以得到插座。IO 实例。

假设你有一个组件需要使用这个插座。IO 实例，您可以获得单个套接字。IO 实例是这样的:

对了，如果有人对我正在搭建的聊天室 app 感兴趣，可以在这里找到:[https://react.chatboard.com.au](https://react.chatboard.com.au)。你也可以在 GitHub 上找到它的源代码:【https://github.com/spencerfeng/chat-board-react[。](https://github.com/spencerfeng/chat-board-react)
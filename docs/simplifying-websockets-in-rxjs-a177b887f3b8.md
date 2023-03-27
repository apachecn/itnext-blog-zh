# 简化 RxJS 中的 WebSockets。

> 原文：<https://itnext.io/simplifying-websockets-in-rxjs-a177b887f3b8?source=collection_archive---------1----------------------->

![](img/dc5b0515c9520414da64e01b59884559.png)

## 对复杂管道使用 Redux-Observable。

*我是 Redux-Observable 的狂热用户，虽然它掩盖了使用 RxJS 的许多困难，但您仍然需要对 RxJS 有更深入的了解，以处理真正复杂的用例。* *使用 WebSockets，* ***我将向您展示如何制作自己的复杂管道并理解它们！***

# Node.js 中与 RxJS 的 WebSocket 连接

在 Node.js 中，设置与 RxJS 的`webSocket`连接有点困难，因为与浏览器不同，Node.js 没有原生的`WebSocket`对象。有了`ws`库，你需要做的就是覆盖`WebSocketCtor`值，然后你就可以开始了:

很简单，对吧？但是这个版本没有伸缩性。所有这些只是创造了一个可观测的`webSocketConnection$`。它只建立一个连接，您必须在您的消费者中编写重新连接逻辑。

有时服务器会关闭，您希望通过连接到下一台可用的服务器来确保您可以恢复运行。

另外请记住，`webSocketConnection$`是“热”的意思，其他任何监听它的东西都不会启动一个新的，但会监听这个特定的实例。把它想象成一个独生子。

知道了这些限制，您可以改为像这样编写代码:

我们通过使用递归调用增加了复杂度。因为我们正在使用`timer` ( `setTimeout`)，所以我们不必担心抛出堆栈溢出错误。

如果你想要更多的功能呢？取消怎么办？假设您的服务得到了它所需要的，并且不再需要侦听这个 WebSocket 连接？如果您的回调需要知道连接何时终止并重新连接，该怎么办？

我经常在前端和后端应用程序中使用像这样更复杂的功能。例如，当您离开关心 WebSocket 连接的视图时，您通常希望关闭该连接。

这么简单的例子你怎么能做到呢？

为此，您需要监听一个事件，或者更好的是，使用 Redux-Observable。

# redux-可在前端或后端观察到

我已经使用 Redux-Observable 作为前端和后端框架两年多了，几乎用在了我的每个项目上。我甚至为 Node.js 编写了我自己的 [Redux-Observable 后端](https://github.com/Sawtaytoes/Redux-Observable-Backend)库，我已经在许多生产项目中使用了它，例如[智能家居服务](https://github.com/Sawtaytoes/Smart-Home-Services)。该库允许您在 Node.js 中使用 Redux 和 Redux-Observable 而不使用 React 来创建事件源应用程序，如 Kafka、RabbitMQ 等。

我创建的其中一个包可以轻松实现高级 WebSocket 连接。在 Redux-Observable 中设置 WebSockets 需要一些思考，所以让我们一步步来。

# 可重复观察的风格

使用 Redux，首先调用一个动作来建立 WebSocket 连接:

这是可重复观测的。我们不只是在不知名的地方行动。对于本例，我们将在应用程序启动时启动它:

那更好。当`APP_STARTED`消息通过我们的管道时，*然后*我们将开始连接。

## 这是一个更简单的版本

作为免责声明，本文中您将看到的所有内容都是基于单个 WebSocket 连接的。我在生产中使用的代码支持您给它的所有连接。

我本来打算在这些例子中使用这段代码，但是它增加了大量的复杂性，并且需要对 RxJS 和 Redux-Observable 有更深入的理解。

如果您对多插座代码感兴趣，请查看本节中的代码。

## 设置监听器

现在我们有一个动作告诉我们需要建立一个 WebSocket 连接，现在我们需要编写代码来实际完成它。现在不要担心去理解这一切；我们将一点一点地来看:

这最初看起来应该很复杂，但是在把它分解成几个部分之后，它会变得更容易理解。

# 把它拆开

## 开始简单

如果没有这段代码，建立 WebSocket 连接是没有用的，因为这是其余逻辑存在的全部原因。

当一个`webSocketConnection$`消息进来时，调用`receivedWebSocketMessage`动作创建者来格式化该消息，并使其对所有 Redux 和 Redux-Observable 可用。

## 最聪明的 RxJS 运算符:` startWith '

RxJS 给人印象最深的一个运营商是`startWith`。我用了太多疯狂的方式使用这个操作符。这部史诗居然用了两次！

一旦我们创建了一个`webSocketConnection`，我们需要向侦听器发送一条消息，告诉它们连接已经准备好了。使用`startWith`，我们可以立即通过管道推送`CONNECTION_READY`动作，一旦可观察对象被订阅，该动作就会被自动调度；甚至在任何数据传来之前。

在我们使用`startWith`的另一个地方，它相当于:

```
const someFunction = () => { /* do stuff */ }someFunction()
```

这部分代码代码说“一旦这个可观察对象被订阅，立即从这个时间点开始并开始执行”。这让我们可以重用重连监听器在订阅时创建`webSocket`连接。这非常巧妙，也是我认为`startWith`如此强大的原因。

## 连接错误时重新连接

这部分订阅了`webSocketConnection$` observable 并捕捉任何连接错误，就像我们在 Node.js 示例中所做的那样。它甚至还设置了计时器！

我们绝不是递归调用函数；相反，我们正在发送一个动作`RECONNECT_TO_SERVER`,这个动作恰好是我们在这个内在可观察对象上听到的动作。

## 取消

使用 RxJS 最重要的原因之一是**取消**。与没有内置取消方法的承诺和回调不同，有许多方法可以取消可观察性。

当一个可观察对象在 RxJS 中被取消时，它可能是通过一个错误或完成可观察对象。在我们的例子中，我们使用`takeUntil`来监听一个动作，该动作最终在执行的不同阶段完成可观察的动作。

听`action$`，我们可以检查各种动作类型，并保持我们的可观察对象运行，直到一个完成它的动作类型出现，如`DISCONNECT_FROM_SERVER`。

当`takeUntil`打开`RECONNECT_TO_SERVER`时，我们说“完成这个可观察到的连接，因为我们正在重新连接”。

`takeUntil`的第一个实例只监听`DISCONNECT_FROM_SERVER`。这是因为它是`RECONNECT_TO_SERVER`的监听器。如果我们让它在`RECONNECT_TO_SERVER`之后取消，它会在启动后立即取消，并且永远不会启动 WebSocket 连接。

## 为什么这么多取消？

我们在三个地方监听具有`RECONNECT_TO_SERVER`和`DISCONNECT_FROM_SERVER`类型的动作。虽然你会认为当父可观测量完成时，子可观测量也会完成，但事实并非如此。

从`catchError`和`switchMap`订阅的 observables 在处理完之前不会完成。如果父进程发送一个触发`switchMap`操作符的值，并且父进程完成，`switchMap`中的可观察对象仍然有效，直到它也完成。这就是为什么我们需要在这三个地方定义`takeUntil`操作符。

# 再次看到这一切

概括地说，我们首先看了在 Node.js 中用 RxJS 创建 WebSocket observables，然后讨论了如何使用 Redux-Observable 在浏览器或 Node.js 中做同样的事情。

然后我们把这个史诗分解成几个部分，学习 WebSocket 消息如何转换成动作，为什么`startWith`非常有用，如何`catchError`帮助我们处理重新连接，以及如何取消是可能的，但需要一些思考。

# 结论

再看一遍代码，现在更有意义了吗？为什么要这样写以及我们获得的好处清楚吗？在经历了这一切之后，请随意评论你的想法。我很想听听你学到了什么，以及这对你自己的项目有什么帮助。

如果你想看一个现实世界中允许多个 WebSocket 连接的相同代码的生产示例，只需看一下[本节 my Redux-Observable-back end repo](https://github.com/Sawtaytoes/Redux-Observable-Backend/tree/2d561c327421328d64f42dc95c2e24dde5d81bea/packages/websocket/redux/externalConnections)。

# 更多阅读

如果你喜欢你所读的，请查看我关于类似的令人大开眼界的主题的其他文章:

*   [Redux-Observable 将解决您的状态问题](/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [Redux-Observable 的最佳实践是反模式](/the-best-practice-anti-pattern-5e8bd873aadf)
*   [修复一切的 Redux-Observable 代码](/redux-observable-code-that-fixes-everything-b7832b904b6b)
*   [回访:权威指南](/the-definitive-guide-to-callbacks-in-javascript-44a39c065292)
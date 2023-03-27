# 大规模使用 Redux 的秘密

> 原文：<https://itnext.io/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a?source=collection_archive---------1----------------------->

![](img/3e66ebc10b04b313c9d3d8b85d18c674.png)

## 打开一个你从来不知道的工具。

每次我在 Redux 上看到一个演讲或观看一个视频，或者在 React 中使用哪种状态，都是一样的——谈论什么时候使用组件状态、新的上下文 API 或 Redux。

我觉得人们忽略了一些东西，真的。这就是`createNamespaceReducer`的用武之地。当**你总是可以使用 Redux** 的时候，为什么要选择 3 个完全不同的状态系统中的哪一个呢？

本文并不是专门讨论在 Redux 中处理所有状态，而是使用`createNamespaceReducer`来帮助将所有状态放入 Redux 中。

我看到了组件状态的很多好处，但是我不喜欢处理`this`或者使用 JavaScript 类(解释在另一篇文章中)。

当使用 Redux 构建数据驱动的应用程序时，您会很快发现 reducer 的能力有限，并且很难选择要为每个状态部分创建多少个 reducer。你最不想要的就是把你的组件结构和你的减速器连接起来。我创造了`createNamespaceReducer`来解决这些问题。

*免责声明:你为什么要把所有东西都放在 Redux 里？除非它能解决你的问题。我不是说你应该这样做，我只是给这样做的人一个选择。*

# 分离逻辑

我看到了业务逻辑和视图逻辑的巨大分离。Redux 的全部目的是实际开发一个独立于底层视图模型的“ **Redux app** ”。

直接来自文件(有一点解释):

> Redux 可以用作任何 UI 层的数据存储。最常见的用法是与 React 和 React Native 一起使用，但也有可用于 Angular、Angular 2、Vue、Mithril 等的绑定。Redux 只是提供了一个订阅机制，任何其他代码都可以使用。也就是说，当与可以从状态变化推断 UI 更新的声明性视图实现结合使用时，它是最有用的，比如 React 或可用的类似库之一。

没有明说，但是暗示了。如果你有一个独立于用户界面层的数据存储层，你的**应用程序是真正的反应还是冗余的**？

例如，我必须在一个项目中工作，在这个项目中，所有公司项目共享逻辑。所有这些逻辑都在 Redux 中，但应用程序可能会使用传统的 AngularJS、React，或者两者都使用。我需要在许多应用程序中统一这些结构的方法，React 的组件状态和上下文 API 都不适合我们。

用 Redux 做任何事情还有其他好处。由于 Redux 处理所有的状态变化，我的业务逻辑可以由纯无状态函数组成，这使得单元测试非常简单。这些类型的函数很容易推理。

单元测试功能也比反应组件容易得多。你也可以对简单的缩减器进行单元测试，这比复杂的要容易得多。`createNamespaceReducer`消除了对复杂减速器的大量需求，特别是当与异步中间件库配对时，如 [Redux-Observable](https://redux-observable.js.org/) 、 [Redux-Thunk](https://www.npmjs.com/package/redux-thunk) 或 [Redux-Saga](https://redux-saga.js.org/) 。

另一个好处是调试。Redux 有一些很棒的 devtools 来计算出你的应用程序发生了什么变化，状态是什么，以及总体上发生了什么。你也有能力回到过去，调度自己的行动，看看会发生什么。React 的组件状态和上下文 API 都没有这样的东西。

最重要的是，我只需要训练同事改变状态的一种方法，我也知道他们只有一种方法。它简化了很多事情，因为我可以编写一点应用程序逻辑，每个人都可以在我们所有的应用程序中重用。这是相同的动作创建者和相同的执行方式，相同的调试方式和相同的查找问题的地方。

我所看到的唯一缺点是当你必须向精通 React 组件状态的人解释时。对我来说，`createNamespaceReducer`开启了无数的可能性，但是如果你在 Redux 中处理你的所有状态，对一些人来说会觉得有局限性，因为它带走了他们习惯使用的工具。用函数式编程和面向对象编程的相同方式来思考这个问题。当您不能将函数参数和对象属性设置为您想要的任何值时，这将极大地限制您的第一次尝试。

如果我说的听起来不寻常或古怪，这和你在 Redux 文档中看到的是一样的:

> 它帮助您编写行为一致、在不同环境(客户机、服务器和本机)中运行、易于测试的应用程序。最重要的是，它提供了很好的开发者体验，比如结合了时间旅行调试器的[实时代码编辑](https://github.com/reduxjs/redux-devtools)。

那我做了什么？

# 释放力量

最重要的第一步是了解`createNamespaceReducer`如何运作:

```
createNamespaceReducer(
  reducer,
  initialNamespacedState,
)
```

但是你如何真正使用它呢？

首先，编写一个简单的 reducer:

这大概是你以前写过的。它只关心一件事:应用程序是已加载还是正在加载？

接下来，我们设置加载状态操作:

这再简单不过了。我们为应用程序的加载状态创建了一个有限状态机。

问题是，整个应用程序只有一个加载状态。如果我们想在应用程序的其他部分使用这个 reducer 和其他组件，我们将不得不一遍又一遍地编写它。我甚至看到过`isLoading`几乎是每个减速器上的道具的项目！这意味着你会看到复制粘贴逻辑无处不在。

这就是`createNamespaceReducer`的威力发挥的地方。它防止你到处重写相同的逻辑。

下面是一个使用与我们之前相同的加载状态缩减器和操作的示例:

行动现在需要有一个`namespace`道具；否则，您的名称空间缩减器将不知道如何对您正在更改的状态进行分类。另一方面，您的减速器只需要一个改变:将它包在`createNamespaceReducer`中。

为了防止内存泄漏，还需要对 reducer 进行一个重要的修改。注意我们的减速器是如何使用`initialState`而不是`false`进入`LOADED`状态的？在这种特殊的减速器中，`initialState`和`false`完全相同，但在其他减速器中，情况可能并非如此。例如，如果您的初始状态是一个对象或数组，那么您将需要返回到`initialState`，以便您的命名空间缩减器将它从底层状态对象中移除。

另一件要注意的事情是，我是如何从命名空间缩减器中单独导出缩减器的。如果您只导出命名空间缩减器，那么您的单元测试必须测试命名空间缩减器以及`isLoadingReducer`。单元测试的最佳实践是永远不要对别人的库进行单元测试，因为你想测试的只是你自己的功能，所以要把它们分开。

请注意，您的命名空间缩减器总是返回到它以前的状态，除非您传入一个`namespace`prop；因此，传递一个 undefined 的`namespace`会返回之前的状态，而不管动作类型如何。

# 它是如何工作的

为了理解`createNamespaceReducer`是如何工作的，你需要首先理解你的状态在使用它时是什么样子的。

之前:

之后:

很简单。它让您将一个`namespace`绑定到一个状态段。它真的像一个减速器一样工作。它比较前一个和下一个状态，并选择返回一个新的或相同的命名空间对象，就像您通常在 reducer 中做的那样。

在引擎盖下，它实际上只是一个带有一些有用逻辑的对象，但它也可以是其他任何东西，如`Map`或不可变对象。事实上，我在 Node.js 中使用了一个`createMappedNamespaceReducer`,因为我可以使用 WebSocket 连接对象引用来命名空间，而不必为每个 WebSocket 连接提供一个随机字符串。

你甚至可以使用辅助工具`createNamespaceReducerCreator`编写自己的`createNamespaceReducer`。我不打算深入讨论你是如何做的，但是在 GitHub 项目中有相当多的[例子。](https://github.com/Sawtaytoes/Ghadyani-Framework-Redux-Utils)

# 从 REST 调用更新状态

当您从 REST 调用中获得一些东西时，如果您想更新一个条目列表，实际上是一个有效负载，该怎么办呢？与其为你要调用的每一个东西创建一个单独的缩减器，为什么不创建一个超级简单的缩减器，它接受一个`payload`，并真正地把它放入状态？

看起来是这样的:

这实际上与我们最近在一个数据驱动的项目中使用的非常相似，该项目与 REST API 挂钩。它帮助我们快速开发应用程序，并担心我们的异步中间件中的所有 AJAX 调用。

我们能够将这一点推进得更远:

很明显，我大大简化了这部史诗的逻辑。它肯定少了很多东西。如果想了解更多，可以查看我的文章: [*Redux-Observable 可以解决你的状态问题*](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7) *。*

我们有一个专门用于获取列表项的动作，仅此而已。没有存储和重置操作，因为我们的有效负载缩减器处理它。我们所要做的就是让我们的 fetch 动作调用一个`storePayload`动作，当它使用正确的名称空间时，我们就可以开始了！

我们有一个`fetchListItems`动作而不是`fetchPayload`动作的唯一原因是，对于其他类型的数据，这个 epic 不需要以相同的方式触发。如果我展示更多关于实际 epic 的外观，它可能会以不同的方式处理错误，或者在触发 AJAX 调用之前设置加载状态。它也可能以不同的方式格式化返回的数据。重点是，我们可以在状态最终被放入通用 Redux 状态容器之前使用动作来改变状态。

当处理 WebSocket 连接时，如果我想做更复杂的同步逻辑，我会编写更复杂的 reducers。正因为如此，使用`createNamespaceReducer`可以将缩减器简化为可读状态，并减少每种数据类型的大量复制-粘贴逻辑。

# 从命名空间状态中选择

如果你熟悉我的`ReduxConnection`组件，这是它和`payloadReducer`一起使用时的样子:

`createNamespaceSelector`提供了一种通过名称空间抓取项目的机制。通常你会得到一个带有属性`payload`的对象，但是现在你可以得到与你的名称空间同名的属性。

*如果你想了解更多关于* `*ReduxConnection*` *和选择器的知识，请查看我的文章:* [*为什么你不需要从 React-Redux*](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e) *连接。*

# 缺点

我总是确保记录下使用我所建立的某个系统的所有缺点。我认为使用名称空间系统的权衡比创建单独的 reducers 更重要。

我们丢失了一些调试。因为这是实现 Redux 状态的一种不寻常的方式，Redux devtools 不知道什么是`namespace`,除非你点击一个动作，否则不会显示出来。这意味着你必须经历许多名字非常相似的行为，才能找到你在乎的那个。这对我来说是一个主要的缺点，但并不是什么大不了的事情。我做了几个命名的归约器，即使很相似，也能提供一点点的分离，尤其是在存储不同类型的数据时。

名称空间冲突怎么办？这是一个很好的问题！在过去的 9 个月里，我们一直在使用它，这并不是一个问题，但我可以看到它会出现在哪里。如果这是一个特定于组件的动作，我们的过程是将名称空间与组件的名称 1:1 联系起来，如果我们使用数据来控制加载或有效负载状态，我们为正在获取的数据传入一个语义名称，比如`'permissions'`或`'accessToken'`。

两个组件可以被命名为相同的，但同样，它没有出现。解决这样的问题实际上比您想象的要容易，因为您知道名称空间和动作创建者。这很简单，只需搜索其中任何一个，就能很好地理解名称空间冲突可能会在哪里引起问题。

# 释放您自己应用的潜能

代码最终比我计划的更复杂；肯定不是你想自己写的东西，所以我创建了几个带有单元测试的实用函数，并开源给每个人使用。

您可以在这里找到它们:

[](https://github.com/Sawtaytoes/Ghadyani-Framework-Redux-Utils/blob/master/utils/createNamespaceReducer.js) [## sawtaytoes/Ghadyani-Framework-Redux-Utils

### Redux 助手函数和中间件。通过创建……为 Sawtaytoes/Ghadyani-Framework-Redux-Utils 的开发做出贡献

github.com](https://github.com/Sawtaytoes/Ghadyani-Framework-Redux-Utils/blob/master/utils/createNamespaceReducer.js) 

# 更多阅读

如果你喜欢你所读的，你也应该检查我的其他文章；尤其是关于 Redux 的任何事情:

*   [为什么不需要 React-Redux 的“连接”](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)
*   函数式编程的表情爱好者指南:第一部分
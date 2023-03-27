# 谁需要 Redux 呢？

> 原文：<https://itnext.io/who-needs-redux-anyway-1d946e9ffa3b?source=collection_archive---------6----------------------->

![](img/0c8bf5e7b10328577204adb59909ce48.png)

**TL；博士**——【https://github.com/amitayh/no-redux】T2

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fwho-needs-redux-anyway-1d946e9ffa3b)

Redux 是一个非常流行的状态管理库，通常与 React 一起使用。人们喜欢它，因为它提供了一个可预测的数据模型，灵感来自 Elm 架构。然而，JS 社区中有许多人[不](https://medium.com/@machnicki/why-redux-is-not-so-easy-some-alternatives-24816d5ad22d) [喜欢](https://games.greggman.com/game/react-and-redux-are-a-joke-right/) [it](https://news.ycombinator.com/item?id=10940845) 并且更喜欢选择替代品，例如 [MobX](https://github.com/mobxjs/mobx) 。

总结一下通常出现的主要利弊:

## Redux 的优势

*   可预测性和简单性
*   单向数据流和不变性
*   数据和表示的分离
*   可扩展性(通过中间件)
*   受欢迎程度和社区
*   工具支持

## Redux 的弱点

*   样板文件(视图、动作类型、动作创建者、缩减者、选择器等等)
*   没有现成的解决副作用的方法(可通过中间件如 [redux-thunk](https://github.com/gaearon/redux-thunk) 或 [redux-saga](https://github.com/redux-saga/redux-saga) 获得)
*   动作与它们的效果(在 reducer 中定义)是分离的

# 一种相似但不同的方法

在我最近参与的一个项目中，我对自己说“我甚至需要一些第三方库来管理我的状态吗”？React 本身就是一个很棒的库，我可以通过一个不可变的值来控制我的状态，这个值带有在状态之间转换的纯函数。React 会处理剩下的事情。

## 纯洁

我想保持大部分应用程序的纯净，使用过 Scala、Clojure 和 Haskell 等语言，并看到了函数式编程和不变性的好处。

**纯视图**——这个很简单。React 给了我们一个编程模型

```
view = f(state)
```

所以我们需要做的就是写下 *f* 。不需要连接的组件或内部组件状态。我们也可以从 React 的[纯组分](https://reactjs.org/docs/react-api.html#reactpurecomponent)中受益，该纯组分比常规组分具有更好的性能。

**纯态**——再次声明，此处无需赘述。我们可以简单地用一个不可变的值来表示我们整个应用程序的状态。像[不可变. js](https://facebook.github.io/immutable-js/) 或 [Ramda](http://ramdajs.com/) 这样的库可以帮助我们管理不可变状态。

**纯更新**——在 Redux 和 Elm 中，应用程序的一个关键部分是在状态之间转换的功能。在 Elm 中，这是*更新*功能，在 Redux 中，这是*减速器*。它们都具有相同的结构

```
(state, action) => new state
```

传统上，他们在动作/消息类型上使用某种模式匹配或 switch 语句，并返回带有所需更新的状态的新副本。这种方法让我感到困扰的是，每次更新都通过同一个函数，这违反了[单一责任原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)并且可能变得很难维护[即使对于小应用程序](https://github.com/reactjs/redux/blob/master/examples/todomvc/src/reducers/todos.js)。我宁愿使用多态调度，其中更新函数是动作的一部分。

## 行动

与 Redux 类似，我写了“动作创作者”。但是与 Redux 不同的是，这些动作只是更新函数的容器。

这是非常独立和解耦的。我们只需要一种方式来调度这些行动。这并不太难，我们在项目的根部保留了一个可变变量来保存我们的状态。然后我们渲染应用程序，将调度功能作为道具传递。

## 跑步(副作用)

我们希望能够以可控和可预测的方式触发副作用(例如，发出 AJAX 请求)。在 Elm 中，这是通过允许 *update* 函数返回一个[命令](https://www.elm-tutorial.org/en/03-subs-cmds/02-commands.html)以及更新后的模型来实现的。命令本身不会导致副作用，而仅仅是对将要发生的效果的描述。我们将使用类似的方法，允许我们的动作定义一个 *runEffect* 函数。这将保持动作本身的纯净——因为它们实际上并不运行效果。这将在我们的计划之外，在“世界的尽头”的一个地方处理

我们将更新我们的调度功能以允许这种行为:

现在，我们可以定义产生副作用的动作，保持应用程序的其余部分纯净和安全:

这种方法非常简单，但它提供了我喜欢的 Redux 的东西，并消除了一些复杂性和样板文件。它确实有一些问题我想很快解决:

*   **工具:**因为这里没有库，也没有社区支持，所以没有像 [Redux DevTools](https://github.com/gaearon/redux-devtools) 这样棒的工具。然而，我发现添加一个用于调试的实用函数非常容易，它记录每个动作，包括新旧状态。
*   **可串行化:** Redux 的动作只保存动作数据和类型。这使您可以轻松地保存它们，或通过网络传输它们。这对于我描述的动作创建者来说是不可能的。然而，通过在 action 对象上添加一个*类型*属性，并拥有某种可以从其有效负载中重新创建 action 对象的工厂函数，可以很容易地获得同样的好处。

# 结论

这对我来说是一个有趣的项目，向我展示了有时我可以稍微改变一下熟悉的模式来实现更简单的代码库。这并不意味着 Redux 是一个坏的或无用的库——只是你可能并不总是需要它为你的用例提供的一切(对于每个框架/库都是如此)。所以花时间想想你真正需要的是什么，因为简单很重要。

下面是使用这种方法的强制 todo MVC 实现:

*   演示:[https://amitayh.github.io/no-redux](https://amitayh.github.io/no-redux)
*   回购:[https://github.com/amitayh/no-redux](https://github.com/amitayh/no-redux)
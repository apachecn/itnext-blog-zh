# 在反应组分中使用 Redux 还原剂

> 原文：<https://itnext.io/using-redux-reducers-in-react-components-4e92985dd9cb?source=collection_archive---------1----------------------->

![](img/ea3ba7b3f63f9ac1b7c108b0ec72d375.png)

## 脑子，炸了。

丹·阿布拉莫夫最近发了一条让我震惊的微博！

你可以在你的组件中使用减速器？不围！

这让我想起了去年一位同事的挑战。他问我是否可以让无状态的功能组件有状态。长话短说，不是真的，但我最终创建了一个非常简单的 Redux 库，您可以在每个组件的基础上使用。

无状态的功能组件不可能是有状态的，因为如果不传递新的属性，就无法更新功能性的无状态组件。这意味着，在某些时候，您必须在链的某个位置拥有一个有状态的 react 组件。即使使用 React 的上下文 API，您也需要一个有状态的组件来修改它自己的状态，以便将新数据传递给`Provider`的值属性。

我完全忘记了那段代码，并且很难找到它。它被隐藏在一长串名为*Redux-like State Components*的“撰写关于这些主题的文章”中。本文将涵盖我所写的概念，以及它如何允许您在 React 组件中使用 Redux reducers。

如果您对**组件状态**和**状态组件**和**表示组件**(想想上下文 API)的分离感兴趣，这个解决方案可能值得一试。该架构与我今年早些时候编写的 [ReduxConnection 组件](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)非常相似。

# 类冗余状态分量

*免责声明:我去年用了大约 1-2 个小时编写了这个解决方案，从未在生产中使用过。虽然它的功能和工作方式与您预期的一样，但它是一个非常简单的最小依赖性解决方案。*

## 使用状态组件

这个看起来怎么样？让我们从最简单的组件开始，加载状态缩减器:

这个`App`组件在第一次渲染时会显示“正在加载…”。2 秒钟后，它会切换到显示“已加载”。

当你查看`LoadingState`时，你真的无法知道这是一个 ReduxConnection 组件、上下文 API 消费者、状态组件还是有状态容器组件。它遵循与其他所有东西相同的渲染道具模式，并且使得弄清楚发生了什么变得更加容易。

`setLoading`和`setLoaded`功能是预调度的。通常您导入您的动作，然后接收一个`dispatch`函数，但是因为所有的功能都包含在这个状态组件中，所以您可以直接调用那些命名的函数，而不管底层的 API。

## “加载状态”组件

由于我们无法说出`LoadingState`到底是什么，我们需要更深入:

哇，看起来像 Redux 对不对？一个主要的区别是没有中间件(至少在这个版本中没有)，但是无论如何看起来都很相似。您创建一些渲染道具，并将它们传递给一个`createStateSubscriber`函数。我导出了 reducers 和 actions，因为您可以分别对它们进行单元测试。

与 Redux 相比，这在引擎盖下做了很多固执己见的事情。你不直接创建一个 reducer，你创建一个`namespacedReducers`的对象，类似于`createReducers`，`createRenderProps`把你所有的动作包装在一个调度函数里，为你管理状态。

我甚至认为一个组件可以同时处理多个 reducers。我可以预见到一种情况，您可能希望使用不同 Redux 的组合来实现类似 Redux 的功能。

## “FormValidationState”组件

在另一个例子中，我们有这个超级简单的`FormValidationState`组件。它通过检查表单域是否不是空字符串来告诉您是否所有表单域都是有效的。

通常，我会使用像 [Redux-Form](https://redux-form.com) 这样的表单库，但是为了展示一些更复杂的功能，我们将使用我们自己的状态组件。

在`FormValidationState`中，我们有一个单一的动作:`VALIDATE_FIELDS`和一个单一的状态值:`isValid`。

光是看，就能看到 API。使用`isValid`属性检查表单是否有效，并通过传入一个数组`fieldNames`和`formElement`本身来调用验证函数。

这是你食用时的样子:

当您提交表单时，它会进行验证检查，并触发显示或隐藏错误消息。非常简单，可能你永远也不会自己写。

## 真正的魔法

看完`LoadingState`组件后，您可能会对这两个神奇的实用程序感到疑惑:`createRenderProps`和`createStateSubscriber`。

`createRenderProps`本质上是 Redux，但是它对你隐藏了大部分 API。如果我愿意，我可以把它换成 Redux 或 RxJS，而你甚至不会知道。

是我对挑战的一个让步。我不得不使用有状态 React 组件。这个函数的行为就像一个更高阶的组件，但是不包装另一个组件来提供它的状态，它是组件。

您调用`createStateSubscriber`，它会创建一个您可以命名和导出的组件。这消除了大量的样板文件，因此您的 Redux 式状态组件甚至不需要了解底层的视图模型。这意味着你可以得到一个 React、Vue 或 WebComponent，而根本不用改变你的状态组件！

因为这两个函数背后的功能并不重要，所以我将让您自己探索它们:

[](https://snippets.cacher.io/snippet/f3db575bd1f8134ce511) [## 类似 Redux 的状态组件-缓存器片段

### 使用 Redux-like 状态组件和表单验证器以及加载状态组件的完整示例。

snippets.cacher.io](https://snippets.cacher.io/snippet/f3db575bd1f8134ce511) 

# 在反应组分中使用 Redux 还原剂

现在，让我们来看看令人兴奋的丹的推特杰作。您可以使用`createRenderProps`和`createStateSubscriber`在 React 组件中使用 Redux reducers。

我将参考 [*中的`payloadReducer`使用 Redux:createNamespaceReducer*](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)的秘诀:

我们要把这个缩减器变成一个状态容器:

简单明了。同样，您不需要担心 React、Redux 或其他任何东西。只需使用通用的 reducers 和动作创建一个通用组件，就可以得到一个非常非常简单的 Redux 式状态组件。

# 基于 React 的有状态容器组件

丹在他最初的推文中说了些什么？如果我们试图做同样的事情，但比他说的更多，那会是什么样子？这就是`createStateSubscriber`的作用。

我的代码看起来与他的有点不同，但这是我们两个实现的要点:

他的版本需要传递一个`dispatch`函数，而我的版本将它抽象出来。通常我会发现传递一个类似于`ReduxConnection`的`dispatch`函数，但是在这种情况下，我希望这些状态组件与底层架构完全分离。他们内部已经有了完整的 API，所以你应该只调用他们允许的动作。

通过向组件类添加动作并从这些组件中调用 dispatch，您也可以在他的示例中轻松地复制自动分派的函数:

```
someAction(payload) {
  this
  .dispatch({
    payload,
    type: 'SOME_ACTION',
  })
}
```

我的解决方案更复杂，因为它做了很多事情，比如抽象出底层架构，甚至没有 React 或 Redux 的概念。我最初编写这段代码时没有考虑 Redux，最终在处理它时使用了相同的 action-reducer-dispatch 概念。通量概念简化了我想要完成的事情。

如果我有困难，我会照丹的方法去做。当 React 已经处理它时，没有理由写我自己的。如果我需要独立于底层架构的东西，我会使用已经写好的东西。关于`createStateSubscriber`最好的一点是，在引擎盖下发生什么并不重要。你可以使用任何你想要的:Dan 的 tweetable 模型或者一个更复杂的类似 Redux 的订阅者模型，或者两者都用。您应该真正构建最符合您需求的解决方案。

# 尝试它，使用它，打破它

用这个 CodeSandbox 自己尝试类似 Redux 的状态组件:

CodeSandbox 中类似 Redux 的有状态容器组件

*免责声明:我试图关闭 appeller 的自动格式化，但当我将代码粘贴到 CodeSandbox 时，它仍然格式化它。*

# 更多阅读

如果你喜欢你所读的，你也应该检查我的其他文章；尤其是关于 Redux 的任何事情:

*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [为什么不需要 React-Redux 的“连接”](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   函数式编程的表情爱好者指南:第 1 部分
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)
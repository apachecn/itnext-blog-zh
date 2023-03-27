# RxJS:缓存和 AJAX 竞争条件

> 原文：<https://itnext.io/handling-cache-and-ajax-race-conditions-4cb152db8764?source=collection_archive---------3----------------------->

![](img/c86d09e5365b863733b8086d446832d9.png)

照片由 [Victoire Joncheray](https://unsplash.com/@victoire_jonch?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

## RxJS 的“startWith”有一个秘密用例

*我创建了一个非常酷的方法来同时使用缓存和 AJAX 版本的数据。*

# 问题是

当处理存储在缓存中的数据时，您希望确保数据是最新的。有很多方法可以解决这个难题，但是我提出的方法是**首先检索缓存版本，然后检索最新版本**。这样，您的应用程序可以快速地从缓存中读取数据，但是真实的数据仍然可以在后台加载。

为了防止任何奇怪的 UI 问题，另一个重要的部分是只传递更新的 AJAX 数据。不管实现如何，这意味着您需要将缓存的版本存储在 state 中的某个地方，以便随时可以与返回的 AJAX 数据进行比较。

# 承诺的方式

我第一次尝试这样的事情是在 3 年前，为了一个 24 小时的工作编码挑战。感谢一位天才建筑师帮助我思考这个问题，我能够在 6 个小时内完成一个工作原型。尽管如此，这仍然是一个很难解决的问题。

当时，我只知道回调和承诺。您可能知道，承诺只发出一次值，所以我用自己的承诺处理程序逻辑包装了 jQuery 承诺实现，并创建了一个发出两个值的自定义承诺。然而，这对于任何人来说都是不可见的。你只需要创建一个`CachedDataFetcher`对象，然后从那里开始。

因为我没有确切的代码，也不想泄露公司机密，所以我打算根据记忆重写 API。

它看起来像这样:

*jQuery 的 promise 实现与官方的 ECMAScript 标准不同，这就是为什么这个 API 有一个* `*done*` *和* `*error*` *方法。*

`create`函数是工厂模式的一部分，它返回`new CachedDataFetcher()`你传入的任何选项。除了`done`方法触发获取数据的承诺，而不是在`fetch`被调用时，其他一切都与您所期望的相似。

今天改写一下，以下是我对该实现的简化看法:

*我故意没有添加错误处理逻辑，这样更容易理解。真正的实现要复杂得多。*

API 一般，示例和实际实现相当糟糕，但我希望你能明白我的想法。

只要您返回的东西是与普通承诺 API 兼容的，您就可以用`CachedDataFetcher`轻松地切换出代码库中现有的数据调用，并且只需要最小的改动。从其他人的角度来看，这是一个正常的承诺；但它的好处是能够发出两种价值。

既然我已经在 observable 上工作了几年，那么使用 observable 来输出缓存和 AJAX 版本的数据就更有意义了，但是当时，我是在用我所知道的东西工作。尽管这有点像谎言。我也在 Knockout 中使用了 observables，但是由于它们与 DOM 绑定在一起，所以我从未想过尝试将它们用于这个特定的目的。

# 随着时间的推移可观察到的

现在，我会在 Redux-Observable epic 中运行这个逻辑，并触发 Redux 操作。相反，我将向您展示昨晚我是如何使用带有主题的直接 RxJS 和令人惊奇的`startWith`操作符解决这个问题的。谈谈隐藏的潜力！

我们不能使用`race`,因为我想要两个值。唯一的另一种方法是使用`merge`，但是这样做的话，AJAX 响应可能会在缓存响应之前先返回。你不想用坏数据覆盖好数据。

这就是我如何利用`startWith`来模仿缓存和 AJAX 行为的:

如你所见，这比偷工减料的许诺少了许多行，也更容易推理，但是它是如何工作的呢？

一旦我们从缓存中获得了我们想要的值，我们就开始我们的`ajax` observable。当我们的`switchMap`发生时，AJAX 调用立即发生，而`startWith`在任何东西通过管道之前首先运行。这给了我们时间来准备`distinctUntilChanged`中的本地状态，并将缓存的值传递给我们的订户(如果它存在的话),同时保持我们的 AJAX 逻辑简单。

不会让值通过，除非它们与之前通过的值不同。这满足了我们“如果与缓存相同，就不要从 AJAX 更新”的要求。

在这种情况下，我们的订户是一个主题。我们不是私下里做`store.dispatch`，而是通过将主题`itemList$`传入`subscribe`方法来调用`subject.next`。

您可以用这种方式模仿 Redux-Observable 的相同功能，但是如果已经有一个实现可以为您完成所有这些工作，为什么还要编写自己的实现呢？我有一个不是很大的快速项目，所以我自己完成了它，但一般来说，我会使用一个有良好文档的已建立的库，以便未来的开发人员(包括我)可以维护它。

## 缓存和 AJAX 随着时间的推移

虽然这解决了 3 年前的原始问题，但这只是昨晚问题的一部分。我需要它来随着时间的推移更新项目列表值。这意味着我的 AJAX 调用需要每隔一段时间发生一次，而我的缓存值只发生一次。

使用与前面相同的代码，解决方案很简单，只需添加`interval`:

`startWith`的位置很重要，我们已经在同一个管道中使用了两次。这就是诀窍。

正如你可能知道的，`interval`就像`timer`一样，只有在计时器超时后才会触发。在这种情况下，它会在发出 AJAX 调用之前等待 1 个小时。使用`startWith`，我们可以在订阅时立即启动 AJAX 调用，同时通过管道传递缓存的值！

我认为`startWith`非常棒。还有`endWith`，当一个观察完成时，它提供类似的功能。这是一个非常巧妙的技巧，以直观的方式简化了这个解决方案。

# 更多阅读

如果您对更多与 RxJS 和 Redux-Observable 相关的主题感兴趣，您应该看看我的其他文章:

*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)
*   [你不应该需要从 React-Redux 中“连接”](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)
*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
# Redux-Observable 的最佳实践是一种反模式。

> 原文：<https://itnext.io/the-best-practice-anti-pattern-5e8bd873aadf?source=collection_archive---------3----------------------->

![](img/5e37836519d39aed9c45a2ddcd11d10c.png)

[亚历克斯](https://unsplash.com/@worthyofelegance?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

## 从不同的角度看待事物。

我遇到过这样一种情况，我提出的一个想法被称为反模式。实际上，我的印象是这是一种最佳实践，而当前的做事方式会适得其反。

这个问题是关于 Redux-Observable 及其动作的自动调度。我开始使用 **Redux-Observable** 已经快 2 年了。在那段时间里，我已经解决了很多难题，并且习惯了用疯狂的方式来编写复杂的可观测量。

到目前为止，我做了一些事情来改善体验:

*   [消除冗余](/redux-observable-without-redux-ff4a2b5a4b39)的需要。
*   [增加更好的错误记录](https://github.com/redux-observable/redux-observable/pull/674)。
*   将长的内部变量分解成单独的函数(通常会增加不必要的间接性)。

编写复杂的史诗在生产应用程序中很常见。很难跟踪什么时候执行了哪些操作，尤其是对团队中的新成员，并且很难跟踪当您继续创建越来越多的内部观察时发生了什么。

我最近参与了一个使用 Redux-Thunk 的项目，我惊讶地发现分派动作是如此容易。我知道大多数人都是这样开始的，但我不是。

虽然我绝对不喜欢让一些动作创建者返回承诺的做法，但我确实很重视在你需要时**调度的能力。**

主要好处？您的代码更简单，更易于维护，并且位于左侧。代码越多，要处理的闭包就越多，也就越难找出在哪里调用了什么。这也被称为回调地狱。

Observables 应该是回调的替代方案，但是如果没有调度的方法，你最终会得到很长的圣诞树管道。我还发现，将大量可重复观察的史诗分解成运算符和可观察对象会增加不必要的间接性。

我要用更好的方法来证明这一点。但是要注意，这种更好的方式被认为是一种[反模式](https://twitter.com/_jayphelps/status/1184300479809015808)。我不是想嘲笑杰伊·菲尔普斯或其他致力于 Redux-Observable 的人，我只是想对一个非常重要的建筑决策提出强烈的反对意见。

# 删除最佳实践？

我将浏览一下从[PR 删除对 Redux 的](https://github.com/redux-observable/redux-observable/pull/346) `[store](https://github.com/redux-observable/redux-observable/pull/346)`的访问的更新片段。

这可能是你在 **v0** 中看到的:

如果你正在写这样的史诗，你需要重新考虑你的策略。一个动作使用`store.dispatch`显式分派，而另一个使用 Redux-Observable 的自动分派(分派任何通过管道的东西)。

有两种做事方式不是一个好的模式。为了让每个人都简单，你需要有一种做事的方式。

你以前用过 AngularJS 吗？有 3 种不同的几乎相同的数据管理方式(服务、工厂、提供商)，几乎每篇文章都谈到了它们。可悲的是，没有人对“为什么”给出一个好的解释。从我参与的项目来看，每个人都同意使用一种方法，这使得事情变得简单。

当 Redux-Observable 转到 **v1** 时，它也切换到具有自动分派动作的单一方法:

这两个例子非常简单。听一个动作，同时输出两件事情。基于这些例子，这种改变看起来没什么大不了的，但这并不是我工作过的项目中大多数史诗的样子。如果这些能代表大多数史诗的样子，这篇文章就不会存在了。

在失去`store`的时候，另一个选择是`state$`。这肯定是优越的，因为你可以直接订阅它。但是我们也失去了`dispatch`。

最近，我一直在想，对自动调度的依赖可能是反模式的。

让我们看一下[反模式的定义](https://en.wikipedia.org/wiki/Anti-pattern):

> *反模式是对重复出现的问题的常见反应，通常是无效的，并且有可能产生非常不利的后果。*

我提出的方法会增加生产力(在我看来)，而不是像自动调度那样降低生产力。我会陈述我的观点，但最终，你会是法官。

# 真正的问题是

当我开始使用 Redux-Observable 时，`store`仍然可用，但我被告知 v1 将改变它的工作方式。我没有学习旧的方法，而是使用新的模式创建了自己的 API 兼容的`combineEpics`函数，并在 v1 发布时切换到官方的函数。

正因为如此，我从来不知道之前发生了什么，所以我从来没有真正考虑过任何与我被告知的不同的事情。

从教授 Redux-Observable 到 noobs，我发现很难解释自动分派的概念。一旦学会了，概念就简单了，但是自己写那个代码？那真的很难。这也是 Redux-Observable 的 Gitter 上最常见的问题之一。

在我看来，Redux-Observable 是最好的包装器 RxJS，这是一个杀手级应用程序，它使每个人都可以使用复杂的应用程序。要理解自动调度，你必须真正掌握 RxJS，这与你想要的杀手级应用相反。

当使用自动调度编写复杂的史诗时，您知道如何通过将管道分成许多更小的并行管道来将多个动作传递到一个管道。在大多数情况下，您订阅了一个`action$`管道，但是您将它分成了许多其他管道，每个管道发送自己的动作，这些动作会被自动分派。

下面是一个例子:

我知道很多人不垂直写代码，但我会。这样写有很多好处，但这超出了本文的范围。我想提供至少一个典型项目中的例子:

这两个例子都不太清楚；尤其是对一个 RxJS noob。我不知道分派了什么(除了我明确地定义了分派发生的地点和时间。这将 epic 从 3 个管道减少到 2 个管道，我们的最大跳转级别从 10 级减少到 7 级。这大大增加了可读性！

我们还能够将`fetchUserSucceeded`和`setLoaded`移出 AJAX 管道，因为`catchError`的输出不再有机会通过它们。

除了比之前稍微长一点的`catchError`之外，我们的其他逻辑都减少了不少。我们可以通过创建自己的`catchError`包装器来简化这一点:

根据我的经验，这是前端应用程序中非常标准的史诗。根据项目的需要，我已经写了许多更疯狂的程序，内部可观察的程序变得难以处理。

在过去的项目中，缺乏`dispatch`功能实际上使我远离了从单个 epic 调度多个动作(Redux-Observable 的好处之一)，因为它增加了复杂性并降低了可读性。

一部分来自创造许多内在的可观察性，另一部分是派遣的不确定性。有时你认为一个动作正在被分派，但后来发现它没有。如果我直接调用`dispatch`，我就不会遇到这种情况。这是非常确定的。

另一个例子

我们还应该看看相反的例子。与其听一个可观察，调度多个动作，不如听多个动作，调度一次，效果如何？

在这个例子中，唯一改变的是`map(pong)`。现在它看起来像这样:

没那么干净。我用这个例子来说明`dispatch`并不总是最干净的选择，但至少它不会让事情变得更糟。

# 你可能会认为在`dispatch`中包装函数就像你在 React-Redux 的`mapToDispatch`参数中看到的那样会解决这个问题。我会提出相反的观点。这个函数所做的只是让我们更难搞清楚到底发生了什么。毫不奇怪，在 hooks 版本中它被删除了。

另一个提议

史诗般的争论

现在，史诗有三个参数，`action$`、`state$`和`dependencies`。如果你要传入多个参数，我更喜欢把它们作为一个单独的对象:

有了对象，你不必知道顺序，所以你可以只解构你需要的。它们有点难以记忆，但你可能不应该记忆你的史诗。还有，`dependencies`是一个对象，我看不出为什么`action$`和`state$`一定要排除在外。

# 如果 epics 接受单个对象，我们可以动态地改变 Redux-Observable API，而不需要完全重写所有 epics。例如，如果旧版本的 Redux-Observable 在一个对象中传递了`store`,那么当`state$`到来时，您就不必预先更改任何东西。

## 现在有了多个参数，改变 epic API 就困难多了；因此，添加`dispatch`需要从第三个参数中抓取一个对象。

从不自动调度

如果`dispatch`是 Redux-Observable 的一部分，自动调度应该被阻止并完全删除。随着时间的推移，用两种方法做同样的事情会使代码更难维护。

虽然这在今天可能很难做到，但是您总是可以关闭您的`rootEpic`并添加`ignoreElements()`到其中，而无需更改任何其他代码。

当然，这将导致相当多的问题，所以像这样的解决方案可能会更好:

## 有了这些代码，你可以与你当前的代码库集成，并随着时间的推移将`dispatch`添加到 epics 中。

工作示例

对于一个将`dispatch`作为依赖项的工作示例，您可以使用这个 CodeSandbox:

专家假说

用 Redux-Observable 的人不都应该是 RxJS 专家吗？不。这个库太棒了，我讨厌它只限于了解他们的东西的人和团队。为什么要让事情变得不必要的困难呢？

## 引自[可重复观察的文件](https://redux-observable.js.org/docs/basics/Epics.html):

*Redux-observable(因为 RxJS)真正在复杂的异步/副作用方面大放异彩。如果您对 RxJS 还不熟悉，您可以考虑使用 redux-thunk 来处理简单的副作用，然后使用 redux-observable 来处理复杂的事情。这样你就可以保持高效率，并在工作中学习 RxJS。redux-thunk 学习和使用起来要简单得多，但这也意味着它的功能要弱得多。当然，如果你已经像我们一样热爱 Rx，你很可能会用它来做任何事情！*

# 在我看来，Redux-Observable 取代了 Redux-Thunk。我甚至用 Redux-Observable 编写了整个应用程序，除此之外什么都没有。

按照这种逻辑，Redux-Observable 既强大又复杂。如果我们能够显著降低复杂性并保持高水平的能力，会怎么样？你甚至需要建议 Redux-Thunk 吗？

结论

> 我真的很喜欢 Redux-Observable 提高了我编写异步模块的能力。令人惊讶的是，我无意中发现了[面向消息编程](https://medium.com/@Sawtaytoes/redux-without-state-15e2f839055c)的实现，却没有意识到这一点。

经过近两年的使用，[做了一次演讲](https://www.youtube.com/watch?v=v4z2OufSSYY)，[写了许多文章](/redux-observable-can-solve-your-state-problems-15b23a9649d7)，并在许多生产项目中使用它，我对*真正的*反模式有了不同的结论。

有可能我完全错了。也许这个想法根本就不怎么样，我也没有看到什么重要的东西。如果是这样的话，**我很乐意得到一些强硬的反馈和批评**。你喜欢这个想法还是你知道一些我不知道的事情？

# 我肯定不知道人们在`dispatch`被移除之前会陷入什么样的奇怪境地，但我知道它的移除会显著增加许多已经很复杂的史诗的复杂性。这就是我想要解决的问题，我所知道的最好的方法是把一个想法放在那里，看看我是否能得到任何合理的反对意见。

更多阅读

如果你喜欢你所读的，请查看我关于类似的令人大开眼界的主题的其他文章:

[Redux-Observable 将解决你的状态问题](/redux-observable-can-solve-your-state-problems-15b23a9649d7)

[回访:权威指南](/the-definitive-guide-to-callbacks-in-javascript-44a39c065292)

# [承诺:权威指南](/promises-the-definitive-guide-6a49e0dbf3b7)

[带有表情符号的函数式编程基础知识](/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)

*   *原载于 2019 年 10 月 16 日*[*https://dev . to*](https://dev.to/sawtaytoes/the-best-practice-anti-pattern-jj6)*。*
*   [Callbacks: The Definitive Guide](/the-definitive-guide-to-callbacks-in-javascript-44a39c065292)
*   [Promises: The Definitive Guide](/promises-the-definitive-guide-6a49e0dbf3b7)
*   [Functional Programming Basics w/ Emojis](/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)

*Originally published at* [*https://dev.to*](https://dev.to/sawtaytoes/the-best-practice-anti-pattern-jj6) *on October 16, 2019.*
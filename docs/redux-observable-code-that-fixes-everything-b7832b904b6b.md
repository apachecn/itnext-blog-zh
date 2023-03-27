# redux-修复一切的可观察代码。

> 原文：<https://itnext.io/redux-observable-code-that-fixes-everything-b7832b904b6b?source=collection_archive---------3----------------------->

![](img/5d0a115226c640a23d6739c0f004603b.png)

[田宽](https://unsplash.com/@realaxer?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

## 使得复杂的冗余可观测数据更容易查找。

## 一个简单的提示就能让你成功。

如果您曾经使用过 Redux-Observable，那么您可能会遇到这样的情况:您需要在管道的顶部分派一些东西，然后再做其他事情。

这是一个常见的问题，但不是一目了然的问题。因为 Redux-Observable v1 会自动调度任何通过管道的内容，这意味着您必须拆分管道以完成第一个操作，然后切换到另一个继续原始管道的可观察对象。

# 管道分裂

它看起来像这样:

使用这种方法，您最终必须以一种非常复杂的方式分割管道，这对于初学者来说并不明显。这在所有用例中都有效，这也是我目前为止设置所有 Redux-Observable 项目的方式。我甚至说 [*Redux-Observable 里的流水线分裂会解决你的状态问题*](/redux-observable-can-solve-your-state-problems-15b23a9649d7) 。

Redux-Observable 有一些奇怪的地方，管道分割(如本例所示)是影响新手的最大问题之一。可悲的是，这也是*类的事情 Redux-Observable 应该使之变得容易。*

*我经常在 [Redux-Observable 的官方 Gitter chat](https://gitter.im/redux-observable/redux-observable) 上看到关于管道拆分的问题(或者缺乏如何拆分的知识)，一个朋友今天在他使用 Redux-Observable 的第二天问了我同样的问题。*

# *反模式*

*Redux-Observable 自动将任何失败的操作分派给订阅者。因此，我在我的文章中提出了一种不同的模式来解决这个问题: [*Redux-Observable 的最佳实践是一种反模式*](/the-best-practice-anti-pattern-5e8bd873aadf) 。*

*我对管道分割的解决方案是从 Redux 公开`dispatch`方法。我还要求你不要让行动通过管道，以确保你不是有时自动调度，有时手动调度。*

*看起来是这样的:*

*很快，您应该能够看出这是一个重大的改进。不仅保留了顺序执行，而且从逻辑上讲，这对人们来说很有意义。*

*它也减少了你跳转的深度，并且更容易可视化按顺序执行的事件序列。这是一个非常巧妙的解决方案，我还没能找到它崩溃的情况。*

*需要注意的一点是，这种方法依赖于副作用的同步执行。在`tap`中调用`dispatch`是一个副作用。当您调用它时，在下一个操作符执行之前，必须运行一堆 reducer 代码和其他史诗。这可能会也可能不会立即显现出来。由于 Redux 同步执行 reducers，这就是为什么它这样工作的原因。*

*对于大多数人来说，我认为手动分派比自动分派更有意义。它更直观，但不是官方认可的。事实上，它被主要开发者 Jay Phelps 称为反模式。*

*不过，这并不意味着什么坏事。杰伊在 Twitter 上澄清说，我正在以非设计的方式使用该库，而不是说我在做一些破坏的事情。*

# *埃弗特·鲍方法*

*Evert 是谁，这是什么方法？Evert Bouw 是 Redux-Observable 的贡献者，也是该库的生产用户。*

*在 ReactConf 2019 上展示了 Evert 我的反模式方法后，他提出了一个利用自动分派的不同解决方案:*

*他解决的关键是`startWith`和`endWith`。看看这个特殊的例子，它极大地简化了定时动作的输出。它甚至会跳出与反模式版本完全相同的标签。*

*一些警告:*

*   *您会丢失管道操作符的顺序执行。`startWith`必须放在任何计算当前值的管道下面。*
*   *你必须知道`startWith`是存在的，当它在任何其他操作符(包括传入值的操作符)之前被订阅时，它首先在管道中执行。*
*   *如果您需要`endWith`中的`state$`，那么您将需要返回一个带有动作数组的`concatMap`操作符。`concatMap`采用一个回调函数，确保您在那个时间点拥有`state$`，而不是在内部可观察对象被创建的时候。*

*Evert 的方法让我大吃一惊。虽然我以前使用过`startWith`和`endWith`，但是我从来没有考虑过在这个特定的场景中使用它们。当我重构没有`dispatch`可用的现有项目时，这将是一个巨大的帮助。*

## *你可以做得更多！*

*`startWith`和`endWith`都带多个参数。每一个都单独得到输出，就像你使用`of`可观测值一样:*

*和我上面的注释一样，记住`endWith`接受一个值，而不是一个回调。要在运行时获得最新的值，请使用`concatMap`或类似的代码。我再怎么强调这一点也不为过，因为如果你不小心的话，它会引起海森伯格综合症。*

# *结论*

*这些解决方案中哪一个最适合您的项目？**你想出了哪些解决同样问题的方法？***

*我仍然不确定 Evert 的方法是否在每种情况下都有效，但我会尝试一下，看看结果。这里肯定有潜力，但在超级复杂的史诗中(我似乎经常写的那种)，管道分裂或`dispatch`可能是唯一的方法。*

*我鼓励你**评论你对这个话题的想法和体验**，这样我们才能真正改善 Redux-Observable 开发者体验。*

# *更多阅读*

*如果你喜欢你所读的，请查看我关于类似的令人大开眼界的主题的其他文章:*

*   *[Redux-Observable 将解决您的状态问题](/redux-observable-can-solve-your-state-problems-15b23a9649d7)*
*   *[Redux-Observable 的最佳实践是反模式](/the-best-practice-anti-pattern-5e8bd873aadf)*
*   *[回访:权威指南](/the-definitive-guide-to-callbacks-in-javascript-44a39c065292)*
*   *[承诺:权威指南](/promises-the-definitive-guide-6a49e0dbf3b7)*
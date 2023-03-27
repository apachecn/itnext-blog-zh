# React 的错误边界。你还在用它吗？

> 原文：<https://itnext.io/reacts-errorboundary-are-you-using-it-yet-a42f3369e983?source=collection_archive---------3----------------------->

## 2017 年就有了，但是没人知道这件事。

您最近是否浏览过 React 文档？作为开发人员，我们应该尽最大努力保持最新状态。浏览文档是学习新事物的最佳方式。这就是我们如何发现 React 的错误边界。

![](img/ac7ed30f7ee4f45669aef0b8b065b424.png)

承认你的错误，有错误的界限。[布雷特·乔丹](https://unsplash.com/@brett_jordan?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# 我为什么要用它？

在你关闭这篇文章之前，请等待！是的，误差边界是从 2017 年开始的。然而，React 最近为 React DevTools 引入了一个新特性！

这个特性将允许开发人员测试不同的组件将如何处理错误。它将允许您使用 DevTools 模拟一个“崩溃”的 React 组件。

当然，你*应该*实现错误处理逻辑，就像任何其他应用程序一样。然而，准备一些东西，以防万一坏了也不是个坏主意。还记得墨菲定律吗？作为受驱动的开发人员，我们希望避免任何可能对我们的用户产生负面影响的事情。

> 任何可能出错的事情都会出错。
> 
> ~墨菲定律

# 误差边界？那是什么？

`ErrorBoundary`组件从 React 16 就有了。也就是实际上…从 2017 年开始！然而，许多开发人员并不知道它的存在，我也没有看到它被经常使用。

浏览 [React 文档](https://reactjs.org/docs/getting-started.html)我们可以找到[错误边界](https://reactjs.org/docs/error-boundaries.html)的定义。

> UI 某个部分的 JavaScript 错误不应该破坏整个应用程序。为了解决 React 用户的这个问题，React 16 引入了“错误边界”的新概念。
> 
> *~ React 文档*

正如文档所述，`ErrorBoundary`组件不会捕捉任何事件处理程序和异步代码错误。

## 好吧，但是它有什么用？

它限制了由于错误而损坏的用户界面区域。想象一下下面的 React 应用程序。

如果这些组件中的任何一个损坏，整个应用程序的 UI 都可能损坏。我们不想那样。相反，我们可以将错误的范围扩大到应用程序的特定块。例如，假设`Content`和`ReadMore`应该只相互影响，而不影响`Header`和`Footer`组件。

很简单，对吧？只需在要隔离错误的组件上定义错误边界。

# 给我一个现场演示！

是的，肯定的！React 的团队为这个做了一个[的精美演示。通过点击直到 5 次点击，这将打破用户界面。顶部组件没有设置错误边界，因此我们可以看到它们同时断开。然而，对于底部的两个元件，使用了误差边界。如果一个组件损坏，另一个组件仍将运行。](https://codepen.io/gaearon/pen/wqvxGa?editors=0010)

React 团队制作的 React 错误边界演示。

# 这个误差边界成分从何而来？

有一个陷阱。这个组件你得自己做。好消息是，这非常简单，并且在出现错误时允许完全定制！您甚至可以基于 React 文档中给出的示例。

如您所见，错误边界组件只不过是一个 React 组件，带有额外的生命周期方法。我们有`getDerivedStateFromError()`和`componentDidCatch()`可以用来根据错误修改 UI，或者记录错误。这只是一些例子，但还有无限多的可能性。

正如您所看到的，React 的文档建议将错误状态保存在状态中，并使用它来呈现错误而不是实际的组件。

# 功能组件呢？

不错的捕捉，它确实是一个类组件！功能组件尚不支持错误边界生命周期。这可能会在将来添加，因为 React 团队正在为 DevTools 推出关于这方面的更新。

# 这是全新的吗？

不，当然不是。这是对 DevTools 的改进，允许更容易地调试和测试这些错误边界，然而，错误边界自 2017 年以来一直存在于 React 中！这个更新可能会给开发人员带来更多对错误界限的关注。

# 告诉我有关此更新的更多信息！

[布莱恩·沃恩](https://medium.com/u/a72dd4e6268?source=post_page-----a42f3369e983--------------------------------)在推特上发布了一张新更新的 gif 图片。我们可以看到一个新的选项来故意“崩溃”一个特定的组件。反过来，这让我们可以测试错误边界，而不必实际造成组件崩溃。谢谢布莱恩和鲍！

# 下一步是什么？

拓宽您对错误边界的视野，并阅读一些开发人员同行的意见！

Bryn Bennett 有一篇关于错误界限的非常深入的文章，我强烈推荐这篇文章。

[](https://javascript.plainenglish.io/how-to-fail-with-style-error-boundaries-in-react-402d8a8f53de) [## 如何在风格上失败:React 中的错误界限

### 在构建用户界面的过程中，处理意外行为与处理预期行为同等重要

javascript.plainenglish.io](https://javascript.plainenglish.io/how-to-fail-with-style-error-boundaries-in-react-402d8a8f53de) 

如果你想要更多的资料，Anam Soomro 已经写了一篇简短但有力的文章，来全面掌握错误边界背后的概念。

[](https://levelup.gitconnected.com/erro-2a61f82526a) [## React 中的错误边界

### 错误处理为用户保持了一个平滑的界面，即使事情变得不顺利。

levelup.gitconnected.com](https://levelup.gitconnected.com/erro-2a61f82526a) 

现在，您已经准备好开始自己的冒险，开始构建令人惊叹的应用程序。只是不要忘记错误边界！

PS:不要忘了不时看一看 [React 文档](https://reactjs.org/docs/getting-started.html)！

[订阅我的媒介](https://kevinvr.medium.com/membership)到**解锁** **所有** **文章**。通过使用我的链接订阅，你是支持我的工作，没有额外的费用。你会得到我永远的感激。
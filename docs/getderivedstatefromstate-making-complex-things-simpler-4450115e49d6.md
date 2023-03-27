# getDerivedStateFromState —让复杂的事情变得简单

> 原文：<https://itnext.io/getderivedstatefromstate-making-complex-things-simpler-4450115e49d6?source=collection_archive---------1----------------------->

![](img/5512e1f58414a9e18f9f9e5d4572e20c.png)

流动

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fgetderivedstatefromstate-making-complex-things-simpler-4450115e49d6%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

我喜欢反应。我喜欢 React 的行为方式。它不只是试图解决问题，而是试图正确地解决问题。

> 特设，RenderProps，组成，组件，无状态，有状态，twitts 从丹和社区，讨论，解决方案，问题和发现。

在去年的生日派对上，我们得到了一种全新的处理道具变化的方法——静态`getDerivedStateFromProps`。然后屎风暴。然后打关于如何使用它的文章。然后是十几期关于某人如何不能使用它的问题。

> JFYI: `getDerivedStateFromProps`，只是定义了如何让 StateA 和 Props 形成 StateB。而且是单程过境。

问题很简单，有时你必须根据你拥有的状态和道具形成一个状态。有时候你有什么道具也很重要。

有成千上万的情况下，你需要它。有一个[讨论](https://github.com/reactjs/reactjs.org/issues/721#issuecomment-378574657)关于从`componentWillReceiveProps`到`gDSFP`的重构。有一个 [PR 来“修复”它](https://github.com/reactjs/rfcs/pull/40#discussion_r180818891)。而且被拒绝了。应该是这样，因为他们忘记了一个用例。

# 测试|行为驱动的开发

为了更好地理解，让我们创建一个我们将要解决的真实世界问题，并解决它。并让它将简单的任务— **显示一个表格**。

> 显示带有分页和排序的表格。

1.  人们应该做什么来呈现那张桌子？

*   获取表格数据
*   分类
*   在表格数据中选取一些范围
*   展示一下。

2.如果数据被更改了怎么办？

*   从头开始一切

3.怎么换一个页面？

*   this . setstate({ page:new page })；

4.如何改变排序？

*   this . setstate({ sorting:new sorting })；

6.如何应对状态变化？

*   不知道。说真的！

6.我能使用道具而不是政府吗？

*   当然可以。创建一个“智能”组件，它将控制一个“笨”组件。
*   那个“笨”的可以得到它需要的所有东西作为道具，并使用`gDSFP`实际排序它将得到的`data`作为道具，使用它可以从道具中读取的`sort`设置，并在状态中存储从`page`到`page+1`的切片。
*   每次你可能改变任何道具，它会从头开始做所有的事情，同时排序不是一件很快的事情。

7.我可以用州名吗？

*   不，你不能。因为你必须对状态变化做出反应，即有人改变了`page`——它应该得到`state.sortedData`，生成一个数据片并存储为`state.result`。但是没有办法对状态变化做出反应。没有办法`getDeviredStateFromState`，即使这篇文章是以它命名的。

8.没有解谜的方法吗？

*   嗯……我们来谈谈吧。

# 解开谜题

## **状态机方式**

这很复杂。**有限状态自动机**定义了一个状态如何转换到另一个状态。

*   当你在`idle`，收到`page change`，转到`recalculation state`。
*   在`recalculation state`输入—重新计算值，转换到`idle`状态。

您可能需要为每个可能的变化定义一个`calculation state`来优化 CPU 的使用，这就是它应该如何工作。底下。在内心深处。更好—在白板上。

## 反应方式

目前官方解决这个难题的方法，是在渲染时使用**记忆。**

它同一时间([链接](https://github.com/reactjs/rfcs/pull/40#issuecomment-378956267)):

> [一个解决方案建议](https://github.com/reactjs/reactjs.org/issues/721#issuecomment-378574657)记忆该计算并每次调用它，这是一个好主意，但实际上这意味着管理缓存，当你处理一个采用多个参数的函数时，这大大增加了潜在错误和错误的发生。我不得不求助于一个奇怪的多深度`WeakMap`，并决定何时丢弃不同级别的缓存。[本·斯泰尔斯](https://medium.com/u/4e17883ba85d?source=post_page-----4450115e49d6--------------------------------)

让我换个说法——bug、科技债务、宇宙末日。

同时，我希望有一个非常简单的东西——简单、稳定、无 bug 的解决方案。我不再要求什么了。只为这个。

# 好的。怎么会？！

要解决这个任务，任何类似的任务，任何不像这样，但类似的任务，你可能只需要两样东西。你认识他们两个。

## 1.第一件事——瀑布。

我知道瀑布很烂，敏捷法则。但是你知道些什么呢？维基百科的第一行包含了我们需要的一切:

> **瀑布模型**是相对线性的[顺序](https://en.wikipedia.org/wiki/Sequence)设计[设计](https://en.wikipedia.org/wiki/Design)。

它描述了 ***如何一个接一个*** 。可预测的、强制性的方法。

> 首先—排序。第二—切片。第三—渲染。

你可以定义一次“工作方式”,它总是一样的。

## 2.第二件事——重新选择。

重新选择是选择和记忆库。很多人更喜欢它而不是其他级联的记忆库。

> 有两个记忆值，如何形成最终的记忆值。(简单)

## 但是

但是我同意 Ben 的观点——一个接一个的级联内存化函数,*“管理缓存，当你处理一个带有多个参数的函数时，会极大地增加你潜在的错误和错误的空间”*。

重选级联是超级强大的东西，但使用它们并不容易，只要你必须“提取”每一步所依赖的所有“真实”变量。艰难，无谓和脆弱的时刻。

第三件事，“流动”，可以解决这个问题。

# 流动

流程，**记忆组成**，字面`composed`记忆功能列表。您可以定义如何形成最终状态，并执行序列。

这比无组织的，甚至是混乱的以随机的顺序调用一些函数，或者对状态/属性的变化做出反应要好得多，只要你不能预测变化的顺序，结果可能是不确定的。

但是我不喜欢它的样子。

*   你弄脏了一个渲染函数。
*   您不能执行副作用，如获取选定页面的数据。因为渲染必须是纯净的。如果没有，哦，不好的事情可能会发生。
*   有些道具被具体化为流程定义，而有些则没有。

> 一定有更好的办法！

# 同样的，但更好一点的方式

它看起来几乎与 lodash 版本相同，但行为不同:

*   你提供一个`input`
*   `input`将被用来调用第一个函数。
*   `output`将第一个函数合并回来，并用来调用第二个函数。
*   第二步的结果将被用作结果。

并且每一步都知道它将从`input`中读取哪些值，从而知道`input`中的哪些变化以及`flow`中的哪一步应该重新计算。这不是魔术，这是`memoize-state`，代理的力量(在 IE11 中有效)

[](/how-i-wrote-the-worlds-fastest-react-memoization-library-535f89fc4a17) [## 我如何编写世界上最快的反应记忆库

### 实际上我已经试着写了最慢的一个，而“最快的 JavaScript 记忆库”是一年前写的…

itnext.io](/how-i-wrote-the-worlds-fastest-react-memoization-library-535f89fc4a17) 

你从“单一的真理来源”获得数据，并把它返回。而且所有的计算都是在`getDerivedStateFromProps`内部进行的。因为 MemoizedState 是一个“智能”组件,“您肯定可以使用”,只是比通常的更智能。

## 为什么这是一个解决方案？

1.  你可以从任何数据形成`input`。将你的状态和你的道具合并在一起。
2.  您可以根据需要在`flow`中使用任意多的步骤，并且每个步骤将仅对观察到的键的变化做出反应，或者对先前步骤的结果做出反应，如果该键是在先前步骤中形成的。并且每个下一个不能使用来自前一个状态的数据。只是存在于状态中的数据，而不管其来源。
3.  执行“顺序”总是一样的。犯一个错误是非常困难的。而且超级容易预测。
4.  你可以控制副作用。

是的——你可以运行副作用。您可以定义流程中的一个步骤，在页面上触发(例如)更改，以及`setState`什么的。如果你将改变的不是页面，而是别的东西——你将**不会运行**副作用，因为调用参数没有改变——这就是记忆化函数的主要原理。

# 更多证明？

这里有一个全脂的例子。看着简单，其实很简单。但是解决一个复杂的任务。

> 再次:表单输入，再次运行序列，重新运行数据更改的“正确”步骤。

# 你刚才提议的是什么？

我提议利用`react-memoize`的力量，更具体地说是`MemoizedFlow`组件。

[](https://github.com/theKashey/react-memoize) [## 记忆/反应-记忆

### react-memoize -不要忘记缓存您的{props|context|state}。🤞

github.com](https://github.com/theKashey/react-memoize) 

只有记忆国家才能提供的神奇记忆。你得试试。

MemoizedFlow 是没有人添加的 getDerivedStateFromState，是您可能需要的 getDerivedStateFromProps。

## 它没有解决的问题

没有`prevProps`。你可以把它们复制到状态(作为流的最后一步？)但你可能不会“想要”它。您可能想要访问最后一个`derived`状态，甚至可能已经访问过了。

```
<MemoizedFlow 
  input={this.state}
  flow={[
     ({stateVariable}) => ...
     ({oldState}) => ...
     ({oldState,...rest}) => this.setState({oldState: rest})
  ]}
> 
```

但是它会导致无限循环，只要你改变你所依赖的变量。但是没有超能力可能是件好事。

> 不给你创造一个非常复杂的算法的能力，让你把事情变得简单，可能是好的。

“记忆化流程”就是让复杂的事情变得简单。仅此而已。

关于本库(和本文)可以解决的“问题”的更多信息:

[](https://github.com/reactjs/rfcs/pull/40#discussion_r180818891) [## ' getDerivedStateFromProps()'中的' prevProps '由 catamphetamine 拉取请求#40 reactjs/rfcs

### 向 getDerivedStateFromProps()添加 prevProps 参数如果不合适，请不要犹豫关闭这个 RFC:我只是…

github.com](https://github.com/reactjs/rfcs/pull/40#discussion_r180818891) [](https://github.com/reactjs/reactjs.org/issues/721) [## 讨论:componentWillReceiveProps vs getDerivedStateFromProps 问题#721 …

### 你好，我们发现了一个场景，删除 componentWillReceiveProps 会鼓励我们编写更糟糕的代码…

github.com](https://github.com/reactjs/reactjs.org/issues/721)
# 我应该什么时候开始使用 react 挂钩？

> 原文：<https://itnext.io/when-should-i-start-using-react-hooks-745f09318399?source=collection_archive---------4----------------------->

react 挂钩及其创建原因的简要介绍

![](img/e4ef2f8ca4fbc468d28638583a3bee92.png)

[春蕾居](https://unsplash.com/@chunlea?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/hooks?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

你是否在一家仍在使用 react 16.8 之前版本的公司工作？如果是这样，你可能不是唯一一个。他们是许多在大型企业应用程序甚至小型应用程序中工作的人，想要权衡在他们的应用程序中引入钩子的利弊。

好消息是 react 并不打算取消类。所以你仍然可以用 react 类来开发，但是他们已经淘汰了一些东西。我不打算在这里列出它们，因为这完全取决于您使用的版本。如果你有一段时间没有升级，你可能有很多工作要做。

因此，简而言之，如果你应该用 react hooks 重写你的整个代码库，答案是否定的。你应该保留现有的代码，并尝试引入 react hooks。

react 核心团队已经解释了使用 react 钩子的许多[好处。我在这里总结一下他们说的话。](https://reactjs.org/docs/hooks-intro.html#motivation)

**组件间难以共享状态**

历史上，当你开始编写功能组件时，你必须使用某种第三方库来避免钻柱或创建额外的组件。Hooks 试图通过允许你制作可重用的代码块来解决这个问题，而不必使用渲染道具模式或更高阶的函数。

你可能遇到的另一个常见问题是，你首先编写了一堆组件作为功能组件，然后意识到你需要添加一些状态或生命周期方法。因此，您需要不断地在 react 类和功能组件之间转换。Hooks 试图解决这个问题。

> 钩子允许你在不改变组件层次结构的情况下重用有状态逻辑。

## 降低复杂性

我相信每个人都知道这种痛苦。复杂性从来不会让事情变得简单。钩子降低了组件的复杂性。如果将编写 react 类所需的代码量与带有钩子的 react 功能组件进行比较，就会发现代码量要少得多。

> 钩子允许您根据相关的部分将一个组件拆分成更小的功能(比如设置订阅或获取数据)，而不是根据生命周期方法强制拆分

## 课程可能会很紧张

对于学习反应的人来说，课程可能是一个粗略的入门障碍。一节课很多人都不知道这是什么。有了钩子，你可以把任何东西都写成一个功能组件。

hooks 的另一个好处是它开始为未来铺平道路。核心团队认为，有了钩子，他们也许能够[提前](https://en.wikipedia.org/wiki/Ahead-of-time_compilation)使用 react 进行编译。类也带来了 javascript 的热重载和缩小的问题。

> 钩子让你不用类就可以使用更多的 React 特性。

## 如何开始使用挂钩

以下是让 react 挂钩正常运行的一些基本步骤。

1.  升级至 react 的最新版本或至少 16.8
2.  修复任何中断的更改(react 团队声称没有中断的更改，但如果你像我一样在你的一些项目中，你没有保持最新，有一些东西已经中断或被否决。)
3.  [学会反应钩子](https://reactjs.org/docs/hooks-overview.html)。花些时间做一个关于 react 钩子的教程。

希望这篇文章对你有所帮助。如果您有任何问题，请在[推特](https://twitter.com/DuffinAvery)上留言或联系我。

编码快乐！
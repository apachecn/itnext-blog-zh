# Vue 开发人员的 React 路线图

> 原文：<https://itnext.io/reactjs-roadmap-for-vuejs-developers-4edacff6bafb?source=collection_archive---------2----------------------->

我学习 React.js 不到两个月的经历

![](img/c85f1afbf95d7fc38b1b42e2234bfd0f.png)

[图片信用](https://medium.com/javascript-in-plain-english/i-created-the-exact-same-app-in-react-and-vue-here-are-the-differences-e9a1ae8077fd)

我已经做了两年的 Vue.js 开发人员。我不能撒谎我在 Vue.js 上有多开心，开发了各种各样的网站。但在我的新工作中，我被要求学习 React.js。作为一名前端开发人员，你应该总是知道框架来来去去，但永远存在的是 HTML、CSS 和 Javascript。所以我需要为这份新工作学习 React.js，我必须尽快开始工作。

为了学习新概念，我通常从视频教程开始，学习基础知识，然后做一些简单的项目，以便更好地适应新的 API。后来，为了更深入地理解每个概念，我在网上搜索并阅读更深入的文章。

但是使用 React.js，我已经熟悉了很多基本概念，比如组件、V DOM、反应性等等。此外，我在开发像 SPA 和 SSR 应用程序这样的 web 应用程序方面有足够的经验，这使得 React.js 速成班对我来说太无聊了。此外，这次我没有时间去上 30 个小时的课程。所以我想出了一个新的方法，这个方法出乎意料地给了我在短时间内开始 React 开发工作的知识。

在这篇文章中，我将分享我的路线图，并链接到一些资源，这些资源在过去的几个月里帮助我学会了反应并自信地开始我的新工作。

# 第一步:概述

![](img/d699442f62c53eddf47dc3eb0f2ce171.png)

[埃斯特扬森斯](https://unsplash.com/@esteejanssens?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

在我看来，学习任何新的框架或语言，都有两个学习阶段，我给它们取了两个名字:**学习概念**和**学习语法**。

例如，学习 Vue.js 中组件的定义是一个概念，但学习如何在 Vue.js 中编写组件是一个语法范畴。我通常同步学习这两方面。但是这一次，由于两个框架中的一些概念非常相似，我决定先学习概念，再学习语法。

首先，我对 React.js 作为一个库的亮点进行了一般性研究，并学习了最著名的术语。

所以你的第一步是拿起笔在笔记本上记下这些问题的答案(保持答案简短，切中要点):

*   React 是如何工作的？
*   什么是 JSX？
*   什么是`ReactDOM`？
*   类组件与功能组件？
*   状态 vs 道具？
*   生命周期方法有哪些？
*   什么是高阶组件(HOC)？
*   Redux 是什么？
*   什么是`setState`？为什么它是异步函数？
*   什么是`render()`法？它返回什么？
*   什么是 Create React App？

您可以在 React 文档和/或网络上的其他文章或视频中找到这些问题的答案。

这里有一篇文章很好地回答了上述问题:

[*常见问题:React JS 面试问答*](https://medium.com/@vigowebs/frequently-asked-react-js-interview-questions-and-answers-36f3dd99f486)

请记住，这一步不应该超过一天。这里的主要目标是理解一般概念。

# 第二步:映射你的知识

![](img/b1a3b77fd920b4ee5d00f509a386c11a.png)

照片由[诺德伍德主题](https://unsplash.com/@nordwood?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

现在我们知道 React 不仅仅是一个标题，让我们花点时间比较一下 React 和 Vue，并将我们从 Vue.js 中了解到的内容与 React 的共同点对应起来:

1.  希望它们都有组件。尽管 React 和 Vue 中的组件之间存在一些基本差异，但它们都使用组件作为前端应用程序的构建块。
2.  React 和 Vue 中的组件使用 props 接收来自父母的数据。
3.  与 Vue 类似，props 在 React 组件中可以有一个默认值和验证。
4.  React 组件中的状态与 Vue 组件中的数据非常相似。但是状态不应该在 React 中变异，相反，它们应该由 setState 更新。
5.  React 没有指令和修饰符。
6.  它们都有生命周期方法。
7.  …

正如你已经注意到的，React 和 Vue 在概念上有很多相似之处。

# 第二步:让我们写一些反应

![](img/718ff4c5d5f0f3b32ae228db72cbe280.png)

照片由[凯特琳·贝克](https://unsplash.com/@kaitlynbaker?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

既然我们知道的 React 比我们想象的要多，那么让我们直接进入代码，用一些真实的 React 代码来弄脏我们的手。此时，您最好的朋友将成为 React 文档和堆栈溢出。作为热身练习，您可以遵循以下步骤:

*   使用 Create React App 创建一个新项目，并构建您的第一个 [**功能组件**](https://reactjs.org/docs/components-and-props.html#function-and-class-components) ，上面写着“Hello World！”。
*   现在从**道具**传递一个名字给你的组件，这样你的组件就可以对任何提供的名字说“你好”。将“世界”设置为您姓名的**默认值**。
*   现在从外部传递一个颜色代码列表给你的组件，确保你总是收到一个颜色列表，并使它成为必需的。(提示:**道具验证**)
*   阅读 React 钩子和`useState`。
*   有了上一步的知识，您可以在组件中添加一个按钮，将文本颜色随机更改为您从道具中接收到的颜色之一。
*   阅读关于`useEffect`和依赖关系的内容。
*   根据您在上一步中获得的知识，更改您的组件，使其仅在第一次呈现时打印“欢迎使用此组件”。

干得好！现在，您比以前更加了解 React 组件。

# 第三步:更多的学习

![](img/e5be56f4a62835f488ec55feffefcc5d.png)

照片由[王思然·哈德森](https://unsplash.com/@hudsoncrafted?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

在你开始低估 React 的复杂性之前，让我列出一长串你需要掌握的概念，在开始一个真正的项目之前。

*   阅读关于 JSX 的 [**条件渲染**](https://reactjs.org/docs/conditional-rendering.html) 和 [**循环**](https://reactjs.org/docs/lists-and-keys.html)
*   学 [**学**](https://reactjs.org/docs/hooks-intro.html) 更深入
*   了解 [**上下文**](https://reactjs.org/docs/context.html) 及其使用案例。学习如何将 [**上下文与**](https://reactjs.org/docs/hooks-reference.html#usecontext) 挂钩使用。
*   学习 [**Redux**](https://www.freecodecamp.org/news/understanding-redux-the-worlds-easiest-guide-to-beginning-redux-c695f45546f6/) 更重要的是 Redux 的术语和结构。
*   了解 React 应用程序 中的 [**路由。**](https://reacttraining.com/react-router/)
*   在创建第一个类组件之前，回顾一下 JavaScript 中`[this](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch2.md)` [的整个概念。](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch2.md)
*   了解一下 [**反应过来的表现**](https://reactjs.org/docs/optimizing-performance.html) 又有哪些是敌人。
*   安装并发现 React Devtool。

为了学习这些概念，你可以阅读官方文件或任何其他适合你的资料。在这篇文章的最后，我会列出一些我推荐的资源。

# 第四步:现在你更清楚了

![](img/d64c5b82a26235629e0e3627e77338a6.png)

由[卡斯滕·沃思](https://unsplash.com/@karsten_wuerth?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

这一点不是学习 React.js 的终点，而是起点。将本文的标题与 React 文档进行比较，可以看出到目前为止我们所涉及的内容是多么少，但是从现在开始，您已经了解了足够多的基础知识，可以决定接下来需要学习什么。此外，您可以从这一点开始任何简单的项目，并在开发 React 应用程序的同时继续学习。

我希望我的 React 学习路线图能够让更多的 Vue.js 开发人员将这项新技能添加到他们的工具箱中。

# 我推荐的资源

*   [Brian Holt 的《反应》完整介绍](https://btholt.github.io/complete-intro-to-react-v5/)(免费)
*   [丹·阿布拉莫夫《Redux 入门》](https://egghead.io/courses/getting-started-with-redux)(免费)
*   肯特·C·多兹的博客(免费)
*   [前端大师:React 学习路径](https://frontendmasters.com/learn/react/)(付费)
*   [前端大师:状态管理与 Redux & MobX](https://frontendmasters.com/courses/redux-mobx/) (已付费)
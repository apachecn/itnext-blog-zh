# 一个简单的小 JavaScript 状态库，叫做 Jstates📡🛰️

> 原文：<https://itnext.io/a-simple-small-javascript-state-library-called-jstates-%EF%B8%8F-c4c366a6ea1a?source=collection_archive---------3----------------------->

## TL；速度三角形定位法(dead reckoning)👇

**jstates** :核心状态库 https[://www . npmjs . com/package/jstates](https://www.npmjs.com/package/jstates)**jstates-react**:react js 为**jstates**[https://www.npmjs.com/package/jstates-react](https://www.npmjs.com/package/jstates-react)订阅函数

一个**简单的**(一档☝️)，一个**小的**(小于 800B🙉)、**可扩展的** ♻️，最重要的是👀**可理解，** JavaScript **状态库**以及另外一个 Reactjs❤️订阅函数，用作一个没有上下文复杂性的 HOC(高阶组件)。

## 为什么是另一个州立图书馆？😒

一般来说，react 和 JavaScript 有很多很棒的状态库(例如:redux、mobx、unstated 等等)。那么为什么要创造另一个呢？😏

![](img/7fb28926ec537f2734dca1c40e18f9e9.png)

我想要一个我能想到的最简单、最清晰、最实用的解决方案。 **我想在州立图书馆中拥有一些我在一个图书馆中找不到的功能:**

1.  **极小的**捆绑尺寸(这样我就不用在安装前再三考虑)
2.  **小**代码库(一个文件:index.js)
3.  简单易懂(这样其他开发人员可以很快上手，甚至改进它)
4.  **可扩展**(这样我可以添加我需要的功能，其他人也可以添加)
5.  可以有**多个分离状态**
6.  **我使用的州库的最好的 api 部分**(在我看来，你可以随意复制和创建你自己的，或者创建一个拉请求😉)
7.  其他人想要和需要的他们在那里找不到的东西…

状态可以是简单的，没有理由不应该有很多这样的状态，对于很多用例，对于我们周围的开发人员来说，有不同的(有些人可能会说“奇怪”)😝)口味。

![](img/cc8dbef94d55c0a1de3e8365fa1a5df3.png)

当我开始研究反应堆时😍甚至在使用 react 之前，我就被告知要使用 redux 并立即学习它😓。

Redux(“国王”👑有些人可能会称之为)是一个伟大的图书馆👍但是我很难接受它，在我的职业生涯中，更难向人们解释它😣。

此外，解释**react js**中的状态问题和组件通信**以及为什么我们需要一个额外的状态而不是全局对象**已经足够复杂了**😕。我不认为我们需要**另一个**额外的**概念来在途中学习**(同样，依我看😅).**

所以不要再拖延了📣，我想向您介绍…

![](img/a5ae0b810bc48b24ae5f0a69dd670001.png)

一个简单易用的状态库，可以与任何 js 库或框架一起工作🎉

如果你想用 **Reactjs** 来使用它，它不需要任何<提供者>在你的应用程序的根目录中，因为它与组件上下文是分离的😃

最后几句话:

快乐黑客👷感谢开源人士花时间创造了我所学习的伟大工具🙏
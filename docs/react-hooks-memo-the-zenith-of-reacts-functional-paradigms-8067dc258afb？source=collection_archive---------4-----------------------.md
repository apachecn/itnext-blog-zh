# React 功能范式的顶峰——第一部分:前端的革命

> 原文：<https://itnext.io/react-hooks-memo-the-zenith-of-reacts-functional-paradigms-8067dc258afb?source=collection_archive---------4----------------------->

![](img/3dfbd5f8b0ffa1b0da42caaf6e2768bf.png)

葆拉·梅在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

作为一名使用 React 的工程师和 JavaScript 函数爱好者，这是一段艰难的旅程。React 最大的好处之一，也是它的诅咒。

回过头来看，React 相对 Angular 和[许多许多前端模板语言/库的竞争优势(还记得那些吗？)当时与之竞争的是，React 的渲染可以只使用 JavaScript 函数来定义，不需要复杂的文本。](https://codecondo.com/15-javascript-template-engines/)

这个简单的值，即 JavaScript functions > strings，为 React 与函数式编程社区的长期关系(BFFL)奠定了基础。现在前端工程师可以只使用函数来构建他们的前端应用程序，包括它的渲染！

嗯，差不多了。

# 无状态功能组件的革命

向前迈出的一大步是在 2015 年发布的版本 0.14 中引入了无状态功能组件(在最初发布 2 年后)。这似乎再次强调了反应的方式不同于竞争。

现在可以使用普通的 JavaScript 函数来定义应用程序的 UI。

```
const MyUI = () => <em>Revolutionary</em>
```

普通 JavaScript 函数:web 前端开发的一场革命。

然而，一个人有一种感觉，被放在一个魔鬼的肩膀上。一个告诉你，课堂是我们这里做事的方式；上课是必须的；它们是合适的；它们，嗯，很经典。另一个告诉你脱掉所有的衣服，沐浴在美丽简单的功能中。

对于大规模的 React 应用程序，哪一个是更好的选择还不清楚。毕竟，人们似乎可以毫无遗憾地为每个组件使用 createClass 或适当的 JavaScript 类。一个人可以不编写任何无状态的功能组件，也不知道 React 的内部功能特性！该死的无神论者！

然而，相反，没有类和生命周期的生存是不可能的。(进行了尝试——有人丧生)

直到现在。

# 第 2 部分:React 最糟糕的部分

以函数式编程风格构建大型 React 应用程序的实际挑战。

[阅读第 2 部分](https://medium.com/@adamterlson/the-zenith-of-reacts-functional-paradigms-part-2-the-worst-parts-of-react-6011b85ba6a1?postPublishedType=initial)
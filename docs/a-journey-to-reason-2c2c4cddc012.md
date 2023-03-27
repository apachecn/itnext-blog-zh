# 理性之旅

> 原文：<https://itnext.io/a-journey-to-reason-2c2c4cddc012?source=collection_archive---------0----------------------->

*从反应到推理*

> 注意:这是从反应过渡到理性的“理性之旅”系列的一部分。敬请关注。还会有更多这样的事情发生！
> [下一个](https://medium.com/@RRafatpanah/a-journey-to-reason-c408a87a54de) >

![](img/5c4c0e8ef03e73668dffa06f237421c1.png)

[Pixabay](https://www.pexels.com/u/pixabay/) 供图

就像在《绿野仙踪》中一样，我已经踏上了寻找… **自信**的旅程。不是我自己，而是我的代码。在这个系列文章中，我想分享我作为一个 JavaScript/React 开发人员迁移到 Reason 时遇到的问题的答案。

# 什么是理性？

由最初创建 React 的同一位[人](https://twitter.com/jordwalke)创建的 [Reason](https://reasonml.github.io/) 是针对 [OCaml 语言](https://ocaml.org/)的一种[新语法](https://reasonml.github.io/docs/en/syntax-cheatsheet.html)，意在欢迎 JavaScript 开发者。Reason 让你在利用 JavaScript 和 OCaml 生态系统的同时，编写简单、快速、高质量的类型安全代码。

# 当我们有反应时，为什么我们还需要理由？

反应是理性的垫脚石。React 的创始人 Jordan walker[在 React](https://www.reactiflux.com/transcripts/cheng-lou/) 之前就开始了理性的概念。那个时候，compile-to-JS 的格局还没有现在这么成熟。此外，当 React 的想法第一次[在 SML](https://www.reactiflux.com/transcripts/jordan-walke/) 成型时，JavaScript 社区还没有为函数式范式做好准备。四年后，他们现在相信理性的时机已经成熟。

> [Messenger.com 现在有超过 25%的人转化为 Reason(25%的 UI 组件)。我们正在积极地证明，使用语言本身，使用元语言，一个普通的 JavaScript 开发人员可以用它来提高工作效率。这不是一个学术怪胎或闪亮的玩具。你可以用这种语言来提高工作效率，只要付出一点努力，你就可以在生产中使用这种语言。不只是周六。](https://youtu.be/_0T5OSSzxms?t=31m31s)

截至 2017 年 7 月 27 日，messenger . com 50%的 UI 组件已经转换为 Reason。

![](img/0b94cb85c657581b4233b44557ed4218.png)

由[程露](https://medium.com/u/701b0000d771?source=post_page-----2c2c4cddc012--------------------------------)报道关于[的不和](https://discord.gg/reasonml)

此外，使用 [reason-react](https://github.com/reasonml/reason-react) ，您实际上可以编写编译成惯用 ReactJS 的原因代码。这允许增量采用。这里有一个很棒的 [reason-react 示例](https://github.com/reasonml-community/reason-react-example)可以玩玩，看看编译后的输出看起来有多棒。

# 当我们有测试时，为什么我们需要类型？

尽管我确实认为今天的 JavaScript 社区或多或少理解了静态类型系统的需要，但我记得有一段时间我并不理解。

> 如果你有类型，你有测试，你的测试将能够用更少的能量表达更多。

# 我们已经有了 TypeScript/Flow，为什么还需要 OCaml 的类型系统？

这是一个大的。

Reason/OCaml 的类型系统是*音*； **TypeScript 的** [**不是**](https://www.typescriptlang.org/docs/handbook/type-compatibility.html) 。这意味着如果一个 Reason/OCaml 程序被编译，那么你将保证(在数学意义上)不会看到运行时类型错误(即 *undefined 不是一个函数*)。

> OCaml 提供了一个工业级的最先进的类型系统，并提供了非常强大的类型推理(即，与 TypeScript 相比，不需要冗长的类型注释)，这在管理大型项目中被证明是非常宝贵的。OCaml 的类型系统不仅仅是用于工具的，**它是一个健全的类型系统，这意味着它保证在类型检查之后不会有运行时类型错误。**

据我所知，Flow 的类型系统比 TypeScript 的好得多——但是需要注意的是:

1.  流是用 OCaml 构建的
2.  流量是由脸书支持的

如果 Flow 完全解决了 JavaScript 的问题，那么我相信脸书也不会支持 Reason。

从[流程文件](https://flow.org/en/docs/lang/)中:

> 在 JavaScript(和相关语言)中使用类型来管理代码进化和增长的想法并不新鲜。事实上，近年来已经为 JavaScript 构建了几个有用的类型系统。但是，类型系统的目标不同。在这个范围的一端是许可型系统，它针对可能的错误提供一定程度的林挺，而不考虑正确性。另一方面，限制性类型系统可以保证静态代码优化的正确性，但代价是互操作性。

这就引出了我们的下一个问题。

# Reason 与 JavaScript 的互操作性如何？

太棒了！[原因文档](https://reasonml.github.io/guide/javascript/interop/)可以让你开始，这篇[写得很好的博文](https://jaredforsyth.com/2017/06/03/javascript-interop-with-reason-and-bucklescript/)可以让你走得更远。 [BuckleScript 手册](https://bucklescript.github.io/bucklescript/Manual.html)全面深入地介绍了所有可用的惊人功能。

> ReasonML 解决了我在过去五年中在构建 UI 应用程序时观察到的最大问题，并开放了一个语言/编译器工具链，它非常适合 React 的 UI 渲染模型。
> 
> —乔丹·沃克(ReactJS 的创始人)

# 下一步是什么？

请继续关注我的理性之旅。在下一篇文章中，我将介绍:

*   我把[这个角码](https://github.com/persianturtle/angular-3d-carousel)转换成理由**的经验(更新:这里读**[](https://medium.com/@RRafatpanah/a-journey-to-reason-c408a87a54de)****)****
*   **创建一个[材质 UI](http://www.material-ui.com/#/) 像菜单一样既有原因&原因-反应**(更新:此处阅读**[](https://medium.com/@RRafatpanah/a-reasonml-tutorial-building-an-app-shell-dd7cc617d0c5)****)******
*   ****我为 Reason 应用程序编写自动化测试的经验****

> ****请**加入**关于[不和](https://discord.gg/reasonml)的讨论！****
# 你不知道 TDD

> 原文：<https://itnext.io/you-dont-know-tdd-691efe670094?source=collection_archive---------2----------------------->

## 说真的。我也去过那里。

![](img/7b076df220fe104a17f616e2ad38bf4a.png)

这篇文章的封面图片显示了 5 张图片的合成。第一张，从左到右，是一个留着短胡须的秃顶男人。他微笑着在桌子上数钱。他周围笼罩着一层阴影。第二张照片展示了喷火表演产生的巨大火焰。第三张是酒店卧室的照片。第四幅画展示了一个铁匠用锤子敲打铁砧上的一块铁。最后一张图是一个留着短胡须的秃头男子。他在桌子上用电脑，面带微笑。这是第一张照片中的同一个光头男子，只是现在他周围没有了阴影。

作为一名 [Github 开源](https://hackernoon.com/lets-implement-the-open-source-model-but-which-open-source-a89c82d1b494)贡献者，我总是发现很难分享能够清晰解释某些概念和[基础](https://hackernoon.com/the-doctor-and-the-scalpel-78656f508c9a)的内容。我总是发现自己在 Github 评论和工作场所一遍又一遍地重复着同样的事情。

这是我开始写作的原因之一:创造我可以用来分享和传播知识的开放内容。我可以用一些东西来给别人指出正确的方向。

测试驱动开发是被开发人员高度误解的基础知识之一。大多数贴子都只是在表面上谈论它。很少有深入主题的东西。

我想改变这一切。

在过去的几周里，我写了一系列的文章来展示使用 JavaScript 的 TDD 的第一原则。

以下是它们的摘要(详情请点击标题):

## [缺少实用的分步 TDD](/the-missing-practical-step-by-step-test-driven-development-a7140ca4b71)

这篇文章以放债人杰克的故事开始。杰克雇佣了一名开发人员，并设定了完成工作的最后期限。然而，由于 TDD 的存在，开发人员能够在开发整个规范所需时间的一小部分时间内拥有可工作的软件。

这篇文章还展示了为什么你需要小的测试，而不是一个驱动你实现整个事情的测试。

> 创建一个测试，并编写最简单的代码使其通过。不要创建一个测试，然后编写整个测试。

## [如何用 JavaScript 构建货币数据类型](/how-to-build-a-money-data-type-in-javascript-7b622beabe00)

这篇文章解决了使用 JavaScript `Number`类型来表示美元和美分的问题。如果您没有意识到浮点运算问题，那么您只会发现当 JavaScript `Number`类型在生产中失败时使用它来表示货币的问题。

> 测试你所知道的。你不能测试你不知道的东西。

## [你写的代码应该只比](https://medium.com/@fagnerbrack/the-code-you-write-should-only-suck-less-than-the-one-before-5a20e9a543f)之前的少吸一点

这篇文章描述了我在开发这个 TDD 系列时学到的经验。它还表明，在第一次尝试解决问题时，你不应该期望得到一个好的代码。TDD 促进[持续改进](https://hackernoon.com/how-to-understand-the-balance-between-learning-and-progress-6783e7f711ac)；代码只会随着实践变得更好。

> 没有[银弹](https://medium.com/@fagnerbrack/how-to-reject-the-belief-on-the-silver-bullet-1d86b686acbb)，你写的代码只会随着实践变得更好。

## [这是没人告诉你的关于 TDD 的事情](/this-is-the-one-thing-nobody-told-you-about-tdd-7f549254cd74)

这篇文章展示了 TDD 如何与测试无关；是关于开车的。您一个接一个地编写小测试，每个测试都促使您使代码更加通用。

此外，如果您遵循 TDD 的基本原则，您不需要将测试框架中失败的测试与“红色阶段”或者成功的测试与“绿色阶段”联系起来您可以在不同的层次上实践 TDD，其中来自语法错误的消息代表“红色阶段”，而修复代表“绿色阶段”只要第一条和第二条消息都是可预测的，那么使用哪个工具来查看它们并不重要。

这里重要的一点是，不管你使用什么工具，你只需要在运行代码得到一个[负面反馈](https://en.wikipedia.org/wiki/Negative_feedback)后，再修改代码。

> 您只能更改您可以运行的代码。

## [TDD 如何防止过度工程化](https://medium.com/@fagnerbrack/how-tdd-can-prevent-over-engineering-1265a02f8863)

这篇文章展示了 TDD 如何让你在没有破坏代码风险的情况下重构代码。这是一门[学科](https://hackernoon.com/how-to-fix-the-software-industry-c2b627ec3d9d)，它迫使你只写你需要的代码，只要你已经具备了沿着[正确方向前进的技能](https://hackernoon.com/the-journey-for-the-right-question-c3f5b9e90035)。

> 如果你总是创建一个测试来证明你写的代码，那么你永远不用担心写你不需要的代码。

驱使我写这个系列的是对测试驱动开发在编程工作流中的价值的普遍误解。

如果你正在阅读这篇文章，并且认为不值得费心去看这些帖子，因为你已经知道了 TDD，我仍然建议看一看，并给我一些你的想法的反馈。

我们中的一个可能会学到一些东西。

> TDD 允许你理解问题并为人类构建一个好的产品，而不是为了通过测试而写代码。

为什么是放债人？为什么不用别的东西做例子呢？

为 Jack 编写利率计算程序看起来就像你在帮助一个[高利贷者](https://en.wikipedia.org/wiki/Loan_shark)，这是一个非常不道德的工作。毕竟，你是在帮助弱势群体拿钱。可惜，[现实是大多数人都不在乎](https://news.ycombinator.com/item?id=17692005)。

在编程领域，有些人开始他们的职业生涯是在边缘不道德的项目中工作，以获得更多的经验，因为这是他们唯一能得到的工作。当他们获得更多经验后，他们会转向其他不那么不道德的项目，而原来的公司会取代他们的角色。

我写了一个放债人，这样你就可以读这篇文章，假装在一个不道德的环境中工作。这样，你不需要在现实生活中为一个人工作，仍然可以获得经验。

> 假装在不道德的项目中工作，这样你就不需要和一个会造成真正伤害的项目一起工作

“放债人杰克”的编程问题是在我为一家公司做的一次面试练习中受到启发的。那样的话，我进门的时候，不知道问题的细节，也不知道解决的办法。他们给了我机会进行[结对编程](https://medium.com/@fagnerbrack/pair-programming-8cfbf2dc4d00)，而不是带回家练习，我充当导航员。那个机会让我展示了 TDD 的价值，提出了正确的问题并快速解决了问题。

在采访之后，他们比以前更了解 TDD 的可能性。

> 你可以找到解决办法。你不需要事先知道。

我写这个系列是因为互联网已经失去了 TDD 的[基本面](https://hackernoon.com/the-doctor-and-the-scalpel-78656f508c9a)。互联网上充斥着关于 TDD 的文章；只有那些文章没有解释[第一原则](https://en.wikipedia.org/wiki/First_principle)。

当有人问我时，我发现很难提供一个好的来源来解释 TDD。大部分内容都藏在书里面，这和一个有付费墙的网站没什么区别。

互联网是开放的，自由的，但也是遗忘的。

我希望你不要忘记这一个。

感谢阅读。如果你有一些反馈，请在[推特](https://twitter.com/FagnerBrack)、[脸书](https://www.facebook.com/fagner.brack)或 [Github](http://github.com/FagnerMartinsBrack) 上联系我。

感谢 **Danil Suits** 和 **Jamie Isaacs** 对这篇文章的深刻见解。
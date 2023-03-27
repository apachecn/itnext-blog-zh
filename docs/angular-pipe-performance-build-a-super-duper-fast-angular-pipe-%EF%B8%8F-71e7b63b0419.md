# 弯管性能——建造一个速度惊人的弯管⚡️

> 原文：<https://itnext.io/angular-pipe-performance-build-a-super-duper-fast-angular-pipe-%EF%B8%8F-71e7b63b0419?source=collection_archive---------1----------------------->

## 想让你的棱角分明的管子表现得像一只奔跑的猎豹吗？然后，这里是如何创建一个快速，高性能的角管道。✨✨✨

![](img/c72706009bcbc41068a3e52ea51ac341.png)

角形管道几乎不会引起值得注意的性能问题。

为了确保你知道，更常见的是，通过调整[变化检测选项](https://danielk.tech/home/heres-how-you-can-improve-angular-change-detection-performance)或[确保 ngFor 使用 trackBy 函数](https://danielk.tech/home/use-trackby-to-improve-angular-performance)，Angular 应用程序的性能可以得到显著提高。

但是在任何情况下，如果你想节省几毫秒的时间，尽你所能压缩一切，拉动所有杠杆，让你的角度应用程序运行，那么这就是如何让你的角度管道运行得非常快。💥

# 在我们深入研究之前，让我们澄清一些事情。

如果你的角形管道不是**纯函数**，那么我在这篇文章中要教你的东西都没有价值。这意味着当给定相同的输入时，您的角管道总是返回相同的值。如果你是一个聪明的 Angular 开发者，我确信你是，那么你的 Angular 管道已经使用纯函数了。

> *是的，伙计，我想让你成为一个聪明的有棱角的开发者！*

第二个要求是这样的。Angular 应用程序不应该将模板数据与 getter 函数绑定在一起。这是一个糟糕的选择，很可能有一天会反过来咬你一口。

在我们的方式的基础上，让我们以一个例子角管，使其执行速度更快。

# 我们的示例管道

我们将从 Angular 文档中抓取[自定义管道示例](https://angular.io/guide/pipes#example-transforming-a-value-exponentially)。它是一个管道，可以指数地提升任何给定值。

还不错。但是我们如何让这个东西表现得更好呢？

# 使用 Lodash 使角形管运行更快

创建一个终端并安装 [lodash](https://www.npmjs.com/package/lodash) 库和它的 Typescript buddy。这是您需要的命令。

```
npm install lodash @types/lodash --save
```

然后我们将利用 [memoize 缓存函数](https://lodash.com/docs/4.17.15#memoize)来缓存之前计算的结果。

嘣！💥💥💥这就是我们缓存管道结果的方式。

**同样，在大多数情况下，这对于角度应用来说并不是一个巨大的性能提升。但是如果你的管道执行一个昂贵的计算，那么我强烈推荐这个方法。**

# 结论

这就是，我的朋友，你如何建立一个快速，高性能的角管道。🤓

如果你喜欢这篇文章，并发现它很有用，请一定要点击它👏按钮，关注我，获取更多类似本文的精彩文章。

**关注我:** [GitHub](https://github.com/dkreider) ，[中型](https://dkreider.medium.com/)，[个人博客](https://danielk.tech)

[![](img/f1b6577fb74e9686aa28b76134fcd343.png)](https://school.danielk.tech/course/unleash-your-angular-testing-skills?utm_source=medium&utm_medium=banner&utm_campaign=unleash_testing_skills)![](img/73cacbc97b4c8af6ff46ab6436cfd367.png)

*最初发布于*[*https://danielk . tech*](https://danielk.tech/home/how-to-build-a-fast-angular-pipe)*。*
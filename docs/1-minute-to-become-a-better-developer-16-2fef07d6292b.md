# OMBD#16: JavaScript 开发者:你无法预测这 4 行代码的结果

> 原文：<https://itnext.io/1-minute-to-become-a-better-developer-16-2fef07d6292b?source=collection_archive---------6----------------------->

## 或者静态代码分析工具如何让您的生活变得更好，如何防止头痛

欢迎来到第 16 期的**O**ne**M**inute**B**etter**D**eveloper，在这里你可以通过阅读简短的知识，每次一分钟，成为一名更成功的软件开发人员。

## [⏮️](https://jportella93.medium.com/1-minute-to-become-a-better-developer-15-d10746c1700) [🔛](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)⏭️

![](img/8e8e59fea9f40ea9bbca44848ff652ef.png)

我的好友洛尔·尼古拉斯的作品

## 问题是

花点时间看看这个代码片段，试着弄清楚它是做什么的。

先不要滚动到解决方案！

真的去揣摩结果，并不像看起来那么简单…

第 4 行将记录什么？

.

.

.

.

不要再滚动了，直到你知道答案！

.

.

.

.

.

.

好了，准备好了吗？

.

.

.

.

现在，我打赌你认为结果是:

```
'Swapped: 2,1'
```

**错了。**

这段代码抛出一个错误。您可以在浏览器的控制台上亲自尝试:

```
Uncaught TypeError: Cannot set property '2' of undefined
```

但是，为什么呢？

这个错误发生在我的一个学生身上，我困惑了一段时间，直到我发现了罪魁祸首。**代码缺少分号**。通常，这不是问题，因为 [JavaScript 的自动分号插入](https://262.ecma-international.org/7.0/#sec-automatic-semicolon-insertion)很好地解决了这个问题。

但是，在这种情况下，代码运行如下:

示例程序是如何实际运行的。

怎样才能避免这种头痛？

## 一个解决方案

使用[静态代码分析工具](https://blog.logrocket.com/static-analysis-in-javascript-11-tools-to-help-you-catch-errors-before-users-do/)

哪怕是我正在做的一个小脚本，我总是会安装，至少，`eslint`。因为:

1.  它会指出这类问题和其他问题。
2.  它将解决这些问题，并在保存时自动格式化您的代码，因为这是您的 IDE 上的配置应该做的。最终，那个任务应该是机器人的工作，而不是人类！

因此，即使花几分钟时间来设置静态代码分析工具，它总是一个大胜利。此外，你设置它们的次数越多，下一次你就能越快完成。

## 如果你喜欢这个故事，你可能也会喜欢:

[](https://jportella93.medium.com/1-minute-to-become-a-better-developer-15-d10746c1700) [## 1 分钟成为更好的开发人员(#15)

### 了解如何在一分钟内删除所有 node_modules 目录。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-15-d10746c1700) [](https://jportella93.medium.com/1-minute-to-become-a-better-developer-17-75fd019f407c) [## 1 分钟成为更好的开发人员(#17)

### 一分钟内了解正向和反向代理。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-17-75fd019f407c) 

## (T0)↓↓️ [↓↓(T3) (T4)↓↓年(T5)](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)
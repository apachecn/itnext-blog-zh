# OMBD#14:当用户停止输入时如何调用函数

> 原文：<https://itnext.io/ombd-14-how-to-call-a-function-when-a-user-stops-typing-bcc55fb410a0?source=collection_archive---------5----------------------->

## 我们将学习如何用 JavaScript 实现一个浏览器自动保存草稿系统

欢迎来到第 14 期(共 14 期)One**M**inute**B**etter**D**developer，通过阅读简短的知识，每次一分钟，你将成为一名更成功的软件开发人员。

## [⏮️](https://jportella93.medium.com/1-minute-to-become-a-better-developer-13-6929b0eec824) [🔛](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e) [⏭️](https://jportella93.medium.com/1-minute-to-become-a-better-developer-15-d10746c1700)

![](img/aba486ce1395b680b0f58c19d381e2e0.png)

我的好友洛尔·尼古拉斯的作品

## 问题是

我们正在用一个类似 Google-Docs 的草稿系统实现一个文本编辑器，当用户停止输入一秒钟时，内容就会保存在服务器端。

## 一个解决方案

我们将使用`keyup`事件和`window.setTimeout`。

这个代码片段的关键是必须在`keyup`回调的范围内声明`timerId`，以便在每次再次触发`keyup`回调时将它重新分配给新的超时 id。

这是 CodePen 上的一个演示:

当用户停止输入时调用函数。

## 如果你喜欢这个故事，你可能也会喜欢:

[](https://jportella93.medium.com/1-minute-to-become-a-better-developer-12-3eb093c1ef21) [## 1 分钟成为更好的开发人员(#12)

### 了解如何利用窗口。IntersectionObserver 了解一分钟内有多少访问者查看了您的内容。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-12-3eb093c1ef21) [](https://jportella93.medium.com/1-minute-to-become-a-better-developer-18-5edc55190b91) [## 1 分钟成为更好的开发人员(#18)

### 一分钟内学会如何避免连字符换行。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-18-5edc55190b91) 

## [⏮️](https://jportella93.medium.com/1-minute-to-become-a-better-developer-13-6929b0eec824) [🔛](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)⏭️
# OMBD#4:带有异步立即调用函数表达式的快速异步一行程序

> 原文：<https://itnext.io/1-minute-to-become-a-better-developer-4-aeabd6586396?source=collection_archive---------3----------------------->

## 求职面试或节省快速脚本时间的巧妙技巧

欢迎来到第 4 期，通过阅读简短的知识，每次一分钟，你将成为一名更成功的软件开发人员。

## [**⏮**](https://jportella93.medium.com/1-minute-to-become-a-better-developer-3-1d73b6ffd514) **️** [**🔛**](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)**[⏭️](https://jportella93.medium.com/1-minute-to-become-a-better-developer-5-a7645ddb4637)**

**![](img/cfa3048dda98b79c64d2d15ee8d55e6a.png)**

**我的好友[洛尔·尼古拉斯](https://www.instagram.com/loornicolas/)的插图**

## **问题是**

**我们必须创建一个脚本，向 Chuck Norris 笑话 API 发出 GET 请求，并将内容添加到 id 为`#chucky`的 div 中。**

**一种快速简单的方法是将`fetch`和`.json`返回的承诺链接到`.then`方法上:**

**让我们把它重构为更易读的`async/await`。现在，问题是`await`除非在`async`函数中使用，否则无法工作:**

**所以常见的模式是将异步代码包装在一个`async`函数中，然后立即调用它。但是我们通过添加这个新的`go`函数污染了名称空间，我们立即调用这个函数:**

## **一个解决方案**

**这里有一个我过去在求职面试中用过的小技巧。**

**我们可以在一个步骤中完成完全相同的操作，并且不会因为异步[立即调用函数表达式](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)而污染名称空间:**

## **如果您喜欢这篇文章，您可能也会喜欢:**

**[](https://jportella93.medium.com/1-minute-to-become-a-better-developer-3-1d73b6ffd514) [## 1 分钟成为更好的开发人员(#3)

### 欢迎阅读本系列的第 3 期，通过阅读简短的知识，您将成为一名更成功的开发人员…

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-3-1d73b6ffd514) [](https://jportella93.medium.com/1-minute-to-become-a-better-developer-5-a7645ddb4637) [## 1 分钟成为更好的开发人员(#5)

### 欢迎阅读本系列的第 5 期，通过阅读简短的知识，你将成为一名更成功的开发人员…

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-5-a7645ddb4637) 

## (T0)↓↓️ [↓↓(T3) (T4)↓↓年(T5)](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)**
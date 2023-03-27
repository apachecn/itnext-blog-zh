# 如何在结构中使用 Golang 泛型

> 原文：<https://itnext.io/how-to-use-golang-generics-with-structs-8cabc9353d75?source=collection_archive---------0----------------------->

## 了解如何创建泛型结构来充分利用此功能

![](img/5cdac895bc67c5d0d390487f9a524405.png)

从 Go 1.18 开始，我们终于有了泛型的力量。本周，当我浏览 Golang 源代码时，我发现了一个如何使用泛型创建结构的例子。

在这篇文章中，我将向你展示如何做到这一点。

在我们开始之前，让我们假设我们正在用下面列出的两个结构实现一个博客系统。

为了提高性能，我们将实现一个缓存系统，保证没有人能够直接在缓存中修改任何数据。

记住这些规范，让我们创建一个名为 cache 的新包。

这个包需要一个私有结构来保存我们想要缓存的所有数据。但是，在我们创建这个结构之前，让我们为可缓存的类型创建一个接口。

很好！现在我们可以创建自己的私有和泛型结构来保存数据。

为了操作数据，我们要实现两个方法，Set 和 Get。

现在，让我们添加一个通用函数来创建并返回一个指向新缓存的指针。

不错！！！

要使用我们的包，我们需要做的就是为我们想要缓存的类型创建一个新的缓存，并使用如下示例所示的结构的方法。

请在评论中留下你的问题，我会尽快回答。

感谢您的阅读！
# C++17 结构化绑定，提供更安全、功能更强的代码

> 原文：<https://itnext.io/c-17-structured-bindings-for-more-safe-functional-code-c0c5b4d31b0d?source=collection_archive---------1----------------------->

![](img/b80d64b5bdb0c0f0b27fe78849092239.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [C Dustin](https://unsplash.com/@dianamia?utm_source=medium&utm_medium=referral) 拍摄的照片

本文的总体思想是展示如何在 C++17 中尽可能保持作用域(即块)的整洁。我所说的“干净”范围指的是两件事:

1.  尽可能少的变量，
2.  所有变量尽可能保持不变。

注意，在这个例子中，main()主体只包含常量变量。当您想要在一个作用域中布局非常长的算法时，这是一个很好的实践，并且希望避免由于无意中修改变量而导致的任何错误。

让我们从我们需要的内容开始。

我们需要帮助来打印一个向量的内容。

是时候声明 main 函数了，我们将使它的主体范围非常安全。作为一个示例算法，让我们选择计算一个数组的中值以及参与中值计算的值的索引。在 C++17 之前，C++中没有很好地管理多重返回值。我建议读者按照下面代码中的注释来弄清楚发生了什么。

谢谢你陪我学习如何用结构化绑定让你的 C++代码更有功能。请随意评论您对 C++代码开发的现代技术的想法。你可以在下面的链接中找到完整的代码。

[](https://github.com/Obs01ete/structured_bindings) [## OBS 01 ete/结构化 _ 绑定

### C++17 结构化绑定教程。通过创建帐户，为 Obs01ete/structured_bindings 开发做出贡献…

github.com](https://github.com/Obs01ete/structured_bindings)
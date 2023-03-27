# 破解 Python 语法:简介

> 原文：<https://itnext.io/hacking-the-python-syntax-part-0-introduction-9a1c054ec1e6?source=collection_archive---------7----------------------->

![](img/8f0c321081b43fe881fea5a278a1e2d8.png)

图片来自 [Pixabay](https://pixabay.com/illustrations/cubes-wood-tube-squares-2803223/)

## 设置您的环境

**简介** | [三元运算符](https://medium.com/@remykarem/bbcb04aa6ecb) | [交替 lambda 语法](https://remykarem.medium.com/hacking-the-python-syntax-alternate-lambda-syntax-c87c383dd1a3) |函数中无返回关键字(即将推出)|列表理解++(即将推出)

*变更日志:
2022 年 2 月 4 日—更新免责声明*

你是否曾经希望你的酷语法想法存在于你最喜欢的编程语言中？

如果人们真的😍这对你有好处。除了你要花大约一年甚至更长的时间来发布这个特性。

但是如果没有其他人发现它特别有用，那么这个想法就到此为止了。😞

不要只是等待，希望它被发布或者抛弃那个想法，让我们创造我们自己的现实🌏—通过修改源代码！

在这个*破解 Python 语法*系列中，我们将研究 4 种语法思想，并在 CPython 源代码中实现它们。我们将探讨 4 个想法:

1.  三元运算符`**score ≥ 50 ? "good" : "bad"**`，
2.  备选 lambda 语法`**|x| -> x+1**`，
3.  函数`**def add(a,b): a+b**`中没有返回关键字，并且
4.  列表理解++ `**[ch ~ ch<-word ~ word<-["hello","world"]]**`。

## **目标**

本系列的目的是分享我的这些想法的原型之旅，以便您也可以修补解析器，可能将它用于您自己的 [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) ，为代码库做出贡献，或者有希望被启发学习其他语言。

## **范围**

*   我们所指的 Python 是 CPython 实现，它是 Python [ [1](#5d85) ]的原始实现。
*   我们将处理解析器和一点 AST。语法变化不需要我们处理 CPython 的内存管理。
*   我们要改变的文件是记号和语法文件。

## **先决条件**

你只需要知道一些基本的 Python。以下是其他一些不错但可选的东西:

*   源代码编译(解析、词法分析、AST 等。),
*   regex(用于在代码库中搜索文本)，
*   C 或 C++，
*   Makefile(用于构建 Python 解释器)，以及
*   熟悉至少一种其他编程语言。

## **免责声明**

*   如果您还是初学者，本系列可能不适合您，因为语法原型可能会让您感到困惑。
*   本系列并不意味着成为改变 Python 语法或理解源代码编译的权威指南。
*   开发一种经过充分测试的语法不符合本系列的利益。
*   **不要在生产中使用这些分叉的 PYTHON 发行版。**

## **样式**

需要学习的东西很多，所以本文的风格是及时学习概念，这样我们就可以很快熟悉代码库。概念是带有以下表情符号的迷你部分:

*   *📜* 源代码编译
*   💡其他供参考的信息包括理解代码库是如何工作的，以及在旅途中学习一些 C 语言

# 安装

为了坚持到底，我推荐使用 [VS Code](https://code.visualstudio.com) 作为你的 IDE。有用的扩展包括一个 C 语法高亮器和[PEG 语法高亮](https://marketplace.visualstudio.com/items?itemName=lysnikolaou.pegen-peg)。

有用的快捷键是 Cmd+P 用于文件搜索，Cmd+Shift+F 用于全局文本搜索。和 Cmd+F 在当前文件视图中进行文本搜索。(Linux 和 Windows 用户，用 Ctrl 代替 Cmd。)

这个系列会用一个 [Python v3.11.0a2](https://github.com/python/cpython/tree/v3.11.0a2) 的叉子。

按照以下步骤从源代码构建 Python 并运行 Python 可执行文件:

1.  `git clone [https://github.com/python/cpython.git](https://github.com/python/cpython.git)`
2.  `cd cpython && git checkout v3.11.0a2`
3.  `./configure`
4.  `make -j4`
5.  `./python.exe` ( `./python`适用于非 macOS 用户)

接下来:[ [第一部分—三元运算符](https://medium.com/@remykarem/bbcb04aa6ecb)
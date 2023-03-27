# Smalltalk 并不是一门完美的语言。它的缺点是什么？

> 原文：<https://itnext.io/smalltalk-is-not-a-perfect-language-what-are-its-shortcomings-72ac93b1ef0d?source=collection_archive---------1----------------------->

![](img/ea9923722e4c646663c61c241725d8a8.png)

首先，Smalltalk 是动态类型化的。认为静态类型是更好的编码方式的人会认为这是一个缺点。我不知道。

其次，Smalltalk 是通过垃圾收集进行内存管理的。对于一些执行效率至关重要的应用程序，垃圾收集可能会带来不可接受的延迟。

第三，Smalltalk 是基于 VM 的，执行字节码。尽管 Smalltalk 通常是 JIT 编译的，但对于某些应用程序来说，它可能还是不够快。

第四，Smalltalk 的语法极其简单易学。然而，它的简单性无法与 Haskell、Rust 和 Scala 等更复杂的语言语法相媲美。所以缺乏“表现力”经常被认为是一个缺点。

尽管如此，Smalltalk 已经被证明是世界上最简洁、最优雅的编程语言之一。如果表现力是一个问题，它从来没有明显。

第五，Smalltalk 缺乏对抢占式多线程的支持。多线程是协作式的或非抢占式的。

第六，Smalltalk 缺乏其他语言生态系统(如 Java 和 Python)中丰富多样的库。

尽管如此，这并没有阻止 Smalltalk 成为一种不可思议的通用语言。我想不出 Smalltalk 在哪个领域没有被证明是有用的。例如:

1.  后端 web:使用[海边](http://seaside.st/)或[茶壶](https://github.com/zeroflag/Teapot)
2.  前端腹板:使用[琥珀色](https://amber-lang.net/)或[褐色](https://pharojs.github.io/)
3.  手机:使用科尔多瓦/PhoneGap
4.  桌面:使用规格
5.  数据科学:使用[博学者](https://github.com/PolyMathOrg/PolyMath)、 [Roassal](http://agilevisualization.com/)
6.  机器学习:使用[张量流](https://github.com/PolyMathOrg/libtensorflow-pharo-bindings)
7.  物联网:使用[法洛斯](https://github.com/pharo-iot/PharoThings)
8.  机器人:例如， [PharROS](http://car.imt-lille-douai.fr/category/software/pharos/)
9.  虚拟现实
10.  操作系统:只有操作系统内核需要用 C 语言编写(例如 PharoNOS)

第七，Smalltalk 代表了一种新的编程模型，其中*映像*是应用程序(包括 Smalltalk 系统)的总执行状态的快照。这对于软件部署非常有用。

然而，有些人错误地认为这是一个缺点，因为他们认为这个图像会阻止你使用你最喜欢的编程工具，比如 Emacs，GitHub，diff，grep 等等。那实际上不是真的。

GNU Smalltalk 是一个命令行的 Smalltalk。你可以使用任何你喜欢的文本编辑器。您可以使用任何命令行实用程序。

甚至 Pharo 也有一个命令行模式，允许你做同样的事情。然而，问题是您将放弃 Smalltalk 最大的优势之一:强大的实时编码环境，这使得 Smalltalk 成为世界上最高效的编程语言之一。

此外，Pharo 有一个 git 集成工具，可以让你使用 GitHub。

但事实是 Smalltalk 是一种新的编程模型。那些不能接受它，或适应它，或放下过去的人，会把它视为缺点。

尽管有这些缺点，我还是喜欢 Smalltalk。它漂亮、优雅、简洁、高效、可伸缩、可维护，而且有趣。它非常适合企业商务计算。对爱好者来说很棒。这对于学者和教育工作者来说非常好。

我从未见过完美的编程语言，但没有一种语言能比 Smalltalk 更接近。
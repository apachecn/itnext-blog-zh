# 使用验证器库通过 Go 清除防御性编程

> 原文：<https://itnext.io/clear-defensive-programming-with-go-using-verifier-library-6f648810b453?source=collection_archive---------3----------------------->

一些软件对可用性、安全性或保密性的要求比通常的更高。在这样的项目中，人们经常实践务实的偏执狂和被称为[防御性编程](https://en.wikipedia.org/wiki/Defensive_programming)的编码风格。

这种方法背后的思想是，即使在意外的情况下，您的代码也应该以一致和可预测的方式运行。在实践中，你将防御不可能的事情，因为当外部世界发生变化时，事情变得可能:新人加入团队，代码进入维护，依赖性升级……实际上，当涉及到错误时，人类使任何事情都成为可能。

吉姆·伯德[已经收集了](https://dzone.com/articles/defensive-programming-just)一套关于如何在你的代码库中使用防御性编程的规则:

1.  保护你的代码免受来自外*的无效数据，无论你决定*外*在哪里。外部系统、文件或来自模块/组件外部的任何调用。建立“信任边界”——边界外的一切都是危险的，边界内的一切都是安全的。在路障代码中，验证所有输入数据。*
2.  检查坏数据后，决定如何处理它。防御性编程不是吞下错误或隐藏错误。选择处理坏数据的策略:返回一个错误并马上停止(fast fail)，返回一个中性值，代入数据值……确保策略清晰一致。
3.  不要假设代码之外的函数调用或方法调用会像宣传的那样工作。确保您理解并测试了外部 API 和库的错误处理。
4.  使用断言来记录假设并强调“不可能”的情况。这在长期由不同的人维护的大型系统中，或者在高可靠性代码中尤其重要。
5.  智能地添加诊断代码、日志记录和跟踪，以帮助解释运行时发生了什么，尤其是当您遇到问题时。
6.  标准化错误处理。决定如何处理“正常错误”或“预期错误”和警告，并始终如一地完成所有这些工作。

这样的规则确实有效，可以帮助人们制作可靠的软件，但是当你使用这些方法时，你的代码会很快变得一团糟。有时候很难区分验证码和底层业务逻辑。查看在执行转移之前执行的一系列验证:

我想我们都同意，这些线是重复的，甚至在某种程度上容易出错。错误处理容易出错😏。但是我们可以使用[验证器](https://github.com/storozhukBM/verifier)库使它变得更好:

[验证器](https://github.com/storozhukBM/verifier)建立在 Rob Pike 在 Go 博客[中描述的错误处理模式之上，错误是值](https://blog.golang.org/errors-are-values)。这个图书馆是透明的，非个人化的。它不是来为你处理错误的。它不会强迫你采用任何错误处理风格，你将处于控制之中，并决定什么对你的项目更好。

如果你想了解更多，请访问 [Github](https://github.com/storozhukBM/verifier) 页面。
你也可以在这里探讨更多关于[防守](http://wiki.c2.com/?DefensiveProgramming)和[进攻](http://wiki.c2.com/?OffensiveProgramming)编程[的想法。](http://wiki.c2.com/?OffensiveDefensiveProgramming)

![](img/9a296f15cdb96659038418f18cc42811.png)

不可能的魔方由[**NotSaccade**](https://www.reddit.com/user/NotSaccade)**from[Reddit](https://www.reddit.com/r/blender/comments/7kuzve/impossible_cube/)**
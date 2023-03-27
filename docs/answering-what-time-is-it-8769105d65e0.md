# 回答“现在几点了？”

> 原文：<https://itnext.io/answering-what-time-is-it-8769105d65e0?source=collection_archive---------0----------------------->

时间是人类长久以来试图获取的核心信息之一。在不同的年龄，出于不同的原因，记录时间是很重要的。随着软件为人类解决现实世界的问题，时间的概念&测量时间的时钟也进入了计算机。这是我们在日常生活中以及在开发软件时认为理所当然的信息。作为一个好奇的人，我想了解计算机是如何记录时间的。

![](img/adec72de2db6a393af1506fed6695f55.png)

日晷照片由[提莫·丁格尔](https://unsplash.com/@tcdinger?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/sundial?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

尽管我们在 Go 中开始了这一旅程，但我们将需要穿透不同层次的抽象，如库、操作系统层和硬件。随着我们在堆栈中的位置越来越低，事情将变得更加异构，因为不同的操作系统和硬件以不同的形式公开这些功能。像 Go 这样的高级编程语言在隐藏这些方面做得很好，我认为这是它们存在的部分原因。由于您阅读这篇文章的时间有限，而我准备这篇文章的时间也有限，所以我们需要从中挑选一篇，并通过抽象层的一个特定分支。为此，我将尝试留下指针，这样如果您对其他分支感兴趣，您就可以跟踪它们。

让我们从一个非常简单的东西开始:一个打印当前时间的程序。

运行此输出:`2022–06–20T22:14:04.907366Z`。

这看起来很简单！

`time.Now`返回一个`[time.Time](https://pkg.go.dev/time#Time)`对象，代表它被调用时的时间。

> 时间以纳秒的精度表示时间中的瞬间。

请花一点时间想想一纳秒有多小。每秒钟有十亿纳秒。

在`time`包文档中还有另一条信息:

> 操作系统提供了“挂钟”和“单调时钟”，前者会随着时钟同步而变化，后者则不会。总的规则是，挂钟是用来报时的，单调钟是用来计时的。而不是拆分 API，在这个包中时间按时间返回。现在包含挂钟读数和单调时钟读数；后来的计时操作使用挂钟读数，但后来的时间测量操作，特别是比较和减法，使用单调的时钟读数。

在这一点上，既然我们对“告诉时间”更感兴趣，“挂钟”似乎是与我们相关的部分。然而，这个解释提供了一些有用的背景知识。

我们接下来会研究`time.Now`。

`time.Now`被定义为 Go 标准库的一部分，在这里[被定义为](https://github.com/golang/go/blob/530511bacccdea0bb8a0fec644887c2613535c50/src/time/time.go#L1089-L1097)。

见原文来源[此处](https://github.com/golang/go/blob/530511bacccdea0bb8a0fec644887c2613535c50/src/time/time.go#L1088-L1097)

请记住文档中的上述部分，其中一部分与“报时”——挂钟有关，另一部分用于“测量时间”——单调时钟。对我来说，这给我的感觉是，`now`是与读取挂钟值相关的部分，其余部分与单调时钟以及`Time`数据结构如何在内部存储这两个概念相关。通读数据结构中的注释还可以验证:

见原文来源[此处](https://github.com/golang/go/blob/530511bacccdea0bb8a0fec644887c2613535c50/src/time/time.go#L129-L150)

这就是为什么，接下来我们将研究从`time.Now`调用的`now`。这就是我们在`time.go`中看到的:

见原文来源[此处](https://github.com/golang/go/blob/530511bacccdea0bb8a0fec644887c2613535c50/src/time/time.go#L1072-L1073)

我们需要更深入地研究 go 标准库。

在`runtime`包中，这个函数有多种定义。有些(比如[这个](https://github.com/golang/go/blob/f229e7031a6efb2f23241b5da000c3b3203081d6/src/runtime/timestub.go))是用 Go 写的(虽然在别的地方也可能指汇编写的零件)有些(比如[这个](https://github.com/golang/go/blob/f229e7031a6efb2f23241b5da000c3b3203081d6/src/runtime/time_linux_amd64.s))是用汇编写的。这是一种在 Go 源代码中广泛使用的模式。我想这与语义相关的片段放在一起有关。

**寻找合适的实施方案**

为了避免跑题，我不会在本文中涉及太多细节，但是对于不同的构建目标有不同的`now`实现。

首先要知道的是，使用我们在编译期间针对的操作系统和架构的文件有一个命名约定:

> 如果去掉扩展名和可能的 _test 后缀后，文件名与以下任何模式匹配:
> 
> * _ GOOS
> * _ go arch
> * _ GOOS _ go arch
> 
> (示例:source_windows_amd64.go)其中 GOOS 和 GOARCH 分别表示任何已知的操作系统和体系结构值，则该文件被视为具有需要这些术语的隐式构建约束(除了文件中的任何显式约束之外)。

与这些匹配的文件被认为满足隐式约束。此外，还有明确的约束。

如果您查看上面链接的那些实现，它们都有一行包含类似于`//go:build !faketime && !windows && !(linux && amd64)`的内容。如果满足这些约束，Go 编译器将使用有问题的源代码进行编译。例如，如果我们的目标是 Windows，这个约束将不会被满足，而对于`darwin`(或苹果设备)，它将被满足。你可以在 Go [这里](https://pkg.go.dev/cmd/go#hdr-Build_constraints)找到更多关于构建约束的信息。

找到匹配后，Go 编译器在这里使用的另一个构造是`go:linkname`指令。它告诉编译器使用一个特定的局部符号，该符号在其他地方被另一个名字引用。你也可以在这里找到更多关于它的[。](https://pkg.go.dev/cmd/compile#hdr-Compiler_Directives)

在这一点上，由于我们将根据构建目标跟踪不同的源代码，我们将需要假设一个。我决定跟随 Linux arm64 架构的实现，这将我们带到了`time`中提到的[一个](https://github.com/golang/go/blob/f229e7031a6efb2f23241b5da000c3b3203081d6/src/runtime/time_linux_amd64.s)T3 的汇编实现。

`**time.now**` **针对 Linux arm64**

首先是一个简短的免责声明:我必须提到我对汇编语言不是很熟悉。我研究过它，除了出于好奇四处打探之外，仅此而已。我可以阅读大多数基本代码，但我不能说我有太多的经验。如果你注意到任何不准确的地方，通过评论让我知道，我会纠正它。

我将参考[围棋汇编指南](https://go.dev/doc/asm)来导航这个。

纵观[的实现](https://github.com/golang/go/blob/f229e7031a6efb2f23241b5da000c3b3203081d6/src/runtime/time_linux_amd64.s)，我看到 3 个主要部分:

*   为调用其他函数做好准备，比如`runtime·vdsoClockgettimeSym`，我们将在后面讨论
*   尝试调用`runtime·vdsoClockgettimeSym`读取实时&单调时钟
*   如果 [vDSO](https://man7.org/linux/man-pages/man7/vdso.7.html) 调用没有返回任何有用的东西——意味着它们不可用，使用 syscall `clock_gettime`作为后备

**什么是 vDSO？**

大多数操作系统的运行方式分为两种操作模式:内核(或特权)模式和用户模式。这也是为了限制对关键资源的访问，并在进程之间建立隔离。某些事情只能通过询问操作系统内核来完成。这些被称为系统调用或系统调用。当一个用户空间进程调用一个系统调用时，它实际上将控制权交给内核，并等待直到结果准备好。这会导致上下文在用户模式和内核模式之间来回切换，这会影响性能。

这正是虚拟动态共享对象的用武之地。对于诸如挂钟之类的资源，由于该信息不是秘密并且旨在被共享，所以有时内核在用户空间中维护结果的表示，从而不需要在更快的操作模式之间切换。

这就是我们在上面的`time.now`实现中看到的。它试图通过 vDSO 检索时间信息，如果这不可用，它将使用常规的 syscall。

这些 vDSOs 的存储位置取决于系统和架构。然而，简而言之，该进程知道用户存储器空间中的地址，该地址对应于列出不同对象的查找表。然后，用户进程可以使用查找表并读取信息，而无需任何上下文切换。

vDSO 手册页对此进行了如下解释:

> **寻找 vDSO** vDSO 的基址(如果存在的话)由
> 内核通过 AT_SYSINFO_EHDR 标签传递给初始辅助向量中的每个程序(参见
> getauxval(3))。
> 
> 您不能假设 vDSO 映射在用户内存映射中的任何特定位置
> 。每次创建一个新的进程映像
> (在 execve(2)时间)，基址通常会在运行时被
> 随机化。这样做是出于安全原因，为了
> 防止“返回 libc”攻击。

在 Go 代码中，我们可以看到[一个用于 Linux 的初始化代码](https://github.com/golang/go/blob/8542bd8938efc8bb1bc681f4a0603c9f392e70b0/src/runtime/vdso_linux.go#L268)收集可用的 vDSO 条目，它是平台相关的(参见 [this](https://github.com/golang/go/blob/2ebe77a2fda1ee9ff6fd9a3e08933ad1ebaea039/src/runtime/vdso_linux_arm64.go#L16) for arm64)。该代码在 Go 程序启动时使用，并保存 vDSO 条目的位置，以备以后需要。

**回落到**

为了明确起见，正如我们前面所讨论的，时间信息在用户空间中并不总是可用的。在这种情况下，系统调用`[clock_gettime](https://linux.die.net/man/3/clock_gettime)`读取不同类型的时钟。

Brendan Gregg 有一篇有趣的文章，关于使用 vDSOs 与使用 syscalls 相比有多大的不同。你可以在这里找到。

至此，我们已经到达了内核。在 Linux 源代码中，我们应该走两条路:

1.  了解`clock_gettime`是如何实现的
2.  如何建立和维护 vDSOs

当然，一台单独的计算机本身并不知道&也不记录时间。还有另一组进程来同步计算机之间的时间，称为 [NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol) 。

另一个要研究的层面也许是这如何与我们的软件运行的硬件一起运作。

我计划在以后的文章中讨论这些主要部分，并把它总结一下。关注或订阅标签。
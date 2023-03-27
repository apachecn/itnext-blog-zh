# 作为软件开发人员，为什么要关心单调的时钟呢？

> 原文：<https://itnext.io/as-a-software-developer-why-should-you-care-about-the-monotonic-clock-7d9c8533595c?source=collection_archive---------1----------------------->

![](img/f693d6a1fe780eb199d303396511a832.png)

来源: [pixabay](https://pixabay.com/)

先举个具体的例子。您是一名 Java 开发人员，想要测量一次执行的持续时间(时间间隔):

如果你没有注意到这段代码有什么问题，这篇文章值得一读。

一台计算机有两个不同的时钟。首先是**挂钟**，我们都知道它是用来获取一天的当前时间的。

该时钟可能会出现**变化**。比如和 NTP(网络时间协议)同步的话。在这种情况下，在同步之后，我们服务器的本地时钟可以在时间上向后或向前跳**。所以从挂钟上测量持续时间可能会有偏差。**

**第二个时钟被称为**单调时钟**。在这里，我们保证时间总是向前移动，不会受到导致时间跳跃的变化的影响。**

**唯一的变化是潜在的频率调整。基本上，如果我们的服务器检测到它的本地 quartz 比 NTP 服务器移动得快或慢，它可以调整它的时钟频率。但同样，单调时钟没有时间跳跃。**

**因此，如果我们必须测量持续时间，我们必须使用单调时钟。这个经验法则只对**本地持续时间测量**有效。事实上，两个不同服务器的单调时钟根据定义是不同步的。因此，基于这些时钟来测量分布式执行是不准确的。**

**让我们回到之前的例子。问题是因为在 Java 中，`System.currentTimeMillis()`是基于挂钟的，所以会有变化，甚至是负的持续时间。相比之下，`System.nanoTime()`是单调的，所以我们应该使用这个。**

**比如像 Go 这样的另一种语言呢？获取持续时间的标准方法基于以下代码:**

**`time.Sub(time.Duration)`实际上是基于根据[文档](https://golang.org/pkg/time/#hdr-Monotonic_Clocks)的单调时钟。**

**作为结论，您应该检查您的编程语言如何处理这两个不同的时钟，以确保您的持续时间实际上是准确的。**

**![](img/1be1009eeb21be8dacfbf3afe8e56b10.png)**

# **进一步阅读**

**Reddit 对其他语言的讨论**

 **[## clock_gettime - Linux 手册页

### 函数 clock_getres()查找指定时钟 clk_id 的分辨率(精度),如果 res 不为空…

linux.die.net](https://linux.die.net/man/2/clock_gettime)** **![](img/b6578984dc018c44134738b9e16e76e8.png)****![](img/ec9b9710329661db6caf375f69308521.png)**

**在推特上关注我 [@teivah](https://twitter.com/teivah)**
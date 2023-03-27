# 写简单的蒙特卡罗模拟

> 原文：<https://itnext.io/write-simple-monte-carlo-simulation-5ceb4c9b0eb3?source=collection_archive---------3----------------------->

## 使用 Go 作为编程语言

![](img/dbb61c4604df7225b5c12a04cfdcce2e.png)

[M. B. M.](https://unsplash.com/@m_b_m?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

如果你正在交易外汇(或交易任何东西)，很可能你听说过一个叫做*蒙特卡洛模拟*的东西。

简而言之，蒙特卡洛模拟是一个程序，它给你一些伪随机输出(在这种情况下是点数),给定的输入(赢/输的交易数，点数……

但是…你知道…举个例子会更好。

# 例子

假设你做了 100 笔交易。你做得很好，你对自己绝对诚实。

你肯定知道，如果你在现实中交易了这 100 笔交易，结果会和你的纸上交易完全一样。

**假设这是你的纸上交易的结果:**

*   交易数量:100
*   佣金和差价的总点数:100 * 1.1 = 110 点
*   成功交易的数量:60
*   成功交易的平均点数:28.3
*   亏损交易数量:40
*   亏损交易的平均点数:26.5

# 计算

现在你可以做一个简单的计算，看看你的系统是否盈利。

你全部加起来:(60 * 28.3)——(40 * 26.5)——110 =**528 个点。**

最终结果是 528 点，包括佣金和差价。你知道，你的系统是有利可图的。如果你有毅力，不断激发它。

也许你现在会问:*蒙特卡洛和*有什么关系？因为这是非常简单的数学，不是吗？蒙特卡洛模拟可以像这个计算一样简单，但是你可以也应该给它添加一些活力。

因为… **对自己诚实:**你永远不会像现在这样 100%的优秀，而纸上交易。所以，加入一些随机抽取和其他意想不到的东西是明智的。让你 528 点的最终结果更真实。

# 简单蒙特卡罗模拟

我们将从制作一个简单版本的蒙特卡罗模拟开始，只是为了满足我们的数据，如上所述。Go 将成为我们的编程语言选择。

如果你在问，为什么要走，彼得·贾霍达给了你七个理由

[](/7-reasons-you-should-try-use-go-5fb4714015d1) [## 你应该尝试围棋的 7 个理由

### 原因，我们停止使用 Java 和 C#

itnext.io](/7-reasons-you-should-try-use-go-5fb4714015d1) 

我们将只有一个 Go 文件，即`main.go`。打开这个文件并声明您的变量。对于这些变量，我们将使用与上述简单计算中相同的数据。

现在我们添加一些逻辑。一个循环就是一个模拟。

每次模拟都会随机生成一个介于 1 和`numberOfTrades`之间的数字。如果生成的数字在 1 和`profitPercentage`之间(在本例中为 60)，我们将增加我们的点数总和。否则我们将减少它。

最后，我们将减去价差和佣金，并打印结果。

如果你运行程序，这就是结果。当然，数字会因你而异。

有利有弊就是盈利和亏损。

```
Monte Carlo Simulation No.      1
+-++++++-+--++-+++-+-++++-+++-++++++--+-----+-++-+-+--++++-+++-+++-+-+-----+++---+-+-+-+--+++++-++++
Number of profits               61
Number of losses                39
Total sum of pips               582.8 pips
Average trade will give you     5.8 pipsMonte Carlo Simulation No.      2
++-+-++++-+-+-+-+++++++-+----+++-++-+-+++-+++---++----+-+-++-+-++-+++-+-+--+--+-+--+-+---------++-+-
Number of profits               52
Number of losses                48
Total sum of pips               89.6 pips
Average trade will give you     0.9 pipsMonte Carlo Simulation No.      3
+-+--++-++--+-----+-+++-+-+-+++-+--+--+-+--++-++--+-+-+-++++++-++-++-+--+-+--+---+-+-+-++-+-+--+--++
Number of profits               51
Number of losses                49
Total sum of pips               34.8 pips
Average trade will give you     0.3 pipsMonte Carlo Simulation No.      4
+-++----++-+-++-++++-+++-+---+++-++++++++--+-++-++--+-+--+-++++-+--+-++++-+---+++++-++--+++-++++++++
Number of profits               64
Number of losses                36
Total sum of pips               747.2 pips
Average trade will give you     7.5 pipsMonte Carlo Simulation No.      5
+++++---++--+-+-+-++-+++---+++--++-++-++-+-++-++-----+-+-++-+-+--+-+++-++++++++-+++-+-+++--+++-+-+++
Number of profits               61
Number of losses                39
Total sum of pips               582.8 pips
Average trade will give you     5.8 pipsMonte Carlo Simulation No.      6
++++-+++++++----++++-+++-++-+-----+++---+--++---+--+-++-++-++++++++++--+-+-++-+-+--++-++-+++-++++-++
Number of profits               62
Number of losses                38
Total sum of pips               637.6 pips
Average trade will give you     6.4 pipsMonte Carlo Simulation No.      7
--+----+++--++--++++--+++--+--+-+--++++-+++--------+++--+-+++++--++--+--++++++-++-+-+--++++----+-+-+
Number of profits               52
Number of losses                48
Total sum of pips               89.6 pips
Average trade will give you     0.9 pipsMonte Carlo Simulation No.      8
+++---++---+++++---++++-++++-++-------+-+----++++-+--++++-----+-+-++--+-+--+++++++-+++++---++++++--+
Number of profits               56
Number of losses                44
Total sum of pips               308.8 pips
Average trade will give you     3.1 pipsMonte Carlo Simulation No.      9
-+---+--++-++++++-++-+-++-+-+-+++++++++-+++-++++++--+++++---+++-++-+-------++-+++----+---+++--+-+-++
Number of profits               59
Number of losses                41
Total sum of pips               473.2 pips
Average trade will give you     4.7 pipsMonte Carlo Simulation No.      10
+-++-+--+-++++-+++--+-+-+-+--+++++++++++----+++---++---+----+-+-++++---+-+---+--+----++-+-+--++-+--+
Number of profits               51
Number of losses                49
Total sum of pips               34.8 pips
Average trade will give you     0.3 pips
```

如你所见，每一次，你都以盈利告终。但是范围在 34.8 点和 747.2 点之间。这是一个巨大的差异。

尽量增加`numberOfSimulations`。换点别的。玩这个程序，看看是什么和如何最大程度地改变了结果，使范围更窄，结果更一致。

# 高级蒙特卡罗模拟

现在让我们给我们的程序添加一些*伪现实主义*:

*   随机下跌(比如…你忘记设置止损，一秒钟内发生了意想不到的事情)
*   随机价差扩大(就像……一些消息传来，不知何故价差扩大到标准值的 3 倍，你的止损被击中)
*   随机情绪波动(比如…你和你的伴侣吵架了，你感觉很糟糕，但仍在交易)

我们还将添加一些简单的统计数据，对所有模拟进行平均。

下面是更新后的变量声明。

这是更新后的循环。我们正在添加一些随机的下拉菜单(！)、滑点(S)和糟糕的日子(B)。最后我们会做一个简单的总结，把结果打印出来。

如果您现在运行该程序，您将看到类似于

```
Monte Carlo Simulation No.      1
--+-+++-++++++---++-+++--+--++++++++-+-+-+++-++-+-+-+++----------+++++-+++--++++---++-++++-++-+-+++-
Number of profits               60
Number of losses                40
Total sum of pips               528.0 pips
Average trade will give you     5.3 pipsMonte Carlo Simulation No.      2
++--+--+--S+-++++++--+---+++-++++++++++++----+-+-++---+++--++++-+++++-++-++++-+++--+-++++++---++---+-
Number of profits               62
Number of losses                38
Total sum of pips               623.4 pips
Average trade will give you     6.2 pipsMonte Carlo Simulation No.      3
+------+--+++++++--+++++-+--+--+-+-++-++++++--S+-++-+++++++-+++--++-+-+-+++++----+++--+-+-+-+++++++--
Number of profits               61
Number of losses                39
Total sum of pips               568.6 pips
Average trade will give you     5.7 pipsMonte Carlo Simulation No.      4
+++-++++-+++++++-++-+-+++++----+-++-+++++-++-+++--S+-+++-+-++-+----++++--+++----+---++-+++++-+-++-+--
Number of profits               62
Number of losses                38
Total sum of pips               623.4 pips
Average trade will give you     6.2 pipsMonte Carlo Simulation No.      5
-++++++-+++++++--+-++++-----+---+-+-+---+---+++-+---+-++++++++++-+++++++++----+-++++--++++++---+-+--
Number of profits               60
Number of losses                40
Total sum of pips               528.0 pips
Average trade will give you     5.3 pipsMonte Carlo Simulation No.      6
+-+-+++-+++-+-+--+--+--++++--+-S+-++-+++++----++-+++-+++-+-+--+-+-++--+-++-+-+----++--++-++--++---+-+
Number of profits               54
Number of losses                46
Total sum of pips               185.1 pips
Average trade will give you     1.9 pipsMonte Carlo Simulation No.      7
--++---++++-+++-+--+---+-++--++++-+--+--++--+-+-++++++--+-+-++-+--+++-+-++++-+-+++-+-++-+++----+-+-+
Number of profits               56
Number of losses                44
Total sum of pips               308.8 pips
Average trade will give you     3.1 pipsMonte Carlo Simulation No.      8
-+-+-++--+++++++-S++++---S-+-+++-+++--++-+++---+-+--++----+++-+-+-+---++-+-+-+--+-++-++-+-+--++---++--
Number of profits               53
Number of losses                47
Total sum of pips               116.1 pips
Average trade will give you     1.2 pipsMonte Carlo Simulation No.      9
-++++--+++++--+++-+++++++-++-+++-+-+++++-+--++-+++-+++++-+---S--+++-+++++++--S+---S-+--++-+--+---++---+
Number of profits               61
Number of losses                39
Total sum of pips               540.3 pips
Average trade will give you     5.4 pipsMonte Carlo Simulation No.      10
+--++-+----++--+++-+++++-+++++-++++-+-++-+-+-S-++--++-+--++++--+++++--+++++----+-+-++--S++-++++-+-++++
Number of profits               62
Number of losses                38
Total sum of pips               609.3 pips
Average trade will give you     6.1 pipsAverage sum                         463.1 pips
Min sum                             116.1 pips
Max sum                             623.4 pips
```

好吧，我们有一些失误，但没有糟糕的一天或下降。原因是，我们只做了 10 次模拟。

尝试将`numberOfSimulations`增加到 100 或 1000，你会开始看到下降和糟糕的日子。但是你也会开始看到不那么令人满意的结果。

类似这样的…

```
Average sum                         -23.9 pips
Min sum                             -1322.9 pips
Max sum                             1116.6 pips
```

# 结论

那么，你怎样才能让它变得更好呢？如何使用这个程序来帮助你的交易策略。

你主要有三个选择:

*   增加盈利交易和亏损交易的比率
*   增加每笔盈利交易的平均点数
*   减少每笔亏损交易的点数

例如，如果你的策略符合本文开头的数字，你会知道，如果你像机器人一样交易，没有亏损、滑点和糟糕的日子，这个策略可能是有利可图的。但在(模拟)现实中，很可能你不会盈利。

使用蒙特卡洛模拟来对抗你的纸面交易数据，以获得(或失去)对你的交易系统的信心。

完整的代码在这里。

[](https://github.com/petrjahoda/monte_carlo_simulator) [## 彼得贾霍达/蒙特卡洛模拟器

### 在 GitHub 上创建一个帐户，为 petrjahoda/Monte _ Carlo _ simulator 开发做出贡献。

github.com](https://github.com/petrjahoda/monte_carlo_simulator)
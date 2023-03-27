# 融合:更多不可能成为可能！

> 原文：<https://itnext.io/fusion-more-of-impossible-is-possible-36ce5d19989f?source=collection_archive---------3----------------------->

![](img/f8b6c141309736d4d562d62bc1900278.png)

这也是融合。但不是真的。

这是过去 3 个月来添加到 [Fusion](https://github.com/servicetitan/Stl.Fusion) 中的功能的简要概述。其中有些已经在[教程](https://github.com/servicetitan/Stl.Fusion.Samples/blob/master/docs/tutorial/README.md)中有所涉及；剩下的我很快肯定会描述的。

1.  [指挥官](https://github.com/servicetitan/Stl.Fusion.Samples/blob/master/docs/tutorial/Part09.md)(挑战[媒体人](https://github.com/jbogard/MediatR)😎)和[操作框架](https://github.com/servicetitan/Stl.Fusion.Samples/blob/master/docs/tutorial/Part10.md) —这两个部分使得多主机状态同步成为可能。是的，现在您可以将融合服务扩展到多台主机，这几乎与任何其他服务一样简单。看看这个动画中的[是如何工作的。值得注意的是，这两个组件实际上都没有绑定到 Fusion，所以您可以将它们用于分布式失效以外的目的。](https://raw.githubusercontent.com/servicetitan/Stl.Fusion.Samples/master/docs/img/Samples-Template.gif)
2.  [桌游](https://boardgames.alexyakunin.com/)——目前为止最先进的融合样本。播放介绍视频，看看它能做什么。Blazor 服务器和 WASM 模式，2 个游戏，聊天，在线状态，OAuth 登录，用户会话跟踪，以及其他一些 100%实时功能。所有这些都是由 Fusion 和**35 行与实时更新相关的代码提供支持的！**
3.  [HelloCart 样本](https://github.com/servicetitan/Stl.Fusion.Samples/tree/master/src/HelloCart)——可能是新人最感兴趣的一个。它展示了如何从一个非常基本的融合服务过渡到一个融合驱动的 web API，该 API 通过 EF Core 保存数据并能够在多台主机上运行。优点:它以 4 种不同的方式(参见 [v1…v4 文件夹](https://github.com/servicetitan/Stl.Fusion.Samples/tree/master/src/HelloCart))实现了在 [Abstractions.cs](https://github.com/servicetitan/Stl.Fusion.Samples/blob/master/src/HelloCart/Abstractions.cs) 中声明的相同 API，并逐渐增加了复杂性和融合特性的使用。这个样本还没有被归档——这是我现在的首要任务。
4.  重构——大多数与 CommandR / Operations Framework 有关，但值得注意的是， [LiveComponentBase](https://github.com/servicetitan/Stl.Fusion/blob/master/src/Stl.Fusion.Blazor/LiveComponentBase.cs) (融合感知 UI 组件的基本类型)也变得更加用户友好。
5.  许多修复，虽然没有什么非常糟糕的。如果你是真的。网虫们，这里是[被可以说是迄今为止最难找到的问题](https://twitter.com/alexyakunin/status/1363738908043870210)影响的问题。示例代码看起来很简单(嗯，这是我能想到的最简单的相同根本原因的说明)，但是底层问题要复杂得多。

最近活动的视频和材料(不要害羞，分享吧！):

*   【about 用户群】我谈融合 ( [**幻灯片**](https://alexyakunin.github.io/Stl.Fusion.Materials/Slides/Fusion_v1/Slides.html) )
*   [类似的议论还在继续。亚美尼亚网络会议](https://youtu.be/fAHXrfmYVCk)。

成长:GitHub 上 830 颗星，从我[在这里](https://medium.com/swlh/fusion-current-state-and-upcoming-features-88bc4201594b)最后一次状态更新+500。

另外，如果你想以融合的方式(即实时)跟踪项目更新*——[加入我们的不和谐](https://discord.gg/EKEwv6d)，现在我在那里分享一些日常新闻，这肯定是问答的最佳位置。*
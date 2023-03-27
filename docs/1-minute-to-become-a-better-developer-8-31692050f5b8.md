# OMBD#8:从命令行找出我们的公共 IP 地址和国家

> 原文：<https://itnext.io/1-minute-to-become-a-better-developer-8-31692050f5b8?source=collection_archive---------2----------------------->

## 一个快速的解释和几个方便的一行程序

欢迎来到第 8 期，通过阅读简短的知识，每次一分钟，你将成为一名更成功的软件开发人员。

## [⏮](https://jportella93.medium.com/1-minute-to-become-a-better-developer-7-6c9c9fa67a9c)️[t17】🔛](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e) [⏭](https://jportella93.medium.com/1-minute-to-become-a-better-developer-9-43ce61662d72) ️ **️**

![](img/4f2fbea0b6272a719d7b0263389071ab.png)

我的好友洛尔·尼古拉斯的艺术作品

## 问题是

每个连接到互联网的设备都有一个分配的 IP 地址(互联网协议地址)，该地址由以下格式的 4 个数字组成:`<number>.<number>.<number>.<number>`。每个数字的范围可以是 0–255。如`184.152.81.47`。

当计算机通过互联网或本地网络相互通信时，信息共享是通过 IP 地址完成的。像物理地址一样，它们提供了一个发送信息的位置。

您的**公共 IP 地址**是一个面向外部的 IP 地址，由您的互联网服务提供商(ISP)提供。

因此，我们正在与我们在互联网上互动的每一个网站分享这条有价值的信息，它可以用来识别我们和我们的原籍国…

但是，我们如何从命令行找到我们的公共 IP 地址呢？

## 一个解决方案

这里有一个使用`api.ipify.org`免费服务的小程序:

```
curl -w "\n" -s [https://api.ipify.org](https://api.ipify.org)
# 116.164.32.81
```

为了方便起见，让我们将它保存在一个[别名](https://www.unixtutorial.org/create-alias-in-unix-shell/) our 或`~/.bashrc`或`~/.zshrc`中:

```
alias whatsmyip='curl -w "\n" -s [https://api.ipify.org'](https://api.ipify.org')
```

现在我们知道了我们的公共 IP 地址，让我们找出它的来源国。我们可以通过管道将别名结果传递给另一个免费服务`ipinfo.io`，该服务执行地理查找:

```
curl ipinfo.io/$(whatsmyip)
```

更好的是，还为此保存一个别名:

```
alias whatsmycountry='curl ipinfo.io/`whatsmyip`'
```

现在我们知道了这些信息，如果我们能隐藏它甚至改变它就太好了……**出于隐私原因**或者例如**让我们看起来像是在另一个国家**！

这里有一个如何做到这一点的故事:

[](https://jportella93.medium.com/1-minute-to-become-a-better-developer-10-bcb2396b6246) [## 1 分钟成为更好的开发人员(#10)

### 了解如何在一分钟内改变你的公共 IP 地址。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-10-bcb2396b6246) 

## 如果您喜欢这篇文章，您可能也会喜欢:

[](https://jportella93.medium.com/1-minute-to-become-a-better-developer-7-6c9c9fa67a9c) [## 1 分钟成为更好的开发人员(#7)

### 欢迎阅读本系列的第 7 期，通过阅读简短的知识，你将成为一名更成功的开发人员…

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-7-6c9c9fa67a9c) [](https://jportella93.medium.com/1-minute-to-become-a-better-developer-9-43ce61662d72) [## 1 分钟成为更好的开发人员(#9)

### 了解如何在一分钟内防止您的自定义别名被遗忘。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-9-43ce61662d72) 

## [⏮](https://jportella93.medium.com/1-minute-to-become-a-better-developer-7-6c9c9fa67a9c) ️ [🔛](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e) [⏭](https://jportella93.medium.com/1-minute-to-become-a-better-developer-9-43ce61662d72)
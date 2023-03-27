# Windows 11 上的 Android 应用，英特尔的战略举措

> 原文：<https://itnext.io/android-apps-on-windows-11-an-intel-strategic-move-37ccfe0c068d?source=collection_archive---------0----------------------->

![](img/fe382564e722e0ab047a7eba91b0c98e.png)

来自[维基百科](https://en.wikipedia.org/wiki/Android_(operating_system))

过去几天最有趣的消息之一是，Windows 11 将允许原生运行 Android 应用程序(来自亚马逊应用商店)。这项新功能由英特尔桥技术提供支持，目前没有透露太多技术细节，但从一些描述来看:

> 英特尔桥技术是一种运行时后编译器，支持应用在基于 x86 的设备上本地运行，包括在 Windows 上运行这些应用。— [英特尔酷睿处理器和英特尔桥接技术释放 Windows 11 体验。英特尔新闻编辑室](https://www.intel.com/content/www/us/en/newsroom/news/intel-tech-unleashes-windows-experience.html#gs.4yzomb)
> 
> 英特尔在给 *The Verge* 的一份声明中证实:“英特尔认为在所有 x86 平台上提供这种能力非常重要，并设计了英特尔桥技术来支持所有 x86 平台(包括 AMD 平台)。微软进一步证实，Android 应用程序将适用于所有芯片供应商，包括 Arm，尽管该公司尚未谈论这些应用程序的实际运行情况。— [Windows 11 基于英特尔技术的安卓应用也可以在 AMD 和 Arm 处理器上运行。边缘](https://www.theverge.com/2021/6/24/22549303/windows-11-intel-bridge-android-apps-amd-arm-processors)

我们可以了解关于英特尔桥技术的一些事实:

1.  它是一个运行时后编译器。
2.  不限于英特尔硬件，也支持其他硬件(包括 AMD)。

实际上，这并不是一项全新的技术，但对于像英特尔这样的大公司来说，这是第一次，在这里，我想分享一下我对这一举措背后的基本原理的一些看法:

# 为什么我们现在不能在 Windows 上运行 Android 应用程序？不是用 Java/Kotlin 写的吗？

是的，它们都是用 Java/Kotlin 编写的，并且都利用 JVM 来运行程序，JVM 是可移植的。但对于大多数主流 Android 应用程序，Android 应用程序二进制接口(ABI)用于提供最高的性能，因为它允许开发人员使用从 C/C++语言编译的二进制文件。尽管 ABI 支持 arm、arm64、x86 和 x86_64，但大多数时候开发人员可能只包括 arm 和 arm64 二进制文件，如:

1.  维护 ARM 和 x86 版本都需要很大的努力(尤其是在做汇编级优化的时候)
2.  它增加了应用程序大小(不多，但对慢速网络很重要)
3.  几乎没有使用 x86 CPU 的安卓手机(例如。英特尔凌动)自 2016 年以来

因此，如果 Android 应用程序不支持 x86 二进制文件，您需要一个 ARM 到 x86 的翻译工具(英特尔将其定位为运行时后编译器)来运行它们。

# ARM 到 x86 的转换问题

ARM 到 x86 的翻译，就像任何自然语言之间的翻译一样，目前并不完美。这是一个正在进行的研究领域，仍然有许多研究人员试图完美地解决这个问题。因此，用户在 Windows 11 上运行 Android 应用时可能会面临**可靠性**和**性能**问题，但由于英特尔和微软将发布它，我相信至少我们可以运行所有主要的 Android 应用，而不会崩溃和 iresponsive UI。

# 英特尔的战略(也许)

那么，为什么是英特尔开发了这样一个翻译工具来允许 Android 应用程序在 Windows 上运行呢？我的猜测是，英特尔正在准备下一代基于 x86 的智能手机，以与 ARM 竞争。故事是这样的:

1.  人们可以在 Windows 上运行 Android 应用程序，这是市场份额最大的操作系统
2.  Windows 上的安卓应用崩溃或变慢，人们抱怨
3.  英特尔继续改进桥接技术，或者 Android 应用程序开发人员开始进行 x86 交叉编译并包含 x86 二进制文件
4.  现在大多数 Android 应用程序都兼容 x86
5.  英特尔是时候发布新一代基于 Android x86 的智能手机了。

虽然这听起来有点疯狂，但我认为有可能发生。毕竟智能手机 CPU 的市场那么大，英特尔不能忽视。

希望你喜欢这篇文章，并随时留下回复来讨论。😃
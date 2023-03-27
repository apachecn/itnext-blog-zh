# Kubernetes 之旅—启动并运行云计算— Linux 映像

> 原文：<https://itnext.io/kubernetes-journey-up-and-running-out-of-the-cloud-linux-image-c51953e50d0e?source=collection_archive---------4----------------------->

![](img/71d4d7f50888cacc0dab78443dce5d5c.png)

照片由[张秀坤·东布罗夫斯基](https://unsplash.com/photos/KNUp-YBwBSE?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/reflection?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

欢迎回到我们的 [Kubernetes 之旅](/kubernetes-journey-up-and-running-out-of-the-cloud-introduction-f04a811c92a5)。我这边的事情真的很忙，自从我发表这个系列的最后一篇文章[关于法兰绒](/kubernetes-journey-up-and-running-out-of-the-cloud-flannel-c01283308f0e)已经有一段时间了。对此我很抱歉，我会努力保持良好的状态。

在我们继续之前，您可能想回顾一下我们到目前为止所经历的一切。你可以在本文的末尾[找到本系列所有文章的索引。](/kubernetes-journey-up-and-running-out-of-the-cloud-introduction-f04a811c92a5)

在我们的第 9 篇文章中，我们将配置**Debian**/**Ubuntu**Linux 映像，它将作为构成我们技术堆栈的所有其他组件的基础。要了解本系列接下来的文章，请看看我们的第三篇文章，[技术栈](/kubernetes-journey-up-and-running-out-of-the-cloud-technology-stack-9c472aafac4e)，在这里我们解释了什么是组件以及它们的用途。

说够了。让我们最后用一些酷的东西弄脏我们的手吧！

请记住，本主题中提到的信息在你根据需要采取的计划中具有相对重要性，你不一定要用铁腕手段来控制这个主题。

如果你不想等到所有的文章都发表了，又想马上动手，可以随意克隆项目的 Github repo。它很实用，文档也在不断改进:

[](https://github.com/mvallim/kubernetes-under-the-hood) [## 罩下的姆瓦利姆/库伯内特斯

### 它甚至还包括一张幻灯片，解释了它吸引目标受众的原因…

github.com](https://github.com/mvallim/kubernetes-under-the-hood) 

# 分割

配置 Linux 时要做的重大决定是如何划分硬盘空间。

这里提出的设计允许在需要时进行动态增长和微调。在没有更多可用存储空间的情况下猝不及防，除了删除文件之外没有其他即时选项，这绝不是一种好的体验。必须考虑到系统的长期寿命和增长以及预算问题。

隔离根卷，尤其是对于不会随时间增长太多的静态数据来说，是最主要的问题。将其他目录隔离在它们自己的卷中是这里采用的策略，这样它们的动态增长就不会影响根分区。在系统中填充根卷是一件非常糟糕的事情，应该不惜一切代价避免。通过隔离分区，我们有一系列的选项可以选择，例如，增加一个分区和/或减少另一个分区，因为卷不是 100%被逻辑卷(分区)占用。

稍后可能会增加分区，但这是我们的卷在系统安装时最初的划分方式:

*   **boot** — 512 Mb —引导加载程序文件(例如:kernel，initrd)。位于逻辑卷管理器(LVM)之外的单个空间
*   **根** — 2 Gb —操作系统(/bin、/lib、/etc、/sbin)
*   **首页** — 2 Gb —用户目录。
*   **opt** — 1 Gb —静态应用程序包。
*   **tmp** — 1 Gb —临时文件。
*   **usr** — 10 Gb —共享用户数据的二级层次结构，限制只读访问。
*   **var** — 10 Gb —“可变”文件，如日志、数据库、网页、邮件文件、容器图片等。

> *参考:*[*http://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html*](http://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html)

# 软件

有必要安装构成基本映像的软件包，以避免在基于该映像创建的其他虚拟机中重复工作。

由于我们使用 VirtualBox 作为我们的虚拟化系统，应该组成每个映像的一个重要软件是 [VirtualBox 访客添加](https://docs.oracle.com/cd/E36500_01/E36502/html/qs-guest-additions.html)，以及它的依赖项。

要安装的软件如下:

*   这个包包含了一个包的信息列表，这些包被认为是构建 Debian 包所必需的。这个包也依赖于列表中的包，这样就可以很容易地安装构建必需的包。
*   模块助手
    模块助手工具(也称为 m-a)帮助用户和维护人员管理为 Debian 打包的外部 Linux 内核模块。它还包含一些基础设施，供 Debian 中附带的模块源码包中的构建脚本使用。
*   resolvconf 是一个框架，用于保持系统关于域名服务器的最新信息。它将自己设置为提供此信息的程序(如 ifup 和 ifdown、DHCP 客户端、PPP 守护程序和本地名称服务器)和使用此信息的程序(如 DNS 缓存和解析程序库)之间的中介。
*   ntp(网络时间协议)通过互联网或本地网络同步计算机时钟，或者通过遵循解释 GPS、DCF-77、NIST 或类似时间信号的精确硬件接收器，来保持计算机时钟准确。
*   sudo 是一个允许系统管理员给用户有限的根权限并记录根活动的程序。基本理念是尽可能少地给予特权，但仍然允许人们完成他们的工作。
*   **Cloud-init** Cloud-init 为基础设施即服务(IaaS)云平台提供了配置和定制虚拟机实例的框架和工具。例如，它可以设置默认的语言环境和主机名，生成 SSH 私有主机密钥，安装 SSH 公共密钥以便登录到默认帐户，设置临时挂载点，以及运行用户提供的脚本。
*   **VirtualBox Guest Additions** VirtualBox Guest Additions 由设备驱动程序和系统应用程序组成，可优化操作系统以获得更好的性能和可用性。本指南中要求的可用性功能之一是自动登录，这就是为什么您需要在虚拟机中安装来宾附件。

> *参考:*apt-缓存显示包名

# 装置

请按照这里的步骤来。

在下面的步骤中，我们将不会采用 LVM 和多卷分区，请再次记住，本主题中提到的信息在您将根据需要采用的计划中具有相对重要性，并且您不一定要用铁腕手段来解决这个问题。

## 或者，如果您喜欢下载基本映像

## 一种自由操作系统

```
~$ cd ~/VirtualBox\ VMs/
~$ curl -L --progress-bar "https://vms-image.s3.amazonaws.com/debian-base-image.tar.bz2" -o - | tar xjf -
~$ vboxmanage registervm ~/VirtualBox\ VMs/debian-base-image/debian-base-image.vbox
```

## 人的本质

```
~$ cd ~/VirtualBox\ VMs/
~$ curl -L --progress-bar "https://vms-image.s3.amazonaws.com/ubuntu-base-image.tar.bz2" -o - | tar xjf -
~$ vboxmanage registervm ~/VirtualBox\ VMs/ubuntu-base-image/ubuntu-base-image.vbox
```

我希望你喜欢这篇文章以及这个系列的其他文章。现在我们已经准备好使用我们的基本映像，在下一篇文章中，我们将通过创建我们的**网关**实例和一个 **BusyBox** 实例来开始部署我们的解决方案，这将帮助我们管理我们所有的实例。

如果你认为这是有帮助的，请留下你的👏以及下面的反馈。不断完善这个系列的内容非常重要。

同样，我强烈建议您在 Medium 上关注我，这样您就不会错过本系列中发表的任何新文章。如果你错过了这个系列的第一篇文章，你可以在这里查看。

回头见！！

再见
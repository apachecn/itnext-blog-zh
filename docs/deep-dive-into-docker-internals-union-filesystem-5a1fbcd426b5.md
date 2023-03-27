# 深入了解 Docker 内部——Union 文件系统

> 原文：<https://itnext.io/deep-dive-into-docker-internals-union-filesystem-5a1fbcd426b5?source=collection_archive---------0----------------------->

## 探索 OverlayFS 的内部工作方式，overlay fs 是 Docker 映像和容器分层架构背后的文件系统

使用 Docker CLI 非常简单——你只需要`build`、`run`、`inspect`、`pull`和`push`容器和图像，但是你有没有想过 Docker 接口背后的内部是如何工作的？在这个简单的接口背后隐藏着许多很酷的技术，在本文中我们将探索其中之一——联合文件系统——所有容器和图像层背后的底层文件系统...

![](img/7754e69100dc8bd5e010b78d5bf61531.png)

肖恩·斯特拉顿在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# 什么是联合文件系统？

Union mount 是一种文件系统，它可以创建一种假象，将几个目录的内容合并成一个目录，而不修改其原始(物理)源。这很有用，因为我们可能将相关的文件集存储在不同的位置或介质中，但我们希望在一个合并的视图中显示它们。例如，来自远程 NFS 服务器的一群用户的`/home`目录全部*联合*成一个目录，或者将分割的 ISO 映像合并成一个完整的目录。

联合装载或联合文件系统为；然而，*不是*文件系统类型，而是一个有许多实现的概念。其中一些更快，一些更简单，具有不同的目标或不同的成熟度。因此，在我们开始深入研究细节之前，让我们快速浏览一下一些更流行的可用实现:

*   *UnionFS* —让我们从最初的 union 文件系统开始。UnionFS 似乎不再被积极开发了，它的[最近一次提交](http://git.fsl.cs.sunysb.edu/?p=unionfs-2.6.39.y.git;a=summary)是在 2014 年 8 月。你可以在 https://unionfs.filesystems.org/的网站上了解更多。
*   *aufs*——原始 UnionFS 的重新实现，增加了许多新特性，但由于合并到主流 Linux 内核中而被拒绝。Aufs 是 Ubuntu/Debian 上 Docker 的默认驱动程序，但被 OverlayFS 取代(用于 Linux 内核> 4.0)。与 Docker [文档第](https://docs.docker.com/storage/storagedriver/aufs-driver/#aufs-and-docker-performance)页描述的其他 union 文件系统相比，它有一些优势。
*   *OverlayFS* —接下来，OverlayFS 从 3.18(2014 年 10 月 26 日)开始包含在 Linux 内核中。这是默认使用的文件系统`overlay2` Docker 驱动程序(你可以用`docker system info | grep Storage`来验证)。它通常比 aufs 有更好的性能，并且有一些很好的特性，比如 [*页面缓存共享*](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#overlayfs-and-docker-performance) 。
*   *ZFS*——ZFS 是由*太阳微系统*(现在的*甲骨文*)创建的联合文件系统。它有一些有趣的功能，如分层校验和、快照和备份/复制的本机处理或本机数据压缩和重复数据删除。然而，由于由 Oracle 维护，它拥有非操作系统友好许可证(CDDL ),因此不能作为 Linux 内核的一部分提供。然而，你可以使用 Linux 上的*ZFS(ZoL)*项目，Docker 文档中将其描述为*健康且成熟的…，但尚未准备好投入生产*。如果你想尝试一下，那么你可以在这里找到它[。](https://zfsonlinux.org/)
*   *Btrfs* —另一个选项是 Btrfs，它是[多家公司](https://btrfs.wiki.kernel.org/index.php/Contributors) —包括 SUSE、WD 或脸书——在 GPL 许可下发布的联合项目，是 Linux 内核的一部分。Btrfs 是 Fedora 33 的默认文件系统。它还具有一些有用的功能，如块级操作、碎片整理、可写快照和许多其他功能。如果你真的想通过切换到 Docker 的非默认存储驱动程序的麻烦，那么 Btrfs 及其特性和性能可能是正确的选择。

如果你想更详细地研究这些与 Docker 相关的驱动程序，你可以查看 [Docker 文档](https://docs.docker.com/storage/storagedriver/select-storage-driver/)中的驱动程序比较。也就是说，除非你*真的*知道你在做什么(在这一点上你不会读这篇文章)，那么你应该坚持使用默认的`overlay2`，它也将在本文的其余部分用于演示。

# 但是为什么呢？

在上一节中，我们提到了为什么这种类型的文件系统可能是有用的，但是为什么它是 Docker 和容器的好选择呢？

我们用来旋转容器的许多图片都很大，不管是 72MB 的`ubuntu`还是 133MB 的`nginx`。每次我们想从这些图像中创建一个容器时，分配那么多空间是非常昂贵的。由于有了 union 文件系统，Docker 只需要在映像上创建一个薄层，其余部分可以在所有容器之间共享。这还提供了减少启动时间的额外好处，因为不需要复制图像文件和数据。

Union 文件系统还提供了隔离，因为容器对共享图像层具有只读访问权限。如果他们需要修改任何只读共享文件，他们使用*写时复制*策略(稍后讨论)将内容复制到他们的顶部可写层，在那里可以安全地修改内容。

# 它是如何工作的？

现在是时候问一个重要的问题了——它实际上是如何工作的？从上面描述的所有事情来看，整个 union 文件系统似乎是某种魔法，但事实并非如此。让我们首先解释一下它在一般(非容器)情况下是如何工作的——让我们想象一下，我们希望将两个目录(`upper`和`lower`)联合挂载到同一个挂载点，并拥有它们的联合视图:

在 union mount 术语中，这些目录被称为*分支*。这些分支中的每一个都被分配了优先级。如果在多个源分支中有相同名称的文件，这个优先级用于确定哪个文件将出现在合并视图中。看看上面的文件和目录——很明显，如果我们试图覆盖它们，我们就会产生这种冲突(`code.py`文件)。所以，让我们试着看看会出现什么:

在上面的例子中，我们使用类型为`overlay`的`mount`命令来合并`lower`目录(只读；低优先级)和`upper`目录(读写；更高的优先级)到`/mnt/merged`中的合并视图中。我们还包括了`workdir=./workdir`选项，它是在将`lowerdir`和`upperdir`移动到原子操作中的`/mnt/merged`之前，准备合并视图的地方。

同样查看上面的`cat`命令的输出，我们可以看到在合并视图中`upper`目录中的文件内容确实优先。

所以，现在我们知道了如何合并两个目录，如果有冲突会发生什么，但是如果我们试图从合并视图中修改一些文件会发生什么呢？这就是*写时拷贝(CoW)* 发挥作用的地方。那么，到底是什么呢？CoW 是一种优化技术，如果两个调用者请求相同的资源，您可以给他们指向相同资源的指针，而不用复制它。只有当调用者之一试图写入他们的*“副本”*时，复制才变得必要——因此术语*在(第一次尝试)写入*时复制。

在联合挂载的情况下，这意味着当我们试图修改共享文件(或只读文件)时，它首先被复制到顶部可写分支(`upperdir`)，该分支比只读的较低分支(`lowerdir`)具有更高的优先级。然后——当它在可写分支中时——可以安全地修改它，并且它的新内容将在合并视图中可见，因为顶层具有更高的优先级。

我们可能要执行的最后一个操作是删除文件。为了执行*【删除】*，在可写分支中创建一个*空白*文件来清除我们想要删除的文件。这意味着文件实际上并没有被删除，而是隐藏在合并视图中。

我们讨论了很多关于 union mount 的一般工作方式，但是它和 Docker 及其容器有什么关系呢？为了将它们连接起来，让我们看看 Docker 分层架构。一个容器的沙箱由一些图像分支组成——或者我们都知道它们——*层*。这些层是合并视图的只读(`lowerdir`)部分，容器层是瘦的可写顶部(`upperdir`)部分。

除了这个架构术语之外，实际上是同样的事情——从注册表中提取的图像层是`lowerdir`,当您运行容器时，`upperdir`被附加到图像层的顶部，为您的容器提供可写的工作空间。听起来很简单，对吗？所以，我们来试试吧！

# 尝试一下

为了演示 OverlayFS 是如何被 Docker 使用的，我们将尝试模拟 Docker 是如何挂载容器和图像层的。在我们这样做之前，我们首先需要清理我们的工作区，并得到一个图像来玩:

我们有一个图像(`nginx`)来玩，所以接下来，让我们检查它的层。我们可以通过在图像上运行`docker inspect`并检查`GraphDriver`字段或者通过浏览存储所有图像层的`/var/lib/docker/overlay2`目录来检查图像层。所以，让我们两个都做，看看里面有什么:

看看上面的输出，它看起来和我们用`mount`命令看到的非常相似，对吗？更具体地说:

*   `LowerDir`:只读图层目录，用冒号分隔
*   `MergedDir`:来自图像和容器的所有层的合并视图
*   `UpperDir`:写入变更的读写层
*   `WorkDir`:Linux overlay fs 用来准备合并视图的工作目录

接下来，让我们更进一步，运行一个容器并检查它的层:

上面的输出显示，之前在`docker inspect nginx`的输出中列出的与`MergedDir`、`UpperDir`和`WorkDir`(id 为`3d963d191b2101b3406348217f4257d7374aa4b4a73b4a6dd4ab0f365d38dfbd`)相同的目录现在是容器的`LowerDir`的一部分。这里的`LowerDir`是由所有的`nginx`图像层叠加而成的。在它们上面是`UpperDir`中的可写层，包含`/etc`、`/run`和`/var`。同样，如果我们在上面列出了`MergedDir`，您将看到容器可以使用整个文件系统，包括来自`UpperDir`和`LowerDir`的所有内容。

最后，为了模拟 Docker 的行为，我们可以使用这些相同的目录来手动创建我们自己的合并视图:

在这里，我们只是从前面的代码片段中抓取值，并将它们传递给`mount`命令中的适当参数，唯一的不同是我们使用`/mnt/merged`来合并视图，而不是`/var/lib/docker/overlay2/.../merged`。

这也是 Docker 中整个 OverlayFS 的本质——一个`mount`命令跨越多个堆叠层。下面是负责这个的 Docker 代码的一部分——替换`lowerdir=...,upperdir=...,workdir=...`值，然后是`unix.Mount`

# 结论

当从外面看 Docker 接口时，它可能看起来像一个黑盒子，里面有许多模糊的技术。这些技术——虽然晦涩难懂——非常有趣和有用，虽然你不需要理解它们就能有效地使用 Docker，但在我看来——学习和理解它们仍然是值得的。对该工具有了更深入的理解，就可以更容易地做出正确的决策——在这种情况下——关于性能优化或安全影响。另外，它还能帮助你发现一些很酷的技术，这些技术在未来会有更多的应用。

在本文中，我们只探讨了 Docker 架构的一部分——文件系统——还有其他值得深入研究的部分，比如`cgroups`或 Linux 名称空间。因此，如果您喜欢这篇文章，请密切关注后续文章，我们也将深入研究这些技术。😉

*本文最初发布于*[*martinheinz . dev*](https://martinheinz.dev/blog/44?utm_source=medium&utm_medium=referral&utm_campaign=blog_post_44)

[](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [## 用 seccomp 强化 Docker 和 Kubernetes

### 您的容器可能不像您想象的那样安全，但是 seccomp 配置文件可以帮助您解决这个问题…

itnext.io](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [](/building-docker-images-the-proper-way-3c9807524582) [## 以正确的方式构建 Docker 图像

### 让我们优化 Docker 构建，以创建更小、更安全的 Docker 映像，只需普通构建的一小部分…

itnext.io](/building-docker-images-the-proper-way-3c9807524582) [](https://towardsdatascience.com/its-time-to-say-goodbye-to-docker-5cfec8eff833) [## 你不用再用 Docker 了

### Docker 不是唯一的集装箱工具，可能会有更好的替代工具…

towardsdatascience.com](https://towardsdatascience.com/its-time-to-say-goodbye-to-docker-5cfec8eff833)
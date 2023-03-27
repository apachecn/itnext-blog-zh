# 用 Kaniko 构建容器形象

> 原文：<https://itnext.io/container-image-building-with-kaniko-dda7a451e25c?source=collection_archive---------2----------------------->

[![](img/88bbcc04d197a665b237a742a81c80ac.png)](https://www.giantswarm.io/on-demand-webinar-kubernetes-hard-lessons-and-how-to-avoid-them?utm_campaign=Medium%20CTA%20Conversion&utm_source=Kubernetes%20hard%20lessons_Medium&utm_medium=Medium%20CTA&utm_term=Kubernetes%20hard%20lessons)

在我们关于容器图像构建技术的[系列](https://blog.giantswarm.io/building-container-images-with-img/)的下一篇文章中，我们将关注名为 [Kaniko](https://github.com/GoogleContainerTools/kaniko) 的开源项目。

Kaniko 是谷歌在 2018 年初发起的项目。Kaniko 背后的基本原理是寻求在执行容器映像构建时消除对提升特权的需求。如果您一直在阅读本系列文章，那么您现在应该已经知道，非特权容器映像构建是关注安全性的组织最需要的特性之一。这与 Kubernetes 集群中的容器映像构建特别相关。

**那么，Kaniko 是什么，它是如何工作的？**

# 方法

Kaniko 使用您通常用来构建容器图像的熟悉对象；包含构建指令和构建上下文的 other 文件，其中包含 other 文件和映像所需的任何其他工件。

然而，与本地 Docker 客户机构建命令不同，它不需要 Docker 守护进程来执行构建步骤。

相反，Kaniko 使用自己的'[执行器](https://console.cloud.google.com/gcr/images/kaniko-project/GLOBAL/executor@sha256:9c40a04cf1bc9d886f7f000e0b7fa5300c31c89e2ad001e97eeeecdce9f07a29?tag=latest)来执行构建步骤，它在一个容器(可能是一个 Kubernetes pod)中运行。当然，构建上下文需要对容器或 pod 可用，以便构建使用它。因为构建步骤是在容器内部执行的，所以最终仍然依赖于 Docker 守护进程或其他一些容器运行时，但是重要的区别在于构建步骤本身是由执行器代码执行的。

执行器遍历 over 文件中定义的每个构建阶段。它提取为构建阶段指定的基础映像并解压缩其 rootfs。然后，它依次执行每个 Dockerfile 指令，在执行过程中添加或更改 rootfs 的内容。如果对 rootfs 进行了更改，执行器会将文件系统的更改作为“diff”层进行快照，并在必要时更新映像元数据。快照是通过将文件系统的先前状态与 Dockerfile 指令执行后的状态进行比较来实现的。这与 overlayfs 之类的写时复制文件系统的行为没有什么不同，只是该操作完全是在用户空间中执行的。一旦构建步骤完成，差异层将按顺序添加到基础映像中以形成新映像。

**“构建的映像会发生什么变化？我听到你问，“图像不会随着用过的容器或容器消失吗？**

嗯，executor 还希望在命令行上指定一个或多个容器注册库名称，这样它就可以在成功构建完成时推送新的映像。

[![](img/9ecafeb580122d973bbb8fe546c81b45.png)](https://www.giantswarm.io/on-demand-webinar-kubernetes-hard-lessons-and-how-to-avoid-them?utm_campaign=Medium%20CTA%20Conversion&utm_source=Kubernetes%20hard%20lessons_Medium&utm_medium=Medium%20CTA&utm_term=Kubernetes%20hard%20lessons)

# 构建上下文

Kaniko 在定义构建上下文的来源时非常灵活。可以使用卷挂载来指定包含上下文的本地目录(到容器),上面的示例强调了使用 git 存储库作为源。

考虑到 Kaniko 的起源，如果云存储不被认为是构建上下文的来源，那将是一个惊喜。果然，Kaniko 支持使用三大公共云提供商提供的对象存储解决方案。

# 构建缓存

容器映像构建的一个非常重要的组件是能够利用之前对映像构建步骤的调用，这些调用通常会被缓存以用于此目的。如果构建步骤会产生相同的输出，构建器将使用缓存的内容，而不是重新执行构建步骤。

**Kaniko 为增强的图像构建提供了两种缓存功能:**

1.  基本映像可以下载到一个本地的共享卷上，在调用 executor 容器时可以使用这个卷。这就避免了每次调用执行器容器时都要拉同一个基础映像，从而节省了构建时间。Kaniko 为此提供了“加热器”。您只需要指定本地目录来存储您想要缓存的图像和相关图像。
2.  在映像构建过程中创建的中间映像层也是构建缓存的重要组成部分。同样，Kaniko 支持缓存在执行 RUN Dockerfile 指令期间创建的图像，但是这些图像存储在远程容器注册表存储库中。在执行器处理运行指令之前，检查指定的储存库的等效层。如果找到了，它将从存储库中取出，而不是正在执行的指令。

# 安全性

由于 Kaniko 避免了使用 Docker 守护进程的构建 API 端点来执行构建步骤，这对于安全性有很大帮助。执行器可以作为非特权容器运行，不需要在容器中安装 Docker 守护进程套接字。这是寻求在 Kubernetes 上的 pods 中安全运行容器映像构建的重要一步。

缺点是构建步骤可能需要以 root (uid=0)用户的身份执行。一般来说，容器应该使用非 root 用户运行—容器中的 root 用户就是主机上的 root 用户。如果基础映像的 rootfs 的组件需要由 root 用户拥有，或者命令需要以 root 权限执行，这对于 Kaniko 来说是不可避免的。

与作为根用户运行 executor 容器相关的一些问题可以得到部分缓解。通过提供额外的隔离，可以保护运行 executor 容器的主机。谷歌自己的 [gVisor](https://gvisor.dev/) 用户空间内核抽象是这种隔离的一个很好的例子，正如 [Kata 容器](https://katacontainers.io/)运行时一样。

# 结论

Kaniko 是一种新型的图像构建工具，旨在消除对 Docker 守护进程的长期依赖。它很好地做到了这一点，并在 Kubernetes 集群中提供了可信的容器映像构建体验。这是在没有通常与针对 Docker 守护进程的构建相关联的安全恐惧的情况下实现的。出于这个原因，Kaniko 是一个流行的映像构建工具，并经常作为 [Tekton 管道](https://github.com/tektoncd/pipeline)的后端构建任务。

尽管 Kaniko 很受欢迎，但它并没有完全实现无根容器映像构建的涅槃，并且还延续了基于 Dockerfile 的构建的顺序性质。

您是否希望将 Kubernetes 容器投入生产？[联系 Giant Swarm](https://www.giantswarm.io/contact) 开始你的云原生之旅。

由 [Puja Abbassi](https://twitter.com/puja108) 撰写——开发者倡导者@ [巨型蜂群](https://giantswarm.io/)

[](https://twitter.com/puja108) [## Puja Abbassi

### Puja Abbassi 的最新推文(@puja108)。开发者关系&产品@ GiantSwarm 研究员；主题…

twitter.com](https://twitter.com/puja108)
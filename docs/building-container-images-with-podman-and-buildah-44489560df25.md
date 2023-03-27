# 用 Podman 和 Buildah 构建容器图像

> 原文：<https://itnext.io/building-container-images-with-podman-and-buildah-44489560df25?source=collection_archive---------6----------------------->

[![](img/4bf2e48f888ba30cfc7a1858f63698a9.png)](https://www.giantswarm.io/on-demand-webinar-kubernetes-hard-lessons-and-how-to-avoid-them?utm_campaign=Medium%20CTA%20Conversion&utm_source=Kubernetes%20hard%20lessons_Medium&utm_medium=Medium%20CTA&utm_term=Kubernetes%20hard%20lessons)

这是关于构建容器图像的系列博客文章的第二篇。系列开始于[集装箱形象建设的未来是什么？](https://blog.giantswarm.io/what-is-the-future-of-container-image-building/)它着眼于自 Docker 首次推出以来构建图像的变化，以及如何克服使用 Docker 文件的一些限制。[这篇文章关注的是 Podman 和 Buildah](https://www.giantswarm.io/blog/building-container-images-with-podman-and-buildah) ，在以后的文章中，我们将研究这一领域的其他新方法。

[Podman](https://podman.io/) 和 [Buildah](https://buildah.io/) 是最近出现的两种帮助构建容器映像的工具。它们是互补的工具，都是容器工具开放库的组成部分，并且源于 Red Hat 从容器工作流中删除 Docker 守护进程的使命。为什么是两个工具，每个工具给容器映像构建体验带来了什么？先说波德曼。

# 波德曼

Podman 的目的超出了容器映像构建的目标，但是它经常与 Buildah 一起讨论，并且在这里被考虑，因为它对容器映像构建有贡献。

# 无梦构建

Podman 试图再现熟悉的 Docker CLI 的整体，而不需要运行守护程序来服务和处理 API 请求。Podman 实现了一个本地 fork/exec 模型，而不是客户端/服务器模型，在 Red Hat 的眼中，这大大简化了容器生命周期的控制和安全性。

Podman 模仿 Docker 提供的各种客户端命令，一些倡导者甚至鼓励新用户将命令别名为 T1，以便于从一个到另一个的迁移。在波德曼提供的一套类似 Docker 的命令中，有一个是`podman build`命令。它用于构建[符合 OCI 标准的容器映像](https://github.com/opencontainers/image-spec/blob/master/spec.md#open-container-initiative)，使用 docker 文件作为各个构建步骤的源。在这个意义上，它实际上与`docker build command`相同，但是没有 Docker 守护进程的开销。

正如您所料，所有熟悉的`docker build`命令行参数都在`podman build`中可用(除了仍未实现的一个，如`--cache-from`)，需要一些额外的参数来代替 Docker 守护进程通常提供的一些功能(如注册表通信)。从`docker build`切换到`podman build`是一种无缝的体验，除了一些奇怪的地方，比如需要[指定](https://github.com/containers/buildah/blob/master/install.md#man-page-registriesconf5)在哪里找到没有完全限定的图像名的图像。

***有兴趣看看 Giant Swarm 如何与 OpenShift 短兵相接？*** [***阅读我们的文章，该文章将使用巨型 Swarm 的主要优势与 open shift***](https://blog.giantswarm.io/giantswarm-vs-openshift/)***进行了对比。***

# 无根建筑

随着一个大目标——无缝构建体验的实现，Podman 还提供了另一个受欢迎的特性——无根容器构建。从历史上看，由于 Docker 守护进程，使用`docker build`构建容器映像需要 root 权限，这种级别的访问权限在有安全意识的组织中通常被认为过于宽松。通过提供执行无根构建的能力，Podman 回答了这个严重的问题，但它也有其局限性。

从 Dockerfiles 构建映像的过程包括为运行命令临时创建容器，以便安装包、检索远程内容、构建工件等等。创建和运行容器通常需要 root 权限。那么，波德曼如何满足这种明显的二分法呢？为了避免以根用户身份运行构建，Podman 使用了[用户名称空间](http://man7.org/linux/man-pages/man7/user_namespaces.7.html)。[名称空间](http://man7.org/linux/man-pages/man7/namespaces.7.html)为 Linux 进程提供了一种隔离机制，并且是容器抽象的主要组成部分。如果创建容器时使用的名称空间集包括用户名称空间，那么调用容器的代理可以是无特权用户——换句话说，使用用户名称空间，Podman 可以使用容器来实现无根映像构建。

用户命名空间提供了一种将主机的默认用户命名空间中的一系列非特权用户和组 id(uid/GID)映射到与容器相关联的新用户命名空间中的一组不同的 uid/GID 的方式。通过这种方式，主机上的非特权 UID/GID 可以安全地映射到容器内的 root 用户(UID/GID=0 ),为容器的进程提供作为映像构建的一部分可能需要的提升的特权(例如，安装 OS 软件包)。但是由于这种映射，容器在主机上只拥有与发出`podman build`命令的非特权用户相同的文件访问权限。这意味着主机的文件系统受到保护，不会受到意外或恶意的容器损害。

[![](img/3ef8462f8c5043c2475daa2b6bdff075.png)](https://www.giantswarm.io/on-demand-webinar-kubernetes-hard-lessons-and-how-to-avoid-them?utm_campaign=Medium%20CTA%20Conversion&utm_source=Kubernetes%20hard%20lessons_Medium&utm_medium=Medium%20CTA&utm_term=Kubernetes%20hard%20lessons)

# 当前无根的限制

但是有一个问题。新映像通常是从基本映像(docker 文件中的`FROM`指令)构建的，其内容通常由 UID/GID=0 的用户拥有。当容器作为容器构建的一部分由非特权用户启动时，容器的文件由主机上的 UID/GID=0 拥有，而容器的进程将只拥有与非特权用户相关联的文件访问权限。这可能意味着容器的进程无法写入其文件系统，这将严重阻碍容器映像的构建。为了使映像中的文件在容器中拥有正确的所有权，UID/GID 集需要与用户名称空间映射一起“移位”。目前，没有实现这一点的最佳方法。

当一个无根的`podman build`被调用并且一个容器需要所有权“转移”时，文件系统内容被复制并且所有权被改变(chowned)以反映映射。就空间而言，这显然是低效的，并且耗费时间，这会严重影响容器构建所花费的时间长度。容器背后的一个重要想法是，多个容器和图像可以共享一个图像的内容，而不会重复。理想情况下，当一个容器的文件系统是由它的组成层组装而成时，这种转移应该在挂载操作中没有复制的情况下发生。

大多数容器运行时使用 overlayfs 来组装容器的文件系统，这不支持在其挂载上转移 UID/GID，但最近 Ubuntu 成为第一个支持覆盖的内核内机制(shiftfs)的 Linux 发行版，该机制已在 Linux 容器 [LXD 项目](https://discuss.linuxcontainers.org/t/lxd-3-12-has-been-released/4483)中投入使用。

对于 Podman 来说，部分补救措施是在 Linux 内核版本 4.19 中为 overlayfs 引入一个 mount 选项，该选项只将文件和目录的元数据复制到读/写层，而不是内容本身。然而，最终，社区正在等待主流 Linux 内核对 UID/GID 转换的支持，以便实现这个目标。

让我们继续讨论 Buildah，并解释它与`podman build`的关系和区别。

# Buildah

到目前为止我们还没有提到的是`podman build`在幕后使用 Buildah 来执行容器映像构建。这意味着无后台和无根构建也是 Buildah 的一个特性。与 Podman 不同，Buildah 有一个容器映像构建特定的函数，并且有许多特性超出了基于 Dockerfiles 构建映像的范围。

外界的大多数容器映像都是使用 docker 文件作为映像的不可变引用构建的。我们已经讨论了`podman build`如何使用 order 文件来构建映像，Buildah 也可以使用`buildah` bud 命令从 order 文件构建映像。但是再一次，Buildah 受到了寻找一种替代无处不在的 Dockerfile 的方法来构建容器映像的启发。另一种方法的基本原理是，所需要的只是一个代表符合 OCI 标准的映像的“捆绑包”,而如何实现最终目标并不需要 docker 文件。Buildah 的维护者坚持认为 Dockerfile 是一个需要规避的限制。

# Buildah 如何工作

尽管刻意希望从 docker 文件中独立出来，但 Buildah 使用了非常相似的过程来构建容器映像。一个`docker build`运行一个新的容器来处理每个 Dockerfile 指令，这导致在容器作为新的映像提交之前，创建新的或改变的内容或映像元数据。下一条指令在基于先前创建的映像的新容器中处理，然后作为新映像提交，依此类推。Buildah 做同样的事情，但是它执行 Buildah 子命令，而不是使用 Dockerfile 指令，并且在每个子命令执行后不需要“提交”。

构建过程可能从一个`buildah from`命令开始，这会导致一个基于指定为参数的图像的运行容器。这显然类似于`FROM` Dockerfile 指令。为了在一个容器中执行命令作为图像构建的一部分(例如，创建一个新用户，或者从其源构建一个工件)，图像作者可以利用`buildah run`，它可以在需要时进行交互。除了运行为容器图像创建内容的命令之外，Buildah 还提供了使用`buildah config`为图像定义元数据的方法。这使得诸如暴露的端口、默认用户、容器入口点等事情的规范成为可能。`buildah copy`和`buildah add`命令直接类似于`COPY`和`ADD` Dockerfile 指令，用于将外部内容获取到图像中。使用`buildah mount`，甚至可以将容器的根文件系统挂载到主机上合适的位置，以便随后使用主机本身自带的工具进行操作。

一旦图像作者确信他们已经完成了图像的制作，`buildah commit`命令将容器提交给新图像。

# 工作流程

Dockerfile 和`docker build`以及 Buildah 的组合有一些明显的相似之处。但是 Dockerfile 强制依赖指令的顺序执行，那么 Buildah 如何在容器构建中提供类似的顺序和可重复性呢？代替守护进程的构建引擎为`docker build`强加这个顺序，建议使用 Buildah 的容器构建应该以编程方式定义，例如使用 Bash 之类的东西。

上面的简单例子展示了如何在 Bash 脚本中使用 Buildah 实现可重复的构建。

在采用它所采用的映像构建方法时，Buildah 消除了对长时间运行的守护进程的任何依赖，并且进一步将映像构建器从 Dockerfile 语法的约束中解放出来。由 Buildah 创建的映像可以被推送到容器注册中心，然后由 Podman 或 Docker 守护进程提取，并将在支持 [OCI 规范](https://github.com/opencontainers/runtime-spec/blob/master/spec.md#open-container-initiative-runtime-specification)的容器运行时上无缝工作。

# 构建缓存和并行执行

如果图像是使用 Dockerfile 和`buildah bud`构建的，那么图像层将被缓存，以便在后续构建中重用。对于那些从 Docker 环境进入 Buildah 的人来说，这是预期的行为，可以显著提高构建执行的速度。但是，如果您希望在脚本中使用 Buildah 命令构建图像时可以使用缓存，那么您将会大吃一惊。缓存[不可用](https://github.com/containers/buildah/issues/1292)，这意味着所有的构建步骤都需要在每个新的构建迭代中执行，不管内容或命令是否发生了变化。

此外，Buildah 按顺序执行其构建步骤，即使一个构建步骤完全独立于另一个构建步骤。虽然包含并行构建步骤特性[已经被考虑过](https://github.com/containers/buildah/issues/1292)，但它目前在 Buildah 中不可用，这会进一步延长执行复杂容器映像构建所需的时间。

# 结论

虽然 Podman 和 Buildah 之间有明显的区别，但有两条路线可以实现相同的目标，这似乎有点令人困惑。如果有疑问，在使用 Dockerfiles 创作图像时应该使用`podman build`,如果 Dockerfile 语法被认为过于严格，或者倾向于使用类似脚本的方法来实现可重复性，则应该使用 Buildah。值得一提的是，容器图像和 Dockerfile 几乎是同义词，因此 Buildah 是否会在 Red Hat 社区之外获得足够的牵引力以最终篡夺 docker file 还有待观察。

Podman 和 Buildah 提供了两个最受欢迎的容器图像构建功能；无梦和无根的构建。然而，这些工具正在一个日益拥挤的空间中竞争，虽然现在还为时尚早，但它们确实缺乏类似工具目前提供的一些功能。

由[Puja Abbas si](https://twitter.com/puja108)——开发者倡议@ [巨型虫群](https://giantswarm.io/)撰写

[](https://twitter.com/puja108) [## Puja Abbassi @ #KubeCon #KubeKhan

### Puja Abbassi 的最新推文@ #KubeCon #KubeKhan (@puja108)。开发者关系和产品@GiantSwarm…

twitter.com](https://twitter.com/puja108)
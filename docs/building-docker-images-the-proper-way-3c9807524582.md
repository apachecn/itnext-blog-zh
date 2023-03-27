# 以正确的方式构建 Docker 图像

> 原文：<https://itnext.io/building-docker-images-the-proper-way-3c9807524582?source=collection_archive---------0----------------------->

## 让我们优化 Docker 构建，以在通常构建时间的一小部分内创建更小、更安全的 Docker 映像…

在这一点上，可能每个人都听说过 Docker，大多数开发人员都熟悉它，使用它，因此知道如何构建 Docker 映像等基础知识。它和运行`docker built -t name:tag .`一样简单，但是还有更多，特别是在优化构建过程和最终创建的映像时。因此，在本文中，我们将超越基础知识，看看我们如何影响 Docker 映像的构建过程，使其更快，并为我们的应用程序生成更精简、更安全的映像。

![](img/8e68c0d39e60eaa67b6aef38bf89a636.png)

照片由 [Timelab Pro](https://unsplash.com/@timelabpro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

# 用于快速构建的缓存

大部分映像构建时间通常来自系统库和应用程序依赖项的下载和安装。然而，它们并不经常改变，因此很适合缓存。

从系统库和工具开始——通常在初始`FROM`之后立即移动它们的安装就足够了，以确保它们被缓存。无论您使用哪个 Linux 发行版作为基础映像，您最终都会得到如下结果:

或者，你甚至可以把这些都提取到单独的`Dockerfile`中来构建你自己的基础图像。然后，可以将该图像推送到注册表，以便您和其他人可以在多个应用程序中重用它。这样你就不用担心任何系统库/依赖项，除非你需要升级它们或者添加/删除一些东西。

在系统库之后，我们通常希望安装应用程序依赖项。这些可能是存储在`.m2`目录中的来自 Maven 仓库的 Java 库、`node_modules`中的 JavaScript 模块或者`venv`中的 Python 库。这些会比系统依赖项更频繁地改变，但不足以保证每次构建都要完全重新下载和重新安装。但是，如果 Dockerfile 写得不好，您会注意到，即使没有修改依赖关系，也不会使用缓存:

这是为什么呢？嗯，问题出在`COPY . .`上，Docker 在构建的每一步都使用缓存，直到它运行到新的/修改过的命令/层。在这种情况下，当我们将所有内容复制到映像中时——包括*未更改的*依赖项列表以及*已修改的*源代码——Docker 只是继续重新下载并重新安装所有依赖项，因为由于修改了源代码，它无法再使用该层的缓存。为了避免这种情况，我们必须分两步复制文件:

首先，我们添加列出所有应用程序依赖项的文件并安装它们。如果没有对此文件进行任何更改，那么这些都将被缓存。只有这样，我们才能将剩余的(修改后的)源代码复制到映像中，并运行应用程序代码的测试和构建。

对于更多的*【高级】*方法，我们使用 Docker 的*构建工具包*和它的实验特性做同样的事情:

上面的代码展示了我们如何使用`RUN`命令的`--mount`选项来选择缓存目录。如果您希望显式使用非默认缓存位置，这可能会很有帮助。如果您想使用这个特性，您将不得不包含指定语法版本的标题行(如上所述),并使用`DOCKER_BUILDKIT=1 docker build name:tag .`运行构建。关于实验特征的更多信息可以在这些[文档](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#run---mounttypecache)中找到。

到目前为止，所说的都只适用于本地构建——对于 CI 来说，这是一个不同的故事，一般来说，对于每个工具/提供商来说都是不同的，但是对于它们中的任何一个，您都需要一些持久的卷来存储缓存/依赖关系。例如对于[詹金斯](https://www.jenkins.io/doc/book/pipeline/docker/#caching-data-for-containers)，你可以在代理中使用存储。对于在 Kubernetes 上运行的 Docker 构建版本——无论是使用 *JenkinsX* 、 *Tekton* 还是其他——你都需要 Docker 守护进程，可以使用 Docker (DinD) 中的 *Docker 来部署，这是在 Docker 容器中运行的 Docker 守护进程。至于构建本身，您将需要一个连接到 *DinD* 插座并运行`docker build`的 pod(容器)。*

出于演示目的，为了简单起见，我们可以使用以下 pod:

上述 pod 由两个容器组成——一个用于 *DinD* 和一个用于*图像生成器*。要使用 builder 容器运行构建，您可以访问它的 shell，克隆一些存储库并运行 build:

最后的`docker build`使用了一些新的选项——一个是`--cache-from image:tag`,它告诉 Docker 应该使用来自(远程)注册表的指定图像作为缓存源。这样，即使缓存的层没有存储在本地文件系统上，我们也可以利用缓存。另一个选项——`--build-arg BUILDKIT_INLINE_CACHE=1`——用于在创建映像时将缓存元数据写入映像。这必须用于`--cache-from`的工作，更多信息在[文档](https://docs.docker.com/engine/reference/commandline/build/#specifying-external-cache-sources)中。

# 给他们减肥

拥有快速构建固然很好，但是如果你有真正的*“厚”*映像，拉和推它们仍然需要很长时间，并且胖映像很可能还包含许多库、工具等等，这使得映像更容易受到攻击，因为它创建了更大的攻击面。

制作更薄图像的最简单的方法是使用类似 Alpine Linux 的东西，而不是基于 Ubuntu 或 RHEL 的图像。另一个好方法是使用多步 Docker 构建，其中您使用一个映像来构建(第一个`FROM`命令)和不同的、更精简的映像来运行应用程序(第二个/最后一个`FROM`)，例如:

上面显示，我们首先在基本 Python 3.8.7 映像中准备应用程序及其依赖项，该映像相当大，有 332.88 MB。在这个映像中，我们安装了应用程序所需的虚拟环境和库。然后我们切换到更小的基于 Alpine 的图像，只有 16.98 MB。我们将之前创建的整个虚拟环境以及源代码复制到这个映像中。通过这种方式，我们可以得到更小的图像，更少的图层，更少的不必要的工具和二进制文件。

另外要记住的一件事是我们在每次构建时产生的层数。`FROM`、`COPY`、`RUN`和`CMD`是创建层的四个命令，至少在`RUN`的情况下，我们可以通过`&&`将所有`RUN`命令合并成一个命令来轻松减少它创建的层的数量，如下所示:

我们可以把它推得更远，完全去掉可能相当厚的基础图像。为此，我们将使用特殊的`FROM scratch`来通知 Docker 应该使用最小的基本图像，并且下一个命令将是最终图像的第一层。这对于以二进制形式运行且不需要大量工具运行应用程序尤其有用，例如 Go、C++或 Rust 应用程序。然而，这种方法要求二进制文件是静态编译的，因此不适用于 Java 或 Python 这样的语言。一个`FROM scratch` Dockerfiles 的例子可能是这样的:

很简单，对吧？有了这种 Dockerfile 文件，我们可以制作出仅在 3MB 周边的图像！

# 锁定事物

速度和大小效率是大多数人会关注的两件事，而图像的安全性成为事后的想法。有一些简单的方法可以锁定图像中的内容，并限制对手的攻击面。

最基本的建议是锁定所有库、包、工具和基本映像的版本，这不仅对安全性很重要，对映像的稳定性也很重要。如果你对图像使用`latest`标签，或者你没有在 Python 的`requirements.txt`或 JavaScript 的`package.json`中指定版本，你就有可能在构建期间下载的图像/库可能与应用程序代码不兼容，或者它可能暴露你的容器的漏洞。虽然您希望将所有内容锁定到特定版本，但您也应该定期更新所有这些依赖项，以确保您拥有所有最新的安全补丁和修复程序。

即使您非常努力地试图避免所有依赖项中的任何漏洞，仍然会有一些您遗漏了或者尚未修复/发现的漏洞。因此，为了减轻任何可能攻击的影响，最好避免像`root`那样运行容器。因此，您应该在 docker 文件中包含`USER 1001`,以表明从您的 docker 文件创建的容器应该并且可以作为非根用户(理想情况下是任意用户)运行。当然，这可能需要您修改您的应用程序并选择正确的基础映像，因为一些常见的基础映像如`nginx`需要 root 权限(例如，因为特权端口)。

通常很难找到/避免 Docker 映像中的漏洞，但是如果映像只包含运行应用程序所需的最少部分，那么就容易多了。谷歌制作的[发行版](https://github.com/GoogleContainerTools/distroless)就是这样一张图片——或者说是一组图片。Distroless 映像被精简到甚至没有外壳或包管理器，这使得它们在安全性方面比 Debian 或基于 Alpine 的映像好得多。如果你使用多步 Docker 构建，那么大多数时候，切换到 distrolles*runner*镜像就像这样简单:

除了最终图像及其容器中可能存在的漏洞，我们还必须考虑我们用来构建图像的 Docker 守护进程和容器运行时。因此，与我们所有的图像一样，我们不应该允许 Docker 与`root`用户一起运行，而是使用所谓的*无根*模式。在 [Docker docs](https://docs.docker.com/engine/security/rootless/) 中有关于如何设置的完整指南，但是如果你不想弄乱这种配置，那么你可能想要考虑切换到默认情况下在*无根*和*无域*模式下运行的`podman`——在我这里的另一篇文章中有更多关于它的内容:

[](https://towardsdatascience.com/its-time-to-say-goodbye-to-docker-5cfec8eff833) [## 你不用再用 Docker 了

### Docker 不是唯一的集装箱工具，可能会有更好的替代工具…

towardsdatascience.com](https://towardsdatascience.com/its-time-to-say-goodbye-to-docker-5cfec8eff833) 

# 结论

容器和 Docker 已经存在了足够长的时间，每个人除了学习和使用最少的可用特性外，几乎没有什么东西。这篇文章中的提示和例子应该(在我看来)是一个很好的垫脚石，可以增加你的 Docker 知识，改善你使用的 Docker 图片的各个方面。然而，除了构建 Docker 图像之外，我们还可以做更多的事情来改进我们使用图像和容器的方式。这包括例如应用`seccomp`政策(下面是我的文章)👇)，使用`cgroups`或者可能使用完全不同的容器运行时/引擎来限制资源消耗。

*本文最初发布于*[*martinheinz . dev*](https://martinheinz.dev/blog/42?utm_source=tds&utm_medium=referral&utm_campaign=blog_post_42)

[](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [## 用 seccomp 强化 Docker 和 Kubernetes

### 您的容器可能不像您想象的那样安全，但是 seccomp 配置文件可以帮助您解决这个问题…

itnext.io](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [](https://towardsdatascience.com/networking-tools-every-developer-needs-to-know-e17c9159b180) [## 每个开发人员都需要知道的网络工具

### 让我们学习被忽视的网络技能，如检查 DNS 记录，扫描端口，排除连接故障…

towardsdatascience.com](https://towardsdatascience.com/networking-tools-every-developer-needs-to-know-e17c9159b180) [](https://towardsdatascience.com/deploy-any-python-project-to-kubernetes-2c6ad4d41f14) [## 将任何 Python 项目部署到 Kubernetes

### 是时候深入 Kubernetes，使用这个成熟的项目模板将您的 Python 项目带到云中了！

towardsdatascience.com](https://towardsdatascience.com/deploy-any-python-project-to-kubernetes-2c6ad4d41f14)
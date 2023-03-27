# 用 seccomp 强化 Docker 和 Kubernetes

> 原文：<https://itnext.io/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111?source=collection_archive---------0----------------------->

## 您的容器可能不像您想象的那样安全，但是 seccomp 配置文件可以帮助您解决这个问题…

关于容器安全性有很多误解——很多人认为容器在默认情况下是安全的，但不幸的是这并不正确。有相当多的工具可以帮助你提高容器的安全性，从而提高 Docker 和 Kubernetes 的安全性。强化它们的方法之一是应用适当的`seccomp`轮廓。如果您不知道`seccomp`是什么，那么请继续阅读，看看它是什么，以及如何使用它来保护您的 Docker 和 Kubernetes 免受安全威胁！

![](img/9d39e65576e5f8796ee9d5aea8233f43.png)

照片由 [mana5280](https://unsplash.com/@mana5280?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

# seccomp 到底是什么？

如果你使用 Docker 或 Kubernetes 已经有一段时间了，你可能听说过术语`seccomp`，但是很有可能，你还没有真正深入了解这个晦涩的工具，对吗？

最简单和最容易理解的`seccomp`定义可能是*“系统调用防火墙”*。`seccomp`本质上是一种限制进程进行系统调用的机制，因此，就像我们可以阻止来自某些 IP 的数据包一样，我们也可以阻止进程向 CPU 发送系统调用。

这很酷，但这如何帮助我们使系统更加安全呢？Linux 内核有很多系统调用(几百个)，但是大多数都不是任何给定进程所需要的。如果进程受到威胁并被骗去使用这些系统调用，那么它会导致整个系统出现严重的安全问题。因此，限制系统调用进程可以大大减少内核的攻击面。

现在，如果你正在运行 Docker 的最新版本(1.10 或更高版本)，那么你已经在使用`seccomp`。您可以使用`docker info`或查看`/boot/config-*`进行检查:

嗯，太好了！我们已经有了`seccomp`，一切都很安全，我们不需要搞乱这些，对吗？嗯，不完全是这样——有相当多的理由让你想把手弄脏并更深入地研究`seccomp`...

# 为什么要创造一个呢？

Docker 中的默认`seccomp`配置文件可能通常是*“足够好”*，如果您没有使用它的经验，那么*“改进”*或以其他方式定制它可能不是对您时间的最佳利用。然而，有一些原因可能会给你足够的动力来改变默认的`seccomp`配置文件:

不时会发现安全漏洞。这是不可避免的，进一步限制默认`seccomp`配置文件可能会降低安全漏洞影响您的应用程序的机会。一个这样的事件是 [CVE 2016-0728](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-0728) ，该事件允许用户使用`keyctl()` syscall 提升权限。这次事件之后，这个系统调用现在被默认`seccomp`配置文件阻止，但是将来可能会有更多这样的漏洞...

限制`seccomp`概要文件的另一个原因是，如果某个子系统有安全缺陷，攻击者将无法从您的容器中利用它，因为相关的系统调用已经被阻塞。这个子系统可以是依赖项、库或系统中你无法控制的部分。您可能也没有办法修复该子系统中的问题，为了防止该问题被利用，您需要在不同的层阻止它，例如在`seccomp`配置文件中。

当您处理 PII 数据或构建任务关键型应用程序(如医疗保健或电网中使用的软件)时，潜在的漏洞和安全漏洞变得更加重要。在这些情况下，任何微小的安全改进都很重要，定制`seccomp`配置文件可能是值得的。

除了所有这些特定的原因，限制容器可以使用的系统调用限制了可以使用的攻击媒介，例如 Docker 上游映像中的后门或应用程序中可利用的 bug。

# 自定义配置文件

现在，如果我们确定我们有一个很好的理由去改变`seccomp`档案，我们实际上该如何去做呢？编写`seccomp`过滤器的一种方法是使用 BPF 语言。使用这种语言并不简单或方便。幸运的是，我们不必使用它——相反，我们可以编写 JSON，由`libseccomp`编译成 profile。此类配置文件的简单/最小示例如下所示:

通常，最好使用白名单而不是黑名单，并明确列出允许的系统调用和禁止的其他系统调用。这正是上面的配置文件所做的——默认情况下，它将使用动作`SCMP_ACT_ERRNO`,这将导致所有系统调用的`Permission denied`。对于我们想要允许的，我们列出它们的名字并指定`SCMP_ACT_ALLOW`动作。

这些 JSON 概要文件可以使用相当多的选项，并且可能会变得非常复杂，所以上面的概要文件实际上是将其精简到最低限度。要了解真实的简介，你可以点击这里查看码头工人简介[。](https://github.com/docker/labs/blob/master/security/seccomp/seccomp-profiles/default.json)

现在我们已经对`seccomp`概要文件有了更多的了解，让我们来玩一下 Docker。在我们尝试应用任何自定义概要文件之前，让我们先做一点实验，覆盖`seccomp`的默认设置:

上面我们可以看到如果我们完全禁用`seccomp`会发生什么——任何系统调用都是可用的，这允许容器中的用户——除了别的以外——重启主机。本例中的`— security-opt seccomp=…`用于禁用`seccomp`。这也是应用自定义概要文件的方法，所以让我们构建并应用我们自己的自定义概要文件。

从头开始并不是一个好主意，所以我们将修改现有的 Dockers 配置文件(如上所述)，我们将通过删除`chmod`、`fchmod`和`fchmodat` syscall 对其进行一点限制，这实际上拒绝了使用`chmod`命令的权限。这可能不是最合理的改变或*“改进”*但出于演示目的，它工作得很好:

在这个代码片段中，我们从`no-chmod.json`加载自定义概要文件，并尝试`chmod`整个`/etc`。我们可以看到这在容器中导致了什么——任何运行`chmod`的尝试都会导致`Operation not permitted`。

有了这个简单的 *"no* `*chmod*` *"* 配置文件，挑选应该阻塞哪些系统调用就变得非常简单。然而，总的来说，选择什么可以或不可以被阻止并不容易。您可以做出有根据的猜测，但是您可能会因为阻塞系统调用而同时阻塞太多和不够，这将阻止您的应用程序正确运行，而且还会错过一些可以并且应该被阻塞的系统调用。

选择阻塞哪个系统调用的唯一可行方法是调用跟踪。跟踪系统调用的一种方法是使用`strace`:

上面的命令给出了一个命令使用的所有系统调用的列表(在这个例子中是`chmod`，从中我们可以选择阻止/允许什么。如果这个`strace`命令对你来说有点太复杂，那么`strace -c ls`也可以工作，它输出一些额外的信息，比如时间和通话次数。我们显然不能阻止所有这些，因为那会使我们的容器不可用，所以最好是查找每一个，例如使用这个 [syscall 表](https://filippo.io/linux-syscall-table/)。

# Kubernetes 怎么样？

在标题和介绍中，我也提到了 Kubernetes，但是到目前为止我们只讨论了 Docker。那么，Kubernetes 世界`seccomp`周围是什么情况呢？不幸的是，在 Kubernetes `seccomp`中，默认情况下不使用，因此系统调用不会被过滤，除了一些非常危险的调用。因此，使用 Docker，您可以不配置任何配置文件，只使用默认值，但在 Kubernetes 中，确实无法防止一些安全问题被利用。

我们如何“解决”这个问题？这取决于您使用的 Kubernetes 版本——对于 1.19 之前的版本，您需要对您的 pod 应用注释。这看起来会像这样:

对于 1.19(我们将在这里重点介绍)和更高版本，`seccomp`配置文件是 GA 特性，您可以在 pod 的`securityContext`中使用`seccompProfile`部分。此类 pod 的定义如下所示:

现在我们知道如何在 Kubernetes 上解决它，所以是时候做一个小小的演示了。为此，我们需要一个集群——这里我将使用*种类*(Docker 中的 Kubernetes)来建立一个最小的本地集群。此外，在启动集群之前，我们还需要所有的`seccomp`配置文件，因为这些文件需要在集群节点上可用。因此，集群本身的定义如下:

这定义了将本地`profiles`目录挂载到`/var/lib/kubelet/seccomp/profiles`节点的单节点集群。我们在这个`profiles`目录中放什么呢？出于此示例的目的，我们将使用 3 个配置文件:Dockers 默认配置文件和以前使用的配置文件，另外还有一个所谓的*“审计”* / *“投诉”*配置文件。

我们将从审计概况开始。这个不允许/阻止任何系统调用，而是当它们被一些命令/程序使用时，将它们记录到`syslog`日志中。这对于调试和探索应用程序的行为以及发现可以或不可以阻塞的系统调用非常有用。这个概要文件的定义实际上只有一行，如下所示:

显然，我们还需要一个豆荚。这个 pod 将有一个 Ubuntu 容器，它运行`ls`来触发一些系统调用，然后休眠，所以它不会终止和重启:

解决这个问题后，让我们构建集群并应用第一个概要文件:

我们在带有我们的*种类*集群定义、`pods`目录和`profiles`目录的目录中执行上面的例子。我们首先使用`kind`命令从定义中创建集群。然后，我们应用使用`audit.yaml`配置文件的`audit-seccomp` pod。最后，我们检查`syslog`消息，查看`audit-seccomp` pod 记录的审计消息。这些消息中的每一条都包含`syscall=...`，它指定了 syscall ID(对于`x86_64`架构)，可以翻译成每行末尾注释中的名称。

既然我们已经确认了`seccomp`如预期的那样工作，我们可以应用真实的概要文件(Dockers 默认)。为此，我们将使用不同的 pod:

我们在这里做了一些改变。也就是说，我们更改了指定`RuntimeDefault`类型的`seccompProfile`部分，我们还将图像更改为[一个包含的](https://github.com/genuinetools/amicontained)部分，这是一个容器自检工具，它会告诉我们哪些系统调用被阻塞，以及一些其他有趣的安全信息。

应用此 pod 后，我们可以在日志中看到以下内容:

这表明 Dockers 默认配置文件将阻塞上述 61 个系统调用。如果我们不包括类型为`RuntimeDefault`的`seccompProfile`部分，它将只有 22(如果你不相信我，你可以自己测试😉).在我看来，这是很大的安全改进，只需要很少的实际努力。

如果我们认为缺省值不够好，或者我们需要做一些修改，我们可以部署我们的定制概要文件。在这里，我们将使用*“无*`*chmod*`*”*配置文件和以下 pod 进行演示:

此窗格与之前显示的*“审计”*窗格非常相似。我们只是将`localhostProfile`切换为指向不同的文件，并将容器参数更改为包含`chmod`命令，这样我们就可以看到并确认我们修改后的`seccomp`概要文件如预期那样工作:

日志显示了预期的结果— `seccomp`概要文件阻止了对测试文件进行`chmod`的尝试，并返回了`Operation not permitted`。这表明，根据我们的喜好或需要调整配置文件是非常简单的。

但是在修改这些类型的默认配置文件时要小心——如果您最终阻塞过多，并且您的 pod/container 无法启动，那么 pod 将处于`Error`状态，但是您将在日志中看不到任何内容，在`kubectl describe pod …`的事件中也看不到任何有用的内容，所以在调试`seccomp`相关问题时要记住这一点。

# 结论

即使读完这篇文章，修改或创建自己的`seccomp`个人资料可能也不是你的首要任务之一。然而，重要的是要意识到这个强大的工具，并能够在需要时使用它——例如在 Kubernetes 中——默认情况下它没有被强制执行，这很容易成为大的安全问题。出于这个原因，我建议——至少——启用*“审计”*概要文件，这样您就可以监视正在使用的系统调用，并使用这些信息来创建您自己的概要文件，或者验证默认的概要文件是否适用于您的应用程序。

此外，如果你从这篇文章中得到什么，那么可能是，`seccomp`是一个重要的安全层，你应该*永远不要*运行你的容器`uncontained`。

*本文原帖*[*martinheinz . dev*](https://martinheinz.dev/blog/41?utm_source=medium&utm_medium=referral&utm_campaign=blog_post_41)

[](https://towardsdatascience.com/analyzing-docker-image-security-ed5cf7e93751) [## 分析 Docker 图像安全性

### 码头集装箱远没有你想象的那么安全…

towardsdatascience.com](https://towardsdatascience.com/analyzing-docker-image-security-ed5cf7e93751) [](https://towardsdatascience.com/its-time-to-say-goodbye-to-docker-5cfec8eff833) [## 你不用再用 Docker 了

### Docker 不是唯一的集装箱工具，可能会有更好的替代工具…

towardsdatascience.com](https://towardsdatascience.com/its-time-to-say-goodbye-to-docker-5cfec8eff833) [](https://towardsdatascience.com/deploy-any-python-project-to-kubernetes-2c6ad4d41f14) [## 将任何 Python 项目部署到 Kubernetes

### 是时候深入 Kubernetes，使用这个成熟的项目模板将您的 Python 项目带到云中了！

towardsdatascience.com](https://towardsdatascience.com/deploy-any-python-project-to-kubernetes-2c6ad4d41f14)
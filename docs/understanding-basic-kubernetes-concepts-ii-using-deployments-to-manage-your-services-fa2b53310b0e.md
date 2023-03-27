# 理解基本的 Kubernetes 概念 II——使用部署以声明方式管理您的服务

> 原文：<https://itnext.io/understanding-basic-kubernetes-concepts-ii-using-deployments-to-manage-your-services-fa2b53310b0e?source=collection_archive---------4----------------------->

[![](img/32c9166eaf4ea29c6590af2813c4372f.png)](http://www.giantswarm.io)

*这篇文章是一系列关于 Kubernetes 基本概念的博客文章中的第二篇。在第一篇中，我* [*解释了 pod、标签和副本集*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-i-introduction-to-pods-labels-replicas/) *的概念。在本帖中，我们将讨论部署。第三篇文章* [*解释了服务概念*](https://blog.giantswarm.io/basic-kubernetes-concepts-iii-services-give-abstraction/) *，第四篇文章我们来看看* [*秘密和配置图*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-iv-secrets-and-configmaps/) *。在第五篇也是最后一篇文章中，我们将讨论* [*守护进程集和*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-v-daemon-sets-and-jobs/) *任务集。*

你可能听过(或读过)我以前说过，在 Kubernetes 社区，有时很难找到做事情的最佳方式。在服务/应用程序部署方面，这导致人们使用 pod 手动部署容器，然后在顶部添加复制控制器以保持其活动并扩展它们，然后在某个时候使用`kubectl rolling-update`进行更新。这至少在某种程度上是管理软件的必要形式。它通常涉及手工工作/命令，如果你在一个团队中工作的话，可能会非常不透明。

因此，在 Kubernetes 1.2 中添加了声明性的*部署*对象。然而，环顾四周，很少有资源提到它以及为什么应该使用它。当然，它包含在[官方文件](https://kubernetes.io/docs/user-guide/deployments/)中，并且在 4 月 1 日有一篇关于它的很好的[博客文章](http://blog.kubernetes.io/2016/04/using-deployment-objects-with.html)(也许人们认为这是一个笑话)，但是我没有看到很多人使用它，甚至更少在他们的博客文章和例子中明确提到它。所以我想写一篇小文章，谈谈为什么我认为你应该使用部署。

*旁注:在撰写本文时，大多数 Kubernetes 安装的系统服务或插件(如 DNS、Dashboard、Metrics)仍然部署有复制控制器，但一些* [*官方插件*](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons) *已经被转移到部署中。监控已经完成了迁移，我们几天前刚刚合并了一个 PR，将仪表板迁移到部署。在 Giant Swarm 上，所有这些系统服务已经作为部署运行。*

# 陈述性方式与命令性方式

部署的主要论点是在部署和管理软件的声明性和命令性方式之间的一般性争论。它不仅适用于应用程序级别，也适用于基础设施级别(但这是另一篇文章要考虑的)。*注意:如果您已经确信声明式管理是一条可行之路，那么您可以跳过这一章，直接进入下面的 Kubernetes 部署概念。*

# 必要的方式

命令式的方法通常是你用来尝试一些事情并得到一个工作系统的方法。你手动调整它，在某一点上它是你喜欢的。然而，如果您继续使用这种命令式风格来部署和管理您的软件，您将会遇到几个问题(即使您自动化了这些步骤)。

*   如果您想知道最后部署了什么，您只能求助于检查当前部署的版本，这不一定是计划部署的版本。
*   如果您手动更改某些内容，其他人可能会覆盖您的更改，或者它们可能会被恢复。例如，因为您只手动启动了新的 pod，而没有更改复制控制器。
*   跟踪一个服务是如何随着时间的推移而发展的是很困难的，需要在某个地方单独记录。这在团队工作中尤为重要。
*   您需要几个命令来使一个服务达到期望的状态。即使这些步骤实现了自动化，当在不同的环境中运行时，它们也可能导致不同的结果。

# 陈述的方式

另一方面，声明性的方法是你在生产中实际部署和管理软件或者与连续交付管道集成时应该想到的方法。你可能以前尝试过命令式的方法，但是一旦你知道它应该是什么样子，你就坐下来，通过把它写成一个声明式的定义来“使它正式化”。这就避免了上述问题，甚至带来了一些额外的好处。

*   它让你思考并计划你希望事情运行后的样子(计划状态)。
*   它描述的是运行时事情应该是什么样子，而不是达到目的所需的步骤(定义期望的状态，而不是过程)。
*   通过跟踪(或版本化)声明性定义，可以很容易地记录变更，这使得团队中的工作和交流更容易(但对于单个开发人员来说也是一种好的/干净的方法)。

[![](img/bc0006a78be67725ad94f93830d17bdb.png)](https://www.giantswarm.io/guide-cloud-native-stack?utm_campaign=Blog%20CTA%20Conversion&utm_source=Cloud%20native%20stack%20guide_Blog&utm_medium=Blog%20CTA&utm_term=cloud%20native%20stack%20guide)

# 在 Kubernetes 的部署

如上所述，部署资源是相当新的，仍然没有被广泛使用。然而，目前它是我们在 Kubernetes 中部署和管理软件的最好的原语，所以理解它的作用和用途是很重要的。

在部署之前，有复制控制器，它管理 pod 并确保一定数量的 pod 在运行。现在，随着部署的进行，我们转向副本集，这基本上是下一代复制控制器。只是这次我们不管理它们，而是由我们定义的部署来管理它们。因此，这个链如下所示:部署->副本集-> Pod。我们只需要解决第一个问题。

除了复制控制器(或[副本集](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-i-introduction-to-pods-labels-replicas/))提供的功能之外，部署还为您提供了对用于部署的更新策略的声明式控制。这取代了旧的`kubectl rolling-update`更新方式，但在定义`maxSurge`和`maxUnavailable`方面提供了相同的灵活性，即允许多少额外的和多少不可用的 pod。在部署中定义这一点使您能够“一次使用多次”，这在团队工作或管理大量部署时更有帮助。

部署会为您管理更新。他们甚至去检查正在推出的新版本是否正常工作，如果不正常，就停止推出。

您还可以定义一个等待时间(`minReadySeconds`)，在认为一个 pod 可用之前，它需要在没有任何容器崩溃的情况下准备好，这也有助于防止“坏的更新”，并为某些容器提供更多的时间来准备流量。

您可以进一步使用部署的更新/修订功能来同时部署部署的多个修订。这支持蓝/绿部署或金丝雀释放策略。

此外，部署会保留修订的历史记录(在回滚情况下使用)以及事件日志(可用于审计部署的发布和更改)。

# 部署入门

阅读所有关于部署的很酷的东西，您可能想亲自看看。

由于部署(以及它们的副本集)是复制控制器的一种继承者，因此很容易将您已经拥有的复制控制器更改为部署。你只需要改变:

到

并且在`selector:`行之后添加包含`matchLabels:`的一行(在实际的选择器之前，不要忘记修复缩进)，因为部署资源支持[基于集合的标签需求](https://kubernetes.io/docs/user-guide/labels/#resources-that-support-set-based-requirements)。

这将为您提供一个可行的部署。由于部署比其前身具有更多功能，因此还有一些您尚未定义的字段。幸运的是，这些字段在创建时会自动填充默认值。

然而，正如我们在上面了解到的，当部署到不同的集群时，这可能会导致不同的结果。有时这可能是有意的，但如果不是，您可以看看在使用`kubectl edit deployment <deployment name>`将部署应用到集群时得到了什么，然后将它(部分)复制回您的清单。

然而，正确的方法应该是通过阅读[关于部署的官方文档](https://kubernetes.io/docs/user-guide/deployments/)来学习部署的正确用法，然后编写一个适合您的清单。(您仍然可以从“翻译”您现有的复制控制器清单开始)。Kubernetes 在 4 月 8 日发布的博客文章也是一个很好的阅读材料，它用一个很好的例子展示了更新和回滚的工作方式。

享受探索部署的乐趣，并期待下一篇基本概念帖子！

由 [Puja Abbassi](https://twitter.com/puja108) 撰写:开发者拥护者@ [巨型蜂群](https://twitter.com/giantswarm)
# 理解 Kubernetes 的基本概念 III——服务给你抽象

> 原文：<https://itnext.io/understanding-basic-kubernetes-concepts-iii-services-give-you-abstraction-efd80db2465c?source=collection_archive---------9----------------------->

[![](img/1776925bb7f062b84d7d9141c9ec928a.png)](http://www.giantswarm.io)

这篇文章是一系列关于 Kubernetes 基本概念的博客文章中的第三篇。在第一篇中，我 [*解释了 pod、标签和副本集*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-i-introduction-to-pods-labels-replicas/) *的概念。在第二篇文章中，* [*我们谈到了部署*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-using-deployments-manage-services-declaratively/) *。这篇文章将详细阐述服务的概念。第四，我们看看* [*秘密和配置图*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-iv-secrets-and-configmaps/) *。在第五篇也是最后一篇文章中，我们将讨论* [*守护进程集和*](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-v-daemon-sets-and-jobs/) *作业集。*

到目前为止，您应该对 Kubernetes 的一些基本原语有了基本的理解。但是，还是有一些概念缺失。其中一个非常核心的问题是，特别是在使用微服务时，在 pod(和其他服务)上设置一个抽象层，这样您就可以与它们通信，而不必跟踪每个 pod 的开始、结束和重新安排。它是 Kubernetes 中服务发现的基本构件。

# 输入服务

正如在本系列的第一篇博客文章中提到的，pod 是短暂的，必然会被副本集(或复制控制器)动态地终止和启动。因此，与 pod 或其组通信需要抽象出短暂 pod 的概念。

这就是服务的用武之地。它们是一个基本概念，在使用微服务架构时特别有用，因为它们将各个服务相互分离。例如，访问服务 B 的服务 A 不知道 B 中实际在做什么样的工作，也不知道有多少工作单元，实际的工作单元甚至它们的实现都可能完全改变。

# 服务如何工作

服务通过定义一组逻辑单元和访问它们的策略来工作。pod 的选择基于标签选择器(我们在第一篇博文中谈到过)。如果您选择多个 pod，该服务会自动进行负载平衡，并为它们分配一个(虚拟)服务 IP(您也可以手动设置)。

您可以使用选择器选择一组窗格，并定义一个`targetPort`来访问它们。进一步帮助抽象，这个`targetPort`也可以指一个端口的名称，这给你更多的自由来实现服务背后的实际 pods。甚至每个 pod 的端口也可以不同，只要它们具有相同的名称。

此外，服务可以抽象出不是 Kubernetes pods 的其他种类的后端。例如，您可以抽象出一个服务背后的外部数据库集群。例如，通过这种方式，您可以在开发环境中使用简单的本地数据库，在生产环境中使用专业管理的数据库集群，而不必改变其他服务与该数据库服务的通信方式。

如果您的一些工作负载在您的 Kubernetes 集群之外运行，即在另一个 Kubernetes 集群(或[命名空间](http://kubernetes.io/docs/user-guide/namespaces/))上运行，或者甚至完全在 Kubernetes 之外运行，您也可以使用相同的功能。如果您刚刚开始迁移工作负载，后者尤其有趣。

# 与服务对话

您可以通过环境变量或通过 [DNS](https://github.com/kubernetes/kubernetes/tree/release-1.2/cluster/addons/dns) 来发现集群中的服务并与之对话。后者是一个集群插件，在大多数 Kubernetes 安装(包括巨型 Swarm)中都有 OOTB。

[![](img/07afe8e3b2d49509a8dcf7a8238cefc0.png)](https://www.giantswarm.io/guide-cloud-native-stack?utm_campaign=Blog%20CTA%20Conversion&utm_source=Cloud%20native%20stack%20guide_Blog&utm_medium=Blog%20CTA&utm_term=cloud%20native%20stack%20guide)

如果您希望使用另一种类型的服务发现，并且不希望该服务提供负载平衡和单一服务 IP，可以选择创建一个所谓的“headless”服务。如果您已经在使用服务发现或者想要减少与 Kubernetes 系统的耦合，那么您可以使用这个。

# 开始

现在你对服务的概念有了更多的了解，你应该在官方文档[中仔细阅读它们的用法。然后继续构建一些通过服务相互对话的](http://kubernetes.io/docs/user-guide/services/)[部署](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-using-deployments-manage-services-declaratively/)。

提示:如果您有一个服务和一个部署，您通常应该先启动服务，然后再启动部署。您甚至可以通过将两个清单包含在一个 YAML 文件中，用一个命令来部署它们。你只需要用一条包含`---`的线把它们分开。Kubernetes 将一个接一个地创建它们。

由 [Puja Abbassi](https://twitter.com/puja108) : 开发者拥护者@ [巨型虫群](https://twitter.com/giantswarm)撰写
# Kubernetes:人工智能和人工智能实验的虚拟集群

> 原文：<https://itnext.io/kubernetes-virtual-clusters-for-ai-ml-experiments-94309422b060?source=collection_archive---------3----------------------->

虚拟 Kubernetes 集群可以解决人工智能中一些最紧迫的技术挑战。

![](img/c33d1a72d3a8ac812e481a0433b0516c.png)

人工智能(AI)和机器学习(ML)是近年来最热门的 IT 话题。最近的一项 O'Reilly 调查发现，85%的公司已经在使用或正在评估人工智能。当然，如此令人印象深刻的大多数公司都在使用人工智能和人工智能有一个很好的理由:[根据麦肯锡的一项调查，63%的公司能够通过人工智能增加收入，44%的公司能够降低成本](https://www.mckinsey.com/featured-insights/artificial-intelligence/global-ai-survey-ai-proves-its-worth-but-few-scale-impact)。这使得人工智能的高效实施成为一个重要的竞争优势。

# 人工智能采用挑战

尽管如此，许多公司在他们的人工智能采用之旅中面临一些挑战，这是奥赖利的[调查:三大问题之一是缺乏熟练的人工智能/人工智能人才。为此，人们可以得出结论，现有工程师的生产力也是有效采用人工智能的一个重要因素。](https://www.oreilly.com/radar/ai-adoption-in-the-enterprise-2020/)

在这里，技术方面也发挥了作用，其中两个甚至在 10 大主要挑战中提到:工作流再现性和技术基础设施。

虚拟 Kubernetes 集群(vClusters)可以直接应对工程效率、工作流可再现性和技术基础设施这三大挑战，我将在接下来介绍这三大挑战。

> *在另一篇文章中，我们描述了* [*什么是虚拟 Kubernetes 集群以及它们如何工作*](https://loft.sh/blog/introduction-into-virtual-clusters-in-kubernetes/) *。*

# 1.工程生产率

## 挑战

鉴于市场上许多开放的人工智能职位缺乏合适的候选人，可用工程师的生产力应该是最重要的。这意味着工程师应该能够在软件上工作，而不应该浪费时间，例如等待或管理软件的执行环境。

## 虚拟集群的影响

**可用性:**有了虚拟 Kubernetes 集群，工程师们手头总是有一个单独的执行环境，可以按需在几秒钟内启动。这意味着工程师不必等到管理员为他们创建这样的环境，或者等到同时使用共享中央测试集群的同事的测试完成。

**可扩展性:**通常情况下，人工智能和机器学习软件需要大量的计算资源。借助 vClusters，人工智能工程师在需要时总能获得计算资源，这也加快了实验的执行速度。Kubernetes 本身提供的这种可伸缩性甚至在开发的早期阶段就可供工程师使用，而不仅仅是在以后的生产中。

# 2.工作流程再现性

## 挑战

测试和实验需要执行多次，通常参数略有不同。为了获得有意义的结果并保持高效率，启动这些实验的工作流程应该简单、快速且不易出错。

## 虚拟集群的影响

**可恢复性:**因为 Kubernetes 本身是声明性的，所以重新创建相同的 Kubernetes 环境相对容易。虚拟 Kubernetes 集群当然也是如此。借助 vClusters，可以更轻松、更快速地创建完全相同规格的全新环境。

**并行化:**使用[虚拟集群平台](https://loft.sh/)，工程师通常被允许自己创建多个 v cluster，而无需拥有物理集群的集群管理权限。为此，工程师甚至可以并行运行实验，这加快了反馈周期(并再次有助于工程生产率和速度，[参见挑战 1](https://loft.sh/blog/kubernetes-virtual-clusters-for-ai-ml-experiments/#1-engineering-productivity) )。

# 3.技术基础设施

## 挑战

技术基础设施的一个主要挑战是如何让工程师访问它。一般来说，有不同的方法来解决这个问题，例如[使用单个集群或共享集群](https://loft.sh/blog/individual_kubernetes_clusters_vs-_shared_kubernetes_clusters_for_development/)。这里的问题通常是，这些解决方案要么非常不灵活和脆弱(共享集群的有限用户隔离)，要么成为管理员的管理噩梦(需要维护的集群太多)。

## 虚拟集群的影响

**灵活性:**由于虚拟集群具有很强的用户隔离性(与 Kubernetes 名称空间相比)，工程师在使用他们的个人环境时非常灵活。他们可以按需创建 vClusters，根据需要对其进行配置(他们甚至可以选择想要使用的 Kubernetes 版本)，最后还可以在不影响同事的情况下删除它们。

**可管理性:**当工程师们在感觉像“真正的”Kubernetes 集群的独立沙盒环境中工作时，底层的物理集群是共享的。作为一名集群管理员，您只需管理一个物理集群，这相对简单且易于管理，尤其是因为这个集群不需要任何复杂的安装或功能。用户管理，以确定谁可以创建 vClusters，其中有多少也可以集中控制，例如，与工程师友好的平台，如[阁楼](https://loft.sh/use-cases/ai-machine-learning-experiments)。

# 4.成本问题

## 挑战

尽管在前面提到的 O'Reilly 研究中没有提到成本，但成本始终是一个问题，尤其是在资源密集型人工智能或机器学习应用程序的情况下。如果人工智能在组织中的扩散取得进展，成本将成为更大的焦点。

## 虚拟集群的影响

**利用率:**只有一个中央物理集群应该支持自动扩展，因此在不使用时几乎不会产生任何成本，人工智能工程师可以在需要时在其上创建虚拟集群。这消除了对大部分时间不使用的昂贵备用集群的需要。

**睡眠模式:**loft 等虚拟集群解决方案提供了一种[睡眠模式](https://loft.sh/docs/sleep-mode/basics)，在不需要 v cluster 时自动将其发送至睡眠状态，从而减少了代价高昂的空闲时间。当工程师再次需要集群时，它也会自动唤醒，这可以确保工程师不受影响，事实上，他们根本不需要主动与睡眠模式功能进行交互。

# 如何获得虚拟 Kubernetes 集群

虚拟 Kubernetes 集群仍然是一个非常新的概念，但已经有了一些解决方案。社区中有一些早期的概念验证，如 [k3v](https://github.com/ibuildthecloud/k3v) 或来自 [Kubernetes 多租户 SIG](https://github.com/kubernetes-sigs/multi-tenancy/tree/master/incubator/virtualcluster) 的项目。

虽然这些都是相当基础的实现， [loft](https://loft.sh/) 提供了一个更全面的解决方案，除了实际的虚拟集群技术之外，还包括一个单点登录的用户管理、一个工程师友好的图形用户界面和前面提到的睡眠模式。有了这样的解决方案，就有可能将虚拟集群的所有优势用于人工智能和人工智能场景。

> *虚拟集群当然也可以用于 AI 和 ML 之外的场景，比如* [*云原生开发*](https://loft.sh/blog/kubernetes-virtual-clusters-as-development-environments/) *或者* [*CI/CD 和测试*](https://loft.sh/blog/kubernetes-virtual-clusters-for-ci-cd-testing/) *。关于虚拟集群的优势和使用案例的一般概述，* [*看看这篇文章*](https://loft.sh/blog/virtual-clusters-for-kubernetes-benefits-use-cases/) *。*

# 结论

对于已经依赖 Kubernetes 开发人工智能和 ML 软件的公司来说，虚拟集群可以解决人工智能采用中最常见的两个技术障碍:如何管理技术基础设施供应以及如何允许可重复的工作流。此外，vClusters 有助于提高工程师的工作效率，这在缺少熟练工程师的情况下尤为重要。最后，虚拟 Kubernetes 集群还有助于降低成本，这是一个始终相关的话题，如果人工智能在组织内的采用进一步发展，这一话题将变得更加普遍。

为此，虚拟集群可以为进一步采用人工智能铺平道路，从而帮助更多公司实现人工智能的全部好处，如增加收入和降低运营成本。

照片由[乌帕尔·帕特尔](https://unsplash.com/@visualsofupal?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

*原载于*[*https://loft . sh*](https://loft.sh/blog/kubernetes-virtual-clusters-for-ai-ml-experiments/)*。*
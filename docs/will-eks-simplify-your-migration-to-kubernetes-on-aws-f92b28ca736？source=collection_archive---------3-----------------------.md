# EKS 会简化你在 AWS 上向 Kubernetes 的迁移吗？

> 原文：<https://itnext.io/will-eks-simplify-your-migration-to-kubernetes-on-aws-f92b28ca736?source=collection_archive---------3----------------------->

![](img/862c3d5d17cb57a360bf32d591ab0150.png)

一年多以前，在 AWS 上建立一个 Kubernetes 集群并不是一件容易的事情。

下面是在 AWS 上设置 HA Kubernetes 集群所需的一些东西:

1.  设置 AWS 资源:VPC，自动缩放组，ELB，DNS
2.  自动化 etcd 安装和备份
3.  选择和配置网络驱动程序
4.  编写 puppet 代码来部署不同的 Kubernetes 组件

这只是我脑海中的一个部分列表，但是我希望您能够了解整体情况——建立一个 Kubernetes 集群将会从您的整个 Kubernetes 迁移项目中分配出很大一部分。

今天，有像 [kops](https://github.com/kubernetes/kops) 这样的工具可以自动完成上面提到的大部分任务，并且可以帮助你在 AWS 上构建 HA Kubernetes 集群。特别是 Kops 相当成熟，看起来最近[得到了很多开发关注](https://github.com/kubernetes/kops/graphs/code-frequency)。

# 为 AWS 管理 Kubernetes

如果您想避免在 AWS 上建立自己的 Kubernetes 集群，有一些商业选项可用，如 Stackpoint。

去年底，AWS 宣布[接受其托管的 Kubernetes 服务](https://aws.amazon.com/blogs/aws/amazon-elastic-container-service-for-kubernetes/)的测试人员，名为“EKS”。

我不是云分析师，所以我甚至不想深入探讨这一声明的战略含义。不过，我会问另一个问题，这个问题与我们一直在做的客户工作更相关:“**EKS 会大幅减少将公司迁移到运行 AWS 的 Kubernetes 所需的时间吗？**”

# Kubernetes 项目的典型步骤

在过去的两年中，我完成了多个 Kubernetes 迁移项目，在这些项目中，我们采用了一个工作生产环境，并将其迁移到 Kubernetes 集群上运行。

为了回答我上面提到的问题，让我们来看看将现有环境迁移到 Kubernetes 通常需要执行的任务:

# 1.设置一个或多个 Kubernetes 集群

正如我提到的，有了 kops 这样的工具，现在这是一项相对容易的工作，特别是如果你已经对 Kubernetes 的内部有了很好的了解，这将帮助你尽早做出正确的决定，比如使用哪个网络驱动程序。

# 2.码头化

如果您还没有使用 Docker 并以其他方式部署您的应用程序，那么为您的所有服务创建合适的 Docker 映像可能需要几天时间。

# 3.对部署和构建自动化的调整

老实说，我会说这部分大概占了项目时间的一半！

为什么？因为通常您会发现许多关于部署环境的问题或假设，您需要首先克服这些问题。

可能会有一些硬编码的 Jenkins 作业来执行部署的某些部分，或者被触发来执行特定的部署后任务。

此外，詹金斯可以是一个地狱般的烂摊子，你甚至不想碰。因此，启动一个单独的 Jenkins 实例来处理新的 Kubernetes 环境可能更有意义。

如果 Jenkins 不参与，那么可能会有一组部署脚本，比如说 Python，累积了如此多未使用的遗留代码，以至于甚至日常使用它的人都不完全理解它的作用！

最终，我们需要绘制现有的部署流程，并将其转化为 Kubernetes 世界中的工作方式。

# 4.库伯内特人的创造体现了

通常创建这些清单是作为上一步的一部分来完成的，但是我想把它单独归为一类。

大多数情况下，这个过程非常简单，但它可能会变得越来越复杂，这取决于您想要部署的服务数量以及它们与 [12 因素应用宣言](https://12factor.net/)的对应程度。

如果您有一堆无状态服务，一个简单的 envsub 模板就足够了。

然而，如果您有几十个具有复杂关系的服务影响部署，我们可能需要使用 Kubernetes 部署的重炮: [Helm](https://github.com/kubernetes/helm) 。

# 5.监控和记录

这一阶段的复杂程度取决于您当前使用的工具和您所拥有的监控范围。

如果您使用 CloudWatch 来监控和绘制一些内置的 AWS 指标，您可能需要实现并切换到一组与 Kubernetes 集成的新监控工具。

像 Datadog 这样的其他监控工具可能需要较少的调整，但是您仍然需要执行几个步骤来转发新的指标。

# 6.培训和移交

如果在迁移到 Kubernetes 之后，您的开发团队不能自己操作日常任务，那么您就错过了 Kubernetes 的一个核心优势。

不幸的是，尽管 Kubernetes 最近越来越受欢迎，但教授它的主要概念需要时间。文档不是最好的，团队需要时间来学习如何有效地使用 Kubernetes。我以前写过关于[对初学者来说在 Kubernetes](https://opsfleet.com/what-makes-kubernetes-difficult-for-beginners/) 学习有什么困难，所以你可能想看一看以获得更全面的了解。

# 摘要

如果你在 AWS 上运行，EKS 真的简化了到 Kubernetes 的迁移吗？从我上面提到的所有任务来看，**在 AWS 上建立一个 Kubernetes 集群将占用大约 10%的迁移项目**。

这还不包括维护，维护也取决于群集的大小和环境的复杂性。

但是要回答我在本文开始时提出的问题，EKS 可能会减少一些与集群设置相关的时间，甚至可能在将来减少与监控和日志记录相关的时间。由于大部分迁移项目的复杂性与现有基础设施有关，我认为**在 AWS 上迁移到 Kubernetes 将在整个 2018 年继续充满挑战**！

*最初发表于*[T5【opsfleet.com】](https://opsfleet.com/will-eks-simplify-your-migration-to-kubernetes-on-aws/)*。*
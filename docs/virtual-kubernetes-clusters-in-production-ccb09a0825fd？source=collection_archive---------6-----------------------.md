# 生产中的虚拟 Kubernetes 集群

> 原文：<https://itnext.io/virtual-kubernetes-clusters-in-production-ccb09a0825fd?source=collection_archive---------6----------------------->

虚拟 Kubernetes 集群不仅可以用于开发环境，还可以以多种方式改进生产系统。

![](img/0028c78f933572edb6a7f82b193b6f0d.png)

[虚拟 Kubernetes 集群(vClusters)](https://loft.sh/blog/introduction-into-virtual-clusters-in-kubernetes/) 的理念是在另一个 Kubernetes 集群中运行一个全功能集群，以提供高效的抽象，并在共享的底层集群之上直接访问 Kubernetes。

我已经描述了此类虚拟集群用于开发的[优势和用例，特别是用于](https://loft.sh/blog/virtual-clusters-for-kubernetes-benefits-use-cases/)[云原生开发](https://loft.sh/blog/kubernetes-virtual-clusters-as-development-environments/)、 [CI/CD](https://loft.sh/blog/kubernetes-virtual-clusters-for-ci-cd-testing/) 和 [ML/AI 实验](https://loft.sh/blog/kubernetes-virtual-clusters-for-ai-ml-experiments/)。然而，由于 vClusters 与常规的 Kubernetes 集群和名称空间一样灵活，因此除了开发之外，它们还可以用于许多情况。

在本文中，我想描述一些我认为虚拟集群最有价值的生产用例。

# 用 vClusters 替代集群以节省成本

[运行更少的 Kubernetes 集群通常更便宜](https://loft.sh/blog/kubernetes-cost-savings/)，因为更少的集群导致更少的冗余和更高的利用率。此外，集群管理员更容易管理它们。

然而，人们仍然没有在一个集群上运行所有的东西，这是有充分理由的。通常，这是因为您必须运行名称空间中的所有内容，而名称空间在许多情况下不能提供足够的隔离。与名称空间相比，vClusters 提高了租户的隔离性，因此有助于实现一种更难的形式 [Kubernetes 多租户](https://loft.sh/blog/kubernetes-multi-tenancy-best-practices-guide/)。这允许您将应用程序的某些部分集中在一个集群中，这些部分目前由于隔离问题而被分开。(当然，这并不意味着您应该在一个集群上运行所有的东西。)

将不同集群上的应用程序分开的另一个原因是它们需要集群的单独配置。虽然这在名称空间中是不可能的，但是虚拟集群可以灵活地进行不同的配置，例如，甚至可以为虚拟集群使用不同的 Kubernetes 版本。这又让您可以将应用程序集中在一起，减少集群数量，从而节省成本。

为此，虚拟集群解决方案也可能会引起托管和云提供商的兴趣，他们希望为客户提供灵活的 Kubernetes 环境，但也可以以较低的价格销售这种 vClusters 体验，因为并非每个客户都需要实际的物理集群，而只是完全的 Kubernetes 访问。

> *我写了一篇关于进一步选择* [*降低你的 Kubernetes 费用*](https://loft.sh/blog/reduce-kubernetes-cost/) *的专门文章。*

# 用 vClusters 替换命名空间以提高稳定性

当没有必要在集群上分离(部分)应用程序时，名称空间通常被用作运行软件的一种更便宜、更容易的方法。在这些情况下，名称空间可以由虚拟集群代替，因为它们有助于提高系统的安全性，因为它们具有更好的隔离性，并且为应用程序的不同部分提供了更灵活的配置。

因此，虚拟集群集群不仅允许您更频繁地共享集群，而且有助于优化您的生产系统。因此，vClusters 通常是“更好的命名空间”。

# 达到更高的可扩展性

如果您运行具有大量网络流量和负载的非常大的集群，您可能会达到可伸缩性的技术极限。自然，这通常发生在有许多用户或客户的生产用例中。

在这种情况下，您通常需要额外的集群来处理大量的流量。但是，仅仅因为 Kubernetes API 服务器或集群的 etcd 达到了极限，运行额外的集群并不一定是最优的。相反，运行多个 API 服务器或 etcds 来处理负载可能更有效。

由于每个虚拟集群都有自己的 API 服务器和 etcd，因此可以使用 vClusters 实现高效的 [Kubernetes 集群分片](https://loft.sh/use-cases/kubernetes-cluster-sharding)。这是可能的，因为对虚拟集群的请求通常由虚拟集群本身来处理，这减少了底层集群上的负载，因为只有一些请求由虚拟集群传递和处理。因此，您可以将负载“分散”到不同的虚拟集群上，从而避免物理集群的早期瓶颈问题。

因此，有可能突破集群所能处理的流量的技术限制，这样您就可以节省成本并优化系统，只需在合理的情况下添加集群。

# 更容易、更便宜地提供托管产品

虽然虚拟集群的第一个生产用例相当普遍，但也有一些更具体的场景非常有用。其中之一是[向您的客户](https://loft.sh/use-cases/cloud-native-managed-products)提供被管理产品。

通常有两种主要的技术方法来为您的客户提供软件即服务(SaaS)体验:[多租户和多实例](https://blog.scaleway.com/saas-multi-tenant-vs-multi-instance-architectures/)。

如果您为您的 SaaS 产品提供一种多租户方法，您运行一个系统实例和一个由不同客户共享的数据库。在这里，vClusters 可以通过提供更好的租户隔离和更大的可伸缩性来提高系统效率，再次为您提供帮助。

使用多实例方法，每个客户都有自己的系统和数据库实例。虚拟集群允许您非常容易地做到这一点，因为您可以在虚拟集群中复制您的系统。因此，每个客户都有一个专用的软件实例，但所有实例仍在单个集群上运行，这减少了系统的管理工作量。由于不必为每个客户启动新的集群，新实例的设置也更快，并且整个系统运行更有效，因为底层计算资源可以共享，因此不会空闲运行。

总之，无论您想使用哪种体系结构方法，vClusters 都有助于向客户提供 SaaS 产品。

# 快速轻松地运行交互式演示

虚拟 Kubernetes 集群的最后一个生产用例是[为你的产品](https://loft.sh/use-cases/live-demos)制作互动演示。尽管演示并不是真正的生产系统，但是它们的使用和功能更接近于传统的生产场景，而不是开发，这就是为什么我要在这里提到它们。

如果您销售一个内部产品，您的客户(尤其是 B2B 客户)通常希望在决定购买之前，甚至在他们自己在基础设施上安装之前，看到一个演示。你的产品的演示版本既可以被销售人员使用，也可以被客户自己使用。

在任何情况下，如果演示总是在相同的干净状态下开始，并且随时可用，也就是说，不必由管理员首先进行调配，这通常是有意义的。这些要求使它们成为虚拟集群的一个非常好的用例:v cluster 可以在几秒钟内从头开始启动，而不是在云环境中启动“真正的”集群需要几分钟。如果底层集群正在运行，它们也总是可用的。

此外，由于许多 v cluster 只需要一个底层集群，因此几名销售人员(或客户)可以按需独立启动演示，这样就不会有一个演示应用程序在每次演示后都必须重置。当演示结束时，整个演示环境(即 vCluster)也可以被删除，这一过程甚至可以通过[自动删除设置](https://loft.sh/docs/self-service/sleep-mode)实现自动化。或者，如果在同一阶段再次使用该环境，也可以保留该环境。

总之，由于共享的基础设施和自动清理，在虚拟集群中运行演示是便宜的，并且由于按需可用的独立环境是容易的。这使得在您的销售活动中使用演示更具吸引力，最终将有助于推动销售和收入。

# 结论

虚拟集群有很多好处，这使得它们不仅适用于开发场景，还适用于各种生产环境。通过提供更多的隔离和稳定性，或者通过增加集群可伸缩性的限制，它们可以用来从技术上改进您的系统。此外，如果您可以用 vClusters 替换整个集群，虚拟集群可以有效地降低您的成本。他们还可以帮助您解决如何高效地向客户提供被管理产品和产品演示的实际问题。

总的来说，考虑到 Kubernetes 应用程序的丰富性，我相信在生产过程中还会有更多的虚拟集群用例尚未被探索。

照片由[沃尔夫冈](https://www.pexels.com/@wolfgang-1002140?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)从[派克斯](https://www.pexels.com/photo/photo-of-people-watching-a-concert-2747449/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

*最初发布于*[*https://loft . sh*](https://loft.sh/blog/virtual-kubernetes-clusters-in-production/)*。*
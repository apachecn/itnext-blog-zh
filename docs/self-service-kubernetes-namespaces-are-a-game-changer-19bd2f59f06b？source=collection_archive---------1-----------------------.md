# 自助式 Kubernetes 名称空间是游戏规则的改变者

> 原文：<https://itnext.io/self-service-kubernetes-namespaces-are-a-game-changer-19bd2f59f06b?source=collection_archive---------1----------------------->

丹尼尔·塞利

![](img/632cf2b873818986a915fe63ba432ebb.png)

最近很多公司都采用了 Kubernetes。然而，他们中的大多数仍然没有意识到它的全部潜力，因为 Kubernetes 在这些组织中的实际使用是非常有限的。由于 Kubernetes 已经发生了巨大的变化，它现在不仅是一项运营技术，而且非运营工程师也可以使用它。因此， [Kubernetes 的采用不应该就此结束](https://loft.sh/blog/why-adopting-kubernetes-is-not-the-solution/)，而是刚刚开始。

因此，现在让工程师也参与 Kubernetes 的采用过程通常是有意义的，正如最新的[Stack Overflow Developer Developer Survey](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-platforms-wanted5)所显示的，工程师们很欣赏这一点，因为如果他们目前没有使用 Kubernetes，他们都希望使用它，而且在他们开始使用后也喜欢使用它。

让更多开发人员开始使用 Kubernetes 的一个简单方法是为他们提供自助式名称空间。在本文中，我将描述什么是自助式名称空间，为什么它们是 Kubernetes 采用的游戏规则改变者，以及如何获得它们。

# 什么是自助式 Kubernetes 名称空间？

自助式名称空间是 Kubernetes 名称空间，用户可以按需创建，而无需成为运行名称空间的集群的管理员。因此，自助式命名空间运行在共享的 Kubernetes 集群上，并由用户以简单和标准化的方式创建，例如通过[自助式命名空间平台](https://loft.sh/features/self-service-kubernetes-namespaces)的 UI。

因此，自助式名称空间为工程师提供了对 Kubernetes 的轻松且随时可用的访问，这与可能的替代方案相比是一个巨大的优势:虽然本地 Kubernetes 解决方案(如 [minikube](https://github.com/kubernetes/minikube) )总是必须由工程师自己设置和配置，因此永远不会随时可用，但为每个开发人员提供一个自己的云中集群是非常昂贵的。由于受限的云访问权限，单个集群 ar 通常也是不可行的，并且不必要，因为简单的名称空间对于大多数标准用例来说已经足够了。

因此，与让管理员手动创建命名空间相比，以自助方式提供命名空间是一个决定性的特性，因为只有这样才能消除“等待中央 IT 提供对基础架构的访问”这一[最重要的开发效率障碍](https://tanzu.s3.us-east-2.amazonaws.com/campaigns/pdfs/VMware_State_Of_Kubernetes_2020_eBook.pdf)。

总的来说，自助式名称空间是为工程师提供随时可用的 Kubernetes 访问的最简单方式。

# 自助命名空间的优势

向用户提供自助式名称空间解决方案对用户(工程师)和管理员双方都有好处。

# 命名空间用户的优势:

**1。Velocity:** 自助式名称空间总是可用的，只要用户需要，就可以快速轻松地创建。这使得它们成为各种工程任务的非常有用的解决方案，从[云原生开发](https://loft.sh/use-cases/cloud-native-development)到 [CI/CD 管道](https://loft.sh/use-cases/ci-cd-pipelines)和 [AI/ML 实验](https://loft.sh/use-cases/ai-machine-learning-experiments)。

**2。独立性:**自助服务方面使工程师能够独立于管理员工作，因为他们不必等待管理员创建工作环境后才能开始工作。

**3。更容易的实验:**用户的这种独立性也使得对名称空间进行更多的实验成为可能，因为名称空间现在可以被丢弃，并由用户自己重新创建。因此，用户不必担心破坏什么东西，最终可以把名称空间当作“牲口”而不是“宠物”。

> *通过使用自助服务* [*虚拟集群(vClusters)*](https://loft.sh/blog/introduction-into-virtual-clusters-in-kubernetes/) *，可以进一步增强用户的独立性，并减少对中断的担心，这些虚拟集群与名称空间非常相似，但提供了更强的隔离，并为工程师提供了更多配置 Kubernetes 的自由。*

# 集群管理员的优势:

**1。更好的稳定性:**由于所有名称空间都是由用户以相同的标准化方式创建的，因此在整个名称空间创建过程中几乎没有人为错误，这提高了底层 Kubernetes 集群的稳定性。此外，用户被封装在名称空间中，这可以防止他们相互干扰。

**2。更少的工作和压力:**用户获得的独立性减轻了集群管理员的压力。他们不必总是为工程师提供工作环境，因此不再是使用 Kubernetes 的[工程工作流程的瓶颈。管理员只需首先设置自助服务平台，然后确保它可用并且底层群集正在运行。](https://loft.sh/blog/kubernetes-development-workflow-3-critical-steps/)

**3。关注稳定性和安全性:**由于管理员不再需要参与每个名称空间的创建过程，他们现在可以更加关注底层集群的稳定性和安全性。

> *提供* [*自助式虚拟集群*](https://loft.sh/features/virtual-kubernetes-clusters) *可以再次改进系统，因为 vClusters 提供了更强大的多租户和用户隔离形式。它们还允许用户在他们的 vCluster 中自行配置更多内容，因此底层主机群集可以非常简单，从而提供更少的攻击面和人为错误空间，进一步提高稳定性和安全性。*

# 如何获得自助式 Kubernetes 名称空间

# 底层集群

自助式名称空间系统需要的第一部分是一个底层的 Kubernetes 集群，名称空间应该在这个集群上运行。如果自助式命名空间将用于开发和测试流程，那么创建一个独立于运行生产工作负载的集群的新集群是有意义的。

由于自助式命名空间解决方案的优势之一是可以由许多用户使用和共享，因此集群需要是基于云的集群，并且不能在本地运行(即使您可能首先使用本地集群测试您的设置，然后在云中使用“真实”版本重新开始)。

在这里，无论是运行在公共云还是私有云中的集群，无论是自我管理还是由云提供商管理，都没有关系。然而，使用类似于生产集群的集群通常是有意义的(例如，如果您的生产集群是 AWS，则使用 AWS ),因为这使得开发、测试和其他您想要使用自助服务名称空间的过程更加现实。

# 用户管理

自助式命名空间解决方案的第二个核心组件是权限和用户管理。这使得管理员可以控制谁可以创建名称空间，以及谁在使用什么。

尤其是在大型团队中，拥有单点登录解决方案非常有用，因为管理员不必手动添加用户，用户可以立即开始使用。如果您自己构建一个自助式名称空间系统，那么像 [dex](https://github.com/dexidp/dex) 这样的解决方案可能会对这个任务有所帮助。

# 用户限制

虽然您希望用户能够按需创建名称空间，但是您还希望防止 CPU、内存以及其他潜在因素(如容器、服务或入口的数量)的过度使用。这样的限制对控制成本很有帮助，但是你需要注意不要限制用户的工作。为此，应该由用户决定他们想要如何分配他们允许的资源。

与由用户按需创建的动态名称空间相比，使用手动供应和静态分配的名称空间实现有效的用户限制要容易得多。这是因为 Kubernetes 对资源配额的限制是基于名称空间的，而不是基于用户的。

但是，您希望限制用户而不是名称空间，所以您需要解决这个问题来获得合理的用户限制。为此，您需要使用聚合资源配额，这可以通过开源解决方案 [kiosk](https://github.com/kiosk-sh/kiosk) 来实现。

既然您已经了解了自助式命名空间系统的最基本组件，那么您需要决定是自己构建这个系统，还是购买现有的现成解决方案。

几个大型组织已经为名称空间建立了内部 Kubernetes 平台[。Spotify 就是一个很好的例子，因为在](https://loft.sh/blog/building-an-internal-kubernetes-platform/) [KubeCon 北美 2019](https://www.youtube.com/watch?v=vLrxOhZ6Wrg) 上甚至有一场关于他们平台的公开演讲，所以你可以学习他们的经验。然而，即使在使用一些开源组件如 [dex](https://github.com/dexidp/dex) 或 [kiosk](https://github.com/kiosk-sh/kiosk) 时，构建一个自己的名称空间自助服务平台也需要花费很多精力，这可能是主要较大的组织或有非常特殊需求的公司走这条路的原因。

与此相反，购买现有的现成解决方案对于任何规模的组织都是可行的，并且具有无需大量前期投资即可快速启动的优势。此外，您还可以获得专门的服务，这种服务超出了您可能自己构建的最低需求。这种现成解决方案的一个例子是 [Loft](https://loft.sh) 。Loft 内部构建在 Kiosk 上，除了在任何连接的集群之上的自助服务名称空间之外，它还提供了一些有用的附加功能:它可以与多个集群一起工作，具有 GUI、CLI 以及[睡眠模式](https://loft.sh/features/kubernetes-cost-management)以节省成本，并且它提供了一种虚拟集群技术，可用于创建比名称空间更好隔离的自助服务 Kubernetes 工作环境。

# 结论

如果您让您的工程师能够独立地按需创建名称空间，这将改变 Kubernetes 在您的组织中的使用方式。特别是如果您已经采用了 Kubernetes，并且现在想要在您的组织中的更多人之间推广它的使用，那么自助式名称空间系统是一个非常好的解决方案。它回答了如何为工程师提供简单而独立的 Kubernetes 访问的基本问题，同时它也是管理友好的，因为管理员可以轻松地管理它，因此有更多的时间来关心底层集群的稳定性。

要获得自助式名称空间系统，您需要决定是自己制造还是购买。对于有非常特殊需求的公司来说，开发它是正确的解决方案，但是即使这样，您也可以在已经存在的开源组件的基础上进行构建，这将使您的生活更加轻松。对于大多数公司来说，购买仍然是一种更实际的方法，因为你可以从专业供应商那里获得完整的解决方案，而无需大量的前期投资。

无论您如何决定，拥有一个自助式名称空间平台将有助于您在组织中更有效地使用 Kubernetes。

照片由 [Thiago Patrevita](https://www.pexels.com/@thiagopatrevita?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从 [Pexels](https://www.pexels.com/photo/four-man-cooking-1121482/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)

*最初发布于*[*https://loft . sh*](https://loft.sh/blog/self-service-kubernetes-namespaces-are-a-game-changer/)*。*
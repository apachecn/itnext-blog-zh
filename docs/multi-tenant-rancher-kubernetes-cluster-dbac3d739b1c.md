# 多租户牧场主 Kubernetes 集群

> 原文：<https://itnext.io/multi-tenant-rancher-kubernetes-cluster-dbac3d739b1c?source=collection_archive---------3----------------------->

## 探索如何扩展 Rancher 项目网络隔离以构建安全的多租户 Kubernetes 集群

![](img/0462586b06c287e0b3b46370f7322004.png)

[https://unsplash.com/photos/KZa4fREZoKk](https://unsplash.com/photos/KZa4fREZoKk)

在这里，我们探索使用牧场主、牧场主项目和项目网络隔离来创建多租户 Kubernetes 集群的想法。

**我们将调查一些安全和使用问题**。

如果您想创建一个多租户 Kubernetes 集群，或者只是想获得更多关于 Kubernetes 安全性的知识，您可能会感兴趣。

## 问题

我们能否创建一个(大型)Kubernetes 集群，并在不同的用户甚至不同的组织之间共享？当我们这样做时，我们希望用户之间有某种程度的隔离。如果用户来自同一家公司的不同部门，那么与来自不同公司的用户相比，隔离可能不那么严格。

## TL；速度三角形定位法(dead reckoning)

这是可能的，但需要很大的努力。根据用例，维护多个 Kubernetes 集群可能更好。

# 先决条件

## 库伯内特斯

你应该知道 Kubernetes 是什么:)

## 服务

你应该了解 [Kubernetes 服务](https://medium.com/swlh/kubernetes-services-simply-visually-explained-2d84e58d70e5?source=friends_link&sk=49a5e832662a689111a6087e1fe1232a)。

## 服务 DNS 名称解析

一个 Kubernetes 集群通常包含一个 DNS 服务器，它允许通过服务名称联系服务，而不是直接使用 IP。因为服务属于某个特定的名称空间，所以不同名称空间中的两个服务可能具有相同的名称。

与服务在同一名称空间中的资源可以简单地使用服务名通过 DNS 检索 IP。不同名称空间中的资源作为服务也需要以格式`<service-name>.<namespace>`指定名称空间。

## 网络策略

网络策略资源可以与防火墙规则相比较，防火墙规则允许或拒绝对某些 pod 或来自某些 pod 的访问。也可以控制对整个名称空间或 IP 地址 CIDRs 的访问。

> 网络策略是一种规范，规定了允许多组 pod 如何相互通信以及如何与其他网络端点通信。网络策略资源使用标签来选择 pod 和 dene 规则，这些规则指定允许哪些 trac 访问选定的 pod。([来源](https://kubernetes.io/docs/concepts/services-networking/network-policies/))

# 大牧场主

> Rancher 是为采用容器的团队提供的一个完整的软件栈。它解决了跨任何基础设施管理多个 Kubernetes 集群的运营和安全挑战，同时为 DevOps 团队提供了运行容器化工作负载的集成工具( [source](https://rancher.com/what-is-rancher/overview) )

它提供了一个 web 界面和一个 REST API，用户可以通过它来管理可访问的资源。一方面，Rancher 允许跨各种云提供商和自托管连接创建和管理多个 Kubernetes 集群。它为所有集群提供集中的身份验证和访问控制。

另一方面，它提供了对单个 Kubernetes 集群的扩展管理，例如，提供了将名称空间捆绑到 Rancher 项目中的能力。

## 项目

Rancher 项目是在 Kubernetes 名称空间之上构建的。一个集群包含多个项目，一个项目可以包含多个命名空间，一个命名空间最多可以属于一个项目。

Rancher 项目允许将规则(如资源配额)应用于项目中的所有名称空间，只需在项目级别进行一次必要的配置。仅使用 Kubernetes，大多数规则定义不能应用于许多名称空间，并且需要多次创建。

> 一个[Rancher]项目是一个集群中的一组多个名称空间和访问控制策略。项目是由 Rancher 引入的概念，而不是 Kubernetes，它允许您将多个名称空间作为一个组进行管理，并在其中执行 Kubernetes 操作。([来源](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/projects-and-namespaces/)

## 项目网络隔离

牧场主项目网络隔离限制了项目之间的通信。它是在 Kubernetes 网络策略资源的基础上实现的，在 network policy 一节中有描述。因此所使用的 CNI 必须实现网络策略的使用，这里选择。

每当在一个项目中创建一个名称空间时，Rancher 都会给它分配带有 project-ID 的 Kubernetes 标签。基于这些标签，Rancher 实现了网络策略，该策略只允许该名称空间的 pod 拥有来自属于同一项目的名称空间的 pod 的传入连接。

# 多租户集群

> 多租户集群由多个用户和/或工作负载共享，这些用户和/或工作负载称为租户。多租户集群的操作员必须将租户相互隔离，以最大限度地减少受损或恶意租户对集群和其他租户造成的损害。([来源](https://cloud.google.com/kubernetes-engine/docs/concepts/multitenancy-overview))

运行多个用户一起使用的多租户集群的优势在于共享资源，从而节省 CPU 或内存等资源。也可以在一个中心位置向所有租户提供服务，比如 NFS 储物。

主要的挑战是在安全性和可用性之间找到一个好的平衡。不同组织的用户应该相互隔离，尽管使用集群环境的体验不应该与拥有完全访问权限有太大区别。

Rancher 通过实施(隔离的)项目来提供 Kubernetes 的扩展管理。使用 Rancher 项目作为不同团队或组织的代表，允许每个团队或组织控制多个名称空间。

# 安全考虑

应该注意的是，这个列表可能不完整。欢迎评论补充或讨论。

## 可以停用项目网络隔离

网络策略资源是在命名空间中创建的，并且可以由有权访问该命名空间所属的 Rancher 项目的用户删除。通过这种方式，用户可以取消他/她的项目的隔离，并允许其他项目的 pod 与其项目中的 pod 进行通信。

这可以通过利用 RBAC(基于角色的访问控制)实现特定的 Kubernetes 角色来限制。

## 节点端口和负载平衡器服务打破了隔离

为了确保正确的项目隔离，需要禁止 NodePort 和 LoadBalancer 类型的服务。这是因为两种服务类型都在每个 Kubernetes 节点的外部和内部 IP 地址上打开一个 TCP 端口。一个项目中的 pod 可以连接到节点的 IP 地址，并与创建服务的另一个项目中的 pod 通信。

牧场主项目隔离的目的只是为了控制豆荚间的直接接触。一旦用户使用服务暴露了 pod，Rancher 项目网络隔离不会提供保护([来源](https://github.com/rancher/rancher/issues/15250))。

这些服务也需要被禁止，因为在相同或不同的项目中，没有两个服务可以打开同一个节点端口。这将在创建过程中导致 Kubernetes 错误，因此是一个跨项目关联。可以使用 Kubernetes 资源配额禁止 NodePort 和 LoadBalancer 服务类型。

## 优先

可以通过分配优先级来区分 pod 的优先级。通过优先级，Kubernetes 规定当可用集群资源不足时，哪些 pod 被调度，哪些被终止[ [来源](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/) ]。但是因为 Kubernetes 优先级类资源是全局的，没有绑定到名称空间，所以访问特定项目的 Rancher 用户没有创建新项目的权限。

另外，系统内部优先级类 system-cluster-critical 和 system-node-critical 只能分配给 Kubernetes 内部名称空间中的 pod。考虑到这一点，不应该有用户使用优先级来强制其他项目的 pod 终止的可能性。

## 强力集群信息收集

蛮力是尝试各种不同组合的过程，希望最终猜对。

因为名称空间需要在 Rancher 项目中是唯一的，所以用户可以通过尝试创建具有不同名称的名称空间来找出其他项目的名称空间名称。如果 Kubernetes API 或 kubectl 返回一个错误，说名称空间已经存在，那么用户知道它的存在。

使用这个过程，用户可以找出所有项目中所有可用命名空间的所有名称。Kubernetes DNS 在“服务 DNS 名称解析”一节中进行了描述，它适用于整个集群。每个 Kubernetes 服务都可以通过一个 IP 地址和一个 DNS 名称到达。一旦用户获得了关于集群中现有名称空间的信息，他们就可以通过使用强力方法找出所有的服务名称。

用户仍然不能连接到另一个项目的 pod，但是用户将能够找到在特定名称空间中运行的服务的名称。没有解决方案可以防止使用 Kubernetes 默认资源进行强力名称空间或服务 DNS 名称检查，但这可以通过异常检测来识别。

保护 DNS 解析的一个解决方案可能是利用纤毛作为 CNI:

> Cilium 1.4 扩展了现有的 DNS 安全策略模型，可以识别各个 pod 发出的 DNS 请求以及它们收到的 DNS 响应。([来源](https://cilium.io/blog/2019/02/12/cilium-14/))

## Pod 安全策略

Rancher 提供了两种默认的 pod 安全策略，受限和无限制。无限制对集群中部署的 pod 没有限制。另一方面，Restricted 被描述为“显著限制可以部署到集群或项目的 pod 类型”[来源](https://rancher.com/docs/rancher/v2.x/en/admin-settings/pod-security-policies)，例如“防止 pod 作为特权用户运行，并防止特权升级。”【[来源](https://rancher.com/docs/rancher/v2.x/en/admin-settings/pod-security-policies)】。

这意味着牧场主项目应该要么有受限的默认 pod 安全策略，要么分配一个更严格的自定义策略。

## 读取其他项目中容器的日志

用户可以创建一个 pod，甚至可以设置一个守护进程在每个节点上创建一个 pod，它使用 host path provider[[source](https://v1-15.docs.kubernetes.io/docs/concepts/storage/volumes/)]挂载它所调度的节点的本地文件夹。

如果一个 pod 挂载了该节点的文件夹`/var/lib/docker`，它就可以访问该节点上运行的所有容器的 docker 日志。这提供了跨项目的信息交换。应该可以使用 pod 安全策略和 AllowedHostPaths 选项，或者通过完全禁止卷类型 hostpath[[源](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) ]来对此进行限制。这种用户行为也可以使用异常检测来识别。

## 异常检测

异常检测，或异常值检测，“旨在识别数据库中不寻常的对象，即不同于大多数数据，因此由污染、错误或欺诈导致的可疑对象”[Zimek A .，Schubert E. (2017)异常值检测。载于:刘主编的《数据库系统百科全书》。纽约州纽约市斯普林格】。

在 Kubernetes 中，可以使用审计来实现，审计“提供了一组与安全相关的按时间顺序排列的记录，记录了个人用户、管理员或系统的其他组件影响系统的一系列活动”。

Falco 是一个开源应用程序，它利用 Kubernetes 审计，并为常见情况提供预先配置的规则，如集群内资源的权限提升尝试[ [source](https://v1-15.docs.kubernetes.io/docs/tasks/debug-application-cluster/falco/) ]。

# 使用注意事项

本节列出了在运行共享的 Rancher Kubernetes 集群时，在使用和管理体验方面需要考虑的几点。

## 名称空间

名称空间需要在 Rancher 项目中是唯一的，这意味着没有两个项目可以包含同名的名称空间。Rancher 不提供配置自动命名空间名称前缀的能力，例如项目名称。

现有的 Kubernetes 应用程序通常将资源安装在以应用程序命名的名称空间中，比如包含 MySQL 数据库的名称空间`mysql`。这意味着两个用户将无法一步一步地遵循相同的安装说明，而是必须调整他们的名称空间名称。这可以通过向用户传达强制使用名称空间前缀的规则来解决。

当用户使用 kubectl 或 Kubernetes API 创建名称空间时，它不会被分配给项目。它处于浮动状态，只能由管理员分配给项目。这意味着用户只需使用 Rancher web 界面创建名称空间。在线分步安装指南通常会使用 kubectl 来创建必要的名称空间。使用 de 时，不可能完全遵循说明。NBI 共享集群。

## 不常见的用法

与完全集群访问相比，使用 kubectl 或 Kubernetes API 会有所不同。用户不能查询他/她有权访问的所有可用名称空间。这只有使用 web 界面或 Rancher API 才有可能。

现有的 Kubernetes 应用程序或 Helm charts 可能无法开箱即用，因为它们使用节点端口或负载平衡器服务。用户需要将它们转换成 ClusterIP 类型的服务。

牧场主集群需要所有的 pod 来定义资源限制。这导致用户不得不为他们或现有的 Kubernetes 应用程序创建的每个 pod 取消资源限制。这在完全访问集群中是不需要的。尽管 Rancher 允许为某个项目中创建的每个容器定义默认资源限制。

## 维护可能会影响许多用户

Kubernetes 集群维护可能需要关闭或重启特定的节点。如果某个节点的 Kubernetes 版本需要更新，首先要将其标记为不可用于安排任何 pods。这将启动将该节点上运行的所有 pod 重新调度到其他节点。只有由其他资源(如部署)控制的单元可以重新计划。未经管理直接创建的 pod 无法自动重新创建[ [source](https://v1-15.docs.kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) ]。

必须考虑这一点，这意味着单个节点的维护可能会影响不同项目和组织的众多用户，因为所有用户在某种程度上都可能共享同一个节点。

## 在共享环境中工作的知识

在完成完全隔离之前，需要通知用户他们正在使用一个共享的 Kubernetes 环境，其中可能只应用了有限的隔离。

# 概述

可以创建一个多租户 Kubernetes 集群(使用 Rancher)，但是隔离很困难。根据不同的用例，创建和维护多个 Kubernetes 集群可能更容易，每个租户一个集群。Rancher 允许并提供了以更简单的方式管理多个集群的工具。
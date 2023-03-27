# Kubernetes 最佳实践

> 原文：<https://itnext.io/kubernetes-best-practices-d2bfb6fe3c19?source=collection_archive---------1----------------------->

## 成功 K8s 管理的 15 个最佳实践，包括示例和技巧！

在这篇文章中，我将介绍一些使用 Kubernetes (K8s)的最佳实践。

作为最流行的容器编排系统，K8s 是现代云工程师需要掌握的事实上的标准。K8s 是一个众所周知的使用和维护复杂的系统，所以很好地掌握您应该做什么和不应该做什么，并知道什么是可能的，这将使您的部署有一个坚实的开端。

这些建议涵盖了 3 大类中的常见问题，即应用程序开发、治理和集群配置。

![](img/024113dad37b0c8e7d85e8f93554b6e7.png)

[伊恩·泰勒](https://unsplash.com/@carrier_lost?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/container?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照

## 最佳实践目录

1.  使用名称空间
2.  使用就绪性和活性探测器
3.  使用资源请求和限制
4.  将 pod 作为部署、DaemonSet、ReplicaSet 或 StatefulSet 的一部分跨节点部署。
5.  使用多个节点
6.  使用基于角色的访问控制(RBAC)
7.  在外部托管您的 Kubernetes 集群(使用云服务)
8.  升级您的 Kubernetes 版本
9.  监控您的群集资源和审计策略日志
10.  使用版本控制系统
11.  使用基于 Git 的工作流(GitOps)
12.  缩小你的容器的尺寸
13.  用标签组织您的对象
14.  使用网络策略
15.  使用防火墙

## 使用名称空间

K8s 中的名称空间对于组织对象、在集群中创建逻辑分区以及安全目的来说非常重要。默认情况下，K8s 集群中有 3 个名称空间，`default, kube-public`和`kube-system.`

RBAC 可用于控制对特定名称空间的访问，以限制一个组的访问，从而控制可能发生的任何错误的影响范围，例如，一组开发人员可能只能访问名为`dev`的名称空间，而不能访问`production`名称空间。限制不同团队使用不同名称空间的能力对于避免重复工作或资源冲突很有价值。

还可以针对名称空间配置 LimitRange 对象，以定义名称空间中部署的容器的标准大小。ResourceQuotas 还可以用来限制命名空间中所有容器的总资源消耗。可以针对名称空间使用网络策略来限制 pod 之间的流量。

## 使用就绪性和活性探测器

就绪和活性探测本质上是健康检查的类型。这些是 K8s 中使用的另一个非常重要的概念。

准备就绪探测器确保当 pod 准备好服务请求时，对 pod 的请求才被定向到它。如果它没有准备好，那么请求将被定向到其他地方。为每个容器定义就绪探测器很重要，因为 K8s 中没有为这些容器设置默认值。例如，如果一个 pod 需要 20 秒钟才能启动，并且就绪探测丢失，则在启动期间定向到该 pod 的任何流量都会导致失败。准备就绪探测应该是独立的，不考虑对其他服务的任何依赖，比如后端数据库或缓存服务。

活性探测器测试应用程序是否正在运行，以便将其标记为健康。例如，可以测试 web 应用程序的特定路径，以确保它正在响应。否则，pod 将不会被标记为健康，探针故障将导致`kubelet`启动新的 pod，然后再次进行测试。这种类型的探测器用作恢复机制，以防进程变得无响应。

## 使用自动缩放

在适当的情况下，可以使用自动扩展来动态调整单元的数量(水平单元自动扩展)、单元消耗的资源量(垂直自动扩展)或群集中的节点数量(群集自动扩展)，具体取决于对资源的需求。

水平 pod 自动缩放器还可以根据 CPU 需求缩放复制控制器、副本集或有状态集。

使用缩放也带来了一些挑战，比如不在容器的本地文件系统中存储持久数据，因为这将阻止水平自动缩放。相反，可以使用持久卷。

当集群上存在高度可变的工作负载时，集群自动缩放非常有用，这些工作负载可能会根据需求在不同时间需要不同数量的资源。自动删除未使用的节点也是省钱的好方法！

## 使用资源请求和限制(不使用 CPU 限制)

应该设置资源请求和限制(容器中可以使用的最小和最大资源量),以避免容器在没有分配所需资源的情况下启动，或者群集用完可用资源。

如果没有限制，pod 可以利用比所需更多的资源，导致总可用资源减少，这可能会导致集群上的其他应用程序出现问题。节点可能会崩溃，并且调度程序可能无法正确放置新的单元。

在没有请求的情况下，如果不能为应用程序分配足够的资源，它可能会在尝试启动或不稳定地执行时失败。

资源请求和限制定义了可用的 CPU 和内存量，单位为毫核心和兆字节。请注意，如果您的进程超出了内存限制，该进程将被终止，因此在所有情况下设置该值并不总是合适的。如果你的容器超过了 CPU 的限制，进程就会被限制。

最佳实践建议简而言之:

1.  对所有东西(CPU 和内存)使用请求。
2.  将内存限制设置为等于内存请求
3.  根本不要使用 CPU 限制

关于这一点的更多解释，请看这篇来自 [**纳坦耶林**](https://www.linkedin.com/in/natanyellin/) **的文章。**

[](https://home.robusta.dev/blog/stop-using-cpu-limits/) [## 看在上帝的份上，停止在 Kubernetes 上使用 CPU 限制吧

### 许多人认为 Kubernetes 需要 CPU 限制，但这不是真的。在大多数情况下，Kubernetes 的 CPU 限制做得更多…

home.robusta.dev](https://home.robusta.dev/blog/stop-using-cpu-limits/) 

## 将 pod 作为部署、DaemonSet、ReplicaSet 或 StatefulSet 的一部分跨节点部署。

不应单独运行单个 pod。相反，为了提高容错能力，它们应该始终是部署、DaemonSet、ReplicaSet 或 StatefulSet 的一部分。然后，可以在部署中使用反关联性规则跨节点部署 pod，以避免所有 pod 都在单个节点上运行，这可能会导致停机。

## 使用多个节点

如果您想内置容错功能，在单个节点上运行 K8s 并不是一个好主意。您的集群中应该使用多个节点，这样工作负载就可以在它们之间分散。

## 使用基于角色的访问控制(RBAC)

在 K8s 集群中使用 RBAC 对于确保系统安全至关重要。可以为用户、组和服务帐户分配权限，以对特定命名空间(角色)或整个群集(ClusterRole)执行允许的操作。每个角色可以有多个权限。为了将定义的角色绑定到用户、组或服务帐户，使用了 RoleBinding 或 ClusterRoleBinding 对象。

RBAC 角色应设置为使用最小特权原则进行授予，即只授予所需的权限。例如，admins 组可以访问所有资源，而 operators 组可以部署，但不能读取机密。

## 在外部托管您的 Kubernetes 集群(使用云服务)

在自己的硬件上托管 K8s 集群可能是一项复杂的任务。云服务提供 K8s 集群即平台即服务(PaaS)，比如 Azure 上的 AKS (Azure Kubernetes 服务)，或者 Amazon Web Services 上的 EKS (Amazon Elastic Kubernetes 服务)。利用这一点意味着底层基础设施将由您的云提供商管理，并且围绕扩展您的集群的任务(如添加和删除节点)可以更容易地完成，让您的工程师管理 K8s 集群本身上运行的内容。

## 升级您的 Kubernetes 版本

除了引入新功能，新的 K8s 版本还包括漏洞和安全修复，这使得在您的集群上运行最新版本的 K8s 变得非常重要。对旧版本的支持可能不如新版本好。

然而，迁移到新版本应谨慎对待，因为某些功能可能会被削弱，也可能会添加新功能。此外，在升级之前，应该检查集群上运行的应用程序是否与较新的目标版本兼容。

## 监控您的群集资源和审计策略日志

监控 K8s 控制平面中的组件对于控制资源消耗非常重要。控制平面是 K8s 的核心，这些组件保持系统运行，因此对于正确的 K8s 操作至关重要。Kubernetes API、kubelet、etcd、控制器-管理器、kube-代理和 kube-dns 组成控制平面。

控制平面组件可以以 Prometheus(最常见的 K8s 监控工具)可以使用的格式输出指标。

应使用自动化监控工具，而不是手动管理警报。

K8s 中的审计日志可以在启动 kube-apiserver 时打开，以便使用您选择的工具进行更深入的调查。audit.log 将详细描述对 K8s API 的所有请求，并且应该定期检查集群上可能存在的任何问题。Kubernetes 集群默认策略在 audit-policy.yaml 文件中定义，可以根据需要进行修改。

Azure Monitor 等日志聚合工具可用于将日志从 AKS 发送到日志分析工作区，以便将来使用 Kusto 查询进行询问。在 AWS 上可以使用 Cloudwatch。第三方工具也提供更深入的监控功能，如 Dynatrace 和 Datadog。

最后，应该为日志确定一个保留期，通常为 30–45 天左右。

## 使用版本控制系统

K8s 配置文件应该在版本控制系统(VCS)中进行控制。这带来了许多好处，包括提高安全性、支持变更的审计跟踪，以及提高集群的稳定性。应该为所做的任何更改设置审批关口，以便团队可以在将更改提交到主分支之前对其进行同行评审。

## 使用基于 Git 的工作流(GitOps)

K8s 的成功部署需要对您的团队使用的工作流程进行思考。使用基于 git 的工作流通过使用 CI/CD(持续集成/持续交付)管道实现自动化，这将提高应用程序部署效率和速度。CI/CD 还将提供部署的审计跟踪。Git 应该是所有自动化的唯一来源，并且将支持 K8s 集群的统一管理。您还可以考虑使用专用的基础设施交付平台，如最近推出 Kubernetes 支持的 [Spacelift](https://spacelift.io/) 。

## 缩小你的容器的尺寸

较小的映像大小将有助于加快构建和部署，并减少容器在 K8s 集群上消耗的资源量。在可能的情况下，应该删除不重要的包，应该优先选择 Alpine 之类的小型操作系统分发映像。较小的图像可以比较大的图像拉得更快，占用的存储空间也更少。

遵循这种方法还将提供安全优势，因为对于恶意参与者来说，潜在的攻击媒介会更少。

## 用标签组织您的对象

K8s 标签是为了组织集群资源而附加到对象上的键值对。标签应该是有意义的元数据，提供一种机制来跟踪 K8s 系统中不同组件如何交互。

[官方 K8s 文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)中推荐的吊舱标签包括`name, instance, version, component, part-of,`和`managed-by.`

标签也可以以类似于在云环境中在资源上使用标签的方式使用，以便跟踪与业务相关的事情，例如对象`ownership`，以及对象应该属于的`environment`。

还建议使用标签详细说明安全要求，包括`confidentiality`和`compliance.`

## 使用网络策略

应该采用网络策略来限制 K8s 集群中对象之间的流量。默认情况下，所有容器都可以在网络中相互通信，如果恶意行为者获得对容器的访问权限，允许他们遍历集群中的对象，这将带来安全风险。网络策略可以在 IP 和端口级别控制流量，类似于云平台中安全组的概念来限制对资源的访问。通常，默认情况下应该拒绝所有流量，然后应该制定允许规则来允许所需的流量。

## 使用防火墙

除了使用网络策略来限制 K8s 集群上的内部流量之外，您还应该在 K8s 集群前面放置一个防火墙，以便限制来自外部世界的对 API 服务器的请求。IP 地址应该列入白名单，开放端口应该受到限制。

## 摘要

在设计、运行和维护您的 Kubernetes 集群时，遵循本文中列出的最佳实践将使您在现代化应用程序之旅中走向成功！

想要更多 Kubernetes 内容？查看我的其他 K8S 文章[这里](https://jackwesleyroper.medium.com/kubernetes-k8s-related-articles-index-54718769e390)！

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper) 

*原载于*[*space lift . io*](https://spacelift.io/)*。*
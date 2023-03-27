# Kubernetes StatefulSets 坏了

> 原文：<https://itnext.io/kubernetes-statefulsets-are-broken-8cafde6544af?source=collection_archive---------1----------------------->

![](img/7d8614dd0da3014b0109edc4ea86c0e0.png)

这实际上不是一个有状态的集合。Unsplash.com 的克利姆·穆萨利莫夫的照片。

*Kubernetes 最初旨在充当无状态工作负载的容器编排平台，而不是有状态应用。*

不要误会我；我们是 Kubernetes 的坚定支持者。这是[我们项目](https://github.com/pluralsh/plural) t 的一个关键部分，正确使用的话会提供巨大的价值。但是，Kubernetes 的初衷是作为无状态工作负载的容器编排平台，**而不是有状态应用**。

在过去几年中，Kubernetes 社区通过创建 StatefulSets(这是 Kubernetes 对以存储为中心的工作负载的解决方案)在发展项目以支持有状态工作负载方面做了大量工作。

状态集涵盖了从数据库、队列和对象存储到出于某种原因需要修改本地文件系统的旧 web 应用程序。它们为开发人员提供了一套非常强大的保证:

*   **每个 pod 的一致网络身份:**这允许您在应用中轻松配置 pod 的 DNS 地址。它对于数据库连接字符串或配置复杂的 Kafka 客户端非常有用。我们有时也用它来建立 erlang 的网状网络。
*   **持久性卷自动化:**每当 pod 重新启动时，即使它被重新安排到不同的节点上，持久性卷也会重新附加到它所在的节点上。这在某种程度上受到您所使用的 CSI(容器存储接口)功能的限制。例如，在 AWS 上，这仅在同一区域 AZ 内有效，因为 EBS 卷是与 AZ 链接的。
*   **顺序滚动更新:** StatefulSet 更新被设计为滚动和一致的。它将总是以相同的顺序更新，这有助于保护具有微妙协调协议的系统。

这些保证涵盖了运行有状态工作负载所需的大量操作。特别是，它几乎完全处理可用性部分。鉴于 EBS 的正常运行时间和冗余保证非常强大，StatefulSet 的重新调度自动化几乎可以保证您获得高可用性服务。然而，一些警告确实适用(例如，您的集群中有空间，不要搞糟 AZ 设置。)

Kubernetes 在这一领域大有可为，从理论上讲，它肯定会发展成为一个平台，可以轻松地运行大多数开发人员使用的有状态工作负载和无状态工作负载。

[](https://github.com/pluralsh/plural) [## GitHub - pluralsh/plural:以创纪录的时间在 kubernetes 上部署开源软件。🚀

### Plural 使您能够在 Kubernetes 上构建和维护云原生和生产就绪的开源基础设施…

github.com](https://github.com/pluralsh/plural) 

# Kubernetes StatefulSet 中缺少什么？

那么，为什么我们认为状态集是坏的呢？嗯，如果您在头脑中思考有状态工作负载的操作需求，您可能会注意到缺少一个关键组件:

*当你需要调整底层磁盘大小时，你会怎么做？*

数据集是一种常见的数据库存储，通常以相当恒定的正速率增长。除非您支持水平扩展和分区，否则随着数据集的增长，您需要在磁盘中增加扩展空间。这就是 Kubernetes 的失败之处。

目前，StatefulSet 控制器[没有对音量大小调整](https://github.com/kubernetes/kubernetes/issues/68737)的内置支持。尽管事实上几乎所有的 CSI 实现都支持控制器可以挂接的卷大小调整。有一个解决办法，但它几乎是可笑的迂回:

*   在孤立 pod 时删除状态集以避免停机，方法是:kubectl **删除**STS<name>—级联=孤立
*   将每个 pod 的永久卷手动编辑为新的存储大小
*   使用新的存储大小手动编辑 StatefulSet 卷声明，并添加虚拟 pod 注释以强制滚动更新
*   使用新规范重新创建 StatefulSet，这允许控制器收回对孤立单元的控制，并开始滚动更新，这将触发 CSI 应用卷大小调整

> 💡我们实际上将整个过程自动化为复数操作符的[部分。我们知道，我们需要构建存储大小调整自动化，让非 Kubernetes 专家也能操作使用复数运行的有状态应用程序。这在现实中是一个不小的逻辑量，如果有人被要求在高压情况下做这件事，失败的可能性非常高。](https://github.com/pluralsh/plural-operator/blob/main/controllers/statefulsetresize_controller.go#L70)

好吧，所以在 Kubernetes 的 StatefulSets 中有一个相当值得注意的缺陷，但是有*是*一个变通办法，即使它有点滑稽。

那应该不会太糟，对吧？

# 但情况变得更糟！

当您意识到这种限制的影响以及许多 Kubernetes 操作符是为管理有状态工作负载而构建的时，情况会变得非常糟糕。

一个很好的例子是 [Prometheus operator](https://github.com/prometheus-operator/prometheus-operator) ，这是一个很棒的项目，既提供 Prometheus 数据库，又允许基于 CRD 的工作流配置指标、刮刀和警报。

出现这个问题是因为操作符的内置控制器没有管理 StatefulSet resize 的逻辑，但是如果它发现触发其删除的事件，它有重新创建其基础 StatefulSet 的逻辑。这意味着您实际上没有办法使用上述解决方法，因为当您执行级联孤立删除时，操作符将根据旧规范重新创建 StatefulSet，并阻止正确调整大小。唯一的解决方案是删除整个 CRD 或找到一个可以欺骗操作者不协调对象的调整(有时缩放到零会做到这一点)。

无论如何，作为这个缺陷的结果，实际上没有办法在不造成大量停机或数据丢失的情况下用操作者调整 Prometheus 实例的大小。考虑到 StatefulSets 中的自动化在所有其他情况下是多么健壮，这仍然是一种潜在的失败模式，这是非常令人震惊的。

我们的社区负责人 Abhi 在开源 Vitess 操作符中实现 StatefulSet volume resizes 时，实际上已经解决了这个问题。

> *“考虑到 Vitess 部署的自然复杂性，您可以推断磁盘大小调整也是成比例的复杂。* [*Vitess*](https://vitess.io/) *是一个位于 MySQL 之上的数据库共享系统，这意味着卷大小调整必须是分区感知和分片感知的。我们不得不手动编写我们自己的分片安全的* [*滚动重启，创建一个*](https://github.com/planetscale/vitess-operator/blob/eeff200e5acaf67db6f01f0f7f945ea7e89d4540/pkg/controller/vitessshard/reconcile_rollout.go#L18) [*级联条件*](https://github.com/planetscale/vitess-operator/blob/eeff200e5acaf67db6f01f0f7f945ea7e89d4540/pkg/operator/rollout/rollout.go#L82) *以便与 Vitess 自定义资源的父子结构一起工作，并使用* [*解决每一个可以想到的故障*](https://github.com/planetscale/vitess-operator/blob/eeff200e5acaf67db6f01f0f7f945ea7e89d4540/pkg/controller/vitessshard/reconcile_disk.go#L21) *条件以防止停机。感谢著名的 Kubernetes 撰稿人*[*enisoc*](https://mobile.twitter.com/enisoc)*设计了这一功能。”*

其他广泛使用和著名的数据库操作符，如 [Zalando 的 Postgres 操作符](https://github.com/zalando/postgres-operator)，在他们自己的代码库中有效地重新实现了我们在复数操作符中实现的相同过程。这导致在一个只需修复一次的问题上浪费了大量的开发周期。

# Kubernetes 的潜力

总的来说，我们非常看好 Kubernetes 让几乎任何工作量的操作变得微不足道的潜力，我们在 [Plural](https://github.com/pluralsh/plural/) 的任务的很大一部分就是让这成为可能。

也就是说，我们还需要清楚地看到 Kubernetes 生态系统中仍然存在的差距，这样我们就可以解决它们，或者从上游弥补它们。我认为很明显这是一个重大的差距，如果优先考虑，这可以在 Kubernetes 的未来版本中很容易地修复[。](https://github.com/kubernetes/kubernetes/pull/110522)

如果你认为这很有趣，看看我们在 Kubernetes [这里](https://github.com/pluralsh/plural)构建了什么。
# 使用 Spot 实例在 EKS 运行弹性工作负载

> 原文：<https://itnext.io/running-production-workloads-in-eks-using-spot-instances-fc6808a7b462?source=collection_archive---------1----------------------->

![](img/76b630eed5e2c24c0204ca67f78af913.png)

# 定点实例概述

一个 **Spot 实例**是一个使用备用 EC2 容量的实例，可用价格低于按需价格(便宜 90%)，这使它成为一个非常经济的选择，但也有一些缺点。在所谓的 *Spot 实例中断中，Spot 实例可被 AWS EC2 Spot 服务**中断**。*以下是 Amazon EC2 中断您的 Spot 实例的可能原因:

*   **价格:**现货价格大于你的最高限价。
*   **容量:** Amazon EC2 可以在需要时中断您的 Spot 实例。EC2 回收实例主要是为了重新利用容量，但也可能是因为其他原因，如主机维护或硬件退役。
*   **约束:**如果您的 Spot 请求包含一个约束，如启动组或可用性区域组，当约束不再满足时，Spot 实例将作为一个组终止。

> 您可以在 [Spot Instance Advisor](http://aws.amazon.com/ec2/spot/instance-advisor/) 中查看您的实例类型的历史中断率。

如果 Spot 实例被停止、休眠或终止，可以使用 CloudTrail 查看 Amazon EC2 是否中断了 Spot 实例。在 AWS CloudTrail 中，事件名称`BidEvictedEvent`表示 Amazon EC2 中断了 Spot 实例。要在 CloudTrail 中查看 BidEvictedEvent 事件:

1.  打开 CloudTrail 控制台
2.  在导航窗格中，选择**事件历史**。
3.  在过滤器下拉列表中，选择**事件名称**，然后在右边的过滤器字段中，输入**bidevictevent**。
4.  在结果列表中选择 **BidEvictedEvent** 查看其详细信息。在**事件记录**下，可以找到实例 ID。

Spot 实例通常适用于无状态、容错的应用程序，这些应用程序能够检查点并在中断后继续运行，也适用于批处理作业。

# 使用定点实例时的注意事项

在 giffgaff，我们使用 100%的 Spot 实例在 EKS 集群中运行所有应用程序。我们利用 Spot 和[海洋集群](https://docs.spot.io/ocean/overview-kubernetes)特征。

Spot 持续监控跨操作系统、实例类型、可用性区域和地区的不同容量池，以便在中断发生之前，实时决定选择哪些实例进行配置，以及哪些实例需要主动重新平衡和替换。

一天之内，我们会被打断 40 到 60 次。虽然 Spot 做得很好，但有时实例在重新平衡完全发生之前就被删除了，这可能会导致某些应用程序停机。

为了最大限度地减少中断的可能性并避免任何停机时间，这些是我们在过去几年中采取的一些措施，使我们能够在不影响可用性的情况下利用现场实例。

## 副本的最小数量

如上所述，spot 实例可被 AWS EC2 Spot 服务中断。我们经常看到两个实例同时被删除，有时甚至是 4 个。

有可能某个特定应用程序的所有 pod 都在被取走的一个实例中运行。有几种方法可以避免这种情况，比如配置[反亲和](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)规则，或者更好的是 [pod 拓扑分布约束](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)。我们稍后会讨论这些。

在我们的关键应用程序中，默认情况下我们将 pod 的最小数量设置为 4，并且我们强制在不同的实例中调度这些 pod。这样，我们可以承受同时丢失多达 3 个实例，为一个特定的应用程序运行 3 个 pods，而不会导致任何停机。

## **实例类型和可用区域**

在不同的可用性区域中跨许多不同的实例类型部署您的应用程序将进一步增强可用性。当多个实例同时关闭时，它们通常属于同一系列和实例类型，在特定的可用性区域中，因为对该特定实例类型的需求很高。

我们配置我们的集群来运行各种各样的实例类型。同时，我们将特定应用的 pod 分布在多个可用性区域，最大限度地降低了服务中断的风险。

## Pod 中断预算

一个 [Pod 中断预算](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets) (PDB)允许您在其 Pod 遭受自愿中断时限制对应用程序的中断:

*   删除管理 pod 的部署或其他控制器
*   更新部署的 pod 模板导致重新启动
*   直接删除 pod
*   排出节点以进行修复或升级。
*   从集群中排出节点以缩小集群
*   从一个节点上移除一个 pod 以允许在该节点上安装其他东西。

PDB 限制了因自愿中断而同时停机的复制应用程序单元的数量。这在集群升级过程中特别有用，因为节点将按照特定的顺序被清空和升级，因此 pdb 将被考虑。

```
**apiVersion**: policy/v1
**kind**: PodDisruptionBudget
**metadata**:
  **name**: test-svc
**spec**:
  **maxUnavailable**: 1
  **selector**:
    **matchLabels**:
      **app**: test-svc
```

## Pod 反亲和性

正如[官方文档](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity)中所述，pod 反亲缘关系允许您根据已经在节点 上运行的 pod 上的标签，而不是根据节点上的标签，来约束哪些节点您的 pod 有资格被调度 ***。规则的形式是*“如果 X 已经在运行一个或多个符合规则 Y 的 pod，则该 pod 不应在 X 中运行”。* Y 表示为 LabelSelector。***

从概念上讲，X 是一个拓扑域，如节点、云提供商区域、云提供商区域等。你用一个`topologyKey`来表达。

在下面的示例中，不能在已经运行标签为`app: app-name`的 pod 的节点中安排 pod。

```
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - app-name
        topologyKey: kubernetes.io/hostname
```

直到最近，我们一直使用 pod 反亲缘关系在 spot 实例之间传播 pod，从而降低停机风险。然而，当您的应用程序有大量副本，或者根据 cron 计划伸缩时，pod 反相似性就不太好用了(请看我关于[事件驱动自动伸缩](/event-driven-autoscaling-503b5cefaa49)的文章)。例如，一个扩展到 30 个 pod 的应用程序需要 30 个不同的节点来调度所有 30 个 pod。这既没有效率，也没有性能，因为应用程序必须等待新节点添加到集群中，而这些节点很可能有一半是空的。

Pod 拓扑分布约束修复了此问题。

## Pod 拓扑分布约束

在 Kubernetes v1.19 中提升为稳定， [Pod 拓扑分布约束](https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/)帮助您控制 Pod 如何在您的集群中的域(如区域、分区、节点和其他用户定义的拓扑域)之间分布。这有助于实现高可用性以及高效的资源利用。

通过以下配置，我们能够跨不同的可用性区域和不同的节点均匀地调度 pod:

```
topologySpreadConstraints:
  - labelSelector:
      matchLabels:
        app: test-svc
    maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
  - labelSelector:
      matchLabels:
        app: test-svc
    maxSkew: 2
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: ScheduleAnyway
```

与 pod 反关联性不同，通过 Pod 拓扑分布约束，我们可以让多个 Pod 用于在同一个节点上运行的同一个应用程序。但是，由于我们使用 3 个可用性区域，第一个拓扑约束将确保至少有 3 个 pod 在 3 个不同的实例中运行。第二个拓扑约束将尝试在不同的实例(`topologyKey: kubernetes.io/hostname`)中调度第 4 个 pod，但这是没有保证的(`ScheduleAnyway`告诉调度器仍然调度它，同时优先考虑使偏斜最小化的节点)。

**maxSkew** 描述了豆荚分布不均匀的程度。您必须指定此字段，并且数字必须大于零。它的语义根据`whenUnsatisfiable`的值而不同:

*   `whenUnsatisfiable: DoNotSchedule`:然后`maxSkew`定义目标拓扑中匹配 pod 数量与*全局最小值*之间的最大允许差值。例如，如果您有 3 个区域，分别有 2 个、2 个和 1 个匹配窗格，`MaxSkew`设置为 1，则全局最小值为 1。
*   如果您选择`whenUnsatisfiable: ScheduleAnyway`，调度程序会优先考虑有助于减少偏差的拓扑。

## **其他有助于可靠性的设置**

在 Kubernetes 中还有其他一些设置有助于运行高可用性的应用程序，无论您是否在 Spot 实例中这样做

**Horizontal Pod auto scaling(HPA):**自动更新工作负载资源(如[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)或 [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) ，目的是自动扩展工作负载以匹配需求(即 CPU、内存或任何其他指标，如果您使用指标适配器，如 [KEDA](/event-driven-autoscaling-503b5cefaa49) )。

**终止宽限期:**SIGTERM 信号发送到 pod 后，Kubernetes 会等待一段指定的时间，称为终止宽限期。默认情况下，这是 30 秒。

如果你的应用在 terminationGracePeriod 结束前关闭并退出，Kubernetes 会立即进入下一步。如果您的 pod 关闭时间通常超过 30 秒，请确保延长宽限期。

**滚动更新策略:**为滚动更新策略中的`maxSurge`、`maxUnavailable`参数设置正确的值，以避免在部署期间出现应用程序的多个副本。

# 结论

如果应用程序按照本文中提到的一些(或全部)参数进行配置，那么使用 Spot 实例运行弹性生产工作负载并不像听起来那么可怕。我们从一开始就这样做，利用成本节约，同时实现高水平的可靠性，为我们的成员提供最佳体验。
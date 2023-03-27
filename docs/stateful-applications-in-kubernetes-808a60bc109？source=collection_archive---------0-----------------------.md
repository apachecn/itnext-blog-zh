# Kubernetes 中的有状态应用程序

> 原文：<https://itnext.io/stateful-applications-in-kubernetes-808a60bc109?source=collection_archive---------0----------------------->

Kubernetes 以管理无状态服务而闻名，但它并不局限于无状态服务。下面这篇文章收集了我关于在 Kubernetes 中运行有状态应用程序的笔记。如果您发现任何不准确之处，想要分享您的经验或想要提出更改建议，请随时联系我；)

在一个地方写下关于有状态应用程序的所有内容是一个过于雄心勃勃的目标。因此，您可以在这里找到简短的陈述，并参考包含每个特定陈述更多详细信息的页面。

![](img/5e346c6b42148090a1fcdf3154ecd7b9.png)

[Susuwatari](https://ghibli.fandom.com/wiki/Susuwatari) 携带他们的资产[吉卜力](http://www.ghibli.jp/gallery/chihiro014.jpg)

有状态应用程序—是一种使用本地文件系统来保存自己的数据的应用程序。在 Kubernetes 中有两种方法可以运行这样的应用程序:

*   StatefulSets — Kubernetes 对象，它管理一组 pod，并提供关于这些 pod 的排序和唯一性的保证。
*   [单实例有状态应用](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/) —具有单个副本和附加卷的部署对象。就我个人而言，我想象不出任何有效的用例能比使用单个副本的 StatefulSet 更有价值。

我认为，要在 Kubernetes 中运行有状态应用程序，我必须:

*   理解什么是状态集合以及如何使用它们；
*   持久层在 Kubernetes 中是如何表示的；
*   为了正确处理应用程序状态，我必须理解 pod 生命周期。

## 1.状态集

1.1. [*StatefulSets*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) 为其中的每个 pod 提供两个稳定的唯一身份:

*   *网络标识*:无论重启次数多少，pod 都分配有相同的 DNS 名称。请注意，IP 地址可能仍然不同，因此消费者应该依赖于 DNS 名称(或者观察变化并更新内部缓存)
*   *存储标识*:相同的网络标识总是接收相同的存储实例，不管它在哪个节点上被重新调度。

1.2.每个 StatefulSet 应该有关联的 Kubernetes 服务实例来管理其网络身份。如果 StatefulSet 不需要内部或外部 IP 的服务，它可以使用[无头服务](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)

每个 pod 根据模式接收 DNS 名称:`{statefulset_name}-{0..N-1}.{service_name}.{namespace_name}.svc.cluster.local`。比如`cassandra-0.cassandra-headless.dev-namespace.svc.cluster.local`。

1.3.声明集是围绕安全第一原则构建的——在每一个有争议/有问题的情况下，Kubernetes 团队都会选择数据安全。例如，默认情况下，当 StatefulSet 被删除或扩展时，关联的卷不会被删除。

1.4.由于 StatefulSets 处理数据，我们在停止 pod 实例时应该小心:给足够的时间将数据从内存持久化到磁盘，完成启动的操作，等等。仍然可能有有效的理由执行[强制删除](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/)，例如，当 [Kubernetes 节点失败](https://medium.com/tailwinds-navigator/kubernetes-tip-how-statefulsets-behave-differently-than-deployments-when-node-fails-d29e36bca7d5)时。

1.5.[部署和升级](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#deployment-and-scaling-guarantees)

默认情况下，StatefulSet 按顺序转出:按从 0 到 N-1 的顺序创建；从 N-1 到 0 终止。可以将其更改为并行模式，但大多数情况下并不需要

一个有趣的属性，部署参数是[分区](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions):它允许金丝雀从盒子中释放。

1.6.我强烈建议浏览这些精彩的教程:

*   [StatefulSet Basics](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/) ，主要讲述基本操作
*   [运行 ZooKeeper](https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/) ，它考虑了一个真实世界的场景，并显示了更高级的主题，例如，pod-affinity 和 disruption-budget。

## 2.持久卷

2.1.主要对象:

*   [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV) —集群中参考物理数据位置的一块存储
*   [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)(PVC)—用户的存储请求。通常用于将状态集映射到 PVs
*   [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) 定义 PV 的参数:provisioner (AzureDisk、AWSElasticBlockStore 等)、回收策略、允许调整大小等

> 它类似于一个豆荚。Pods 消耗节点资源，PV 消耗 PV 资源。Pods 可以请求特定级别的资源(CPU 和内存)。声明可以请求特定的大小和访问模式

2.2.卷配置:

*   *静态*:管理员预先创建卷和 PV，然后 PVC 消耗现有 PV。
*   *动态*:创建 PVC 时，集群尝试配置卷。

当没有定义*存储类*时，集群使用`DefaultStorageClass`

2.3.[回收](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming)

有以下回收政策:

*   *保留*:如果 PVC 被删除，PV 会保留，但对其他用户不可用(因为可能会有数据！).操作员应决定如何处理卷:按原样将其分配给新的 pod，在分配前清理或删除。
*   *删除*:从 Kubernetes 中删除 *PersistentVolume* 对象，以及外部基础设施中的相关存储资产(并非所有提供商都支持)

2.4.卷插件:

*   *in-tree* :源代码在主 Kubernetes git-repository 中
*   *树外*:插件，使存储供应商能够创建定制的存储插件，而无需将其插件源代码添加到 Kubernetes 存储库:[容器存储接口](https://github.com/container-storage-interface/spec/blob/master/spec.md) (CSI)和 FlexVolume

很可能，CSI 是 Kubernetes 的未来，但并不是每个人都支持它(例如，目前亚马逊 EBS [CSI 驱动程序处于测试阶段](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)，Azure 只有预览版中的 [CSI 驱动程序)。此外，CSI 卷可以在 PVC 创建时被克隆。](https://docs.microsoft.com/en-us/azure/aks/azure-disk-csi)

2.5.[卷快照](https://kubernetes.io/docs/concepts/storage/volume-snapshots)(1.17 中的测试版)

*   如*持续卷*和*持续卷声明* — *卷快照内容*和*卷快照*
*   可以被安排

## 3.Pod 生命周期

3.1.通常，在 StatefulSet 中，我们运行更复杂的应用程序，需要预热、领导者选举、数据处理等。因此，StatefulSet 中的 pod 可能对如何开始/停止、何时接收流量等有特殊的要求。

我认为，一旦你不得不使用状态集，你就必须仔细阅读 [Pod 生命周期文档](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)，它解释了所有可能的状态以及它们之间的转换

3.2.[探头](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes):

*   `startupProbe`表示 pod 准备好接收其他探针的时间。该探针仅在开始时执行，而接下来的两个探针在整个 pod 生命周期内执行
*   `livenessProbe`表示容器是否正在运行。当探针出现故障时— pod 重新启动。
*   `readinessProbe`表示 pod 是否应该接收入口流量。当探针出现故障时— pod 从端点移除(例如 iptables)

3.3.[挂钩](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks)

*   `PostStart`钩子在容器创建后立即执行(甚至可能在容器入口点之前！)
*   `PreStop`这个钩子在容器终止之前被调用。可能有助于准备终止 pod:停止接收新请求，将内存中的数据转储到磁盘，等等。

3.4. [Pod 终止](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)

当 API 服务器收到删除 Pod 的命令时，它:

*   更新*端点*，导致 iptables、Ingress 和 DNS 发生变化
*   向 *kubelet* 发送命令以停止 pod，这将调用 *preStop* 钩子并等待*terminationgraceperiodes*秒(默认为 30 秒)

关于 pod 正常关闭的非常详细的解释:【https://learnk8s.io/graceful-shutdown.它给出了很好的可视化效果，并在最后提出了一个有趣的模式来处理长时间运行的 pod:在新版本发布期间，不是更新相同的部署，而是创建一个新的部署(我不想重复文章内容，如果你很好奇，可以去看看)。

3.5.[中断](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)可能是:

*   *自愿*:删除部署或吊舱、秤等
*   *非自愿*:虚拟机故障，硬件故障， [OOM](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/)

为了减少中断:

*   使用资源配额
*   复制应用程序
*   使用[中断预算](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)

借助中断预算，您可以:

*   定义最小可用或最大不可用的实例数量；
*   除非中断预算被消除，否则防止任何重新安排

## 4.多方面的

以下主题不属于以上任何一组，但仍然值得一提

4.1.最初，状态集被命名为[petset](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/apps/stateful-apps.md#history)

4.2.

> 在相应的后端工作负载之前创建服务

*   k8s 在创建时为每个容器提供带有服务主机/端口的环境变量，如果 pod 是在服务之前创建的—没有环境变量
*   服务创建导致新的 DNS 记录。如果 pod 在创建服务之前查询 DNS 不存在的结果会被缓存一段时间

4.3.`imagePullPolicy: Always`并不是每次都随便拉图。它每次都会查询容器注册表，并且仅当(a)映像不在本地虚拟机缓存中或者(b)本地缓存的映像摘要与容器注册表中的不同时，才会提取映像。

4.4.

> 支持所有权管理的卷被修改为由 fsGroup 中指定的 GID 拥有和可写

因此，您可以使用`securityContext.fsGroup`,而不是编写定制的所有权变更脚本
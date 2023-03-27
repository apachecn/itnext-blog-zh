# Kubernetes 节点关联:将 pod 放在特定的节点上

> 原文：<https://itnext.io/kubernetes-node-affinity-placing-pods-on-specific-nodes-8ea918dda9b9?source=collection_archive---------3----------------------->

![](img/b1ecf00053a6edb82504ecf3e5d3eec7.png)

有些情况下，我们需要在 Kubernetes 集群中的特定节点上运行一些应用程序。例如，有一个应用程序需要的资源比群集中的任何单个节点都多。在这种情况下，我们需要创建一个新节点，并将这个应用程序放在这个新节点中。我们如何做到这一点？Kubernetes 会允许我们将 pod 分配到我们决定的特定节点上吗？当然可以。但是要做到这一点，我们需要给 Kubernetes scheduler 一些小的输入。Kubernetes 调度器可以被限制为使用几个不同的选项将一个 pod 放在特定的节点上。

1.  [节点选择器](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)
2.  [节点亲和力](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity)
3.  [Pod 亲和力](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity)
4.  [污点和宽容](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

在这篇文章中，我将重点介绍第二种方法，因为它允许我们将 pod 放置到特定的节点，无论是作为硬规则还是软规则。硬性规则意味着，必须满足该规则才能将一个 pod 调度到给定的节点上。软规则意味着如果调度程序不能满足我们的约束，pod 仍然会被调度到不同的节点。因此调度程序会尝试强制执行，但不会保证 pod 满足我们的调度约束。

让我们来看看如何使用节点关联机制将 pod 放置到特定节点上的步骤。

1.  列出群集中的节点及其标签:

```
kubectl get nodes --show-labels
```

2.选择要运行应用程序的节点，并为其添加标签:

```
kubectl label nodes <node-name> <label-key>=<label-value>
```

可以选择任何键:值对来标记节点。我将使用“type:t2medium”作为我的节点的示例键值对。

3.现在我们需要创建一个调度到所选节点的 pod:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
     ** affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: type
                operator: In
                values:
                - t2medium**
      containers:
      - image: nginx
        name: nginx
```

将上述部署 yaml 文件应用到您的集群，并验证 pod 正在所选节点上运行:

```
kubectl get pods --all-namespaces -o wide
```

上面的命令将向我们显示 pod 信息以及它运行的节点。然后，您可以验证 pod 是否运行在具有相同键值对(例如:type:t2medium)的所选节点上。

除了`requiredDuringSchedulingIgnoredDuringExecution`类型的节点关联性之外，还存在`preferredDuringSchedulingIgnoredDuringExecution`。第一个可以被认为是一个“硬”规则，而第二个构成了一个“软”规则，Kubernetes 试图强制执行，但不会保证。

您可以指明该规则是“软”的，而不是硬的要求，因此如果调度程序不能满足它，pod 仍然会被调度。

在调度器过滤掉不符合特定要求的节点后，它会为每个剩余节点计算一个优先级分数，以找到“最佳”节点。这个分数是由多个内置的优先级函数相加得到的，如`LeastRequestedPriority`、`BalancedResourceAllocation`或`SelectorSpreadPriority`。`NodeAffinityPriority`也是这些功能之一，因此当一个 pod 进入调度程序并且`preferredDuringSchedulingIgnoredDuringExecution`被设置为按需节点时，它将为集群中的每个节点计算一个优先级分数，并且仅对`matchExpression`为真的节点将`weight`字段的值添加到该分数中。因此，这些节点对于`NodeAffinityPriority`将具有更高的优先级分数，但是可能发生的情况是，其他节点上的其他优先级函数平衡了这个更高的优先级。因此，即使在`matchExpression`为真的节点上仍然有空闲容量，pod 也可以放在其他地方。

那么对于将`weight`设置在 1-100 范围内有什么好的建议？很简单:您无法预先计算每个节点的优先级分数，因此根据经验，您越希望满足自己的偏好，就需要设置越高的权重。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  affinity:
 **   nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: type
            operator: In
            values:
            - t2medium**
  containers:
  - name: nginx
    image: nginx
```

同样，通过节点关联，我们可以使用每个节点上的标签告诉 Kubernetes 需要在哪个节点上调度哪些 pods。希望这篇博客能帮助你理解什么是节点亲和性，以及在调度你的 pod 时如何使用它。让我们很快在另一篇文章上见面吧！！
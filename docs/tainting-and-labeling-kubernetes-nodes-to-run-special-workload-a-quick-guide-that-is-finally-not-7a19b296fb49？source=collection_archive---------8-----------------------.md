# 污染和标记 Kubernetes 节点以运行特殊的工作负载——这是一个最终不会令人困惑的快速指南

> 原文：<https://itnext.io/tainting-and-labeling-kubernetes-nodes-to-run-special-workload-a-quick-guide-that-is-finally-not-7a19b296fb49?source=collection_archive---------8----------------------->

![](img/4cd98066b7f15097e0134cd714bc569f.png)

好了，伙计们，我打算长话短说，这就是我要做的。我的意思是，这应该很容易，但官方文档( [1](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) ， [2](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node) )让它变得不必要的混乱。所以我想也许我可以填补这个空白。

我将在这个项目中使用我们在 Buffer [中的一个业务需求，作为这篇博文的例子。](/how-to-set-kubernetes-resource-requests-and-limits-a-saga-to-improve-cluster-stability-and-a7b1800ecff1)

# 快速回顾

因此，我们需要几个专用于运行 cronjobs 的节点，除此之外别无其他。同时，我们希望确保玉米作业被调度到这些节点，而不是其他节点。这意味着我们需要两样东西

*   不承担其他工作负载的受感染节点
*   仅到达目标节点的工作负载

现在，让我们从节点开始，然后是工作负载

# 节点

由于需求被分解为两个方面(见上文)，我们需要为节点指定两件事情。一如既往，kops 是我的首选武器。

在 kops 中，你可以这样做`kops edit ig <INSTANCE GROUP IN INTEREST>`

# 污染节点

这将阻止其他工作负载被调度给它们。这是通过这两条线实现的

```
taints: 
- dedicated=frequent-cronjob-nodes:NoSchedule
```

# 标记节点

这有助于专门的工作负载定位节点。这是通过这两条线实现的

```
nodeLabels: 
  kops.k8s.io/instancegroup: frequent-cronjob-nodes
```

我知道有些人不使用 kop。如果你是其中之一，这里有两个命令可以帮助你

`kubectl taint nodes <NODE IN INTEREST> dedicated=frequent-cronjob-nodes:NoSchedule`

`kubectl label nodes <NODE IN INTEREST> kops.k8s.io/instancegroup=frequent-cronjob-nodes`

# 工作量

与节点类似，我们需要对 deployment/cronjob yaml 文件做两件事。我包含了一个完整的 yaml 来保护我们的眼睛[免受这个](https://twitter.com/caged/status/1039937162769096704)的伤害(是的，你知道我在说什么)。

# 容忍污点

这确保了可以将工作负载调度给受感染的节点。这是通过这些线实现的

```
tolerations: 
- key: dedicated 
  value: frequent-cronjob-nodes 
  operator: "Equal" 
  effect: NoSchedule
```

# 指定目标节点

这确保了只将工作负载调度到指定的节点。这是通过这两条线实现的

```
nodeSelector: 
  kops.k8s.io/instancegroup: frequent-cronjob-nodes
```

# 利润

这就是了。现在，我们可以放心了，正确的工作负载将被分配到正确的节点。这样，我们可以开始为专门的工作负载构建一些专门的节点组，比如用于机器学习的 GPU 节点或用于本地缓存的内存密集型节点。

我希望这有所帮助。下次，如果您有任何问题，请随时通过 [Twitter](https://twitter.com/stevenc81) 联系我。:D

*最初发表于*[T5【http://github.com】](https://gist.github.com/stevenc81/494044f7b90c75efdc0914200824dce2)*。*
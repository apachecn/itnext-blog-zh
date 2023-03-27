# 如何加快 Kubernetes 集群的节点扩展

> 原文：<https://itnext.io/how-to-faster-node-scale-your-kubernetes-cluster-2c988c7c7d08?source=collection_archive---------1----------------------->

## 再也不要在这可怕的几分钟里等待新节点上线

![](img/3b0436fce43e052be4f796292fa30990.png)

Kubernetes 集群自动缩放器(或节点自动缩放器)非常棒。启用后，如果群集中有挂起的 pod，它会请求新节点。

集群自动缩放器[是 Kubernetes](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) 的一部分，提供特定节点设置的建议。如果有人听了这个建议(像 GCP，AWS，Azure，DigitalOcean)，节点将被添加或删除。

## 待定窗格

如果没有更多资源可用于调度新的 pod，K8s 中的 HPA 和 VPA 会导致 pod 处于未决状态。这将触发集群自动缩放。

## 问题

根据您的云提供商，启动新节点可能需要一些时间。对于 Azure 和一个 2CPU-2GB 的节点，5 分钟以上。

## TL；速度三角形定位法(dead reckoning)

创建具有较低优先级的占位符单元，作为将由 HPA 创建的单元。这样，当创建其他窗格时，占位符窗格将立即被替换。**缺点**:如果需要的话，至少还有一个节点在运行。这是你为速度付出的代价。

## 有关系的

1.  [如何使用 K8s 优先级的实际例子](https://medium.com/faun/kubernetes-cka-hands-on-challenge-6-pod-priority-1fe95f613ac5?source=friends_link&sk=7f9ea22abbe4b0c5c1758e756c3ff2cf)
2.  [Kubernetes 卧式容器自动缩放实用指南](https://codeburst.io/practical-guide-to-kubernetes-scaling-1-pods-5a7ed08f4e8b?source=friends_link&sk=22602bf9789af6112fa53e9d20c05ed0)
3.  [Kubernetes 节点自动缩放实用指南](https://codeburst.io/practical-guide-to-kubernetes-node-scaling-5a7fc3499a56?source=friends_link&sk=ac4e04e5bc9a21197871ecdc8ccec911)
4.  [垂直 Pod 自动缩放](/k8s-vertical-pod-autoscaling-fd9e602cbf81?source=friends_link&sk=df7289cb35bcfdfa7f191546e6d555b6)

# 优先级占位符

我们想要创建占位符窗格。

## 负优先级

具有负优先级的单元将被集群自动缩放器忽略。这些被认为是“填充物”:

> 非关键工作负载可能有更多可以放入集群的单元。如果您为非关键工作负载设置了负优先级，那么当非关键 pod 挂起时，Cluster Autoscaler 不会向您的集群添加更多节点。因此，你不会产生更高的费用。当您的关键工作负载需要更多计算资源时，调度程序会抢占非关键单元并调度关键单元。([来源](https://kubernetes.io/blog/2019/04/16/pod-priority-and-preemption-in-kubernetes/))

因此，我们需要给 pod 的优先级≥ 0。

## 创建优先级类别

```
**apiVersion**: scheduling.k8s.io/v1
**kind**: PriorityClass
**metadata**:
  **name**: placeholder
**value**: 0
**preemptionPolicy**: Never
**globalDefault**: false
**description**: 'placeholder'
---
**apiVersion**: scheduling.k8s.io/v1
**kind**: PriorityClass
**metadata**:
  **name**: normal
**value**: 1
**preemptionPolicy**: Never
**globalDefault**: true              # default
**description**: 'normal'
```

我们将 PriorityClass `normal`定义为所有 pod 的默认值。

## 创建占位符部署

```
**apiVersion**: apps/v1
**kind**: Deployment
**metadata**:
  **name**: placeholder
**spec**:
  **replicas**: 2
  **selector**:
    **matchLabels**:
      **app**: placeholder
  **template**:
    **metadata**:
      **labels**:
        **app**: placeholder
    **spec**:
      **terminationGracePeriodSeconds**: 0        # important
      **priorityClassName**: placeholder          # important
      **containers**:
      - **image**: nginx
        **name**: placeholder
        **resources**:
          **requests**:
            **cpu**: 1500m
```

我们需要根据节点大小来定义资源请求。如果我们将 CPU 请求设置为`2000m`，它可能不会触发集群自动缩放，因为它甚至不能在新的 2CPU 节点上进行调度，所以我们将其设置得稍低一些。副本数量指定了我们想要保留的节点数量。在上面的示例代码中，我们在 Azure 上保留了 2 个 2CPU-2GB 的节点。

**还要检查 terminationGracePeriodSeconds**，这确保占位符 pod 不会在 30 秒内(默认)处于终止状态，在此期间，它仍然被调度并且仍然会使用资源。

# 虚拟库伯勒

也看看[的虚拟库伯莱](https://virtual-kubelet.io/)和[我关于它的文章](/kubernetes-without-nodes-caedd172f940?source=friends_link&sk=a7cd94bbb7e8776fa4f0802de4e41094)。它将允许您在云容器引擎而不是节点上运行 pods，并可以让您更快地扩展。

# 概述

速度还是金钱，这是这里的问题。

# 成为 Kubernetes 认证

[![](img/cf3901a56841fcb55f9e4e17b9f07672.png)](https://killer.sh)

[https://killer.sh](https://killer.sh)
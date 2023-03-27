# 如何用 Kubectl 重启 Kubernetes Pods

> 原文：<https://itnext.io/how-to-restart-kubernetes-pods-with-kubectl-2a7834a6b961?source=collection_archive---------0----------------------->

豆荚是 Kubernetes (K8S)的最小单位。它们应该一直运行，直到被新的部署所取代。正因为如此，没有办法重启一个 pod，反而要更换。

![](img/8ca69e3676a79c4bd8719c5e0525da06.png)

照片由 [Gia Oris](https://unsplash.com/@giabyte?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/start?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

K8S 没有使用`kubectl restart [podname]`命令(使用 Docker 你可以使用`docker restart [container_id]`)，所以有几种不同的方法用`kubectl`实现 pod 重启’。

在几种情况下，您可能需要“重新启动”pod，通常是当 pod 状态停留在“终止”状态时，或者当 pod 出现错误并且不重新启动无法修复该错误时。在某些情况下，您可能需要在修改配置后重新启动 pod，例如修改资源限制以限制 pod 的内存限制，或者更改 pod 正在使用的永久卷存储。

请注意，要重新启动 pod 中的特定容器，您可以使用下面详述的方法 6，而不必重新创建 pod。

## Pod 状态

pod 有五种可能的状态:

1.  **待定:**此状态表示 pod 中至少有一个容器尚未创建。
2.  **正在运行:**所有的容器都已经创建，并且 pod 已经绑定到一个节点。此时，容器正在运行，或者正在启动或重新启动。
3.  **成功:**pod 中的所有容器都已成功终止，不会重新启动。
4.  **失败:**所有容器都已终止，至少有一个容器失败。失败的容器处于非零状态。
5.  **未知:**无法获得 pod 的状态。

如果您注意到 pod 处于不良状态，状态显示为“错误”,您可以尝试“重启”作为故障排除的一部分，以恢复正常操作。您也可能会看到“CrashLoopBackOff”状态，这是遇到错误时的默认状态，K8S 会尝试自动重启 pod。

## 重启一个 pod

您可以使用以下方法通过`kubectl.`来“重启”一个 pod。一旦新的 pod 被重新创建，它们将具有与旧的不同的名称。使用`kubectl get pods`命令可以获得吊舱列表。

## 方法 1

*   `**kubectl rollout restart**`

这种方法是推荐的第一停靠港，因为它不会在吊舱运行时引入停机时间。首次展示重启将一次杀死一个单元，然后新单元将按比例增加。此方法可从 K8S v1.15 开始使用。

```
kubectl rollout restart deployment <deployment_name> -n <namespace>
```

## 方法 2

*   `**kubectl scale**`

这种方法会导致停机，不推荐使用。如果停机不是问题，可以使用这种方法，因为它是`kubectl rollout restart`方法的更快的替代方法(在重新部署之前，您的 pod 可能必须经过漫长的持续集成/持续部署过程)。

如果没有与部署相关联的 YAML 文件，您可以将复制副本的数量设置为 0。

```
kubectl scale deployment <deployment name> -n <namespace> --replicas=0
```

这终止了豆荚。缩放完成后，副本可以根据需要按比例放大(至少放大到 1 倍):

```
kubectl scale deployment <deployment name> -n <namespace> --replicas=3
```

在缩放过程中，可以使用以下方法检查 Pod 状态:

```
kubectl get pods -n <namespace>
```

## 方法 3

*   `**kubectl delete pod**` **和** `**kubectl delete replicaset**`

如果需要，可以单独删除每个 pod:

```
kubectl delete pod <pod_name> -n <namespace>
```

这样做将导致 pod 被重新创建，因为 K8S 是声明性的，它将基于指定的配置创建一个新的 pod。

然而，在大量的 pod 运行的情况下，这实际上并不是一种实用的方法。但是，如果许多窗格具有相同的标签，您可以使用该标签一次选择多个窗格:

```
kubectl delete pod -l “app:myapp” -n <namespace>
```

另一种方法如果有许多 pod，则可以删除 ReplicaSet:

```
kubectl delete replicaset <name> -n <namespace>
```

## 方法 4

*   `**kubectl get pod | kubectl replace**`

使用`kubectl get pod` 获取当前正在运行的 pod 的 YAML 语句，并通过指定的`--force`标志将其传递给`kubectl replace`命令，可以检索要更换的 pod，以实现重启。如果没有可用的 YAML 文件并且 pod 已启动，这将非常有用。

```
kubectl get pod <pod_name> -n <namespace> -o yaml | kubectl replace --force -f -
```

## 方法 5

*   `**kubectl set env**`

设置或更改与 pod 关联的环境变量将导致它重新启动以进行更改。以下示例将环境变量`DEPLOY_DATE`设置为指定的日期，导致 pod 重新启动。

```
kubectl set env deployment <deployment name> -n <namespace> DEPLOY_DATE="$(date)"
```

## 方法 6

这种方法将允许您杀死一个特定容器的主进程，这将触发该容器的重启，而无需 K8S 杀死 pod 并重新创建它。

```
kubectl exec -it <pod_name> -c <container_name> --/bin/sh -c "kill 1"
```

这将向进程 1 发送一个`SIGTERM`信号，进程 1 是在容器中运行的主进程。所有其他进程都将是进程 1 的子进程，并将在进程 1 退出后终止。请参见 [kill 联机帮助页](http://linuxcommand.org/lc3_man_pages/kill1.html)了解您可以发送的其他信号。

## 摘要

当遇到 K8S 中的 pod 问题时，工具箱中有一系列命令可供使用，这将使您能够使用适当的方法来重启它们，具体取决于您部署 pod 的方式、应用程序正常运行的必要性以及您需要重启的速度。一般来说，最好的方法是使用上述的`kubectl rollout restart`方法，因为它可以避免应用程序停机。

重启并不能解决最初导致 pod 出现问题的问题，因此需要进一步调查根本原因。

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper) 

*原载于*[*space lift . io*](https://spacelift.io/)*。*
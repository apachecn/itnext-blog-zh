# kubernetes crash loop 回退状态

> 原文：<https://itnext.io/kubernetes-crashloopbackoff-status-d254dd642d10?source=collection_archive---------0----------------------->

## 排除故障并修复错误！

Kubernetes (K8S)群集中的一个 pod 的状态可能会显示“CrashLoopBackoff”错误，这是在一个 pod 崩溃并多次尝试重新启动时显示的。在本文中，我们将介绍如何发现错误，如何修复错误，以及出现错误的一些原因。

![](img/91d032b7eb1b6a94214343d18b7d0f4e.png)

马库斯·斯皮斯克在 [Unsplash](https://unsplash.com/s/photos/crash?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

## 到底什么是 CrashLoopBackoff 错误？

K8S `kubelet`在尝试再次启动吊舱之前，将在两次碰撞之间等待一个不断增加的“后退”时间。K8S `restartPolicy`默认设置为“始终”，如果`restartPolicy`已更改为“onFailure”，也会出现这种情况。重新启动之间的这段时间让我们有时间进行故障诊断，以尝试修复问题。最大“回退”时间为 5 分钟，从 10 秒开始，呈指数增长，直到达到最大值。当 pod 显示 CrashLoopBackoff 状态时，这意味着它正在等待“Backoff”指定的给定时间来重新启动，并且可能会再次失败。在再次崩溃之前，容器可能会短暂地显示为“正在运行”。

## 如何找到“CrashLoopBackoff”错误

要显示窗格的状态，请运行以下命令:

```
kubectl get pods -n <namespace>
```

状态部分将显示 pod 状态。如果 pod 处于 CrashLoopBackOff 状态，它将显示为*未*就绪，(如下图 0/1 所示)，并将显示 0 次以上的重启。

```
NAME                     READY     STATUS             RESTARTS   AGE
nginx-5796d5bc7d-xtl6q   0/1       CrashLoopBackOff   4         1m
```

“正常”状态包括:

*   运转

pod 运行正常，没有任何问题。

*   等待

pod 仍在启动，它可能正在提取容器图像，或者正在接收秘密数据。一旦完成，它应该转换到运行状态。

*   终止的

具有此状态的 pod 要么运行完成，要么由于某种原因失败。

## 故障排除具有 CrashLoopBackOff 状态的 pod

下面列出的四个 kubectl 命令是开始对出错的 pod 进行故障诊断的推荐方法。

1.  `kubectl describe pod`
2.  `kubectl logs`
3.  `kubectl get events`

从`kubectl describe`命令开始，您可以使用它来获得更多细节。

```
kubectl describe pod <pod name> -n <namespace>
```

pod 状态部分将显示与 pod 相关的任何错误消息。输出的 events 部分将为您提供关于 pod 状态的信息。查找包含“回退重启失败的容器”的条目，如下例所示。

```
Name:         pod name
Namespace:    default
Priority:     0
…
State:        Waiting
Reason:       CrashLoopBackOff
Last State:   Terminated
Reason:       Error
…
Warning  BackOff                1m (x5 over 1m)   kubelet, ip-10-0-9-132.us-east-2.compute.internal  Back-off restarting failed container
…
```

接下来，使用`kubectl logs`命令检查故障 pod 的日志。`-p`(或`--previous`)标志将从 pod 的最后一个失败实例中检索日志，这有助于了解应用程序级别发生了什么。可以使用`--all-containers`标志指定所有容器或一个容器的日志。您可以通过添加`---tail`标志来查看日志文件的最后一部分。

```
kubectl logs <pod name> -n <namespace> -pkubectl logs <pod name> -n <namespace> --previouskubectl logs <pod name> -n <namespace> --all-containerskubectl logs <pod name> -n <namespace> -c mycontainerkubectl logs <pod name> -n <namespace> --tail 50
```

接下来，您应该使用`kubectl get events`命令检查 K8S 事件，并查看 pod 崩溃之前的事件。您可以使用`--sort-by=`标志按时间戳排序。要查看单个 pod 中的事件，请使用`--field-selector`标志。

```
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestampkubectl get events -n <namespace> --field-selector involvedObject.name=<pod name>
```

输出将显示在列表中，示例如下:

```
kube-system 60m Normal Pulling pod/node-problem-detector-vmcf2                        pulling image "k8s.gcr.io/node-problem-detector:v0.7.0"kube-system 60m Normal Pulled pod/node-problem-detector-vmcf2                        Successfully pulled image "k8s.gcr.io/node-problem-detector:v0.7.0"kube-system 60m Normal Created pod/node-problem-detector-vmcf2                        Created containerkube-system 60m Normal Started pod/node-problem-detector-vmcf2
```

## CrashLoopBackOff 错误的原因及防止方法

导致 CrashLoopBackOff 错误的原因有很多，下面列出了一些常见的原因:

1.  容器的错误配置—检查配置文件中的拼写错误或错误配置的值。
2.  内存或资源不足—检查是否正确指定了资源限制。这可能是由交通或活动的突然或意外增加引起的。检查配置文件的“资源:限制”部分。
3.  两个或多个容器被配置为使用同一个端口，如果它们在同一个 pod 中，这将导致错误。
4.  由于其他 pod 正在使用某个文件或数据库，该文件或数据库被锁定，pod 正在尝试连接该文件或数据库。
5.  pod 可能引用不存在的资源或包，例如可以在容器或永久存储卷中找到的脚本。
6.  部署软件时出现一般错误—软件特有的任何错误和异常。
7.  命令行参数可能不正确或丢失。
8.  活性探测器配置不正确—请检查配置文件。
9.  指定的权限不正确或授予的权限不足。
10.  pod 试图写入的文件系统或文件夹是只读的。
11.  连接问题—网络配置不正确或 DNS 不可达。`kube-dns`可能未运行且容器无法联系外部服务。
12.  设置了不正确的环境变量—使用 env 检查环境变量。
13.  在将身份分配给 pod 的情况下，例如在 Azure Kubernetes 服务中，Azure Active Directory 中的托管身份被错误分配或无法访问。

## 摘要

CrashLoopBackoff 状态是一个通知，通知 pod 由于错误正在重新启动，并且正在等待指定的“回退”时间，直到它将尝试再次启动。

运行上面详细介绍的步骤应该有助于您找到状态的根本原因并纠正问题。

想要更多 Kubernetes 内容？查看我在 K8S 上的其他文章[这里](https://jackwesleyroper.medium.com/kubernetes-k8s-related-articles-index-54718769e390)！

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper) 

*原载于*[*space lift . io*](https://spacelift.io/)*。*
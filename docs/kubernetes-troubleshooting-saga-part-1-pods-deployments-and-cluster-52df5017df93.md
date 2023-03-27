# Kubernetes 故障排除传奇第 1 部分:豆荚，部署和集群。

> 原文：<https://itnext.io/kubernetes-troubleshooting-saga-part-1-pods-deployments-and-cluster-52df5017df93?source=collection_archive---------2----------------------->

![](img/e2b65601749b1eec9ed7283587de13c9.png)

当你迷失在日志中…

Kubernetes 于 2015 年发布，世界各地的许多公司已经接受它作为他们的容器编排器。在许多组织中，整个平台本身都构建在 kubernetes 之上。尽管 kubernetes 似乎是许多容器和平台用例的灵丹妙药，但对它进行故障排除仍然是一件痛苦的事情。

在过去的三年里，我一直在与 kubernetes 合作，我遇到了许多问题。我想写这个系列来帮助像我一样是 kubernetes 爱好者的开发者和 SRE。这应该是这个系列的第一部分，我会继续添加更多不同主题的指南。以下是该系列的第二部分:

[Kubernetes 故障排除传奇第二部分:网络和 DNS](https://medium.com/@krishna.sharma1408/kubernetes-troubleshooting-saga-part-2-networking-and-dns-connectivity-7f11013f6148)

# 有用的工具:

拥有像 Prometheus 或 Datadog 这样的监控工具必须能够更深入地了解您的集群。为了进一步调试一些问题，Kubectl 之类的 CLI 工具很方便。此外，还有许多其他 CLI 工具可用于调试错误或轻松获取有关部署的信息。下面是一些有用的例子:

*   **斯特恩**:有没有想过如何方便地同时查看所有部署单元的日志？？如果是，那么斯特恩就是你要找的人。[将](https://github.com/wercker/stern)链接到文档。
    举例:`stern kubernetes-dashboard`
*   kubectx :如果您正在使用多个集群，那么 kubectx 可以帮助您在集群之间进行切换。[将](https://github.com/ahmetb/kubectx)链接至文档。
    例如:`kubectx production`
*   **kube-resource-report** :很多时候，问题不在于您的部署，而在于您的节点耗尽了资源。kube-resource-report 有助于了解集群中资源的使用情况。[将](https://github.com/hjacobs/kube-resource-report)链接至文档。
*   **kube-shell** :如果你是自动补全的粉丝。将链接到文档。

除了上面列出的工具之外，还有许多其他可用的 CLI 工具。你可以在谷歌上查看，例如 **kubens** 。

# 部署、pod、Daemonsets 的故障排除:

![](img/fbdd15e82befc9062fef0ad12b502413.png)

让我们假设您的部署或 daemonset webapp 已关闭。起点应该是检查部署 webapp 的 pod 的状态。下面是一些有用的命令，以便更深入地了解:

*   获取属于部署 webapp 的所有窗格的状态:
    `kubectl get pod -l app=webapp`
*   如果上面的命令显示，pod 处于*****【逐出】*** 等状态。，那我们就要深入挖掘了。以 webapp-wertr:
    `kubectl describe pod webapp-wertr`的名称返回故障 pod 状态的详细报告**
*   **如果 describe 命令不能帮助你，你可以查看日志:
    `kubectl logs webapp-wertr`或`stern webapp`**
*   **有时，pod 正在运行，但您的服务仍然不能按预期工作。下面的命令可以帮助你在 pod 的容器上运行命令:
    `kubectl exec -it webapp-wertr -- ps -aux`**
*   **节点上的资源不足可能会导致 pod 出现问题。对于 ***daemonsets*** 来说，每个节点都有足够的资源是很重要的。找出 pod 运行的节点:
    `kubectl get pod webapp-wertr -o wide`**
*   **上一点中的命令将告诉我们 pod 正在哪个节点上运行(假设 worker-1)。找出工人-1 的资源:
    `kubectl top node worker-1`**
*   **我们还可以使用:
    `kubectl describe node worker-1`获得关于 worker 节点的详细报告**

# **pod 的常见问题:**

**以下是部署和 pod 最常见的问题:**

## **创建 Pod 需要很长时间:**

*   **拉码头形象可能需要时间。要对此进行检查，请使用 ssh 进入 worker 节点并运行`journalctl -f -u docker`**
*   **如果您使用持久卷声明，那么请确保您使用的是符合您的云的正确配置。创建卷本身也可能需要时间。**
*   **如果 pod 为`pending`或`evicted` ，则可能会因可用资源而出现问题。为 pod 定义的结帐限制和资源。还要检查集群中的资源。**
*   **签出资源配额或限制范围配置。**

## **ImagePullBackoff 或 ErrImagePull:**

*   **确保您输入了正确的图像名称和标签。**
*   **请确保节点具有提取图像的权限。**

## **Pod 在创建后立即崩溃:**

*   **如果 pod 引用任何配置映射或机密，则应该在 pod 之前创建它们。**
*   **如果 pod 中有多个容器，则检查单个容器的状态。为此，您可以使用`kubectl describe`。**
*   **活动性或就绪性探测器可能会出现问题。**
*   **查看日志以获取更多信息。**

## **持续量声明的问题:**

**对于有状态的 pod，卷可能是一个棘手的问题。以下是获取 pvc 信息的一些要求:**

```
**## Persistent volume information:
kubectl get pv
kubectl describe pv <persisten volume name>## Persistent volume claims information:
kubectl get pvc
kubectl describe pvc <pvc name>**
```

**Kubernetes 关于持久性卷的文档非常棒，包含大量信息:**

**[](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) [## 持久卷

### 编辑此页面此文档描述了 Kubernetes 中持久卷的当前状态。熟悉卷…

kubernetes.io](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) 

> ***注意:*** 如果你的 pullpolicy 是 **IfNotPresent** 那么确保你所有的构建都有不同的标签。带有现有标签的新构建将不会被提取，这可能会导致一些奇怪的错误。

# Kubernetes 日志路径:

根据节点的类型，从节点访问日志的方式会有所不同。

## 对于非 systemd 主节点:

*   **API server**:/var/log/kube-API server . log
*   **调度器**:var/log/kube-scheduler . log
*   **控制器管理器**:var/log/kube-scheduler . log

## 对于非 systemd 工作节点:

*   **kube let**:/var/log/kube let . log
*   **kube-proxy**:/var/log/kube-proxy . log

## 对于 systemd 节点:

检查日志的最佳方式是使用 journalctl 命令。使用`journalctl --unit kubelet`检查 kubelet 日志的示例** 

## **以下是更多的参考资料:**

****故障排除服务:**[https://kubernetes . io/docs/tasks/debug-application-cluster/debug-application/](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)**

****集群故障排除:**[https://kubernetes . io/docs/tasks/debug-application-cluster/debug-cluster/](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)**

****Kubernetes dashboard:**[https://Kubernetes . io/docs/tasks/access-application-cluster/we b-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)**

# **最后的话:**

**希望这对你有帮助。如果还有什么可以补充的，请留言**
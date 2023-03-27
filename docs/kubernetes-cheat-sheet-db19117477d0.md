# Kubernetes 备忘单

> 原文：<https://itnext.io/kubernetes-cheat-sheet-db19117477d0?source=collection_archive---------1----------------------->

## 开门见山！

在这篇文章中，我们将列出并描述 Kubernetes (K8S)的每个常用类别或组件，并使用适当的`kubectl`命令进行快速参考！

这些描述直接取自官方文档，网址为 [kubernetes.io](https://kubernetes.io/) ，如果您需要更多信息或更深入的了解，每个标题都将链接到相应的文档页面。

![](img/adf5960398ad724195b194ba17340f53.png)

格伦·卡斯滕斯-彼得斯在 [Unsplash](https://unsplash.com/s/photos/list?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

## 索引

*   常见选项
*   配置文件(清单文件)
*   集群管理和环境
*   达蒙塞特
*   部署
*   事件
*   日志
*   名称空间
*   节点
*   分离舱
*   复制控制器
*   复制集
*   秘密
*   服务
*   服务帐户

# Kubernetes 对象和 Kubectl 命令清单

## 常见选项

在 Kubectl 中，您可以指定可选标志用于各种命令。

`alias`为 Kubectl 设置一个别名。

```
alias k=kubectl
echo 'alias k=kubectl' >>~/.bashrc
```

`-o=json`JSON 中的输出格式

```
kubectl get pods -o=json
```

`-o=yaml`YAML 输出格式

```
kubectl get pods -o=yaml
```

`-o=wide`以纯文本格式输出任何附加信息，对于 pods，包括节点名

```
kubectl get pods -o=wide
```

`-n`别名为`namespace`

```
kubectl get pods -n=<namespace_name>
```

`-f`用于创建资源的文件名、目录或文件的 URL。

```
kubectl create -f ./<file name>
```

`-l`使用指定标签过滤

```
kubectl logs -l name=<label name>
```

`-h`对 kubectl 的帮助

```
kubectl -h
```

## [配置文件](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)(也称为清单或 YAML 文件)

> 通过将多个对象配置文件存储在一个目录中，并根据需要使用`*kubectl apply*`递归地创建和更新这些对象，可以创建、更新和删除 Kubernetes 对象。此方法保留对活动对象的写入，而不将更改合并回对象配置文件。`*kubectl diff*`也给你一个`*apply*`将会做出什么改变的预览。

```
kubectl apply -f <configuration file>
```

创建对象

```
kubectl create -f <configuration file>
```

在目录中的所有清单文件中创建对象

```
kubectl create -f <configuration file directory>
```

从 URL 创建对象

```
kubectl create -f <‘url’>
```

删除对象

```
kubectl delete -f <configuration file>
```

## 集群管理和上下文

集群管理是指查询关于 K8S 集群本身的信息。

显示群集中主服务器和服务的端点信息

```
kubectl cluster-info
```

显示在客户机和服务器上运行的 Kubernetes 版本

```
kubectl version
```

获取集群的配置

```
kubectl config view
```

获取用户列表

```
kubectl config view -o jsonpath='{.users[*].name}'
```

显示当前上下文

```
kubectl config current-context
```

显示上下文列表

```
kubectl config get-contexts
```

设置默认上下文

```
kubectl config use-context <cluster name>
```

列出可用的 API 资源

```
kubectl api-resources
```

列出可用的 API 版本

```
kubectl api-versions 
```

列出所有名称空间中的 pod、服务、daemonsets、部署、副本集、状态集、作业和 cronjobs，而不是自定义资源类型。注意`--all-namespaces`的别名是`-A`

```
kubectl get all --all-namespaces
```

## [达蒙塞特](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

> DaemonSet **确保所有(或一些)节点运行 Pod** 的副本。随着节点添加到集群中，单元也会添加到其中。随着节点从集群中移除，这些 pod 将被垃圾收集。删除 DaemonSet 将清理它创建的 pod。

列出一个或多个 daemonsets

```
kubectl get daemonset
```

编辑和更新一个或多个数据集的定义

```
kubectl edit daemonset <daemonset_name>
```

删除守护集

```
kubectl delete daemonset <daemonset_name>
```

创建新的 daemonset

```
kubectl create daemonset <daemonset_name>
```

管理 daemonset 的首次展示

```
kubectl rollout daemonset
```

显示命名空间内 daemonsets 的详细状态

```
kubectl describe ds <daemonset_name> -n <namespace_name>
```

## [部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

> 部署为[pod](https://kubernetes.io/docs/concepts/workloads/pods/)和[复制集](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)提供声明性更新。
> 
> 你在展开中描述一个*期望状态*，展开[控制器](https://kubernetes.io/docs/concepts/architecture/controller/)以受控的速率将实际状态改变为期望状态。您可以定义部署来创建新的副本集，或者删除现有的部署并在新部署中采用它们的所有资源。

列出一个或多个部署

```
kubectl get deployment
```

显示一个或多个部署的详细状态

```
kubectl describe deployment <deployment_name>
```

编辑和更新服务器上一个或多个部署的定义

```
kubectl edit deployment <deployment_name>
```

创建新部署

```
kubectl create deployment <deployment_name>
```

删除部署

```
kubectl delete deployment <deployment_name>
```

查看部署的首次展示状态

```
kubectl rollout status deployment <deployment_name>
```

执行滚动更新(K8S 默认)，将容器的映像设置为特定部署的新版本

```
kubectl set image deployment/<deployment name> <container name>=image:<new image version>
```

回滚以前的部署

```
kubectl rollout undo deployment/<deployment name>
```

执行替换部署— *强制替换、删除然后重新创建资源。*

```
kubectl replace --force -f <configuration file>
```

## 事件

列出系统中所有资源的最近事件

```
kubectl get events
```

仅列出警告

```
kubectl get events --field-selector type=Warning
```

列出按时间戳排序的事件

```
kubectl get events --sort-by=.metadata.creationTimestamp
```

列出事件，但不包括 Pod 事件

```
kubectl get events --field-selector involvedObject.kind!=Pod
```

为具有特定名称的单个节点提取事件

```
kubectl get events --field-selector involvedObject.kind=Node, involvedObject.name=<node_name>
```

从事件列表中过滤出正常事件

```
kubectl get events --field-selector type!=Normal
```

## [日志](https://kubernetes.io/docs/concepts/cluster-administration/system-logs/)

> 系统组件日志记录集群中发生的事件，这对调试非常有用。您可以配置日志详细度来查看更多或更少的详细信息。日志可以是粗粒度的，如显示组件中的错误，也可以是细粒度的，如显示事件的逐步跟踪(如 HTTP 访问日志、pod 状态更改、控制器操作或调度程序决策)。

打印 pod 的日志

```
kubectl logs <pod_name>
```

打印 pod 最近 6 小时的日志

```
kubectl logs --since=6h <pod_name>
```

获取最近的 50 行日志

```
kubectl logs --tail=50 <pod_name>
```

从服务获取日志，并可以选择哪个容器

```
kubectl logs -f <service_name> [-c <$container>]
```

打印 pod 的日志并跟踪新日志

```
kubectl logs -f <pod_name>
```

打印 pod 中容器的日志

```
kubectl logs -c <container_name> <pod_name>
```

将 pod 的日志输出到名为“pod.log”的文件中

```
kubectl logs <pod_name> pod.log
```

查看以前出现故障的 pod 的日志

```
kubectl logs --previous <pod_name>
```

## [命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

> 在 Kubernetes 中，*名称空间*提供了一种在单个集群中隔离资源组的机制。资源的名称在一个名称空间内需要是唯一的，但在不同的名称空间之间不需要。基于命名空间的作用域仅适用于命名空间对象*(例如部署、服务等)*，不适用于集群范围的对象*(例如存储类、节点、持久卷等)*。

创建名称空间

```
kubectl create namespace <namespace_name>
```

列出一个或多个名称空间

```
kubectl get namespace <namespace_name>
```

显示一个或多个命名空间的详细状态

```
kubectl describe namespace <namespace_name>
```

删除名称空间

```
kubectl delete namespace <namespace_name>
```

编辑和更新名称空间的定义

```
kubectl edit namespace <namespace_name>
```

显示节点或单元上命名空间的资源(CPU/内存/存储)使用情况

```
kubectl top <node / pod> --namespace=<namespace_name>
```

## [节点](https://kubernetes.io/docs/concepts/architecture/nodes/)

> Kubernetes 通过将容器放入 Pods 中在*节点*上运行来运行您的工作负载。根据群集的不同，节点可以是虚拟机或物理机。每个节点由[控制平面](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane)管理，并包含运行[pod](https://kubernetes.io/docs/concepts/workloads/pods/)所需的服务。
> 
> 通常，一个群集中有几个节点；在学习或资源有限的环境中，您可能只有一个节点。
> 
> 一个节点上的[组件](https://kubernetes.io/docs/concepts/overview/components/#node-components)包括 [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) ，一个[容器运行时](https://kubernetes.io/docs/setup/production-environment/container-runtimes)，以及 [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) 。

更新一个或多个节点上的污点

```
kubectl taint node <node_name>
```

列出一个或多个节点

```
kubectl get node
```

删除一个或多个节点

```
kubectl delete node <node_name>
```

显示节点的资源使用情况(CPU/内存/存储)

```
kubectl top node <node_name>
```

在节点上运行的 pod

```
kubectl get pods -o wide | grep <node_name>
```

注释节点

```
kubectl annotate node <node_name>
```

将节点标记为不可调度

```
kubectl cordon node <node_name>
```

将节点标记为可调度

```
kubectl uncordon node <node_name>
```

排空节点以准备维护

```
kubectl drain node <node_name>
```

添加或更新一个或多个节点的标签

```
kubectl label node
```

## [豆荚](https://kubernetes.io/docs/concepts/workloads/pods/)

> *Pods* 是可以在 Kubernetes 中创建和管理的最小可部署计算单元。
> 
> 一个 *Pod* (如鲸或豌豆荚中的 Pod)是一组一个或多个[容器](https://kubernetes.io/docs/concepts/containers/)，具有共享存储和网络资源，以及如何运行容器的规范。一个 Pod 的内容总是位于同一位置并被共同调度，并且在一个共享的上下文中运行。Pod 模拟一个特定于应用程序的“逻辑主机”:它包含一个或多个相对紧密耦合的应用程序容器。在非云环境中，在同一物理或虚拟机上执行的应用类似于在同一逻辑主机上执行的云应用。
> 
> 和应用程序容器一样，Pod 可以包含在 Pod 启动期间运行的 [init 容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)。如果您的集群提供的话，您还可以注入[临时容器](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)进行调试。

列出一个或多个窗格

```
kubectl get pod
```

按重新启动计数排序的列表窗格

```
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'
```

获取命名空间中所有正在运行的窗格

```
kubectl get pods --field-selector=status.phase=Running
```

删除窗格

```
kubectl delete pod <pod_name>
```

显示窗格的详细状态

```
kubectl describe pod <pod_name>
```

创建一个 pod

```
kubectl create pod <pod_name>
```

对 pod 中的容器执行命令

```
kubectl exec <pod_name> -c <container_name> <command>
```

在单容器 pod 上获得交互式外壳

```
kubectl exec -it <pod_name> /bin/sh
```

显示 pod 的资源使用情况(CPU/内存/存储)

```
kubectl top pod
```

添加或更新 pod 的注释

```
kubectl annotate pod <pod_name> <annotation>
```

添加或更新 pod 的标签

```
kubectl label pods <pod_name> new-label=<label name>
```

获取窗格并显示标签

```
kubectl get pods --show-labels
```

监听本地机器上的端口，并转发到指定 pod 上的端口

```
kubectl port-forward <pod name> <port number to listen on>:<port number to forward to>
```

## [复制控制器](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)

> **注意:**配置`[ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)`的`[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)`现在是设置复制的推荐方式。
> 
> 一个*复制控制器*确保指定数量的 pod 副本在任一时间运行。换句话说，复制控制器确保一个 pod 或一组同类的 pod 总是可用的。

列出复制控制器

```
kubectl get rc
```

按名称空间列出复制控制器

```
kubectl get rc --namespace=”<namespace_name>”
```

## [复制集](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

> 副本集的目的是在任何给定时间保持一组稳定的副本盒运行。因此，它通常用于保证指定数量的相同 pod 的可用性。

列出副本集

```
kubectl get replicasets
```

显示一个或多个副本集的详细状态

```
kubectl describe replicasets <replicaset_name>
```

缩放副本集

```
kubectl scale --replicas=[x]
```

## [秘密](https://kubernetes.io/docs/concepts/configuration/secret/)

> 机密是包含少量敏感数据(如密码、令牌或密钥)的对象。否则，这些信息可能会放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) 规范或[容器图像](https://kubernetes.io/docs/reference/glossary/?all=true#term-image)中。使用秘密意味着您不需要在应用程序代码中包含机密数据。
> 
> 因为可以独立于使用它们的窗格创建机密，所以在创建、查看和编辑窗格的工作流程中，机密(及其数据)暴露的风险较小。Kubernetes 和在集群中运行的应用程序还可以采取额外的保密措施，比如避免将保密数据写入非易失性存储器。
> 
> 机密与[配置图](https://kubernetes.io/docs/concepts/configuration/configmap/)相似，但专门用于保存机密数据。

创造一个秘密

```
kubectl create secret
```

列出秘密

```
kubectl get secrets
```

列出秘密的详细信息

```
kubectl describe secrets
```

删除秘密

```
kubectl delete secret <secret_name>
```

## [服务](https://kubernetes.io/docs/concepts/services-networking/service/)

> 一种将运行在一组[pod](https://kubernetes.io/docs/concepts/workloads/pods/)上的应用程序公开为网络服务的抽象方式。
> 
> 使用 Kubernetes，您不需要修改应用程序来使用不熟悉的服务发现机制。Kubernetes 为一组 Pods 提供它们自己的 IP 地址和一个 DNS 名称，并且可以在它们之间进行负载平衡。

列出一项或多项服务

```
kubectl get services
```

显示服务的详细状态

```
kubectl describe services
```

将复制控制器、服务、部署或 pod 作为新的 Kubernetes 服务公开

```
kubectl expose deployment [deployment_name]
```

编辑和更新一个或多个服务的定义

```
kubectl edit services
```

## [服务账户](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

> 服务帐户为在 Pod 中运行的进程提供身份。
> 
> **注意:**本文是对服务帐户的用户介绍，描述了服务帐户在 Kubernetes 项目推荐的集群中的行为。您的集群管理员可能已经自定义了集群中的行为，在这种情况下，本文档可能不适用。
> 
> 当您(一个人)访问集群时(例如，使用`kubectl`)，您被 apiserver 认证为一个特定的用户帐户(目前通常是`admin`，除非您的集群管理员已经定制了您的集群)。pod 内容器中的进程也可以联系 apiserver。当他们这样做时，他们被认证为一个特定的服务帐户(例如，`default`)。

列出服务帐户

```
kubectl get serviceaccounts
```

显示一个或多个服务帐户的详细状态

```
kubectl describe serviceaccounts
```

替换服务帐户

```
kubectl replace serviceaccount
```

删除服务帐户

```
kubectl delete serviceaccount <service_account_name>
```

## [statefullset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

> StatefulSet 是用于管理有状态应用程序的工作负载 API 对象。
> 
> 管理一组[pod](https://kubernetes.io/docs/concepts/workloads/pods/)、*的部署和扩展，并提供关于这些 pod 的排序和唯一性*的保证。
> 
> 像[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)一样，StatefulSet 管理基于相同容器规范的 pod。与部署不同，StatefulSet 为它们的每个 pod 维护一个粘性身份。这些 pod 由相同的规范创建，但不可互换:每个 pod 都有一个持久标识符，它在任何重新调度中都维护这个标识符。
> 
> 如果您希望使用存储卷为您的工作负载提供持久性，您可以使用 StatefulSet 作为解决方案的一部分。尽管状态集中的单个 Pod 容易出现故障，但永久 Pod 标识符使得将现有卷与替换任何出现故障的 Pod 的新 Pod 相匹配变得更加容易。

列表状态集

```
kubectl get statefulset
```

仅删除状态集(不删除窗格)

```
kubectl delete statefulset/[stateful_set_name] --cascade=false
```

## 摘要

使用 K8S 时，开门见山，用小抄！

想要更多 Kubernetes 内容？查看我的其他 K8S 文章[这里](https://jackwesleyroper.medium.com/kubernetes-k8s-related-articles-index-54718769e390)！

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper) 

*原载于*[*space lift . io*](https://spacelift.io/)*。*
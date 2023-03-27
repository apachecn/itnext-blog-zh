# 如何在 Kubernetes 中编排无状态应用？

> 原文：<https://itnext.io/stateless-apps-in-kubernetes-beyond-pods-f0f584eda91b?source=collection_archive---------3----------------------->

你好 Kubernauts！欢迎来到“果壳中的库伯内特”博客系列:-)

这是介绍用于管理无状态应用程序的原生 Kubernetes 原语的第一部分。Kubernetes 最常见的用例之一是编排和操作无状态服务。在 Kubernetes 中，您需要一个`Pod`(或者在大多数情况下需要一个`Pod`的*组*来表示一个服务或应用程序——但是还有更多！我们将超越基本的`Pod`，探索其他高级组件，即`ReplicaSet`和`Deployment`

和往常一样，代码[可以在 GitHub](https://github.com/abhirockzz/kubernetes-in-a-nutshell) 上获得

![](img/28d47c3d6b86b97615e153222e63c28c.png)

首先，您需要一个 Kubernetes 集群。这可能是一个简单的使用`[minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)`、`[Docker for Mac](https://blog.docker.com/2018/01/docker-mac-kubernetes/)`等的单节点本地集群。或者来自[Azure](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu)[Google](https://cloud.google.com/kubernetes-engine/)[AWS](https://aws.amazon.com/eks/)等的托管 Kubernetes 服务。要访问你的 Kubernetes 集群，你需要`[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)`，它很容易安装。

例如，要为 Mac 安装`kubectl`,您只需

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && \
chmod +x ./kubectl && \
sudo mv ./kubectl /usr/local/bin/kubectl
```

如果你已经安装了 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu) ，你需要做的就是`[az acs kubernetes install-cli](https://docs.microsoft.com/cli/azure/acs/kubernetes?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-acs-kubernetes-install-cli)`。

> *如果你有兴趣学习 Kubernetes 和 Containers 使用*[*Azure*](https://azure.microsoft.com/services/kubernetes-service/?WT.mc_id=medium-blog-abhishgu)*，只需* [*创建一个* ***免费*** *账号*](https://azure.microsoft.com/en-us/free/?WT.mc_id=medium-blog-abhishgu) *就可以开始了！一个好的起点是使用文档中的* [*快速入门、教程和代码示例*](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu) *来熟悉该服务。我也强烈推荐查看一下* [*50 天 Kubernetes 学习路径*](https://azure.microsoft.com/resources/kubernetes-learning-path/?WT.mc_id=medium-blog-abhishgu) *。高级用户可能希望参考* [*Kubernetes 最佳实践*](https://docs.microsoft.com/azure/aks/best-practices?WT.mc_id=medium-blog-abhishgu) *或观看一些* [*视频*](https://azure.microsoft.com/resources/videos/index/?services=kubernetes-service&WT.mc_id=medium-blog-abhishgu) *以了解演示、主要功能和技术会议。*

让我们从理解`Pod`的概念开始。

# `Pod`

一个`Pod`是 Kubernetes 中最小的可能抽象，它可以运行一个或多个容器。这些容器共享资源(存储、卷)，并且可以通过`localhost`相互通信。

使用下面的 YAML 文件创建一个简单的`Pod`。

> `*Pod*` *只是一个 Kubernetes 资源或对象。YAML 文件是描述其期望状态以及一些基本信息的东西——它也被称为* `*manifest*` *、* `*spec*` *(规范的简写)或* `*definition*` *。*

作为`Pod`规范的一部分，我们传达了在 Kubernetes 中运行`[nginx](https://www.nginx.com/)`的意图，并使用`spec.containers.image`指向其在 [DockerHub](https://hub.docker.com/_/nginx) 上的容器图像。

使用`[kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)`命令向 Kubernetes 提交`Pod`信息。

> *为了简单起见，直接从*[*GitHub repo*](https://github.com/abhirockzz/kubernetes-in-a-nutshell)*中引用 YAML 文件，但是您也可以将文件下载到您的本地机器上，并以同样的方式使用它。*

```
$ kubectl apply -f https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/stateless-apps/kin-stateless-pod.yamlpod/kin-stateless-1 created$ kubectl get podsNAME                               READY   STATUS    RESTARTS   AGE
kin-stateless-1                    1/1     Running   0          10s
```

这应该如预期的那样工作。现在，让我们删除`Pod`，看看会发生什么。为此，我们需要使用`kubectl delete pod <pod_name>`

```
$ kubectl delete pod kin-stateless-1
pod "kin-stateless-1" deleted$ kubectl get pods
No resources found.
```

….就这样，`Pod`消失了！

![](img/f05fad2a660feada01a44ea727446689.png)

对于严肃的应用程序，您必须注意以下几个方面:

*   **高可用性和弹性** —理想情况下，您的应用程序应该足够健壮，能够自我修复并在出现故障时保持可用，例如`Pod`由于节点故障导致的删除等。
*   **可伸缩性**——如果你的应用程序的单个实例(`Pod`)不够用怎么办？您不想运行副本/多个实例吗？

一旦有多个应用程序实例跨集群运行，您将需要考虑:

*   **缩放** —你能指望底层平台自动处理水平缩放吗？
*   **访问您的应用程序** —客户端(内部或外部)如何访问您的应用程序，多个实例之间的流量是如何调节的？
*   **升级** —您如何以无中断方式(即不停机)处理应用程序更新？

问题说够了。让我们来看看一些可能的解决方案！

# Pod 控制器

虽然可以直接创建`Pod` s，但是使用 Kubernetes 在`Pod` s 之上提供的更高级别的组件来解决上述问题是有意义的。简单来说，这些组件(也称为`Controllers`)可以创建和管理一组`Pod`

以下控制器在`Pod`和无状态应用的环境中工作:

*   **复制集**
*   **部署**
*   **复制控制器**

> *还有其他的* `*Pod*` *控制器，如*`*StatefulSet*`*`*Job*`*`*DaemonSet*`*等。但是它们与无状态应用程序无关，因此不在这里讨论***

# **复制集**

**一个`ReplicaSet`可以用来确保应用程序的固定数量的副本/实例(`Pod`)总是可用的。它在(用户定义的)选择器的帮助下识别需要管理的一组`Pod`并协调它们(创建或删除)以维护期望的实例计数。**

**下面是一个常见的`ReplicaSet`规格**

**让我们创建`ReplicaSet`**

```
**$ kubectl apply -f  https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/stateless-apps/kin-stateless-replicaset.yamlreplicaset.apps/kin-stateless-rs created$ kubectl get replicasetsNAME               DESIRED   CURRENT   READY   AGE
kin-stateless-rs   2         2         2       1m11s$ kubectl get pods --selector=app=kin-stateless-rsNAME                     READY   STATUS    RESTARTS   AGE
kin-stateless-rs-zn4p2   1/1     Running   0          13s
kin-stateless-rs-zxp5d   1/1     Running   0          13s**
```

**我们的`ReplicaSet`对象(名为`kin-stateless-rs`)是和两个`Pod`一起创建的(注意`Pod`的名称包含一个随机的字母数字字符串，例如`zn4p2`)**

**这是按照我们在 YAML 提供的(规格):**

*   **`spec.replicas`被设置为`two`**
*   **`selector.matchLabels`被设置为`app: kin-stateless-rs`并匹配`Pod`规格中的`.spec.template.metadata.labels`字段。**

> **[*标签是简单的键值对*](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) *，可以添加到对象中(比如本例中的* `*Pod*` *)。***

**我们在`kubectl get`命令中使用了`--selector`来根据标签过滤`Pod`，在本例中是`app=kin-stateless-rs`。**

**试着删除一个`Pod`(就像你在前面的例子中做的那样)**

> ***请注意，* `*Pod*` *名称在您的情况下会有所不同，所以请确保使用正确的名称。***

```
**$ kubectl delete pod kin-stateless-rs-zxp5dpod "kin-stateless-rs-zxp5d" deleted$ kubectl get pods -l=app=kin-stateless-rsNAME                     READY   STATUS    RESTARTS   AGE
kin-stateless-rs-nghgk   1/1     Running   0          9s
kin-stateless-rs-zn4p2   1/1     Running   0          5m**
```

**我们还有两个`Pod` s！这是因为创建了一个新的`Pod`(突出显示)来满足`ReplicaSet`的副本计数(两个)。**

**要水平扩展您的应用程序，您需要做的就是更新清单文件中的`spec.replicas`字段并再次提交它。**

> **作为一个练习，试着把它扩大到五个副本，然后再回到三个。**

**到目前为止一切顺利！但这并不能解决所有问题。其中之一是处理应用程序更新，特别是以不需要停机的方式。Kubernetes 提供了另一个在`ReplicaSet` s 之上工作的组件来处理这个和更多的问题。**

# **部署**

**一个`Deployment`是管理一个`ReplicaSet`的抽象——回想一下上一节，一个`ReplicaSet`管理一组 pod。除了弹性可伸缩性之外，`Deployment`还提供了其他有用的特性，允许您管理更新、回滚到以前的状态、暂停和恢复部署过程等。让我们来探讨这些。**

**Kubernetes `Deployment`从其底层`ReplicaSet`借用了以下特性:**

*   ****弹性** —由于`ReplicaSet`，如果一个 Pod 崩溃，它会自动重启。唯一的例外是当您将`Pod`规格中的`restartPolicy`设置为`Never`时。**
*   ****缩放** —这也由底层`ReplicaSet`对象负责。**

**这就是典型的`Deployment`规格的样子**

**创建`Deployment`并查看创建了哪些 Kubernetes 对象**

```
**$ kubectl apply -f  https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/stateless-apps/kin-stateless-deployment.yaml
deployment.apps/kin-stateless-depl created$ kubectl get deployment kin-stateless-dp
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
kin-stateless-dp   2/2     2            2           10$ kubectl get replicasets
NAME                         DESIRED   CURRENT   READY   AGE
kin-stateless-dp-8f9b4d456   2         2         2       12$ kubectl get pods -l=app=kin-stateless-dp
NAME                               READY   STATUS    RESTARTS   AGE
kin-stateless-dp-8f9b4d456-csskb   1/1     Running   0          14s
kin-stateless-dp-8f9b4d456-hhrj7   1/1     Running   0          14s**
```

**`Deployment` ( `kin-stateless-dp`)与`spec.replicas`字段中指定的`ReplicaSet`和(两个)`Pod`一起创建。太好了！现在，让我们看一下`Pod`看看我们使用的是哪个`nginx`版本——请注意，在您的情况下`Pod`名称会有所不同，所以请确保您使用的是正确的名称**

```
**$ kubectl exec kin-stateless-dp-8f9b4d456-csskb -- nginx -v
nginx version: nginx/1.17.3**
```

**这是因为`nginx`图像的`latest`标签是从 [DockerHub](https://hub.docker.com/_/nginx) 中获取的，而该标签在编写时恰好是`v1.17.3`。**

> *****什么是*** `***kubectl exec***` ***？*** *简单来说，它允许你在一个* `*Pod*` *内的特定容器中执行一个命令。在这种情况下，我们的* `*Pod*` *有一个单独的容器，所以我们不需要指定一个***

# **更新一个`Deployment`**

**您可以通过修改`Pod`规范的模板部分来触发对现有`Deployment`的更新——一个常见的例子是更新到容器映像的较新版本(标签)。您可以使用`Deployment`清单中的`spec.strategy.type`来指定，有效选项为- `Rolling`更新和`Recreate`。**

****滚动更新****

**滚动更新确保您不会在更新过程中导致应用程序停机——这是因为更新一次只发生一次`Pod`。在某个时间点，应用程序的早期版本和当前版本共存。一旦更新完成，旧的`Pod`将被删除，但是在某个阶段，部署中的`Pod`的总数将超过指定的`replicas`计数。**

**可以使用`maxSurge`和`maxUnavailable`设置进一步调整该行为。**

*   **`spec.strategy.rollingUpdate.maxSurge` —除了指定的副本数量之外，可以创建的最大数量的`Pod`**
*   **`spec.strategy.rollingUpdate.maxUnavailable` —定义不可用的`Pod`的最大数量**

****再造****

**这非常简单——在推出新版本之前，旧的一组 pod 会被删除。你可以使用`ReplicaSet` s 获得同样的结果，首先删除旧的，然后用更新的规格创建一个新的(例如，新的 docker 图像等)。)**

**让我们尝试通过指定一个显式的 Docker 图像标签来更新应用程序——在这种情况下，我们将使用`1.16.0`。这意味着一旦我们更新了我们的应用程序，这个版本应该在我们反省我们的`Pod`时反映出来。**

**下载上面的`Deployment`清单，更新它以将`spec.containers.image`从`nginx`更改为`nginx:1.16.0`，并提交给集群——这将触发更新**

```
**$ kubectl apply -f deployment.yaml
deployment.apps/kin-stateless-dp configured$ kubectl get pods -l=app=kin-stateless-dp
NAME                                READY   STATUS    RESTARTS   AGE
kin-stateless-dp-5b66475bd4-gvt4z   1/1     Running   0          49s
kin-stateless-dp-5b66475bd4-tvfgl   1/1     Running   0          61s**
```

**您现在应该会看到一组新的`Pod`(注意名称)。要确认更新:**

```
**$ kubectl exec kin-stateless-dp-5b66475bd4-gvt4z -- nginx -v
nginx version: nginx/1.16.0**
```

> ***请注意，* `*Pod*` *名称会因您的情况而异，所以请确保使用正确的名称***

# **反转**

**如果当前的`Deployment`不尽如人意，您可以恢复到以前的版本，以防新版本不能如预期那样工作。这是可能的，因为 Kubernetes 以修订版的形式存储了`Deployment`的首次展示历史。**

**![](img/772b59f859e1bf0085f6a6f04ba6d599.png)**

**要检查`Deployment`的历史记录:**

```
**$ kubectl rollout history deployment/kin-stateless-dpdeployment.extensions/kin-stateless-dpREVISION  CHANGE-CAUSE
1         <none>
2         <none>**
```

**注意有两个版本，其中`2`是最新版本。我们可以使用`kubectl rollout undo`回滚到前一个**

```
**$ kubectl rollout undo deployment kin-stateless-dp
deployment.extensions/kin-stateless-dp rolled back$ kubectl get pods -l=app=kin-stateless-dp
NAME                                READY   STATUS        RESTARTS   AGE
kin-stateless-dp-5b66475bd4-gvt4z   0/1     Terminating   0          10m
kin-stateless-dp-5b66475bd4-tvfgl   1/1     Terminating   0          10m
kin-stateless-dp-8f9b4d456-d4v97    1/1     Running       0          14s
kin-stateless-dp-8f9b4d456-mq7sb    1/1     Running       0          7s**
```

**注意中间状态，Kubernetes 忙于终止旧的`Deployment`的`Pod`，同时确保创建新的`Pod`来响应回滚请求。**

**如果你再次检查`nginx`版本，你会看到这个应用确实已经回滚到`1.17.3`。**

```
**$ kubectl exec kin-stateless-dp-8f9b4d456-d4v97 -- nginx -v
nginx version: nginx/1.17.3**
```

**![](img/86cd26d9496631b41b08dabb7758873b.png)**

# **暂停和继续**

**也可以暂停`Deployment`卷展栏，并在对其应用更改后将其恢复(在暂停状态期间)。**

# **`ReplicationController`**

**一个`ReplicationController`类似于一个`Deployment`或者`ReplicaSet`。然而，对于无状态应用程序编排来说，它不是推荐的方法，因为 T3 提供了更丰富的功能(如前一节所述)。你可以在 Kubernetes 文档中读到更多关于它们的内容。**

# **参考**

**查看 Kubernetes 文档，了解我们在本文中讨论的资源的 API 细节，即`Pod`、`ReplicaSet`和`Deployment`**

*   **[Pod API 参考](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#pod-v1-core)**
*   **[复制集 API 参考](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#replicaset-v1-apps)**
*   **[部署 API 参考](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#deployment-v1-apps)**

****敬请关注本系列的下一部分！****

**我真的希望你喜欢这篇文章，并从中学到了一些东西！如果你做了，请喜欢并跟随。很高兴通过 [@abhi_tweeter](https://twitter.com/abhi_tweeter) 获得反馈或发表评论。**
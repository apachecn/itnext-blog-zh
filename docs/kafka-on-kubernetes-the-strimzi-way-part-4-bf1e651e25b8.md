# 库伯内特斯上的卡夫卡，斯特里姆兹之路！(第四部分)

> 原文：<https://itnext.io/kafka-on-kubernetes-the-strimzi-way-part-4-bf1e651e25b8?source=collection_archive---------0----------------------->

![](img/e699688d3b052551b5f76396ac6abc55.png)

欢迎来到这个关于在 Kubernetes 上运行卡夫卡的博客系列:

*   [第一部分](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788)
*   [第二部分](/kafka-on-kubernetes-the-strimzi-way-part-2-43192f1dd831)
*   [第三部分](/kafka-on-kubernetes-the-strimzi-way-part-3-19cfdfe86660)
*   第 4 部分:这个博客

到目前为止，我们已经有了一个带有 TLS 加密的 Kafka 单节点集群，在此基础上，我们配置了不同的身份验证模式(`TLS`和`SASL SCRAM-SHA-512`)，使用 User Operator 定义了用户，使用 CLI 和 Go 客户端连接到集群，并看到了使用 Topic Operator 管理 Kafka 主题是多么容易。到目前为止，我们的集群使用了`ephemeral`持久性，在单节点集群的情况下，这意味着如果 Kafka 或 Zookeeper 节点(`Pod` s)由于任何原因重启，我们将会丢失数据。

让我们继续前进！在这一部分，我们将涵盖:

*   如何配置 Strimzi 以增加集群的持久性:
*   探索`PersistentVolume`和`PersistentVolumeClaim`等组件
*   如何修改存储质量
*   尝试扩展我们 Kafka 集群的存储大小

> *代码在 GitHub 上可以找到—*[*https://github.com/abhirockzz/kafka-kubernetes-strimzi/*](https://github.com/abhirockzz/kafka-kubernetes-strimzi/)

# 我需要什么来浏览这个教程？

`kubectl`-[https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

我将使用[Azure Kubernetes Service(AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes?WT.mc_id=medium-blog-abhishgu)来演示这些概念，但总的来说，它是独立于 Kubernetes 提供商的。如果你想使用`AKS`，你所需要的就是一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)，如果你还没有的话，你可以[免费获得](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu)。

> *在本系列的本部分或后续部分中，我不会重复一些常见的部分(如安装/设置(Helm、Strimzi、Azure Kubernetes 服务)、Strimzi 概述)，我会要求您* [*参考第一部分*](https://dev.to/azure/kafka-on-kubernetes-the-strimzi-way-part-1-57g7)

# 添加持久性

我们将从创建一个持久集群开始。下面是规范的一个片段(你可以在 GitHub 上访问完整的 YAML

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-kafka-cluster
spec:
  kafka:
    version: 2.4.0
    replicas: 1
    storage:
      type: persistent-claim
      size: 2Gi
      deleteClaim: true
....
  zookeeper:
    replicas: 1
    storage:
      type: persistent-claim
      size: 1Gi
      deleteClaim: true
```

需要注意的关键事项:

*   `storage.type`是[中的`persistent-claim`(相对于`ephemeral`)之前的例子](https://github.com/abhirockzz/kafka-kubernetes-strimzi/blob/master/part-2/kafka.yaml#L20)
*   `storage.size`对于卡夫卡和动物园管理员来说，节点分别是`2Gi`和`1Gi`
*   `deleteClaim: true`表示删除/取消部署集群时，相应的`PersistentVolumeClaim`也会被删除

> *可以看一下*`*storage*`[*https://strim zi . io/docs/operators/master/using . html # type-PersistentClaimStorage-reference*](https://strimzi.io/docs/operators/master/using.html#type-PersistentClaimStorage-reference)

要创建集群，请执行以下操作:

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent.yaml)
```

让我们看看创建集群后会发生什么

# Strimzi Kubernetes 魔术…

[Strimzi](https://strimzi.io/) 负责创建运行集群所需的 Kubernetes 资源。我们在[第 1 部分](https://dev.to/azure/kafka-on-kubernetes-the-strimzi-way-part-1-57g7) — `StatefulSet`(和`Pods`)、`LoadBalancer`服务、`ConfigMap`、`Secret`等中涵盖了其中的大部分。在这篇博客中，我们将只关注与持久性相关的组件- `PersistentVolume`和`PersistentVolumeClaim`

> *如果你正在使用 Azure Kubernetes 服务(AKS)，这将创建一个* [*Azure 托管磁盘*](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview?WT.mc_id=medium-blog-abhishgu)*——稍后将有更多相关内容*

检查`PersistentVolumeClaim` s

```
kubectl get pvc NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-my-kafka-cluster-kafka-0       Bound    pvc-b4ece32b-a46c-4fbc-9b58-9413eee9c779   2Gi        RWO            default        94s
data-my-kafka-cluster-zookeeper-0   Bound    pvc-d705fea9-c443-461c-8d18-acf8e219eab0   1Gi        RWO            default        3m20s
```

…以及他们被`Bound`到的`PersistentVolume`

```
kubectl get pv NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                       STORAGECLASS   REASON   AGE
pvc-b4ece32b-a46c-4fbc-9b58-9413eee9c779   2Gi        RWO            Delete           Bound    default/data-my-kafka-cluster-kafka-0       default                 107s
pvc-d705fea9-c443-461c-8d18-acf8e219eab0   1Gi        RWO            Delete           Bound    default/data-my-kafka-cluster-zookeeper-0   default                 3m35s
```

> *请注意，磁盘大小是在清单 ie 中指定的。*`*2*`*`*1*`*Gib 分别为卡夫卡和动物园管理员**

## *数据在哪里？*

*如果我们想看到数据本身，让我们首先检查存储 Kafka 服务器配置的`ConfigMap`:*

```
*export CLUSTER_NAME=my-kafka-clusterkubectl get configmap/${CLUSTER_NAME}-kafka-config -o yaml*
```

*在`server.config`部分，您会发现一个条目:*

```
*##########
# Kafka message logs configuration
##########
log.dirs=/var/lib/kafka/data/kafka-log${STRIMZI_BROKER_ID}*
```

*这告诉我们，卡夫卡的数据存储在`/var/lib/kafka/data/kafka-log${STRIMZI_BROKER_ID}`中。在这种情况下,`STRIMZI_BROKER_ID`就是`0`,因为我们只有一个节点*

*有了这些信息，让我们看看卡夫卡`Pod`:*

```
*export CLUSTER_NAME=my-kafka-clusterkubectl get pod/${CLUSTER_NAME}-kafka-0 -o yaml*
```

*如果您查看`kafka` `container`部分，您会注意到以下内容:*

*`volumes`配置之一:*

```
*volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-my-kafka-cluster-kafka-0*
```

*名为`data`的`volume`与`data-my-kafka-cluster-kafka-0` PVC 相关联，相应的`volumeMounts`使用该卷来确保 Kafka 数据存储在`/var/lib/kafka/data`中*

```
*volumeMounts:
    - mountPath: /var/lib/kafka/data
      name: data*
```

*要查看内容，*

```
*export STRIMZI_BROKER_ID=0kubectl exec -it my-kafka-cluster-kafka-0 -- ls -lrt /var/lib/kafka/data/kafka-log${STRIMZI_BROKER_ID}*
```

> **你也可以对 Zookeeper 节点重复同样的操作**

# *…云呢？*

*如前所述，在 AKS 的情况下，数据将最终存储在 Azure 管理的磁盘中。磁盘类型取决于 AKS 集群中的`default`存储类别。对我来说，就是:*

```
*kubectl get scazurefile           kubernetes.io/azure-file   58d
azurefile-premium   kubernetes.io/azure-file   58d
default (default)   kubernetes.io/azure-disk   2d18h
managed-premium     kubernetes.io/azure-disk   2d18h//to get details of the storage class
kubectl get sc/default -o yaml*
```

> **更多关于语义为* `*default*` *的存储类在* `*AKS*` [*文档中*](https://docs.microsoft.com/azure/aks/concepts-storage?WT.mc_id=medium-blog-abhishgu)*

*要在 Azure 中查询磁盘，使用`kubectl get pv/<name of kafka pv> -o yaml`提取`PersistentVolume`信息，并获取 Azure 磁盘的 ID，即`spec.azureDisk.diskURI`*

*您可以使用 Azure CLI 命令`[az disk show](https://docs.microsoft.com/cli/azure/disk?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-disk-show)`命令*

```
*az disk show --ids <diskURI value>*
```

*您将看到在`sku`部分定义的存储类型是`StandardSSD_LRS`，它对应于一个[标准固态硬盘](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-blog-abhishgu#standard-ssd)*

> **此表提供了不同* [*天蓝色磁盘类型*](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-blog-abhishgu#disk-comparison) 的对比*

```
*"sku": {
    "name": "StandardSSD_LRS",
    "tier": "Standard"
  }*
```

*……`tags`属性突出显示了`PV`和`PVC`的关联*

```
*"tags": {
    "created-by": "kubernetes-azure-dd",
    "kubernetes.io-created-for-pv-name": "pvc-b4ece32b-a46c-4fbc-9b58-9413eee9c779",
    "kubernetes.io-created-for-pvc-name": "data-my-kafka-cluster-kafka-0",
    "kubernetes.io-created-for-pvc-namespace": "default"
  }*
```

> *你也可以对 Zookeeper 磁盘重复同样的操作*

# *快速测试…*

*按照以下步骤确认集群是否按预期运行..*

*创建制作人`Pod`:*

```
*export KAFKA_CLUSTER_NAME=my-kafka-clusterkubectl run kafka-producer -ti --image=strimzi/kafka:latest-kafka-2.4.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list $KAFKA_CLUSTER_NAME-kafka-bootstrap:9092 --topic my-topic*
```

*在另一个终端中，创建一个消费者`Pod`:*

```
*export KAFKA_CLUSTER_NAME=my-kafka-clusterkubectl run kafka-consumer -ti --image=strimzi/kafka:latest-kafka-2.4.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_CLUSTER_NAME-kafka-bootstrap:9092 --topic my-topic --from-beginning*
```

# *如果…会怎么样*

*让我们探讨一下如何解决您将会遇到的几个需求:*

*   *使用不同的存储类型——例如，在 Azure 的情况下，您可能希望使用 [Azure Premium SSD](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-blog-abhishgu#premium-ssd) 来处理生产工作负载*
*   *重新调整存储规模—在某个时候，您会想要向 Kafka 集群添加存储*

## *更改存储类型*

*回想一下，Strimzi 的默认行为是创建一个引用`default`存储类的`PersistentVolumeClaim`。要对此进行定制，您可以简单地在`spec.kafka`(和/或`spec.zookeeper`)中的`storage`规范中包含`class`属性。*

*在 Azure 中，`managed-premium storage`类对应于高级 SSD: `kubectl get sc/managed-premium -o yaml`*

*这里是存储配置的一个片段，其中添加了`class: managed-premium`。*

```
*storage:
      type: persistent-claim
      size: 2Gi
      deleteClaim: true
      class: managed-premium*
```

*请注意，您不能更新现有群集的存储类型。要尝试这一点:*

*   *删除已有的集群— `kubectl delete kafka/my-kafka-cluster`(稍等)*
*   *创建新的集群— `kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent-premium.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent-premium.yaml)`*

```
*//Delete the existing cluster
kubectl delete kafka/my-kafka-cluster//Create a new cluster
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent-premium.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent-premium.yaml)*
```

*为了确认，检查卡夫卡节点的`PersistentVolumeClain`——注意`STORAGECLASS`列*

```
*kubectl get pvc/data-my-kafka-cluster-kafka-0NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-my-kafka-cluster-kafka-0   Bound    pvc-3f46d6ed-9da5-4c49-87ef-86684ab21cf8   2Gi        RWO            managed-premium   21s*
```

> **我们只配置了 Kafka broker 使用高级存储，所以 Zookeeper* `*Pod*` *将使用* `*StandardSSD*` *存储类型。**

## *调整存储大小(TL；灾难恢复—还不工作)*

*Azure Disks 允许你添加更多的存储空间。在 Kubernetes 的情况下，是存储类定义了是否支持这一点——对于 AKS，如果您检查 default(或`managed-premium`)存储类，您会注意到属性`allowVolumeExpansion: true`，这表明您也可以在 Kubernetes PVC 的上下文中这样做。*

*Strimzi 使得增加 Kafka 集群的存储变得非常容易——您所需要做的就是将`storage.size`字段更新为所需的值*

*现在检查 PVC:`kubectl describe pvc data-my-kafka-cluster-kafka-0`*

```
*Conditions:
  Type       Status  LastProbeTime                     LastTransitionTime                Reason  Message
  ----       ------  -----------------                 ------------------                ------  -------
  Resizing   True    Mon, 01 Jan 0001 00:00:00 +0000   Mon, 22 Jun 2020 23:15:26 +0530           
Events:
  Type     Reason              Age                From           Message
  ----     ------              ----               ----           -------
  Warning  VolumeResizeFailed  3s (x11 over 13s)  volume_expand  error expanding volume "default/data-my-kafka-cluster-kafka-0" of plugin "kubernetes.io/azure-disk": compute.DisksClient#CreateOrUpdate: Failure sending request: StatusCode=0 -- Original Error: autorest/azure: Service returned an error. Status=<nil> Code="OperationNotAllowed" Message="Cannot resize disk kubernetes-dynamic-pvc-3f46d6ed-9da5-4c49-87ef-86684ab21cf8 while it is attached to running VM /subscriptions/9a42a42f-ae42-4242-b6a7-dda0ea91d342/resourceGroups/mc_my-k8s-vk_southeastasia/providers/Microsoft.Compute/virtualMachines/aks-agentpool-42424242-1\. Resizing a disk of an Azure Virtual Machine requires the virtual machine to be deallocated. Please stop your VM and retry the operation."*
```

*注意`"Cannot resize disk...`错误信息。发生这种情况是因为 Azure 磁盘当前与 AKS 集群节点相连，这是因为`Pod`与`PersistentVolumeClaim`相关联——这个[是一个记录在案的限制](https://docs.microsoft.com/azure/virtual-machines/linux/expand-disks?WT.mc_id=medium-blog-abhishgu)*

*当然，我不是第一个遇到这个问题的人。详情请参考[问题，如本](https://github.com/kubernetes/kubernetes/issues/68427)问题。*

> *有一些解决方法，但是在这篇博客中没有讨论。我加入了这一部分，因为我想让你知道这个警告*

# *最后倒计时…*

*我们想高调离开，不是吗？好了，总结一下，让我们将群集从一个节点扩展到三个节点。这非常简单！*

*您需要做的就是将副本增加到所需的数量——在本例中，我将其配置为 3(针对 Kafka 和 Zookeeper)*

```
*...
spec:
  kafka:
    version: 2.4.0
    replicas: 3
  zookeeper:
    replicas: 3
...*
```

*除此之外，我还添加了一个外部负载平衡器监听器(这将创建一个 Azure 负载平衡器，正如在[第 2 部分](https://dev.to/azure/kafka-on-kubernetes-the-strimzi-way-part-2-1210)中所讨论的)*

```
*...
    listeners:
      plain: {}
      external:
        type: loadbalancer
...*
```

*要创建新的，只需使用新的清单*

```
*kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent-multi-node.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-4/kafka-persistent-multi-node.yaml)*
```

> **请注意，总体集群就绪需要时间，因为将有额外的组件(Azure 磁盘、负载平衡器公共 IP 等。)这将在 pod 被激活之前创建**

## *在您的 k8s 集群中，您将看到…*

*卡夫卡和动物园管理员各三个*

```
*kubectl get pod -l=app.kubernetes.io/instance=my-kafka-clusterNAME                           READY   STATUS    RESTARTS   AGE
my-kafka-cluster-kafka-0       2/2     Running   0          54s
my-kafka-cluster-kafka-1       2/2     Running   0          54s
my-kafka-cluster-kafka-2       2/2     Running   0          54s
my-kafka-cluster-zookeeper-0   1/1     Running   0          4m44s
my-kafka-cluster-zookeeper-1   1/1     Running   0          4m44s
my-kafka-cluster-zookeeper-2   1/1     Running   0          4m44s*
```

*三副(卡夫卡和动物园管理员各一副)`PersistentVolumeClaims` …*

```
*kubectl get pvcNAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-my-kafka-cluster-kafka-0       Bound    pvc-0f52dee1-970a-4c55-92bd-a97dcc41aee6   3Gi        RWO            managed-premium   10m
data-my-kafka-cluster-kafka-1       Bound    pvc-f8b613cb-3da0-4932-acea-7e5e96df1433   3Gi        RWO            managed-premium   4m24s
data-my-kafka-cluster-kafka-2       Bound    pvc-fedf431c-d87a-4bf7-80d0-d43b1337c079   3Gi        RWO            managed-premium   4m24s
data-my-kafka-cluster-zookeeper-0   Bound    pvc-1fda3714-3c37-428f-9e4b-bdb5da71cda6   1Gi        RWO            default           12m
data-my-kafka-cluster-zookeeper-1   Bound    pvc-702556e0-890a-4c07-ae5c-e2354d74d006   1Gi        RWO            default           6m42s
data-my-kafka-cluster-zookeeper-2   Bound    pvc-176ffd68-7e3a-4e04-abb1-52c54dcb84f0   1Gi        RWO            default           6m42s*
```

*…以及它们所绑定的各自的`PersistentVolumes`*

```
*NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                       STORAGECLASS      REASON   AGE
pvc-0f52dee1-970a-4c55-92bd-a97dcc41aee6   3Gi        RWO            Delete           Bound       default/data-my-kafka-cluster-kafka-0       managed-premium            12m
pvc-176ffd68-7e3a-4e04-abb1-52c54dcb84f0   1Gi        RWO            Delete           Bound       default/data-my-kafka-cluster-zookeeper-2   default                    8m45s
pvc-1fda3714-3c37-428f-9e4b-bdb5da71cda6   1Gi        RWO            Delete           Bound       default/data-my-kafka-cluster-zookeeper-0   default                    14m
pvc-702556e0-890a-4c07-ae5c-e2354d74d006   1Gi        RWO            Delete           Bound       default/data-my-kafka-cluster-zookeeper-1   default                    8m45s
pvc-f8b613cb-3da0-4932-acea-7e5e96df1433   3Gi        RWO            Delete           Bound       default/data-my-kafka-cluster-kafka-1       managed-premium            6m27s
pvc-fedf431c-d87a-4bf7-80d0-d43b1337c079   3Gi        RWO            Delete           Bound       default/data-my-kafka-cluster-kafka-2       managed-premium            6m22s*
```

*…和负载平衡器 IP。请注意，这些是为每个 Kafka 代理创建的，还有一个从客户端应用程序连接时推荐使用的引导 IP。*

```
*kubectl get svc -l=app.kubernetes.io/instance=my-kafka-clusterNAME                                        TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
my-kafka-cluster-kafka-0                    LoadBalancer   10.0.11.154    40.119.248.164   9094:30977/TCP               10m
my-kafka-cluster-kafka-1                    LoadBalancer   10.0.146.181   20.43.191.219    9094:30308/TCP               10m
my-kafka-cluster-kafka-2                    LoadBalancer   10.0.223.202   40.119.249.20    9094:30313/TCP               10m
my-kafka-cluster-kafka-bootstrap            ClusterIP      10.0.208.187   <none>           9091/TCP,9092/TCP            16m
my-kafka-cluster-kafka-brokers              ClusterIP      None           <none>           9091/TCP,9092/TCP            16m
my-kafka-cluster-kafka-external-bootstrap   LoadBalancer   10.0.77.213    20.43.191.238    9094:31051/TCP               10m
my-kafka-cluster-zookeeper-client           ClusterIP      10.0.3.155     <none>           2181/TCP                     18m
my-kafka-cluster-zookeeper-nodes            ClusterIP      None           <none>           2181/TCP,2888/TCP,3888/TCP   18m*
```

*要访问集群，您可以使用[第 2 部分](https://dev.to/azure/kafka-on-kubernetes-the-strimzi-way-part-2-1210)中概述的步骤*

# *这是一个总结！*

*这就是这个博客系列的全部内容，其中涵盖了使用开源 Strimzi 操作符在 Kubernetes 上运行 Kafka 的一些方面。*

*如果您对这个主题感兴趣，我鼓励您查看其他解决方案，如[合流运算符](https://docs.confluent.io/current/installation/operator/index.html)和[坂仔云卡夫卡运算符](https://github.com/banzaicloud/kafka-operator)*
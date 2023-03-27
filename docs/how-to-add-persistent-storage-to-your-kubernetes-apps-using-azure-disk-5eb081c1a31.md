# 如何使用 Azure Disk 向您的 Kubernetes 应用添加持久存储

> 原文：<https://itnext.io/how-to-add-persistent-storage-to-your-kubernetes-apps-using-azure-disk-5eb081c1a31?source=collection_archive---------2----------------------->

![](img/9cd931e2b34723530f2732b9162fdeab.png)

在这篇博文中，我们将看一个例子，如何使用 [Azure Disk](https://azure.microsoft.com/services/storage/disks/?WT.mc_id=medium-blog-abhishgu) 作为部署到 [Azure Kubernetes 服务](https://azure.microsoft.com/services/kubernetes-service/?WT.mc_id=medium-blog-abhishgu)的应用的存储介质。

您将:

*   在 Azure 上设置 Kubernetes 集群
*   创建一个 Azure 磁盘和相应的`PersistentVolume`
*   为应用程序`Deployment`创建一个`PersistentVolumeClaim`
*   测试一下，看看它是如何端到端工作的

# 概观

您可以使用 Kubernetes `Volume` s 为您的应用程序提供存储。Kubernetes 支持多种类型的卷。将它们分类的一种方法如下

*   **短暂的** — `Volume`与`Pod`寿命紧密相关(如`emptyDir`卷)，即如果`Pod`被移除(出于任何原因)，它们将被删除。
*   **持久性** — `Volume`用于长期存储，独立于`Pod`或`Node`生命周期。在托管 Kubernetes 产品的情况下，这可能是`NFS`或基于云的存储，如 Azure Kubernetes 服务、Google Kubernetes 引擎等。

Kubernetes 可以通过`static`或`dynamic`方式进行配置。在“静态”模式下，手动创建存储介质，例如 Azure 磁盘，然后使用如下的`Pod`规范进行引用:

```
volumes:
        - name: azure
            azureDisk:
            kind: Managed
            diskName: myAKSDisk
            diskURI: /subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

> *我强烈推荐阅读关于如何* [*“在 Azure Kubernetes 服务(AKS)中手动创建和使用带有 Azure 磁盘的卷”的优秀教程*](https://docs.microsoft.com/azure/aks/azure-disk-volume?WT.mc_id=medium-blog-abhishgu)

# 有没有更好的办法？

![](img/bb1365afa141f04bd81a1306c3221c58.png)

在上面的`Pod`清单中，存储信息直接在`Pod`中指定(使用`volumes`部分)。这意味着开发人员需要知道存储介质的所有细节，例如在 Azure Disk 的情况下——`diskName`、`diskURI`(磁盘资源 URI)，它是`kind`(类型)。这里肯定有改进的余地，就像软件中的大多数事情一样，可以使用持久卷和持久卷声明的概念，通过另一个层次的间接或抽象来实现。

关键思想围绕着“职责分离”以及将存储创建/管理与其使用分离开来:

*   当一个应用程序需要持久存储时，开发者可以通过在 pod 规范中“声明”来请求——这是通过使用`PersistentVolumeClaim`来完成的
*   实际的存储配置，例如创建 Azure 磁盘(使用 azure CLI、门户等)。)并在 Kubernetes 集群中表示它(使用`PersistentVolume`)可以由另一个实体来完成，比如管理员

让我们看看这是怎么回事！

# 先决条件:

您将需要以下内容:

*   [微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)—[注册一个免费账户吧！](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu)
*   [Azure Kubernetes 服务(AKS)集群](https://azure.microsoft.com/services/kubernetes-service/?WT.mc_id=medium-blog-abhishgu)——本博客将指导您创建一个集群
*   Azure CLI 或 Azure Cloud Shell——如果你还没有安装 Azure CLI ，你可以选择安装它(应该很快！)或者直接从你的浏览器使用 [Azure 云壳](https://azure.microsoft.com/features/cloud-shell/?WT.mc_id=medium-blog-abhishgu)。
*   `[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)`与你的 AKS 集群进行交互

> *在你的 Mac 上，你可以安装* `*kubectl*` *如下:*

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s [https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl)chmod +x ./kubectlsudo mv ./kubectl /usr/local/bin/kubectl
```

代码可以在 GitHub 的[上找到。请在继续之前克隆该存储库](https://github.com/abhirockzz/aks-azuredisk-static-pv)

```
git clone https://github.com/abhirockzz/aks-azuredisk-static-pv
cd aks-azuredisk-static-pv
```

# Kubernetes 集群设置

你只需要一个命令就可以在 Azure 上建立一个 Kubernetes 集群。但是，在此之前，我们必须创建一个资源组

```
export AZURE_SUBSCRIPTION_ID=[to be filled]
export AZURE_RESOURCE_GROUP=[to be filled]
export AZURE_REGION=[to be filled] (e.g. southeastasia)
```

切换到您的套餐并调用`az group create`

```
az account set -s $AZURE_SUBSCRIPTION_ID
az group create -l $AZURE_REGION -n $AZURE_RESOURCE_GROUP
```

您现在可以调用`az aks create`来创建新的集群

> 为了简单起见，下面的命令创建了一个单节点集群。根据您的要求随意更改规格

```
export AKS_CLUSTER_NAME=[to be filled]az aks create --resource-group $AZURE_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --node-count 1 --node-vm-size Standard_B2s --node-osdisk-size 30 --generate-ssh-keys
```

使用`az aks get-credentials`获取 AKS 集群凭证——因此，`kubectl`现在将指向您的新集群。你可以证实这一点

```
az aks get-credentials --resource-group $AZURE_RESOURCE_GROUP --name $AKS_CLUSTER_NAMEkubectl get nodes
```

> *如果您对使用*[*Azure*](https://azure.microsoft.com/services/kubernetes-service/?WT.mc_id=medium-blog-abhishgu)*学习 Kubernetes 和 Containers 感兴趣，一个好的起点是使用文档中的* [*快速入门、教程和代码示例*](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu) *来熟悉该服务。我也强烈推荐查看一下* [*50 天 Kubernetes 学习路径*](https://azure.microsoft.com/resources/kubernetes-learning-path/?WT.mc_id=medium-blog-abhishgu) *。高级用户可能希望参考* [*Kubernetes 最佳实践*](https://docs.microsoft.com/azure/aks/best-practices?WT.mc_id=medium-blog-abhishgu) *或观看一些* [*视频*](https://azure.microsoft.com/resources/videos/index/?services=kubernetes-service&WT.mc_id=medium-blog-abhishgu) *以了解演示、主要特性和技术会议。*

# 为持久存储创建 Azure 磁盘

Azure Kubernetes 集群可以使用 Azure 磁盘或 Azure 文件作为数据卷。在这个例子中，我们将探索 Azure 磁盘。你可以选择由标准硬盘或[高级固态硬盘](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-blog-abhishgu#premium-ssd)支持的 [Azure 硬盘](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-blog-abhishgu#standard-hdd)

获取 AKS 节点资源组

```
AKS_NODE_RESOURCE_GROUP=$(az aks show --resource-group $AZURE_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query nodeResourceGroup -o tsv)
```

在节点资源组中创建 Azure 磁盘

```
export AZURE_DISK_NAME=<enter-azure-disk-name>az disk create --resource-group $AKS_NODE_RESOURCE_GROUP --name $AZURE_DISK_NAME --size-gb 2 --query id --output tsv
```

> *我们正在创建一个容量为 2 GB 的磁盘*

您将获得 Azure 磁盘的资源 ID 作为响应，这将在下一步使用

```
/subscriptions/3a06a10f-ae29-4242-b6a7-dda0ea91d342/resourceGroups/MC_testaks_foo-aks_southeastasia/providers/Microsoft.Compute/disks/my-test-disk
```

# 将应用程序部署到 Kubernetes

`azure-disk-persistent-volume.yaml`文件包含了`PersistentVolume`的详细信息。我们创建它是为了在 AKS 集群中映射 Azure 磁盘。

> *注意，容量(* `*spec.capacity.storage*` *)是 2 GB，与我们刚刚创建的 Azure 磁盘的容量相同*

用 Azure 磁盘信息更新`azure-disk-persistent-volume.yaml`

*   `diskName` -您之前选择的 Azure 磁盘的名称
*   `diskURI`-Azure 磁盘的资源 ID

创建`PersistentVolume`

```
kubectl apply -f azure-disk-persistent-volume.yamlpersistentvolume/azure-disk-pv created
```

接下来，我们需要创建`PersistentVolumeClaim`，我们将在`Pod`规范中使用它作为参考

> *我们请求 2 GB 的存储空间(使用* `*resources.request.storage*` *)*

要创建它:

```
kubectl apply -f azure-disk-persistent-volume-claim.yamlpersistentvolumeclaim/azure-disk-pvc created
```

检查`PersistentVolume`

```
kubectl get pv/azure-disk-pv NAME  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM    STORAGECLASS   REASON   AGE
azure-disk-pv   2Gi        RWO            Retain           Bound    default/azure-disk-pvc            8m35s
```

检查`PersistentVolumeClaim`

```
kubectl get pvc/azure-disk-pvcNAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
azure-disk-pvc   Bound    azure-disk-pv   2Gi        RWO                           9m55s
```

> *注意(在上述输出的* `*STATUS*` *部分)的* `*PersistentVolume*` *和* `*PersistentVolumeClaim*` *是相互* `*Bound*`

# *试验*

*为了进行测试，我们将使用一个简单的 Go 应用程序。它所做的只是将日志语句推送到`/mnt/logs`中的一个文件`logz.out`——这是挂载到`Pod`中的路径*

```
*func main() {
    ticker := time.NewTicker(3 * time.Second)
    exit := make(chan os.Signal, 1)
    signal.Notify(exit, syscall.SIGTERM, syscall.SIGINT)
    for {
        select {
        case t := <-ticker.C:
            logToFile(t.String())
        case <-exit:
            err := os.Remove(fileLoc + fileName)
            if err != nil {
                log.Println("unable to delete log file")
            }
            os.Exit(1)
        }
    }
}*
```

*将我们的应用程序创建为`Deployment`*

```
*kubectl apply -f app-deployment.yaml*
```

*等待一段时间，使部署处于`Running`状态*

```
*kubectl get pods -l=app=logzNAME                               READY   STATUS    RESTARTS   AGE
logz-deployment-59b75bc786-wt98d   1/1     Running   0          15s*
```

*为了确认，检查`Pod`中的`/mnt/logs/logz.out`*

```
*kubectl exec -it $(kubectl get pods -l=app=logz --output=jsonpath={.items..metadata.name}) -- tail -f /mnt/logs/logz.out*
```

*您将每 3 秒钟看到一次日志(只有时间戳)。这是因为 Azure 磁盘存储已经安装在您的`Pod`中*

```
*2019-09-23 10:00:18.308746334 +0000 UTC m=+3.002071866
2019-09-23 10:00:21.308779348 +0000 UTC m=+6.002104880
2019-09-23 10:00:24.308771261 +0000 UTC m=+9.002096693
2019-09-23 10:00:27.308778874 +0000 UTC m=+12.002104406
2019-09-23 10:00:30.308804587 +0000 UTC m=+15.002130219*
```

*一旦你完成了测试，你可以删除资源以节省成本。*

# *打扫卫生*

```
*az group delete --name $AZURE_RESOURCE_GROUP --yes --no-wait*
```

> **这将删除资源组*下的所有资源*

*这个博客到此为止！您看到了如何使用标准的 Kubernetes 原语(如`PersistentVolume`和`PersistentVolume`)将 Azure Disk 实例附加和安装到在 AKS 中运行的应用程序。敬请关注更多内容😃😃*

*我真的希望你喜欢这篇文章，并从中学到了一些东西！如果你做了，请喜欢并跟随。很高兴通过 [Twitter](https://twitter.com/abhi_tweeter) 获得反馈，或者发表评论👇👇*
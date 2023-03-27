# 使用托管 Kubernetes 服务

> 原文：<https://itnext.io/using-managed-kubernetes-services-6201258e0df2?source=collection_archive---------6----------------------->

![](img/aab8738a4ffbb131cff9cebf2ed7dfbb.png)

约瑟夫·巴里恩托斯在 [Unsplash](https://unsplash.com/search/photos/ship-wheel?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片。

本文概述了主要云提供商的托管 Kubernetes 服务，并解释了如何使用它们。

# 手动与托管 Kubernetes

Kubernetes 是一种可以安装在现有计算基础设施上的软件，通常安装在一组互连的物理机器上。

在出现托管 Kubernetes 服务之前，您通常会配置机器(在本地或云中)，在这些机器上安装 Kubernetes，将它们绑定到一个集群，然后开始使用带有`kubectl`的集群。

特别是，您需要选择其中一台机器作为主节点，在这个主节点上安装 Kubernetes，然后在所有其他机器上安装 Kubernetes，这些机器将成为工作节点，然后将工作节点加入主节点以形成一个集群。

有了托管 Kubernetes 服务，这变得更加容易。这些服务提供了创建整个 Kubernetes 集群的简单命令。这包括机器的供应、Kubernetes 的安装和集群的形成。所有这些都是在幕后自动完成的。执行这个命令后，Kubernetes 集群就可以使用了。您只需要将集群的地址提供给本地机器上的`kubectl`，然后您就可以开始使用带有`kubectl`的集群。

因此，使用托管 Kubernetes 服务的典型工作流程如下:

1.  使用托管 Kubernetes 服务提供的命令启动 Kubernetes 集群。
2.  将本地机器上的`kubectl`指向这个集群。
3.  用`kubectl`开始使用集群。

# 现有的托管 Kubernetes 服务

所有主要的云提供商，**亚马逊网络服务(AWS)** 、**谷歌云平台(GCP)** 、**微软 Azure** 和 **IBM Cloud** ，目前都提供托管的 Kubernetes 服务。具体而言，这些是:

## [谷歌 Kubernetes 引擎(GKE)](https://cloud.google.com/kubernetes-engine/)

自 2014 年 Kubernetes 成立以来就一直存在(如你所知，Kubernetes 诞生于谷歌的 Borg 项目)。

## [Azure Kubernetes 服务(AKS)](https://docs.microsoft.com/en-us/azure/aks/)

于【2017 年秋季推出。目前(2018 年春季)在预览中，但已正式上市。

## [亚马逊弹性集装箱服务(EKS)](https://aws.amazon.com/eks/)

目前(2018 年春季)在预览中，但尚未全面上市。

**编辑:**EKS 已于 2018 年 6 月公开发售(见[此处](https://aws.amazon.com/blogs/aws/amazon-eks-now-generally-available/))，但目前(2018 年 8 月)仅在*美东-1* 和*美西-2* 地区发售。

## [IBM 云容器服务](https://www.ibm.com/cloud/container-service)

2017 年春季[推出。](https://www.ibm.com/blogs/bluemix/2017/03/kubernetes-now-available-ibm-bluemix-container-service/)

# 托管 Kubernetes 服务命令

下面是一些使用**AK**和 **GKE** 的托管 Kubernetes 服务的有用命令，分别是微软 Azure 和谷歌云平台的托管 Kubernetes 服务。

注意:将来我可能会为 AWS 和 IBM Cloud 的 Kubernetes 服务添加等效的命令。

深入研究这些命令的最好方法是查看内置的帮助消息，您可以通过向任何命令添加`-h`选项来显示这些消息。

## 创建一个集群

```
az aks create -n my-cluster
    [--node-count n]
    [--node-vm-size type]gcloud container clusters create my-cluster
    [--num-nodes n]
    [--machine-type type]
```

AKS 和 GKE 的默认集群节点数都是 3。Azure 的可用计算实例类型在这里列出[，GCP 的可用计算实例类型在这里列出](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes)[。](https://cloud.google.com/compute/docs/machine-types)

## 列出所有集群

```
az aks listgcloud container clusters list
```

## 列出计算实例

```
az vm listgcloud compute instances list
```

这包括由*创建集群*命令创建的计算实例。

## 删除集群

```
az aks delete -n my-clustergcloud container clusters delete my-cluster
```

这还会删除与群集关联的所有计算实例。

## 将 kubectl 指向一个集群

```
az aks get-credentials -n my-clustergcloud container clusters get-credentials my-cluster
```

这编辑`$HOME/.kube/config`。

# 享受你的集群

在执行上面的最后一个命令(将`kubectl`指向集群)后，您就有了一个正常工作的集群，并且您可以将它与`kubectl`一起使用。

从这一点开始使用`kubectl`总是以相同的方式工作，不管您是使用 AKS、GKE 还是手动创建集群。

托管 Kubernetes 服务只是帮助您创建集群，但是一旦完成，您就不必再处理它了。

你现在是在纯粹的 Kubernetes 世界。

用`kubectl cluster-info`或`kubectl get nodes`进行测试。

# 附录:自定义 Azure CLI

![](img/ef170afd14c05ef795c56958cf8f87ae.png)

[蒂姆·福斯特](https://unsplash.com/photos/xyGfJ2cKCl4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/ocean?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照。

下面是一些定制 Azure CLI 的命令。

特别是，设置默认的资源组和位置非常方便，因为它使您不必分别用`-g`和`-l`选项为每个命令指定资源组和位置。

**资源组**是一组可以作为一个整体来管理(例如，删除)的资源。对于您创建的每个资源(例如，一个计算实例)，您必须定义它属于哪个资源组。

**资源提供者**只是一个提供资源的 Azure 服务(例如微软。提供计算实例计算机)。

查看[关于资源组和相关概念的官方文档](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview)。

## 设置默认位置

```
az configure --defaults location=CanadaEast
```

## 创建资源组

```
az group create -n my-resource-group
```

刚开始用 Azure 的时候要创建一个资源组。

## 设置默认资源组

```
az configure --defaults group=my-resource-group
```

## 注册资源提供者

```
az provider register -n Microsoft.Compute
```

当您第一次使用 AKS 时，这对于几个资源提供者可能是必要的。

## 将默认输出格式设置为“*表格”*(默认为“*JSON”*)

```
az configure -o table
```

这会将所有命令的默认输出格式设置为人类可读的格式。
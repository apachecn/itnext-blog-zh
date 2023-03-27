# Kubernetes 提示:如何使用“storageClass”属性🤔

> 原文：<https://itnext.io/kubernetes-tip-how-to-use-the-storageclass-attribute-75cf47e7c6b0?source=collection_archive---------3----------------------->

这是一篇简短的博客文章，介绍了 Kubernetes 动态预配置的一个方面，当人们第一次探索它的时候，经常会被绊倒(我肯定被弄糊涂了！)

![](img/9cd931e2b34723530f2732b9162fdeab.png)

为了使用一个`StorageClass`，你需要做的就是从一个`PersistentVolumeClaim`中引用它。但是`StorageClass`不是一个强制字段——所以有可能根本不使用它，或者提供一个空字符串(`""`)作为它的值。理解这些选项及其影响非常重要。

让我们来澄清这些排列组合，一劳永逸！

> *Kubernetes* `*StorageClass*` *概念(以及更多)在* [*《教程:Kubernetes 基础知识卷(第二部分)】*](/tutorial-basics-of-kubernetes-volumes-part-2-b2ea6f397402) 中有深入的介绍

就像`PersistentVolume`封装了存储相关的细节一样，`StorageClass`提供了一种描述存储“类别”的方式。这里是一个[天蓝色磁盘](https://azure.microsoft.com/services/storage/disks/?WT.mc_id=medium-blog-abhishgu)的`StorageClass`的例子。

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    kubernetes.io/cluster-service: "true"
  name: default
parameters:
  cachingmode: ReadOnly
  kind: Managed
  storageaccounttype: Standard_LRS
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

# 当`*storageClass*`属性缺失时

如果想在集群中使用`default`存储类，可以选择不在`PersistentVolumeClaim`中包含`storageClass`属性。例如， [Azure Kubernetes 服务](https://azure.microsoft.com/services/storage/disks/?WT.mc_id=medium-blog-abhishgu)包括两个预播种的存储类。

您可以通过运行`kubectl get storageclass`命令进行检查

```
NAME                PROVISIONER                AGE
default (default)   kubernetes.io/azure-disk   6d10h
managed-premium     kubernetes.io/azure-disk   6d10h
```

*   `default`存储类别:提供由标准硬盘支持的标准 [Azure 磁盘](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-blog-abhishgu#standard-hdd)
*   `managed-premium`存储类:调配高级【由高级 SSD 支持的 Azure 磁盘】([https://docs . Microsoft . com/Azure/virtual-machines/windows/disks-types？wt . MC _ id = medium-](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types?WT.mc_id=medium-)blog-abhishgu # premium-SSD)

> *在这种情况下，即使* `*PersistentVolume*` *存在并且与* `*PersistentVolumeClaim*` *中的要求相匹配，也将使用* `*default*` *存储类(及其关联的* `*provisioner*` *)。*

# 当`storageClass`属性设置为空字符串时

这是个棘手的问题！

您已经有了一个`PersistentVolume`以及相应的存储介质，并且想要使用它(不涉及自定义存储类或`default`类)

在这种情况下，只需在`PersistentVolumeClaim`中将`storageClass`设置为空字符串(`""`)。这将抑制动态预配置！

> *查看* [*“如何使用 Azure Disk 将持久存储添加到您的 Kubernetes 应用程序”*](/how-to-add-persistent-storage-to-your-kubernetes-apps-using-azure-disk-5eb081c1a31) *以查看实际操作*

# 当引用特定的`storageClass`属性时

> *[*“如何在 Kubernetes 中使用自定义存储类？”*](/how-to-use-custom-storage-classes-in-kubernetes-edc568acfdfe) *博文用一个例子*展示了这个场景*

*在许多情况下，除了已经安装在 Kubernetes 集群中的存储类之外，您可能还需要创建自定义的存储类，例如，一旦删除了`PersistentVolumeClaim`，您需要改变处理`PersistentVolume`和存储介质的移除的方式。*

*在这种情况下，自定义存储类中指定的`provisioner`将用于创建存储介质(和相应的`PersistentVolume`对象)以满足`PersistentVolumeClaim`中的存储请求细节。*

***注意**:如果从`PersistentVolumeClaim`引用无效的存储类，动态预配置将会失败*

*目前就这些。我希望得到您的反馈和建议！只需[发推文](https://twitter.com/abhi_tweeter)或发表评论。还有，别忘了喜欢和关注😃😃*
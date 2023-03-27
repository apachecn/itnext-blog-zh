# K8s 标签和选择器

> 原文：<https://itnext.io/k8s-labels-selectors-9ad2fcf78a4e?source=collection_archive---------4----------------------->

在这篇文章中，我们将看看

*   什么是 Kubernetes(K8s)标签和选择器
*   我们为什么需要它们
*   如何使用它们

让我们试着用类比来理解这个概念。如果你看过不同类型的房子，你会看到联排别墅、公寓、独栋住宅、公寓等等。

![](img/4420aeb27308b53865af00bef966bfdc.png)

公寓 vs 公寓 vs 独栋住宅 vs 联排别墅

比方说，我们想对这些房屋进行分类

```
- Size(sqft area)- Type(condo, apartment, sfh, townhome)- Number of people(2,4,6,8+)
```

这些不过是**标签。选择器**帮助过滤掉所有 SFH 类型且面积大于 500 平方英尺的房子。人们可以混合搭配不同类型的选择器。

随着时间的推移，K8s 中可能会有 1000 个不同的对象，并且需要标记和选择某些类型的对象。 ***标签*** 是附在对象上的键/值对，比如 pod。

标签示例:

*   “释放”:“稳定”，“释放”:“金丝雀”
*   “环境”:“开发”、“环境”:“质量保证”、“环境”:“生产”
*   "层":"前端"，"层":"后端"，"层":"缓存"
*   "分区":"客户 a "，"分区":"客户 b "
*   “跟踪”:“每日”、“跟踪”:“每周”

## 为什么我们需要这些标签和选择器？

即使是一个小的 Kubernetes 集群也可能有数百个容器、pod、服务和许多其他 Kubernetes API 对象。

浏览 kubectl 输出的页面来找到你的对象很快变得很烦人，**标签**完美地解决了这个问题。

您应该使用标签的主要原因是:

*   使您能够在所有集群中逻辑地组织所有 Kubernetes 工作负载，
*   使您能够非常**选择性地过滤** kubectl 输出，只过滤您需要的对象。
*   使您能够**理解所有 API 对象的层和层次**。

> **我们如何使用相同的？**

让我们看看，我们想要创建一个带有标签的 Pod。使用 minikube 作为下面的例子，您可以在任何其他 K8 集群上尝试这样做。

```
**environment=**production
```

我们有两种方法可以做到这一点:命令式的方法或声明式的方法

```
kubectl run nginx-imperative --image=nginx --restart=Never -l environment=production
```

> **让我们尝试使用标签**来确认 POD

```
kubectl get pods -l environment=production
NAME               READY   STATUS    RESTARTS   AGE
nginx-imperative   1/1     Running   0          13s
```

让我们尝试使用声明的方式

```
# declarative.yml
apiVersion: v1
kind: Pod
metadata:
  name: declarative-nginx
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

让我们应用 YAML 文件，我们看到具有给定标签的 POD 被创建。通过这里的选项，我们选择了一个特定的标签

```
kubectl apply -f declarative.yml
pod/declarative-nginx createdkubectl get pods -l environment=production
NAME                READY   STATUS    RESTARTS   AGE
declarative-nginx   1/1     Running   0          74s
nginx-imperative    1/1     Running   0          2m43skubectl get pods -l environment=production --no-headers
declarative-nginx   1/1   Running   0     2m8s
nginx-imperative    1/1   Running   0     3m37s
```

> **我们可以使用多个选择器吗？**

当然，下面的例子使用了选择器 env、bu 和 type

```
kubectl get pods -l env=prod,bu=finance,type=app1 --no-headerskubectl get pods -l 'environment in (production, qa)'
```

> **如何查看所有正在使用的标签？**

```
kubectl get all --show-labels
kubectl get pod --show-labels
```

标签的 K8 文档&选择器有更多的细节，如果你想了解的话。作为后续，我们有一个帖子描述标签&注释[这里](https://sandeepbaldawa.medium.com/k8s-labels-annotations-c61c00237c77)的区别。就这样吧，下次再见！
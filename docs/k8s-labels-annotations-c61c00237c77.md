# K8s 标签和注释

> 原文：<https://itnext.io/k8s-labels-annotations-c61c00237c77?source=collection_archive---------4----------------------->

![](img/e3ccbadf89344433e1bef1b097658016.png)

我们在文章[这里](https://sandeepbaldawa.medium.com/k8s-labels-selectors-9ad2fcf78a4e)看到了标签的介绍。在本文中，让我们试着理解标签和注释之间的区别&何时使用一个而不是另一个。

让我们快速总结一下我们对标签的了解

> **K8s 为什么要用标签？**

标签允许 K8s 对一组相关资源进行分组(例如:-所有产品资源)。选择器用于查询这些标签(例如:-获取所有产品资源)。公文[这里](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。在撰写本文时需要注意的是，下面是在 K8s 中必须如何定义标签的限制

有效标签值:

*   必须等于或少于 63 个字符(不能为空)，
*   必须以字母数字字符([a-z0–9A-Z])开始和结束，
*   可以包含破折号(-)、下划线(_)、点(。)，以及介于之间的字母数字。

> 为什么我们现在在 K8s 中需要一个叫做注释的东西？

注释用于为 K8 对象添加元数据，K8s 并不关心这些。因此，注释中使用的键和值没有限制(不像标签)。

> **但是为什么不能用标签代替注释，为什么要增加更多的复杂性**？

如果有人想要为其他人添加关于给定资源的信息，并且不希望 K8s 关心它，注释是最好的工作。另外，注释非常灵活，可以无限制地包含任何内容。

**下面是注释**的一个例子

```
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  **annotations:**
    why_use_this_demo: "[I](https://hub.docker.com/)t's to learn about annotations"
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  selector:
    env: prod
```

> 让我们试着注释一个 pod

我们首先创建一个名为 pod10 的 pod。

```
MyK8sInstance> **kubectl run pod10 --image=nginx:alpine --restart=Never**
pod/pod10 created
```

然后用 msg 注释它，注意它必须是一个键/值对

```
MyK8sInstance> **kubectl get po | grep -i pod10**
pod10                           1/1     Running            0          22s
MyK8sInstance> **kubectl annotate pod pod10 msg="I like pod10"**
pod/pod10 annotated
```

为了验证注释是否有效

```
MyK8sInstance> **kubectl get pod pod10 -o yaml | grep  msg**
    msg: I like pod10
```

就这样吧，下次再见！
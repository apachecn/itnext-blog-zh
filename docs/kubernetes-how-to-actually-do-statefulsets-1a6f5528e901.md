# Kubernetes:如何实际做陈述集

> 原文：<https://itnext.io/kubernetes-how-to-actually-do-statefulsets-1a6f5528e901?source=collection_archive---------4----------------------->

![](img/258c3b74bb10cd90d76567d1d8f322cb.png)

[点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fkubernetes-how-to-actually-do-statefulsets-1a6f5528e901)

编辑:谷歌已经更新了他们的文档并修复了这个问题…

> TL；StatefulSets 博士刚刚在 k8s 1.9 中正式发布。这是我的故事，讲述了在与 Google 的教程和 Kubernetes 的官方文档争论之后，我是如何设法将 StatefulSet 部署到我的 GKE 集群的。此外，我在凌晨 1:00 写完这篇文章，所以如果你看到错别字，请在评论中注明。

下面是完整的 yaml 文件。这个故事的要点是，我不知道如何让一个国家设置部署在 GKE。部分原因是谷歌[没有在他们的 CLI sdk 工具包](https://github.com/kubernetes/kubernetes/issues/59867)中发布 kubectl 1.9，另一部分原因是谷歌的教程没有完全充实，另一个原因可能是我就是我，把事情搞砸了。我设法部署了我的 StatefulSet，这是我如何做的。如果你知道更好的方法或者知道如何工作，请在评论中给我留言:)

正如我之前在 [Medium](https://medium.com/u/504c7870fdb6?source=post_page-----1a6f5528e901--------------------------------) 和我的个人网站上提到的，我目前正在开发一个 slackbot(并在[https://categorylinksbot . io](https://categorylinksbot.io)上寻找 beta 用户)来帮助人们组织他们的链接。我的长期目标是为每个主要的消息平台建立一个 messenger 集成。然后利用这些数据建立一个比谷歌搜索更好的广告产品。在 GCP 之上建立一个击败谷歌搜索的广告平台的讽刺意味并没有被我忽略😜。

我的 slackbot 需要请求之间的有状态交互。我用 redis 来管理这些交互。Redis 很棒，但是如果 redis 运行的实例重新启动，那么所有的数据都会丢失。由于我在我的 GKE 集群中将 redis 作为可抢占实例的容器运行(因为我很便宜)，我的实例可以在任何时候重启，几乎没有任何警告。我需要能够定期在我的 redis 数据库上运行`[BGSAVE](https://redis.io/commands/bgsave)`,以便在可抢占实例再次启动时最小化最终用户问题。

在 Kubernetes 中，扩展有状态应用程序的正确方式是利用 StatefulSets。StatefulSets 允许您创建应用程序的副本(读:scale it ),然后它可以访问您用来存储由`BGSAVE`创建的文件的挂载卷。

首先，你需要定义一个`PersistentVolume` (pv)。在 [GCP 教程](https://cloud.google.com/kubernetes-engine/docs/how-to/stateful-apps)中，pv 描述了将要挂载的卷。GCP 教程中的例子有一个错误。它需要一个叫做`storageClassName`的东西被我们创建的 pvc 引用。下面的示例 [](#440f) 更新为 yaml 定义中的`storageClassName`参数。没有它，我们就不能绑定到这个 pv。

说到装订，`PersistentVolumeClaims` (pvc)将帮助我们做到这一点。看一下那个例子 [](#de84) 会再次显示**错误** - >否`storageClassName`。**免责声明:**如果你没有注意，只是复制粘贴了下面的文件，我已经为你添加了。所以不客气:)

现在，当您运行`kubectl apply -f my-pv.yaml`并对您的`my-pv-claim.yaml`文件(分别称为文件 [](#440f) 和文件 [](#de84) )执行同样的操作时，您应该看到该声明能够绑定到 pv，并且运行`gcloud compute disks list`或 w/e 也应该显示您创建的磁盘。请阅读教程中的[部分](https://cloud.google.com/kubernetes-engine/docs/how-to/stateful-apps#submitting_a_persistentvolumeclaim),因为它解释了当 pvc 没有绑定到 pv 时应该寻找什么。免责声明..

您需要的第三部分是与您的 StatefulSet 接口的服务。我有一个 nginx 容器来处理路由到我的应用程序的所有请求，所以这个服务已经创建好了，但是为了清楚起见，我把它放在了 [](#a90a) 下面，可能会让那些只是复制粘贴我的内容到页脚的读者感到困惑。

最后，我把 StatefulSet 的定义放在下面，因为制作页脚链接是一个皮塔饼(这里还有一个[链接](https://medium.com/@siranachronist/it-would-be-great-if-we-could-deeplink-footnotes-or-better-yet-show-them-in-the-margins-like-how-f5c8e93464af)，我在那里学会了如何做)。Google 教程和 kubernetes 官方文档的最后一个问题是 StatefulSet 定义中的`volumeClaimTemplates`部分的用法。对于我来说，当我运行`kubectl apply`时，我不能得到这种验证或任何东西。我试着做了很多没有运气的事情。最终我搜索了一下，找到了这篇博文。基本要点是，您应该在 StatefulSet 定义的`volumes`部分中使用`persistentVolumeClaim`关键字。

基本上，您的 StatefulSet 定义应该是这样的..

```
apiVersion: apps/v1beta2
kind: StatefulSet
metadata:
  name: [STATEFULSET_NAME]
spec:
  serviceName: [SERVICE_NAME]
  replicas: 3
  template:
    metadata:
      labels:
        app: [APP_NAME]
    spec:
      updateStrategy:
        type: RollingUpdate
      containers:
      - name: [CONTAINER_NAME]
        image: ...
        ports:
        - containerPort: 80
          name: [PORT_NAME]
        volumeMounts:
        - name: my-volume
          mountPath: /data-or-something
      **volumes**:
        - ...
        - name: my-volume
          **persistentVolumeClaim**:
            claimName: **$$YOUR-PVC-NAME**
```

应该就是这样！在评论区发表你的任何问题。我希望这可以作为一个很好的故障排除参考页面，帮助那些陷入 StatefulSets 的人。k8s 社区中的每个人都说 k8s 仍然在努力让有状态的应用变得正确。我认为陈述集是朝着正确方向迈出的一步。它们是一个巨大的特性，将为像我这样的小开发者提供大量的价值。我希望实际上摆脱谷歌的云 SQL 实例，只在 k8s 中运行 StatefulSet postgres 容器，这样可以节省一些钱。我也希望更多的人会在 k8s 中使用 StatefulSets 来构建真正酷的东西。

[【1】](#9d01):

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: $$NAME-ME-SOMETHING
spec:
  capacity:
    storage: 20Gi
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    pdName: [PD_NAME]
    fsType: ext4
```

[【2】](#11a9):

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: $$NAME-ME-SOMETHING
spec: 
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

[【3】](#3fd2):

```
apiVersion: v1
kind: Service
metadata:
  name: myApp
  namespace: default
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  type: NodePort
  selector:
    run: myApp
```
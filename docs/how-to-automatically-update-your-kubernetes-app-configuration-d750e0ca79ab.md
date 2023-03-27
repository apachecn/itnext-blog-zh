# 使用这种技术来自动更新您的 Kubernetes 应用程序配置

> 原文：<https://itnext.io/how-to-automatically-update-your-kubernetes-app-configuration-d750e0ca79ab?source=collection_archive---------0----------------------->

[“实践指南:使用 ConfigMap 对象配置您的 Kubernetes 应用程序”](/learn-how-to-configure-your-kubernetes-apps-using-the-configmap-object-d8f30f99abeb)博文介绍了如何使用 Kubernetes 中的`ConfigMap`对象将配置与代码分离。

通过`ConfigMap`在应用程序中使用环境变量(`Pod`或`Deployment`)带来了一个挑战——万一`ConfigMap`被更新，你的应用程序将如何获取新的值？显然，您可以删除并重新创建`Deployment`，但这在大多数情况下是不可取的。

一种可能的方法是将`ConfigMap`内容作为`Volume`加载。在这种情况下，Kubernetes 确保如果`ConfigMap`被更新，那么`Volume`的内容(包含配置值的文件)被刷新。

> 我非常希望得到您的反馈和建议！不要害羞，就 [*推文*](https://twitter.com/abhi_tweeter) *或者掉个评论。*

让我们用一个实际的例子来学习这个吧！

![](img/dc426662b9b2ac16e02cba0002aad0fc.png)

该代码可在 [GitHub](https://github.com/abhirockzz/kubernetes-configmap-auto-update) 上获得

# 先决条件

首先，您需要一个 Kubernetes 集群。这可能是一个简单的使用`[minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)`、`[Docker for Mac](https://blog.docker.com/2018/01/docker-mac-kubernetes/)`等的单节点本地集群。或者来自[Azure](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu)[Google](https://cloud.google.com/kubernetes-engine/)[AWS](https://aws.amazon.com/eks/)等的托管 Kubernetes 服务。

要访问您的 Kubernetes 集群，您将需要`[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)`，它很容易安装。

> *本例使用* `[*minikube*](https://kubernetes.io/docs/setup/learning-environment/minikube/)`

# 部署应用程序

先造出`ConfigMap`。

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kubernetes-configmap-auto-update/master/config.yaml](https://raw.githubusercontent.com/abhirockzz/kubernetes-configmap-auto-update/master/config.yaml)
```

> *为了简单起见，直接从*[*GitHub repo*](https://github.com/abhirockzz/kubernetes-configmap-auto-update)*中引用 YAML 文件，但是您也可以将文件下载到您的本地机器上，并以同样的方式使用它。*

下面是`ConfigMap`的内容——它包含三个键值对，作为`data`部分的一部分

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  foo: bar
  hello: world
  john: doe
```

创建使用`ConfigMap`的应用程序(作为 Kubernetes `Deployment`

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kubernetes-configmap-auto-update/master/app.yaml](https://raw.githubusercontent.com/abhirockzz/kubernetes-configmap-auto-update/master/app.yaml)
```

让我们看看`Pod`规格部分:

```
spec:
  containers:
  - name: configmaptestapp
    image: abhirockzz/configmaptestapp
    volumeMounts:
    - mountPath: /config
      name: appconfig-data-volume
    ports:
    - containerPort: 8080
  volumes:
    - name: appconfig-data-volume
      configMap:
        name: app-config
```

如[之前的一篇文章](/learn-how-to-configure-your-kubernetes-apps-using-the-configmap-object-d8f30f99abeb)中所解释的那样，`ConfigMap`中的每个键都作为一个文件添加到规范中指定的目录中，即`spec.containers.volumeMount.mountPath`，其值就是文件的内容。

通过使用这种方法，您可以受益于这样一个事实，即如果`ConfigMap`发生变化，卷会自动更新。

## 快速说明该应用程序的功能

这是一个简单的 [Go 应用程序](https://raw.githubusercontent.com/abhirockzz/kubernetes-configmap-auto-update/master/src/main.go)，它:

*   加载内存中的配置数据`map`
*   并允许您使用 REST 端点获取特定配置键的值

两个休息终点:

*   `/readconfig`:获取配置键的值
*   `/reload`:强制 app 从磁盘读取(刷新)配置数据到内存

为了访问 REST 端点，让我们使用一个`NodePort`服务来公开应用程序，并获取随机端口值

```
kubectl expose deployment configmaptestapp --type=NodePort
PORT=$(kubectl get service configmaptestapp -o=jsonpath='{.spec.ports[0].nodePort}')
```

> *不知道什么是* `*NodePort*` *服务也不用担心。目前，jsut 认为这是一种在 Kubernetes* 中提供应用程序访问的方式

# 测试一下…

获取您的`minikube`主机的 IP 地址

```
MINIKUBE_IP=$(minikube ip)
```

自省 app 的 REST 端点。在这种情况下，`foo`和`john`对应于配置密钥——它们只是目录`/config`中的文件

```
curl http://$MINIKUBE_IP:$PORT/readconfig/foo
//output - bar curl http://$MINIKUBE_IP:$PORT/readconfig/john
//output - doecurl http://$MINIKUBE_IP:$PORT/readconfig/junk
// output - Configuration 'junk' does not exist
```

可以直接自省`Pod`来(加倍)检查。从获得`Pod`名称开始

```
POD_NAME=$(kubectl get pods -l=app=configmaptestapp -o=jsonpath='{.items[0].metadata.name}')
```

查看`/config`目录—列出所有文件(配置密钥)

```
kubectl exec $POD_NAME -- ls /config///output
foo
hello
john
```

看特定的键

```
kubectl exec $POD_NAME -- cat /config/foo
//output - barkubectl exec $POD_NAME -- cat /config/john
//output - barkubectl exec $POD_NAME -- cat /config/junk
//output - cat: can't open '/config/junk': No such file or directory
```

好吧，这只是一个理智测试…

# …自动更新怎么样？

获取`config.yaml` ( `ConfigMap`)文件

```
curl https://raw.githubusercontent.com/abhirockzz/kubernetes-configmap-auto-update/master/config.yaml -o config.yaml
```

更改数值(例如将`foo`的数值更改为`baz`并将`hello`的数值更改为`universe`)并更新`ConfigMap`

```
kubectl apply -f config.yaml
```

如果您立即检查`Pod`文件系统，您将**而不是**看到更新。

```
kubectl exec $POD_NAME -- cat /config/foo
```

这是怎么回事？？

![](img/5c5f31e7d917cccb9e17b5930a0f972a.png)

这是一种最终一致的机制，其中包含延迟—原因是这由`kubelet`同步过程的频率和`kubelet` `ConfigMap`缓存的 TTL(生存时间)来处理。几秒钟后，您应该会看到更新后的值。

让我们确认 REST API 也以同样的方式运行

```
curl [http://$MINIKUBE_IP:$PORT/readconfig/foo](http://$MINIKUBE_IP:$PORT/readconfig/foo)
```

为什么我们会看到旧值？这是因为在这个特定的例子中，我们的应用程序从文件系统中读取数据(在启动阶段)并将其保存在`map`中。它还通过 REST 端点提供了一个`reload`工具——调用它

```
curl [http://$MINIKUBE_IP:$PORT/reload/](http://$MINIKUBE_IP:$PORT/reload/)
```

您现在应该会看到更新后的值

```
curl http://$MINIKUBE_IP:$PORT/readconfig/hello
//output - universecurl http://$MINIKUBE_IP:$PORT/readconfig/foo
//output - baz
```

就是这样。您看到了我们的应用程序如何能够在不重启的情况下使用更新后的 ConfigMap。

# 结束语…

这种方法有其优点、缺点(和警告)

**优点**

最大的好处是，你不需要重启你的应用程序(`Deployment`)，他们就可以开始使用`ConfigMap`中的更新数据。

**缺点**

*   代码变化:如果你使用环境变量(大多数应用程序都是这样)，你需要更新你的代码来从文件系统中读取，你还需要提供一个重新加载的能力
*   最终一致:你必须考虑实际的`ConfigMap`更新和批量数据同步之间的时间延迟——你的应用程序是否对此敏感是你需要考虑的事情。

> ***注意事项*** *—如果您将* `*ConfigMap*` *作为* `*subPath*` *卷*加载，此功能将不起作用

这个博客到此为止。如果你觉得这篇文章有用，请点赞并关注😃😃

> *如果你有兴趣学习 Kubernetes 和 Containers 使用*[*Azure*](https://azure.microsoft.com/services/kubernetes-service/?WT.mc_id=medium-blog-abhishgu)*，只需* [*创建一个* ***免费*** *账号*](https://azure.microsoft.com/en-us/free/?WT.mc_id=medium-blog-abhishgu) *就可以开始了！一个好的起点是使用文档中的* [*快速入门、教程和代码示例*](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu) *来熟悉该服务。我也强烈推荐查看一下* [*50 天 Kubernetes 学习路径*](https://azure.microsoft.com/resources/kubernetes-learning-path/?WT.mc_id=medium-blog-abhishgu) *。高级用户可能希望参考* [*Kubernetes 最佳实践*](https://docs.microsoft.com/azure/aks/best-practices?WT.mc_id=medium-blog-abhishgu) *或观看一些* [*视频*](https://azure.microsoft.com/resources/videos/index/?services=kubernetes-service&WT.mc_id=medium-blog-abhishgu) *以了解演示、主要功能和技术会议。*
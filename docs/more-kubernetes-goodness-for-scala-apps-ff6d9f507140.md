# 从 Scala 应用到 Kubernetes，第 2 部分

> 原文：<https://itnext.io/more-kubernetes-goodness-for-scala-apps-ff6d9f507140?source=collection_archive---------0----------------------->

我的副业 [Caterina](http://ticofab.io/Caterina-side-project/) 是关于(试图)将一种加密货币的具体事件与其价格波动联系起来。事实上，这更像是一个借口，从 A 到 Z 运行一个项目，同时[学习东西](https://medium.com/@ticofab/doing-things-the-proper-way-b085068cba71)。

一切都在云端运行，这是我的一个主要学习点。这篇文章和[上一篇](https://medium.com/@ticofab/from-scala-app-to-kubernetes-pod-d67e0cd6bfaf)是关于我如何把我的 JVM 服务从我舒适的本地环境带到云和更远的地方。

![](img/4f947c7015d0c7a554829218023331df.png)

攀登通往天堂的阶梯，版权所有

## 将 pod 生命周期委托给控制器

我首先将服务作为单个 Pod 运行，这是 Kubernetes 上下文中的基本执行单元。但是，[这不是推荐的方式](https://cloud.google.com/kubernetes-engine/docs/concepts/pod)。相反，我们应该将 Pod 的生命周期委托给控制器，比如部署。控制器管理 pod 的生命周期，这意味着他们可以在崩溃时重新启动它，或者旋转同一 pod 的多个实例以实现水平扩展。我们只需要稍微修改一下我们的 yaml 文件:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  selector:
    matchLabels: 
      app: my-service-pod     // <- which pod? (see metadata below)
  replicas: 1                 // <- only one instance of this pod 
  template:
    metadata:
      name: my-service
      labels:
        app: my-service-pod
    spec:
      containers:
      - image: eu.gcr.io/my-project-id/my-app:latest
        name: my-app-image
        env:
        - name: MY_ENV_VARIABLE
          value: the_value_of_my_env_variable
```

我知道——有点冗长。有一些工具添加了额外的抽象层，为 yaml 文件提供模板，以减少复制/粘贴和样板文件。最受欢迎的是[赫尔姆](https://helm.sh)。我的边项目只有[两个服务](https://medium.com/@ticofab/from-event-storming-to-architecture-c2dc49e9c2d0)，所以[我用基本 yaml 文件](https://en.wikipedia.org/wiki/Rule_of_three_(computer_programming))就可以了。

检查我们集群的状态现在也需要查看部署:

```
kubectl get pods,deployment
```

它将向我们展示，我们确实有一个运行一个单元的部署:

```
NAME                            READY     STATUS    RESTARTS   AGE
pod/my-service-pod-85f8-ctcdq   1/1       Running   0          10mNAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
deployment/my-service   1        1        1           1          10m
```

注意，我们的 pod 的名称带有一个惟一的字符串(`85f8-ctcdq`)，它是由控制器添加的。我们只要求一个实例(在`replicas: 1`中)，但可能有更多，随机字符串确保每个实例都有唯一的名称。

如果 pod 由于任何原因(例如您没有管理的内部故障)而终止，控制器将检测它的过早消失并启动一个新的实例。相当狡猾。

## 在 Kubernetes 的秘密中保护您的敏感数据

![](img/3a868a52db2595f575e685d4e7d226b4.png)

Kubernetes 提供“秘密服务”。您可以将密码或 API 密钥嵌入到 Kubernetes 会为您安全存储的一个 [Secret](http://kubernetesbyexample.com/secrets/) 对象中，而不是将它们写在 yaml 文件中。然后，您可以将机密数据作为公开的环境变量或装入卷中的文件来访问。

在上面的 yaml 文件中，我们有一个环境变量:

```
env:
  - name: MY_ENV_VARIABLE
    value: the_value_of_my_env_variable
```

用它创建一个秘密对象是通过`kubectl`完成的。我可以创建一个名为`my_env_var`的文件，其中包含`the_value_of_my_env_variable`。命令

```
kubectl create secret generic my-env-secret --from-file=./my_env_var
```

创建一个名为`my-env-secret`的键-值秘密，其中键`my_env_var`指向值`the_value_of_my_env_variable`。我们现在可以将环境变量 inclusion 重新连接为:

```
env:
  - name: MY_ENV_VARIABLE
    valueFrom:
      secretKeyRef:
        name: my-env-secret
        key: my_env_var
```

就像之前一样，我们的服务将看到一个名为`MY_ENV_VARIABLE`和值为`the_value_of_my_env_variable`的环境变量。

在挂载卷上以文件形式访问机密有一个基本情况:存储凭证以从 pod 内部访问其他 Google 云服务。程序略有不同，在此以[为例](https://cloud.google.com/kubernetes-engine/docs/tutorials/authenticating-to-cloud-platform)进行说明。

## 记录

![](img/7381c9dd30bdb7712e9eb6c2fc220daf.png)

您可以通过`kubectl logs`从您的容器中访问日志，但是这不会被持久化。使用谷歌的云日志监控资源 [Stackdriver](https://cloud.google.com/stackdriver/) 要好得多。事实上，它在所有新创建的 Kubernetes 集群中都是默认启用的。

我唯一悬而未决的问题是日志的严重性。我使用的是[机身日志](https://github.com/wvlet/airframe/tree/master/airframe-log)，一个 Scala 的轻量级日志记录器。出于某种原因，所有日志都作为错误出现在 Stackdriver 中。这显然很烦人，一旦我找到解决办法，我会尽快更新这个帖子。

**更新**:解决了！我显然没有看过文件。我将日志格式切换到 Airframe `BareFormatter`(简单地按原样记录字符串)，并添加了一个`severity`字段。典型的日志现在看起来像这样

```
{ "msg" : "....", "severity" : "INFO" }
```

Stackdriver 现在从这个有效载荷中剥离出`severity`部分，并相应地标记它的消息——它包含了另一个丰富的信息。

## 连续交货？

![](img/7d4f6b91572b1f0fbce0fee4826aad6f.png)

在我的 Caterina 项目的[架构概述](https://medium.com/@ticofab/from-event-storming-to-architecture-c2dc49e9c2d0)中，我提到我选择 BitBucket 作为云版本控制提供商，因为我很想尝试他们的流水线特性。实际上，我在 BitBucket 上托管了我的回购，我对管道做了一点小小的尝试，但最终选择了*而不是*进行任何连续交付设置。

这个决定是在注意到设置这一切的工作量之后做出的。当我想要部署服务的新版本时，我需要:

*   构建 docker 图像(`sbt docker:publishLocal`)
*   上传到 Google 容器注册表(`docker push --[SERVICE]:latest`)
*   停止当前正在运行的部署(`kubectl delete -f all.yaml`
*   告诉 Kubernetes 重新开始部署(`kubectl apply -f all.yaml`

这样做的缺点是，为了进行部署，我需要在我的机器上安装所有的东西。但是要从管道系统进行构建和部署，我需要安排云凭证、安装 SBT 和下载依赖项，这意味着我应该把我的公共库放在某个可用的地方(比如 [JFrog artifactory](https://jfrog.com/artifactory/) )。总的来说，这将是额外的工作和成本，我不能完全证明。

因为我只是一个有两个服务的单身男人，一个 *shell 脚本*看起来更快。

我的云之旅还远未结束，但我已经学到了很多。最有收获的是随之而来的赋权。下一步:巩固管道，让它运行一段时间。然后我的数据地盘就可以玩了！

[](https://medium.com/@ticofab) [## 法比奥·特里蒂奇科-中等

### 技术领导和社区人员。寻找一个 DevRel 角色！

medium.com](https://medium.com/@ticofab)
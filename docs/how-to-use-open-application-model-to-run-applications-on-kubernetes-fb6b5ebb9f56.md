# 如何使用开放应用模型在 Kubernetes 上运行应用

> 原文：<https://itnext.io/how-to-use-open-application-model-to-run-applications-on-kubernetes-fb6b5ebb9f56?source=collection_archive---------5----------------------->

[开放应用模型](https://oam.dev/) (OAM)是构建云原生应用的规范，以`[Rudr](https://github.com/oam-dev/rudr)`作为其 [Kubernetes](https://kubernetes.io/) 的具体实现。在这篇博客中，我们将通过几个例子来强化前一篇博客中提到的与`OAM`和`Rudr`相关的概念。

[](https://medium.com/@abhishek1987/building-cloud-native-apps-intro-to-open-application-model-and-rudr-bd1b55df9bf3) [## 构建云原生应用:开放应用模型和 Rudr 简介

### 生产级分布式云原生微服务通常由多个合作伙伴构建和运营…

medium.com](https://medium.com/@abhishek1987/building-cloud-native-apps-intro-to-open-application-model-and-rudr-bd1b55df9bf3) 

我们将从使用`Rudr`组件在 Kubernetes 上运行一个简单的应用程序开始，然后看一个如何使用`[Rudr](https://github.com/oam-dev/rudr/blob/master/docs/concepts/traits.md)` [特征](https://github.com/oam-dev/rudr/blob/master/docs/concepts/traits.md)的例子。

> *代码在* [*GitHub*](https://github.com/abhirockzz/rudr-k8s-sample) 上有

# 先决条件

在其核心，`Rudr`是一个定制控制器，作为 Kubernetes `Deployment`运行。要安装 Rudr，您将需要一个版本为`1.15.x`或`1.16.x`的 Kubernetes 集群(在撰写本文时这些是受支持的版本)。任何集群都可以工作，但是我在这篇博客中使用了 [Azure Kubernetes 服务](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu)作为例子。

如果你想使用 AKS，你只需要一个 Azure 订阅([在这里抢一个免费账号！](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu))和 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu) 来使用`[az aks create](https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-aks-create)`命令设置一个托管的 Kubernetes 集群。

下面是一个运行 Kubernetes 版本`1.15.7`的单节点集群的例子

```
az aks create --resource-group <AZURE_RESOURCE_GROUP> --name <AKS_CLUSTER_NAME> --kubernetes-version 1.15.7 --node-count 1 --node-vm-size Standard_B2s --node-osdisk-size 30 --generate-ssh-keys//point kubectl to AKS
az aks get-credentials --resource-group <AZURE_RESOURCE_GROUP> --name <AKS_CLUSTER_NAME>//confirm
kubectl get nodes
```

在[安装](https://helm.sh/docs/intro/) `[Helm 3](https://helm.sh/docs/intro/)`后，您可以继续进行`Rudr`的设置

```
//clone the repo
git clone https://github.com/oam-dev/rudr.git//install it using Helm
helm install rudr ./charts/rudr --wait//confirm Rudr Deployment
kubectl get deployment rudr//check Rudr CRDs
kubectl get crds -l app.kubernetes.io/part-of=core.oam.dev
```

# 用 Rudr 部署一个简单的应用

我们将从一个使用以下 Rudr 对象部署简单应用程序的基本示例开始:`ComponentSchematic`和`ApplicationConfig`。

这个应用程序非常简单——它是一个容器化的应用程序，公开了一个端点，默认情况下用`Hello World!`响应，如果设置了`GREETING`环境变量，则用`Hello <greeting>`响应。要在 Kubernetes 中运行这个，显而易见的方法是使用一个`Deployment`对象。相反，我们将创建`Rudr`自定义资源定义(CRD)来表示我们的应用程序，将它们提交给 Kubernetes，并让`Rudr`控制器/操作员负责处理特定的 Kubernetes 资源。

## 部署 Rudr CRDs

我们将从创建一个`ComponentSchematic`开始。我们来自省一下:

```
apiVersion: core.oam.dev/v1alpha1
kind: ComponentSchematic
metadata:
  name: greeter-component
spec:
  workloadType: core.oam.dev/v1alpha1.Server
  containers:
    - name: greeter
      image: abhirockzz/greeter-go
      env:
        - name: GREETING
          fromParam: greeting
      ports:
        - protocol: TCP
          containerPort: 8080
          name: http
      resources:
        cpu:
          required: 0.1
        memory:
          required: "128"
  parameters:
    - name: greeting
      type: string
      default: abhi_tweeter
```

这是一个名为`greeter-component`的`ComponentSchematic`，它的`workloadType`是`core.oam.dev/v1alpha1.Server`——它决定了为处理这个组件而创建的 Kubernetes 资源的类型。它有一个单独的`container`引用 Docker Hub 上的`abhirockzz/greeter-go`图像。`parameters`部分定义了一个名为`greeting`的可配置属性，其默认值为`abhi_tweeter`。该参数在`containers`部分的`env`属性中作为环境变量被引用。

如此创建`ComponentSchematic`:

```
kubectl apply -f https://raw.githubusercontent.com/abhirockzz/rudr-k8s-sample/master/deploy/component.yaml//outputcomponentschematic.core.oam.dev/greeter-component created
```

这将在 Kubernetes 中创建一个`ComponentSchematic`对象——您可以使用`kubectl get components`来确认。单靠自己真的做不了多少事情。它需要另一个`Rudr`实体与之合作——即`ApplicationConfig`。让我们看看这种情况下是什么样的:

```
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  name: greeter-app-config
spec:
  components:
    - componentName: greeter-component
      instanceName: greeter-app
```

`ApplicationConfig`是一个`ComponentSchematic`的实例——在这种情况下，它指的是`greeter-component`T3。

创造了`ApplicationConfig`:

```
kubectl apply -f https://raw.githubusercontent.com/abhirockzz/rudr-k8s-sample/master/deploy/app-config.yaml//output
applicationconfiguration.core.oam.dev/greeter-app-config configured
```

要确认:

```
kubectl get applicationconfiguration.core.oam.dev/greeter-app-config -o yaml
```

仔细查看`status`部分——您应该会看到类似这样的内容:

```
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
...
status:
  components:
    greeter-component:
      deployment/greeter-app: running
      service/greeter-app: created
  phase: synced
  phase: synced
```

## 检查 Kubernetes 对象

`Rudr`为我们创造了一堆 Kubernetes 资源- `Deployment`、`Pod`和`Service`

检查`Deployment`

```
kubectl get deployment/greeter-appNAME          READY   UP-TO-DATE   AVAILABLE   AGE
greeter-app   1/1     1            1           42s
```

检查`Pod`

```
kubectl get pod -l=app.kubernetes.io/name=greeter-app-configNAME                           READY   STATUS    RESTARTS   AGE
greeter-app-586b5d4ddc-wrqtb   1/1     Running   0          2m30s
```

最后，Kubernetes `Service`资源

```
kubectl get service/greeter-appNAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
greeter-app   ClusterIP   10.0.135.117   <none>        8080/TCP   4m15s
```

# 测试应用程序

访问应用程序最简单的方法是使用端口转发

> *确保你替换了*的名字`*Pod*`

```
kubectl port-forward pod/<pod name> 9090:8080Forwarding from 127.0.0.1:9090 -> 8080
Forwarding from [::1]:9090 -> 8080
```

现在你可以简单地`[curl](https://curl.haxx.se/)`终点

```
curl localhost:9090//output
Hello abhi_tweeter!
```

就这样。这是一个在 Kubernetes 中运行的应用程序的非常简单的例子，它仅使用 Rudr 构造创建。

作为练习，您可以尝试创建下面的`ApplicationConfig`并按照上面概述的步骤来访问应用程序以及结果是什么

> *注意，这个* `*ApplicationConfig*` *使用* `*parameterValues*` *段来覆盖* `*parameters*` *中定义的* `*ComponentSchematic*` *。*

```
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  name: greeter-app-config-2
spec:
  components:
    - componentName: greeter-component
      instanceName: greeter-app-2
      parameterValues:
        - name: greeting
          value: foobar
```

# 利用特质

在前面的示例中，创建的部署对象有一个 Pod(单个应用程序实例)。您可以使用`kubectl scale`命令来扩展它，但是我认为您现在已经得到了流程——我们不会这样做！让我们利用 Rudr 中的手动缩放器特性来实现这一点。

我们将继续使用相同的`ComponentSchematic`，并引入一个新的`ApplicationConfig`定义，以确保我们的应用程序有两个副本。我们将借助手动定标器特性来实现这一点

```
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  name: greeter-app-config-3
spec:
  components:
    - componentName: greeter-component
      instanceName: scalable-greeter-app
      parameterValues:
        - name: greeting
          value: scalable
      traits:
        - name: manual-scaler
          properties:
            replicaCount: 2
```

`greeter-app-config-3ApplicationConfiguration`引用了`greeter-component` `ComponentSchematic`。注意`traits`部分，这里我们使用了一个`manual-scaler`，并将`replicaCount`指定为`2`。为了确保我们能够将它与前面的应用程序区分开来，我们重写了 paramater，将`greeting`的值作为`scalable`传入

创建`ApplicationConfiguration`

```
kubectl apply -f https://raw.githubusercontent.com/abhirockzz/rudr-k8s-sample/master/deploy/manual-scaler-trait/app-config.yaml//output
applicationconfiguration.core.oam.dev/greeter-app-config-3 created
```

等待几秒钟，确认 Rudr 已经触发了`Kubernetes`对象的创建:

```
kubectl get applicationconfiguration.core.oam.dev/greeter-app-config-3 -o yaml
```

您应该会看到一个`status`部分

```
status:
  components:
    greeter-component:
      deployment/scalable-greeter-app: running
      service/scalable-greeter-app: created
  phase: synced
```

> *如果* `*deployment/scalable-greeter-app*` *处于* `*unavailable*` *状态，请在 10 秒左右后重试*

确认`Deployment`对象

```
kubectl get deployment/scalable-greeter-appNAME                   READY   UP-TO-DATE   AVAILABLE   AGE
scalable-greeter-app   2/2     2            2           4m
```

也检查单个的`Pod`

```
kubectl get pods -l=app.kubernetes.io/name=greeter-app-config-3NAME                                    READY   STATUS    RESTARTS   AGE
scalable-greeter-app-6488f64cb4-mj6nw   1/1     Running   0          5m
scalable-greeter-app-6488f64cb4-rpfvp   1/1     Running   0          5m
```

要访问该应用程序，只需运行安装有`curl`的一次性`Pod`即可。一旦进入 Pod，您可以简单地使用`curl $SCALABLE_GREETER_APP_SERVICE_HOST:$SCALABLE_GREETER_APP_SERVICE_PORT`来调用应用程序的端点。

```
kubectl run curl --image=radial/busyboxplus:curl -i --tty --rm[ root@curl-6bf6db5c4f-5hw6t:/ ]$ curl $SCALABLE_GREETER_APP_SERVICE_HOST:$SCALABLE_GREETER_APP_SERVIC
E_PORT
Hello scalable!
```

> `*Rudr*` *创建的* `*ClusterIP*` `*Service*` *使`*SCALABLE_GREETER_APP_SERVICE_HOST*` *和* `*SCALABLE_GREETER_APP_SERVICE_PORT*` *成为环境变量。**

预期的响应是`Hello scalable!`，因为我们已经覆盖了`ApplicationConfiguraton`中的`greeting` `parameter`

# 打扫

您可以使用`[az aks delete](https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-aks-delete)`命令删除整个 AKS 集群或者删除单个的`ApplicationConfiguration`来触发级联删除所有与它相关的 Kubernetes 资源(`Deployment`等)。).要删除`Rudr`部署，如果您还想删除`Rudr` CRDs ( `components`、`configurations`等)，只需使用`helm delete rudr`和`kubectl delete crd -l app.kubernetes.io/part-of=core.oam.dev`即可。)

这就是这个关于`Open Application Model`和`Rudr`的两部分系列的全部内容，还有一个实际操作的例子，让你感受一下如何在 Kubernetes 上实际使用它。如果你觉得这篇文章有帮助，请喜欢并关注🙌很高兴通过 [Twitter](https://twitter.com/abhi_tweeter) 获得反馈，或者随时发表评论。
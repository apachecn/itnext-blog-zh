# 用 Kubeless 探索无服务器计算

> 原文：<https://itnext.io/exploring-serverless-computing-with-kubeless-63c30bd96b2f?source=collection_archive---------1----------------------->

## k3s 上使用 Kubeless 的第一步

![](img/abd6496db0c12c1d1962fd200572318b.png)

照片由[格雷格·拉科齐](https://unsplash.com/@grakozy?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

至少对我来说，是时候不用服务器了。作为一名开发人员，我想尽可能快地编写应用程序代码并让它在“某个地方”运行。

我不想在开发期间换帽子，只是通过创建新的 Docker 映像和更改一些 Kubernetes 配置来将我的代码放入一些云中。

是的，当然，我可以设置一些 CI 系统，但有时这在本地开发环境中有点太多了。

在我看来，无服务器模式进入了正确的方向。是时候感受一下了。

我选择了 Kubeless 作为框架，因为我想在我的本地 k3s 集群上使用它。

[](https://levelup.gitconnected.com/setup-your-own-kubernetes-cluster-with-k3s-take-2-k3sup-a5d9453f709f) [## 使用 K3s — k3sup 设置您自己的 Kubernetes 集群

### 编辑描述

levelup.gitconnected.com](https://levelup.gitconnected.com/setup-your-own-kubernetes-cluster-with-k3s-take-2-k3sup-a5d9453f709f) 

在他们的网站上，他们是这样描述的

> Kubeless 是一个 Kubernetes-native 无服务器框架，允许您部署少量代码(功能),而不必担心底层基础设施。它旨在部署在 Kubernetes 集群之上，并利用所有优秀的 Kubernetes 原语。如果你正在寻找一个开源的无服务器解决方案，克隆你能在 AWS Lambda、Azure Functions 和 Google Cloud Functions 上找到的东西，Kubeless 正适合你！

因此，这篇文章是一个关于 Kubeless 框架的教程，开始探索部署无服务器功能意味着什么。

说够了，让我们开始吧。

等一下，在我们开始之前，我必须提到我的设置是

*   MacBook Pro 作为本地工作站
*   运行在 Ubuntu 上的双节点 k3s 集群

如果你的设置非常不同，请查看 Kubeless 网站，见本文末尾的链接。

## 装置

首先，我们用以下命令安装所有东西:

```
$ export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
$ kubectl create ns kubeless
$ kubectl create -f [https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml](https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml)
```

和其他好的工具一样，Kubeless 也有自己的命令行界面来使事情变得简单一些。

```
$ export OS=$(uname -s| tr '[:upper:]' '[:lower:]')
$ curl -OL https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless_$OS-amd64.zip && unzip kubeless_$OS-amd64.zip && sudo mv bundles/kubeless_$OS-amd64/kubeless /usr/local/bin/
```

检查它是否已正确安装:

```
$ kubeless version
```

让我们继续部署一些功能。

## 部署功能

现在，我们将部署我在开始时提到的那些“小代码”，即所谓的“功能”。

让我们通过将以下内容写入名为`echo.py`的文件来创建代码。

```
def echo(event, context):
  print(event)
  return event['data']
```

然后所有东西都进入集群:

```
$ kubeless function deploy echo --runtime python3.8 --from-file echo.py --handler echo.echo
```

仅此而已！

快速测试一下函数是否创建正确:

```
$ kubeless function ls echo
NAME NAMESPACE  HANDLER     RUNTIME     DEPENDENCIES    STATUS
echo     default      echo.echo python3.8                     1/1 READY
```

特别是检查最后一栏，那个`STATUS`实际上是`READY`。

如果不工作，请查看“故障排除”一节。

## 试验

最后，是时候通过调用`kubeless`来测试这个函数了:

```
$ kubeless function call echo --data 'Hello World!'
Hello World!
```

成功！

## 清除

如果您想再次删除所有内容，这里有三个命令来清理刚刚安装的内容。

```
$ kubeless function delete echo
$ kubectl delete -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
$ kubectl delete ns kubeless
```

## 不使用`kubeless`的情况下使用 Kubeless

好诡异的标题，不过看完这一节还是有道理的。

我们仍然在 Kubernetes 上，一切工作都不需要任何额外的 CLI，如`kubeless`。

在 Kubernetes 中，无库功能只不过是一个“自定义资源定义”(CRD)。因此，无库函数变成了 Kubernetes 对象，我们可以像以前一样使用`kubectl`。

让我们试一试。

现在，让我们删除前面的函数。如果尚未这样做，请参见“清理”一节。

将以下内容放入一个名为`echo.yaml`的文件中。

```
---
apiVersion: kubeless.io/v1beta1
kind: Function
metadata:
  name: echo
spec:
  deployment:
    metadata:
      annotations:
        "annotation-to-deploy": "value"
    spec:
      replicas: 2
      template:
        metadata:
          annotations:
            "annotation-to-pod": "value"
  deps: ""
  function: |
    def echo(event, context):
      print(event)
      return event['data']
  function-content-type: text
  handler: echo.echo
  runtime: python3.8
  service:
    selector:
      function: echo
    ports:
    - name: http-function-port
      port: 8080
      protocol: TCP
      targetPort: 8080
      nodePort: 30333
    type: NodePort
```

然后，像往常一样将其安装到集群上:

```
$ kubectl apply -f echo.yml
```

检查它是否已经部署:

```
$ kubectl get svc echo
NAME   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
echo   ClusterIP   WW.XX.YYY.Z   <none>        8080/TCP   16m
```

一切都好！

现在，为了测试服务，我们需要将集群节点的端口转发到我们的本地机器。

```
$ kubectl port-forward service/echo 8080:8080
```

然后，在另一个终端，我们进入集群`curl`来调用我们的函数。

```
$ curl -L --data '{"echo":"Hello World!"}' --header "Content-Type:application/json" localhost:8080/api/v1/namespaces/default/services/echo:http-function-part/proxy/
```

瞧，这就是我们的输出。希望…如果没有，继续下一节。

## 解决纷争

第一次浏览本教程时，很可能有些地方没有按预期运行。在部署我的第一个函数后，我遇到了一个问题。该服务不可达，我无法调用该功能。我不知道为什么它不起作用。

检查您的功能时，该状态可能是“未准备好”而没有变为“准备好”，并且相关 pod 的状态可能无限期地处于“CrashLoopBackOff”状态。

通常的“策略”是检查 Kubernetes pod 的日志。

调用这个来获取 pod 的名称。

```
$ kubectl get pods
NAME      READY   STATUS        RESTARTS   AGE
echo-xzy  1/1     Running       0          4m35s
```

然后，获取日志。

```
$ kubectl logs echo-xzy
```

直到那时，我才发现，我想使用一个不再可用的 Python 版本，我必须切换到一个更高的版本(> = 3.x)才能让它工作。我还必须改变我的代码的语法，以使它再次工作，因为在 Python 2 中这是有效的代码

```
print "Hello World!"
```

但是在 Python 3.x 中，语法需要

```
print("Hello World!")
```

只是为了开心。

## 结论

希望你的第一次无服务器之旅是成功的。我真的很喜欢它，并且我看到未来有许多改变的可能性，特别是简化我的开发工作流程。

现在是时候探索更多已经存在的函数并自己编写一些了。我在下面放了一些链接，让你继续下去。

如果你喜欢这篇文章，请给我买杯咖啡。。

## 资源

*   [无所不能](https://kubeless.io)
*   [安装](https://kubeless.io/docs/quick-start/)
*   [功能](https://github.com/kubeless/functions)
*   什么是 Serverless or Kubernetes?Serverless ON Kubernetes!(T1 )
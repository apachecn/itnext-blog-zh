# 如何使用 minikube 和您的本地 docker 文件在 Kubernetes 上进行本地实验

> 原文：<https://itnext.io/how-to-experiment-locally-on-kubernetes-with-minikube-and-your-local-dockerfiles-48833fcd90c9?source=collection_archive---------2----------------------->

![](img/0e28ec56f0743444de32593c7c2881a0.png)

尼兰塔·伊兰加穆瓦在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# 介绍

在这篇入门级文章中，我介绍了一种在本地开发环境中使用 Minikube(本质上是计算机上的 Kubernetes)构建、公开和测试 dockerized 应用程序的简单方法。有时，开发人员很难处理实验图像，因为对于每个图像更改，他们都将其推送到注册表，然后在测试 K8s 集群上拉出。下面，我们将只使用本地环境—即使您没有网络连接，这也是可行的。

我们将收集必要的工具，安装 Minikube 并构建和测试我们自己的示例应用程序。

# TL；灾难恢复版本

1.您可以在自己的笔记本电脑上使用 Minikube 轻松运行简单的 Kubernetes 环境，而无需创建虚拟机。用` *docker 驱动*就行了。

2.你可以插入 Minikube 的 Docker deamon，你只需要运行' minikube docker-env '并按照说明操作。

3.您可以使用此连接来构建 Docker 映像，以便 Minikube K8s cluster 自动看到它。

4.然后，您可以基于构建的映像创建 Kubernetes 部署，并将其作为服务公开，而不必将其推送到某个外部注册中心。

# 收集工具集

**获取 Docker**

既然您正在阅读这篇文章，我假设您的机器上已经安装了 Docker。如果没有，最好的地方是 Docker 文档页面，它的子页面致力于流行的发行版，比如 Ubuntu Linux 。

**获取 Kubectl**

为了检查是否安装了“kubectl”工具，只需运行:

```
$ kubectl version --client
```

结果应该看起来有点像这样:

```
Client Version: version.Info{Major:”1", Minor:”19", GitVersion:”v1.19.2", GitCommit:”f5743093fd1c663cb0cbc89748f730662345d44d”, GitTreeState:”clean”, BuildDate:”2020–09–16T13:41:02Z”, GoVersion:”go1.15", Compiler:”gc”, Platform:”linux/amd64"}
```

如果您的系统上没有` *kubectl* 工具，那么获取最新` *kubectl* 工具的流行方法是直接从 *kubernetes.io* 站点获取。我建议你这样放入你的` */usr/local/bin/* `文件夹:

```
$ curl -Lo kubectl “https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" && chmod +x kubectl$ sudo mv kubectl /usr/local/bin/
```

当然，如果你使用的是 Windows 或 MacOS，你需要改变这个方法。您可以访问 [Kubernetes 文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/)获取说明。

**获取 Minikube**

由于“ *minikube* ”是一个独立的应用程序，与“ *kubectl* ”的方式非常相似，因此安装也非常相似:

```
$ curl -Lo minikube [https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64) && chmod +x minikube$ sudo mv minikube /usr/local/bin
```

请记住，移动到“ */usr/local/bin* ”只是一个建议——这是我习惯遵循的方式，以便在我的道路上拥有所有有用的工具；你可以随意改变它。只要确保` *minikube* `命令正常工作即可:

```
$ minikube versionminikube version: v1.13.0
commit: 0c5e9de4ca6f9c55147ae7f90af97eff5befef5f-dirty
```

# **轻松跑迷你库**

Minikube 支持许多关于运行 Kubernetes 组件的环境的选项，比如 KVM 或 VirtualBox。然而，这些都不是必需的，安装后会使您的开发环境变得混乱。

Minikube 还提供“none”和“podman”驱动程序，但这些驱动程序需要特权访问，在某些情况下可能会覆盖一些敏感的系统配置。

我建议你简单地使用“docker”驱动程序，因为它不需要 root 权限，对你运行 Minikube 的系统配置没有威胁。

Minikube 的第一次运行是一个简单的命令行，示例结果如下:

```
$ minikube start — driver=docker
```

> 😄基于用户配置使用 docker 驱动程序的 Linuxmint 19.3
> ✨上的 minikube v 1 . 13 . 0
> 👍启动集群迷你库
> 中的控制平面节点迷你库🚜拉出基础图像…
> 💾正在下载 Kubernetes v1.19.0 预加载…
> >预加载-images-k8s-V6-v 1 . 19 . 0-docker-overlay 2-amd64 . tar . lz4:486.28 MiB
> 🔥正在创建 docker 容器(CPUs = 2，内存=2200MB) …
> 🐳在 Docker 19.03.8 上准备 Kubernetes v 1 . 19 . 0…
> 🔎验证 Kubernetes 组件…
> 🌟启用的插件:默认存储类，存储供应器
> 🏄搞定了。kubectl 现在默认配置为使用“minikube”

完成上述操作后，您可以检查 minikube 部署状态:

```
$ minikube statusminikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

**重用 Docker 守护进程**

正如 [Minikube 文档](https://kubernetes.io/docs/setup/learning-environment/minikube/)所述，“*当对 Kubernetes 使用单个 VM 时，重用 Minikube 的内置 Docker 守护进程是有用的。重用内置的守护进程意味着你不必在你的主机上建立一个 Docker 注册表，然后把镜像推送到里面*。

我已经完整地引用了上面的句子，因为没有更好的词来描述这个想法。但是这个注释很容易在快速入门页面中被忽略，因为它对于在本地机器上进行快速实验是至关重要的！

我们将“插入”minikube 的 Docker 环境，因此我们将构建的任何映像都将在我们的 Minikube 集群中立即可见，就像它被推送到 DockerHub 或其他存储库一样。

在终端中运行的命令是:

```
$ eval $(minikube -p minikube docker-env)
```

请记住，这将为这个特定的终端会话设置环境！从现在开始，你使用' docker '命令做的任何事情，都将由 Minikube 中的 Docker 解释。如果您销毁这个终端会话或启动另一个终端会话，您需要重复如上所示的“eval”命令。

# **准备和构建映像**

现在，我们将进行快速 DevOps 配方中的“开发”部分。我们需要创建两个文件——一个是在容器中运行的应用程序，另一个包含如何构建运行该应用程序的 Docker 容器的说明。

**粗 WWW 应用**

对于我们的简单应用程序，我们将使用 Arundel & Dominguis 的“带有 Kubernetes 的云原生 DevOps”的思想创建 go http 服务器。如果你还没有接触过这本书，我强烈推荐它！

这种选择有一个特定的原因 golang 应用程序可以编译并构建到非常小的优化容器中，当构建到“scratch”容器中时，还会留下非常小的 *_attack surface_* 。基本上，我们将在 Docker 内创建一个应用程序，里面什么也没有。

创建一个“hello.go”文件，并在您选择的编辑器中进行编辑，以实现以下目的:

```
package hello
import (
   "fmt"
   "log"
   "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
   fmt.Fprintln(w, "It works!")
}
func main() {
   http.HandleFunc("/", handler)
   log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**用于 WWW 应用的 Docker 图像**

接下来，在同一个目录中创建一个` *Dockerfile* `,内容如下:

```
FROM golang:1.15-alpine AS base
WORKDIR /src/
COPY hello.go /src/
ENV CGO_ENABLED=0 
RUN go build -o /bin/hello
FROM scratch
COPY --from=base /bin/hello /bin/hello
ENTRYPOINT ["/bin/hello"]
```

现在，如果您在连接到 Minikube 的 Docker deamon 的终端中构建容器，它将立即在 K8s 集群中可用。

若要使用连接，请运行下面的命令。记得留在 Dockerfile 放的目录下！如你所见，我们使用了“1.0”标签。注意末尾的那个点。

```
$ docker build -t go-hello:1.0 .
```

重要提示:*** *始终**** 标记图像。不要对本地图像使用`*最新的*标签。否则，您可能会在 Minikube 的 K8s 中遇到“ *ErrPullBackoff* ”错误。

# **展开和暴露**

在生产中，您应该始终遵循 IaC 和 DevOps 原则。就 Kubernetes 而言，您至少应该有部署和服务 YAML 文件，甚至是重新部署应用程序的导航图。

在您的本地环境中，当您进行试验时，您可以只创建部署，并使用两个简单的命令公开它:

```
$ kubectl create deployment go-hello --image=go-hello:1.0$ kubectl expose deployment go-hello --type=NodePort --port=8080
```

请记住将标签设置为您在上一小节中使用的标签。

部署后，如果服务已成功创建，您可以在 K8s 集群中验证应用程序行为，并使用 Minikube 的内置功能在浏览器窗口中验证服务:

```
$ kubectl get svc go-hello$ kubectl get deploy go-hello -o wide$ kubectl get pods -l=app=go-hello$ minikube service go-hello
```

最后一个命令将打开您的服务浏览器，并显示连接值，如下例所示。

```
| default | go-hello | 8080 | [http://172.17.0.3:32061](http://172.17.0.3:32061) |🎉 Opening service default/go-hello in default browser…
```

当然，你可以使用 curl、Postman 或其他工具来测试你的服务。

每当您更改应用程序并希望在 Kubernetes 中进行测试时，您可以:

*   使用新标签构建图像，例如“2.0”

```
$ docker build -t go-hello:2.0 .
```

*   使用以下命令编辑 Minikube 中的部署:

```
$ kubectl set image deployment/go-hello go-hello=go-hello:2.0
```

更新的图像将导致窗格被重新创建，并且更改将立即在浏览器中可见。

# 就是这样！

您已经成功地部署了一个 minikube Kubernetes 集群，构建了一个优化的 Docker 应用程序，在集群上运行——并且在您的本地环境中完成了所有这些工作。恭喜你！

让我在评论中知道是否有一些部分引起了一些问题。有许多配置和操作系统，我希望我的教程涵盖了大多数，但我肯定不是所有的:)
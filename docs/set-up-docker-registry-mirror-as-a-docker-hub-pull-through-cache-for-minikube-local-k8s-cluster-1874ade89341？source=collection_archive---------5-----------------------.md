# 将 Docker 注册表镜像设置为 Minikube 本地 K8s 集群的 Docker Hub 直通缓存

> 原文：<https://itnext.io/set-up-docker-registry-mirror-as-a-docker-hub-pull-through-cache-for-minikube-local-k8s-cluster-1874ade89341?source=collection_archive---------5----------------------->

![](img/21dac679a2c794c87d674115234bf04e.png)

照片由[丹尼·米勒](https://unsplash.com/@redaquamedia?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

# 为什么 Docker 注册表镜像？

自 2020 年 11 月 20 日起，[匿名和免费 Docker Hub 用户每六小时被限制为 100 和 200 个集装箱图像拉取请求](https://www.docker.com/increase-rate-limits)。如果您经常使用本地 minikube dev 集群构建 docker 映像或部署测试应用程序，最终您会遇到“请求过多”或“达到拉速率限制”的错误。

为了解决这个问题，我们可以在您的 minikube 集群中安装一个 docker 注册表镜像作为一个直通缓存。它从 [Docker Hub registry](https://hub.docker.com/) 中提取图像，并将其存储在本地，然后在您第一次请求图像时将其返回给您。在后续请求中，本地注册表镜像能够从自己的存储中提供映像。

# 安装 Docker 注册表镜像

为了使它更容易，我准备了一个定制的 Docker 注册表舵图[,预先配置注册表在“代理缓存”模式下工作。](https://github.com/t83714/docker-registry-mirror)

首先，添加舵图回购:

```
$ helm repo add docker-registry-mirror https://t83714.github.io/docker-registry-mirror
```

要安装图表，请使用以下方法:

```
$ helm upgrade --install docker-registry-mirror docker-registry-mirror/docker-registry-mirror
```

默认情况下，Docker 注册表镜像将以匿名用户的身份从 [Docker Hub](https://hub.docker.com/) 获取图像。如果您希望它使用现有帐户从 Docker Hub 获取图像，您可以在安装过程中提供用户名&密码:

```
$ helm upgrade --install --set proxy.username=xxxx,proxy.password=xxxx docker-registry-mirror docker-registry-mirror/docker-registry-mirror
```

# 配置 Minikube

1.  找到分配给注册表镜像的“节点端口”

```
kubectl get svc --all-namespaces --selector=app=docker-registry-mirror -oyaml | grep nodePort
```

此命令将列出分配给已安装的注册表镜像服务的“节点端口”。

要验证“节点端口”和注册表镜像安装，请执行以下操作:

```
# Log into Minikube VM via SSH
minikube ssh
curl http://localhost:xxxxx/v2/_catalog
```

这里，“xxxxx”是我们刚刚通过“kubectl”命令找到的“节点端口”号。

我们应该会看到类似于以下内容的响应:

```
{"repositories":[]}
```

2.配置 Minikube Docker 守护程序

用文本编辑器打开 Minikube 配置文件:`~/.minikube/machines/minikube/config.json`。

在键`HostOptions.EngineOptions`下，将`RegistryMirror`键替换(如果不存在，则添加)为:

```
"RegistryMirror": [
    "http://localhost:xxxxx"
]
```

这里，`xxxxx`是我们在步骤 1 中通过“kubectl”命令找到的节点端口号。

3.重启 Minikube

```
# Apply the config and restart minikube
minikube stop
minikube start
```

重新启动后，您应该看到 docker 请求出现在 docker 注册表镜像窗格日志中。
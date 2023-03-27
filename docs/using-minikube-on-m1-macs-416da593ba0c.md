# 在 M1 MAC 电脑上使用 Minikube

> 原文：<https://itnext.io/using-minikube-on-m1-macs-416da593ba0c?source=collection_archive---------0----------------------->

在基于 x86 的 CPU 上，运行 Linux 容器和 Kubernetes 是很正常的。对于新的基于 M1 CPU 的 MAC 电脑，仍然存在挑战。

![](img/3cc5f99edf6907d8da93983bd78facec.png)

[图片基于特雷西·伦德格伦在 Pixabay 上的原创作品](https://pixabay.com/de/photos/äpfel-früchte-rot-reif-vitamine-634572/)

在 [Minikube 主页](https://minikube.sigs.k8s.io/docs/start/)上有说明，其中甚至提到了 M1 MAC，但对我不起作用。我是这样开始的:

首先，我通过 brew 安装了 podman 和 minikube(这包括 kubectl 命令):

```
% brew install podman
% brew install minikube
```

下一步是初始化 podman，并明确地为它分配 2 个 CPU(如果有足够的内核，可以更高)，然后启动它。这在第一次设置时可能需要一段时间，因为 podman 需要下载一个 Linux 映像来运行。

```
% podman machine init --cpus 2 --memory 2048 --rootful
% podman machine start
```

这同样适用于内存。我的 Mac Mini 只有 8GB，所以我用了 2 GB，这也是默认设置。我不确定 rootful 选项是否真的需要，但是它在我的例子中是有效的。

成功后，我们可以安装 Minikube。

```
% minikube start --driver=podman
```

需要传递驱动程序选项，否则 minikube 将找不到 podman，安装将失败。下面显示了一个成功的开始(我的系统被设置为德国语言环境)

> % minikube start—driver = pod man
> 😄达尔文 12.3.1 (arm64)
> ✨👍启动集群迷你库
> 中的控制平面节点迷你库🚜Ziehe das 基本映像…
> e 0510 11:17:02.706500 8561 cache . go:203]下载 kic 工件时出错:尚未实现，请参见问题#8426
> 🔥Erstelle podman 容器(CPUs = 2，Speicher=1957MB) …
> 🐳vorbereiten von Kubernetes v 1 . 23 . 3 auf Docker 20 . 10 . 12…e 0510 11:19:08.175006 8561 start . go:126]无法获取主机 IP: RoutableHostIPFromInside 目前仅针对 linux 实现
> 
> kube let . house keeping-interval = 5m
> Generiere zertifickate und schlüssel…
> 启动控制平面…
> Konfiguriere RBAC·雷格尔恩…
> 🔎gcr.io/k8s-minikube/storage-provisioner:v5 图片
> 🌟Addons aktiviert:存储供应器，默认存储类
> 🏄弗提格。库对象是标准的(缺省的)集合“库和名称空间”缺省的“概念”

仅供参考，在撰写本文时，我的系统运行的是 macOS 12.3.1。

## 重新开始

当出于任何原因需要删除集群并重新启动时，请确保同时删除 minikube 配置。

```
% minikube stop
% minikube delete
```

如果没有这个，在~/中会有一个剩余的配置。妨碍下一次运行成功开始的 minikube/
# 使用 Flux 和 Gitlab 开始使用 GitOps

> 原文：<https://itnext.io/getting-started-with-gitops-with-flux-and-gitlab-9d8ac2cec0cc?source=collection_archive---------4----------------------->

## 了解如何使用 GitOps 方法来简化您的 Kubernetes 部署。

![](img/e5fe54f5e73792cb132147a7609de104.png)

GitOps 是一种 Kubernetes 应用交付方法。它旨在简化 Kubernetes 应用程序的部署和操作。

在本文中，我们将使用 [Flux](https://fluxcd.io/) ，它可以作为 Kubernetes 操作器安装。

Flux 操作符保持集群状态和存储库同步。存储库中的任何配置更改都会自动应用到 [Kubernetes 集群](https://codersociety.com/blog/articles/terraform-azure-kubernetes)中。

在本指南中，我们将通过 Git 库设置 Flux 并部署一个演示应用程序。

## 先决条件

首先，我们需要`kubectl`连接到一个 Kubernetes 集群和 Gitlab 中的一个 Git 存储库。

## 安装 fluxctl

`fluxctl`是与 Flux 交互的命令行工具。在 macOS 上，你可以用自制软件安装`fluxctl`。

```
brew install fluxctl
```

您可以在这里找到其他平台的安装说明:[https://docs.fluxcd.io/en/1.17.1/references/fluxctl.html](https://docs.fluxcd.io/en/1.17.1/references/fluxctl.html)

## 为 Flux 创建一个 Kubernetes 名称空间

让我们首先为 Flux 操作符创建一个名称空间。

```
kubectl create namespace flux
```

## 生成通量清单文件

我们使用`fluxctl`工具为 Flux 操作符生成 Kubernetes 清单文件。您需要指定存储库 url，Git 用户信息。您还需要提供保存 Kubernetes 清单的文件夹的路径，以及应该部署操作员的名称空间。

```
fluxctl install 
--git-url=git@gitlab.com:$tom-code/example-gitops-flux 
--git-user=flux 
--git-email=flux@tomcode.com 
--git-path=namespaces,workloads 
--namespace=flux > flux.yaml
```

## 应用清单文件

让我们使用`kubectl`来应用通量算子。

```
kubectl apply -f flux.yaml
```

## 验证部署

通过检查 flux 名称空间中的状态 Kubernetes pods 来验证部署。应该有 Flux 操作符和一个 Memchached 实例，操作符用它来保存一些内部状态。

```
kubectl get pods -n fluxNAME                         READY   STATUS    RESTARTS   AGE
flux-5dd6d54f5b-2gbws        1/1     Running   0          67s
memcached-5fd8f56fc5-qlpsc   1/1     Running   0          67s
```

## 通过 fluxctl 检索 SSH 密钥

Flux 使用 SSH 密钥连接到 Git 存储库。它在初始启动时生成密钥。您可以通过`fluxctl` cli 检索公共 SSH 密钥。

```
fluxctl identity --k8s-fwd-ns flux
```

## 将 SSH 密钥添加到 Gitlab 存储库的部署密钥中

打开 Gitlab，导航到您的项目，转到`Settings -> Repository -> Deploy Keys`并添加从 fluxctl 检索的 SSH 密钥。确保您选中了允许写访问。

## 克隆存储库

使用 Git 将存储库克隆到您的本地机器上，并将`cd`克隆到目录中。

```
git clone https://gitlab.com/tom-code/example-gitops-flux.git
cd example-flux-gitops
```

## 为清单文件创建文件夹

让我们创建两个文件夹来保存我们的 Kubernetes 清单文件。

```
mkdir namespaces workloads
```

## 为演示命名空间创建 Kubernetes 清单文件

我们首先添加一个 Kubernetes 名称空间资源。复制下面的代码片段，并将其作为`demo-ns.yaml`保存在名称空间目录中。

```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    name: demo
  name: demo
```

## 为 nginx 部署创建 Kubernetes 清单文件

接下来，我们将添加一个演示 nginx 部署。复制下面的代码片段，并将其作为`nginx-deploy.yaml`保存在 workloads 目录中。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: demo
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

## 提交和推送更改

使用 Git 将新文件添加到存储库中。之后，提交更改并将其推送到 Gitlab 存储库。

```
git add namespaces workloads
git commit -m 'Add demo resources'
git push
```

## 验证部署

通过检查演示名称空间中的状态 Kubernetes pods 来验证部署。应该有两个 nginx pods 在运行。

```
kubectl get pods -n demoNAME                                                   READY   STATUS    RESTARTS   AGE
nginx-deployment-6b6bc59c57-fd2w5              1/1     Running   0          57s
nginx-deployment-6b6bc59c57-vh8bh              1/1     Running   0          52s
```

## 摘要

我们部署了 Flux，并将其配置为与 Gitlab 库一起工作。对 Git 存储库的更改会自动部署到集群中。

正如您所看到的，没有外部客户端需要访问 Kubernetes 集群，这使得该过程非常安全，非常适合在高度监管的环境中部署，如保险和金融科技行业。

有一个中央存储库来管理 Kubernetes 清单文件使得部署更容易操作，并且消除了 Kubernetes 从存储库和服务管道带来的复杂性。

*最初发表于*[T5【https://tomcode.com】](https://tomcode.com/blog/getting-started-with-gitops-with-flux-and-gitlab)*。*
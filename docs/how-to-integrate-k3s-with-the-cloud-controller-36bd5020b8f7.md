# 如何将 k3s 与云控制器集成

> 原文：<https://itnext.io/how-to-integrate-k3s-with-the-cloud-controller-36bd5020b8f7?source=collection_archive---------5----------------------->

![](img/aec76ef96f77a4c561b68fce96c09537.png)

照片由[达拉斯里德](https://unsplash.com/@dallasreedy?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

k3s 太棒了！我已经写了一篇关于如何在 60 秒内创建一个 4 节点 k3s 集群的文章(包括虚拟机配置——这里是)。唯一的问题是它没有云提供商的支持，这意味着你不能像在托管的 Kubernetes 上一样容易地使用负载平衡器、存储等。但是有一种方法可以做到这一点，这就是这篇文章的内容。所以让我们开始吧

将 CCM(云控制器管理器)与 k3s 集成意味着 k3s 集群将能够与云提供商 API 对话，以请求和配置负载平衡器(用于入口)、为节点应用适当的标签等。云提供商之间的流程有所不同(这取决于云提供商是否首先提供 CCM)。在这篇文章中，我们将在 DigitalOcean 上安装 k3s(如果你想看不同云的文章，请在评论中告诉我)。

整个过程并不复杂，但是你需要仔细地按照步骤来。首先，主节点。安装 k3s 主节点时，我们需要传递以下参数:

*   —禁用云控制器
*   —不部署服务 lb
*   — kubelet-arg= "云提供商=外部"
*   —kube let-arg = " provider-id = digital ocean://{ master _ node _ id } "

那么，它们是什么意思呢？首先，顾名思义禁用默认的 k3s 云控制器。我们需要这样做，否则 k3s 将使用自己内置的“虚拟”云控制器。第二—我们要求 k3s 不要部署 servicelb，因为它会弄乱 IP 地址(servicelb 会用节点 IP 覆盖入口 IP—我们希望将 DigitalOcean LoadBalancer IPs 用于服务类型 LoadBalancer)。第三，这是几乎所有 CCM 的要求。我们只需要告诉 kubelet 我们将使用外部云提供商。最后一个——这是 DigitalOcean CCM 的特定要求——如果您检查 DO CCM GitHub [repo](https://github.com/digitalocean/digitalocean-cloud-controller-manager) ,您会看到他们只是要求您将此参数传递给 kubelet。[master_node_id]是 droplet id，您可以在 DO Dashboard 中找到它，也可以从 droplet 本身调用 GET 来找到它:

```
curl [http://169.254.169.254/metadata/v1/id](http://169.254.169.254/metadata/v1/id)
```

因此，为了安装为 CCM 准备的 k3s 服务器，我们需要这样做:

```
curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -s — server \
  --disable-cloud-controller \
  --no-deploy servicelb \
  --kubelet-arg=”cloud-provider=external” \
  --kubelet-arg=”provider-id=digitalocean://$master_id”
```

这是第一部分——准备。现在 k3s 服务器启动后，我们需要安装 CCM 本身。为此，首先克隆 git 存储库:

[](https://github.com/digitalocean/digitalocean-cloud-controller-manager) [## 数字海洋/数字海洋-云-控制器-管理器

### digitalocean-cloud-controller-manager 是用于 digital ocean 的 Kubernetes 云控制器管理器实现。阅读…

github.com](https://github.com/digitalocean/digitalocean-cloud-controller-manager) 

接下来，我们需要创建包含 DigitalOcean API 令牌的 k8s secret。要做到这一点，您可以使用 repo 中的秘密生成器(只需遵循说明)或使用下面的一行程序:

```
kubectl -n kube-system create secret generic digitalocean --from-literal=access-token=[YOUR_DO_API_TOKEN]
```

如果保存了机密，那么只需应用存储库中的 yaml 清单:

```
kubectl apply -f releases/v0.1.21.yml
```

就是这样！现在，每当您启动类型为:LoadBalancer 的服务时，DigitalOcean LoadBalancer 将被创建并配置为将流量路由到它。此外——因为默认情况下 k3s 带有 Traefik——也会自动为它创建 DO LB。

还有一件事。到目前为止，我们只创建了 k3s 主节点，对于任何工作节点，您只需使用以下参数安装 k3s:

```
curl -sfL https://get.k3s.io | K3S_TOKEN=${token} sh -s - agent \
  --server https://${master_node_ip}:6443 \
  --kubelet-arg="cloud-provider=external" \
  --kubelet-arg="provider-id=digitalocean://$worker_id"
```

仅此而已。所有节点都将具有适当的标签集(公共/私有 ip、DO 区域等),并且通过 DO LB 的路由也将无缝地发生。

最后但同样重要的是，我创建了一个简单的 bash 脚本来自动化这个过程。该解决方案允许您在大约 2 分钟内使用 DO CCM 在 DigitalOcean 上创建 k3s 集群(1 个主节点+ x 个工作节点):

[](https://github.com/DavidZisky/kloud3s) [## GitHub - DavidZisky/kloud3s:部署虚拟机+ k3s 集群+云控制器

### 60 秒内 Kubernetes 部署者。简单的 bash 脚本，在 GCP/数字海洋中启动虚拟机，在其上安装 k3s，然后…

github.com](https://github.com/DavidZisky/kloud3s)
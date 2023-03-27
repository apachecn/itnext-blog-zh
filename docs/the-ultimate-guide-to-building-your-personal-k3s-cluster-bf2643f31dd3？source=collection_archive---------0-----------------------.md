# 构建您个人 K3S 集群的终极指南

> 原文：<https://itnext.io/the-ultimate-guide-to-building-your-personal-k3s-cluster-bf2643f31dd3?source=collection_archive---------0----------------------->

![](img/00315f0f668cbec7f6b1901879202e51.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上[科学高清](https://unsplash.com/@scienceinhd?utm_source=medium&utm_medium=referral)拍摄的照片

有很多原因可以让你拥有自己的 Kubernetes 集群。就我个人而言，我喜欢构建自己的工具，并更好地了解幕后发生的事情，但你可能出于不同的原因想要尝试 Kubernetes，但由于使用 AWS、GCP、Azure、DigitalOcean 等提供的 Kubernetes 服务的高昂成本而受到阻碍。

[K3s](https://k3s.io/) 是由 [Rancher Labs](https://rancher.com/) 开发的一个轻量级 Kubernetes 发行版。基本上，这是一个完整的 Kubernetes 发行版，但他们将所有进程合并到一个二进制文件中，为 ARM 添加了交叉编译，删除了许多您通常不使用的额外功能，并捆绑了一些额外的用户空间工具。

在本教程中，我将向您展示如何设置一个完整的 k3s 集群，它与上游 Kubernetes 完全兼容，并增加了在虚拟机上简单部署的功能。阅读完本教程后，您可以将任何 Kubernetes 应用程序部署到您的集群上，并利用通常只能在托管 Kubernetes 服务中找到的自动存储供应和软件负载平衡器。

# 设置您的虚拟机(VM)

首先，您需要设置您的虚拟机。如果你已经有一个虚拟机，确保你有任何重要的数据备份，以防万一。如果你想创建一个新的虚拟机，有很多选择。我建议您看一下不同的云选项并比较它们的价格，但是一般来说，您会希望至少有 4GB 的 RAM 才能在上面运行 Kubernetes 集群(但是，1GB 应该适用于无状态部署，不需要存储驱动程序，存储驱动程序的最低要求是 2GB RAM)。

如果你现在不想看太多，这里有一个 100 美元的链接，可以试用 vultr ，他们的好处是他们在世界各地都有数据中心，性价比很高。如果你住在欧洲，或者到欧洲的延迟对你来说还可以，你可以试试 [Hetzner](https://hetzner.cloud/?ref=sxI1XJxyltWU) ，它们真的很便宜(我用它们的时间不长，所以我不能谈论它们的性能或可用性)。

在您选择了您的云提供商和您要使用的虚拟机规模之后，我们开始创建服务器。这通常很简单，创建一个新的实例，选择离你最近的数据中心，选择虚拟机大小(我建议使用 4GB 的内存，但 2GB 也可以尝试)，选择“Ubuntu 18.04”或“Ubuntu 20.04”作为操作系统，添加 SSH 密钥，最后，单击 deploy。

虚拟机启动后，使用 ssh 登录并更改其主机名:

```
sudo hostnamectl set-hostname k3smaster
sudo reboot
```

请注意，如果您计划创建一个包含多个虚拟机的集群，每个虚拟机都应该有一个唯一的主机名。确保您更改了每个虚拟机的主机名。

确保如果你正在为你的虚拟机使用防火墙，你打开你将要使用的端口，TCP 端口 **6443** 和 **10250** ，UDP 端口 **8472** 用于来自虚拟机外部的 k3s 交互。

# 在客户端安装工具

如果您计划使用 Kubernetes 集群，您需要一些额外的工具，比如 kubectl，来帮助您与集群进行通信。这些工具既可以安装在主节点上，也可以安装在您的笔记本电脑上，看哪个更方便。

为了安装很多这样的工具，我将使用 [Arkade](https://github.com/alexellis/arkade) ，这是一个由 [Alex Ellis](https://github.com/alexellis) (他也是 OpenFaaS 的创造者)创建的很棒的工具，可以帮助安装很多工具。

首先，我们可以安装 Docker 和 Docker Compose 来帮助我们进行开发:

```
# Docker
curl -sSL  [https://nimamahmoudi.github.io/cicd-cheatsheet/sh/install-docker.sh](https://nimamahmoudi.github.io/cicd-cheatsheet/sh/install-docker.sh) | bash
# Docker Compose
sudo apt-get update && sudo apt install -qy python3-pip && pip3 install docker-compose
```

然后，我们可以安装 Arkade 及其 bash-completion:

```
# Arkade
curl -SLfs [https://dl.get-arkade.dev](https://dl.get-arkade.dev) | sudo sh
# Add tools bin directory to PATH
echo "export PATH=\$HOME/.arkade/bin:\$PATH" >> ~/.bashrc
# Copy bash completion script
arkade completion bash > ~/arkade_bash_completion.sh
echo "source ~/arkade_bash_completion.sh" >> ~/.bashrc
# starting bash completion
source ~/.bashrc
```

接下来，我们需要 kubectl 与 kubelet 进行通信:

```
# Kubectl
arkade get kubectl
# Kubectl bash completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
# starting bash completion
source ~/.bashrc
```

接下来，kustomize，helm，k3sup，和 kompose:

```
# Kustomize
arkade get kustomize
# Helm
arkade get helm
# K3sup
arkade get k3sup
# Kompose for converting docker-compose
curl -L [https://github.com/kubernetes/kompose/releases/download/v1.22.0/kompose-linux-amd64](https://github.com/kubernetes/kompose/releases/download/v1.22.0/kompose-linux-amd64) -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
echo 'source <(kompose completion bash)' >>~/.bashrc
source ~/.bashrc
```

如果你不知道这些工具中的任何一个，可以去看看，但是我会试着给每个工具一个总结。Kustomize 可以帮助我们动态地改变 kubernetes 清单中的值。Helm 用于在大型项目中进行更专业的定制，或者使用 helm 图表安装外部应用程序。K3sup 是 Alex Ellis 开发的另一个工具，我们将用它在虚拟机上安装 k3s。

# K3s 安装

默认情况下，K3s 带有 CoreDNS、Metrics server 和 Traefik。查看[此视频](https://youtu.be/FmLna7tHDRc)了解更多关于 k3s 的信息。在本教程中，我们将禁用 Traefik，而使用 Nginx Ingress。我们还将启用集群模式(它使用 dqlite 而不是 sqlite 来允许多个主节点)。

要从远程计算机(例如，您的笔记本电脑)安装 k3s，请使用以下命令:

```
k3sup install \
  --ip YOUR_VM_IP \
  --cluster \
  --user ubuntu \
  --k3s-channel stable \
  --local-path ~/.kube/config \
  --merge --context k3s \
  --k3s-extra-args '--no-deploy traefik --write-kubeconfig-mode 644'
```

请确保将您的 _VM_IP 替换为您在云控制台中看到的虚拟机的 IP。更多选项，请查看 k3sup 文档。关于高可用性安装的更多信息，请阅读[本教程](https://github.com/alexellis/k3sup#create-a-multi-master-ha-setup-with-embedded-etcd)。

使用 k3sup，我们在虚拟机上引导 k3s，等待它完成安装，获取 kube 配置文件，将其移动到默认位置，并将我们的集群(上下文)命名为 k3s。我们还可以在虚拟机上移动 kube 配置文件，这样我们也可以在虚拟机上使用 kubectl:

```
mkdir ~/.kube
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
echo "export KUBE_CONFIG=~/.kube/config" >> .bashrc
```

如果您想使用虚拟机本身来安装 k3s，而不是您的笔记本电脑，请使用以下命令:

```
k3sup install \
  --cluster \
  --local
  --k3s-channel stable \
  --local-path ~/.kube/config \
  --merge --context k3s \
  --k3s-extra-args '--no-deploy traefik --write-kubeconfig-mode 644'
```

K3s 自带 kubectl 烘焙的，可以在 VM 上用“k3s kubectl”，也可以设置别名(把这行加到~/。巴沙尔将其永久化):

```
alias kubectl="k3s kubectl"
```

我们现在可以检查我们的安装了:

```
kubectl get nodes -o wide
```

您应该会看到类似这样的内容:

```
NAME        STATUS   ROLES    AGE   VERSION
k3smaster   Ready    master   3m    v1.18.9+k3s1
```

现在我们已经在一个虚拟机上运行了 k3s，我们可能希望添加其他虚拟机:

```
k3sup join \
  --server-ip YOUR_FIRST_VM_IP \
  --ip CURRENT_VM_IP \
  --user ubuntu \
  --k3s-channel stable \
  --k3s-extra-args '--no-deploy traefik --write-kubeconfig-mode 644'
```

关于 k3sup 选项的更多信息，请查看他们的 [Github repo](https://github.com/alexellis/k3sup) 。从所有虚拟机加入 k3s 群集后，检查是否一切正常:

```
kubectl get nodes -o wide
```

干得好，您现在已经安装了 k3s，并准备好部署不需要持久存储的无状态应用程序。接下来，我们将添加几个其他工具，当我们想要在集群上进行重要部署时，这些工具将会派上用场。

![](img/920f5d0684c03d7df25b2261920f404d.png)

照片由 [Alasdair Elmes](https://unsplash.com/@alelmes?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 安装 Nginx 入口控制器

现在我们已经有了一个正在运行的 k3s 集群，我们希望开始在其上部署令人惊叹的应用程序，其中几个可能在相同的 web 端口 80 (HTTP)和 443 (HTTPS)上。在这些情况下，以及出于其他各种原因，您可能需要安装入口控制器。目前使用最多的入口控制器是 Nginx。在他们的网站上了解更多信息。

由于我们已经在开始时安装了适当的工具，安装 nginx 和 cert-manager for let ' s encrypt HTTPS 证书相当容易:

```
# install nginx ingress
arkade install ingress-nginx --namespace default
# cert-manager for letsencrypt certificates
arkade install cert-manager
```

现在，我们已经准备好部署任何使用入口控制器的应用程序。我将很快为一些最有用的写教程，你可以关注我以便在发生时得到通知。

# 安装 Longhorn 存储控制器

[Longhorn](https://github.com/longhorn/longhorn) 是 Rancher Labs 的另一个出色的开源项目，它为您的 Kubernetes 集群提供分布式块存储。它还带有一个预安装的仪表板，可以帮助您进行设置、备份和监控。我们可以使用 helm 轻松安装 longhorn:

```
# add the longhorn helm repository
helm repo add longhorn [https://charts.longhorn.io](https://charts.longhorn.io)
# update helm repos
helm repo update
# create the namespace
kubectl create namespace longhorn-system
# install longhorn
helm install longhorn longhorn/longhorn --namespace longhorn-system
```

检查是否一切顺利:

```
# check everything went smooth
kubectl -n longhorn-system get pod
```

您应该会看到类似这样的内容:

```
NAME                                        READY   STATUS 
longhorn-ui-6fb889895f-klgs7                1/1     Running
instance-manager-e-20feb18d                 1/1     Running
instance-manager-r-2d587ad2                 1/1     Running
engine-image-ei-ee18f965-vrjk8              1/1     Running
longhorn-manager-qg7fz                      1/1     Running
longhorn-driver-deployer-6756bb8fd6-wq757   1/1     Running
csi-provisioner-57d6dbf5f4-jn9rr            1/1     Running
csi-provisioner-57d6dbf5f4-5d67p            1/1     Running
longhorn-csi-plugin-z4z5c                   2/2     Running
csi-attacher-5b4745c5f7-z9zqj               1/1     Running
csi-attacher-5b4745c5f7-hdvxj               1/1     Running
csi-provisioner-57d6dbf5f4-zt8jc            1/1     Running
csi-resizer-75ff56bc48-clx4p                1/1     Running
csi-resizer-75ff56bc48-qcq22                1/1     Running
csi-attacher-5b4745c5f7-q276d               1/1     Running
csi-resizer-75ff56bc48-spmt2                1/1     Running
```

现在，您应该能够访问 longhorn UI 来访问它的配置:

```
# check out the longhorn service
kubectl -n longhorn-system get svc
# forward the port to check the configuration
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8002:80
```

现在，您应该能够从“http://localhost:8002”打开 longhorn 仪表板。添加群集中任何虚拟机上的任何可用磁盘。如果您的 k3s 集群中只有一个节点可用，您将需要启用`Replica Node Level Soft Anti-Afinity`,以便允许在同一节点上放置数据块备份(默认情况下，longhorn 在不同的虚拟机上保留每个数据块的 3 个副本，这样即使虚拟机上的任何数据丢失，您也不会丢失任何数据)。

# 结论

您只需设置一个非常“专业”的 Kubernetes 集群，就可以用于学习、自动化您的个人任务，或者在云上托管您的爱好项目，而不必使用托管服务(这最终会比您自己想要的花费多得多)。如果这对于你想做的事情来说太复杂了，你也可以看看无服务器计算，但它的目标是非常不同的，但它可能是许多人的选择。

在以后的文章中，我将写一些最常用的自托管应用程序的安装过程。你可以关注我，当那些文章出来时，我会通知你。

# 关于我

我是阿尔伯塔大学的博士生，约克大学的客座研究员，塞内卡学院的兼职讲师。我日复一日地研究无服务器计算平台，试图找到提高其性能、可靠性、能耗等的方法。，使用分析或数据驱动的方法(对“我要么使用数学要么使用机器学习来建模无服务器计算平台”的花哨说法)。我在这里写的是我在研究期间对一些无服务器计算平台的深入了解，以及它们的自动伸缩模式文档的简要汇编。

如果你想更多地了解我，可以去我的[网站](https://nima-dev.com/)看看。
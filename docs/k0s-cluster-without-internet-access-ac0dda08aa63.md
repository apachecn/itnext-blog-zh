# 无互联网接入的 K0s 集群

> 原文：<https://itnext.io/k0s-cluster-without-internet-access-ac0dda08aa63?source=collection_archive---------4----------------------->

## 让我们看看 k0s 如何让气隙安装变得简单

![](img/b8028416403fa5532add38087a66cb3f.png)

照片由[卡比尔·拉赫曼·里亚德](https://unsplash.com/@riiyad?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/network?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

🔥根据我在 LinkedIn 上的一些评论，我必须强调，这篇文章只介绍了一种非常简单的气隙安装方法。它并不一定是建立弹性/安全/可观察/可升级生产集群的完整指南。

在具有高度安全性约束的公司中，可能需要在没有任何互联网接入的机器上安装 Kubernetes 集群。这意味着必须预先从另一台机器上下载设置集群所需的所有东西，然后以安全的方式(例如，使用 USB 密钥，从而通过物理方式访问机器)复制到气隙机器上。

在这篇短文中，我们将了解 [k0s](https://k0sproject.io) 如何管理气隙安装。k0s 0.12(火星 2021)增加了这个功能。我们将用使用[多遍](https://multipass.run)创建的 2 个虚拟机来说明整个过程:

*   第一个叫做*工具*，可以上网。它将用于获取 k0s 二进制文件和设置集群所需的所有映像
*   第二个名为 *airgap* 的是一个模拟的 airgap 机器(我们假设这个没有互联网接入)

## 获取图像

在这一步中，我们将从一台可以访问互联网的机器上下载 k0s 二进制文件和所有需要的图像。

首先，我们创建一个新的虚拟机(启动并运行不到一分钟),并在其中装入一个 shell:

```
$ multipass launch -n tools$ multipass shell tools
```

接下来，我们用下面的命令下载 k0s 二进制文件，这个命令也将 k0s 安装到 */usr/local/bin:*

```
**ubuntu@tools:~$ curl -sSLf https://get.k0s.sh | sudo sh**
...
k0s is now executable in /usr/local/bin
```

这里不需要运行 k0s，因为我们只使用这个二进制文件来获取运行集群所需的映像列表。该列表可使用如下所示的`airgap`子命令获得:

```
**ubuntu@tools:~$ k0s airgap list-images**
docker.io/calico/cni:v3.16.2
docker.io/calico/kube-controllers:v3.16.2
docker.io/calico/node:v3.16.2
us.gcr.io/k8s-artifacts-prod/kas-network-proxy/proxy-agent:v0.0.13
docker.io/coredns/coredns:1.7.0
k8s.gcr.io/kube-proxy:v1.20.4
gcr.io/k8s-staging-metrics-server/metrics-server:v0.3.7
k8s.gcr.io/pause:3.2
docker.io/cloudnativelabs/kube-router:v1.1.1
quay.io/k0sproject/cni-node:0.1.0
```

我们不会深入每个映像的细节，它们基本上被 k0s 用来运行集群的控制平面(加上一些额外的项目，如 metrics-server，一个用来获取集群指标的流行工具)。

0.13 `kube-router`的ℹ️是 k0s 默认使用的 CNI 插件

在同一台机器上，我们安装了 Docker，以便更容易下载图像。在 Linux 主机上安装 Docker，下面的命令非常方便:

```
**ubuntu@tools:~$** curl -sSLf https://get.docker.com | sudo sh
```

我们将默认的`ubuntu`用户添加到`docker`组中:

```
**ubuntu@tools:~$** sudo usermod -aG docker ubuntu
```

退出 shell 并启动新的 shell 后，我们可以从`ubuntu`用户那里获取所有图像:

```
**ubuntu@tools:~$** for image in $(k0s airgap list-images); do
  docker image pull $image
done
```

接下来，我们创建一个包含所有这些图像的归档。这可以使用`docker image save`子命令完成，其用法详述如下:

```
**ubuntu@tools:~$ docker image save --help**Usage:  docker image save [OPTIONS] IMAGE [IMAGE...]Save one or more images to a tar archive (streamed to STDOUT by default)Options:
  -o, --output string   Write to a file, instead of STDOUT
```

以下命令创建了`images.tar`归档文件:

```
**ubuntu@tools:~$** docker image save $(k0s airgap list-images) -o images.tar
```

归档包含所有图像，因此非常沉重:

```
**ubuntu@k0s:~$ ls -lrt** total 759408
-rw------- 1 ubuntu ubuntu 777626624 Apr 26 15:29 images.tar
```

在下一步中，我们将把 k0s 二进制文件和归档文件复制到 Air-Gap 机器，并运行 k0s 集群。

## 将图像复制到气隙机

![](img/0dca098bdaf72b5e71108aef5e8e8da7.png)

将数据复制到气隙机并不总是容易的

根据气隙电机的安全级别，我们可能需要对电机进行物理访问，并从 USB 闪存盘复制数据。当然，有时来自另一台机器的 ssh 连接也是可能的。

让我们用一个新的多通道虚拟机来模拟一个气隙电机:

```
$ multipass launch -n airgap
```

接下来，我们将使用本地文件夹和 Multipass' transfer 命令将资产从`tools`复制到`airgap`:

```
# Create a local shared folder
mkdir -p /tmp/shared/k0s# Copy assets from tools to the shared folder
multipass transfer tools:/usr/local/bin/k0s /tmp/shared/k0s
multipass transfer tools:/home/ubuntu/images.tar /tmp/shared/k0s# Copy assets from the shared folder to airgap
multipass transfer /tmp/shared/k0s/k0s airgap:/tmp
multipass transfer /tmp/shared/k0/images.tar airgap:/tmp
```

一旦可以从气隙机访问资产，我们需要将它们放入正确的位置:

*   k0s 二进制需要移到 */usr/local/bin/*

```
$ multipass exec airgap -- /bin/bash -c "
  sudo mv /tmp/k0s /usr/local/bin/
  sudo chmod +x /usr/local/bin/k0s
"
```

*   图像档案需要移动到新的 */var/lib/k0s/images* 文件夹中

```
$ multipass exec airgap -- /bin/bash -c "
  sudo mkdir -p /var/lib/k0s/images
  sudo mv /tmp/images.tar /var/lib/k0s/images/
"
```

所有的资产在气隙机上都不可用。在下一步中，我们将使用这些组件运行一个集群。

## 在气隙电机上运行 k0s

首先我们在`airgap`上得到一个 shell

```
$ multipass shell airgap
```

接下来，我们定义一个配置文件，将`default_pull_policy`设置为`Never`，以确保如果图像不存在，就不会从互联网上下载。

```
**ubuntu@airgap:~$** cat <<EOF > /tmp/k0s.yaml
apiVersion: k0s.k0sproject.io/v1beta1
kind: Cluster 
metadata:   
  name: k0s 
spec:   
  images:     
 **default_pull_policy: Never** EOF
```

然后，我们照常安装 k0s(在本例中，我们将选择单个节点),并提供配置文件:

```
**ubuntu@airgap:~$ sudo k0s install controller --single -c /tmp/k0s.yaml**
INFO[2021-04-26 20:54:49] no config file given, using defaults
INFO[2021-04-26 20:54:49] creating user: etcd
INFO[2021-04-26 20:54:49] creating user: kube-apiserver
INFO[2021-04-26 20:54:49] creating user: konnectivity-server
INFO[2021-04-26 20:54:49] creating user: kube-scheduler
INFO[2021-04-26 20:54:49] Installing k0s service
```

然后，我们启动 systemd 进程:

```
$ sudo systemctl start k0scontroller
```

最后，我们可以检查群集是否正常运行

```
export KUBECONFIG=/var/lib/k0s/pki/admin.conf**$ sudo k0s kubectl get no** NAME     STATUS   ROLES    AGE     VERSION
airgap   Ready    <none>   2m28s   v1.20.5-k0s1
```

## 关键外卖

出于安全原因，气隙安装有时是必要。正如我们在本文中看到的，k0s 的最新版本通过新的`airgap`子命令使整个过程变得非常简单。
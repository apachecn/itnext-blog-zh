# Kubernetes 生产集群，带有 MacOS 上的 vagger 和 Virtualbox

> 原文：<https://itnext.io/kubernetes-production-cluster-with-vagrant-and-virtualbox-on-mac-8718586f179f?source=collection_archive---------0----------------------->

![](img/483e78a1951e14e1627ae889476166de.png)

约瑟夫·巴里恩托斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

出于开发的目的，在本地拥有一个 Kubernetes 生产集群是非常好的，在这篇文章中，我将介绍基本的设置，以便开始使用一个好的集群。

# **库比斯普雷前来救援**

很少有开源解决方案可以完成部署生产就绪的 Kubernetes 集群的任务，但是我选择了 Kubespray。

它附带了可翻译的行动手册(部署、升级、实用程序和附加程序)以及大量测试。

应该已经安装了什么:

*   无赖
*   Virtualbox
*   Python + Pip

让我们克隆一下官方回购:

```
git clone [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
```

之后，进入克隆的 repo，我们可以开始安装所需的工具:

```
cd kubespray && pip3 install -r requirements.txt# ORcd kubespray && pip install -r requirements.txt
```

一旦一切都被正确安装，我们可以开始流浪:

```
vagrant up
```

该命令将生成虚拟机，在其上安装 Kubernetes，并配置网络。在没有定制的情况下调用时，它会获取位于 */inventory/sample* 中的示例配置。内部 YAML 文件允许为计划的集群定制所需的任何内容。此外,*流浪者文件*包含了所产生的虚拟机的所有必要规范(因此它允许我们配置网络、虚拟机的映像、其属性等等)。

当这个过程完成时，集群被成功创建，我们可以使用自动注入到工件文件夹中的配置。

让我们进入*。移动*文件夹以便访问它:

```
cd .vagrant/provisioners/ansible/inventory/artifacts
```

有 3 个文件:

```
admin.conf     kubectl    kubectl.sh
```

我们需要的是“ *admin.conf* ”，它包含我们新创建的本地集群的配置(并设置 Kubectl 使用它)。

```
export KUBECONFIG=$(pwd)/admin.conf
```

因此，让我们检查到我们的集群的连接:

```
$ kubectl cluster-infoKubernetes master is running at [https://172.18.8.101:6443](https://172.18.8.101:6443)To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

太好了，我们现在连接到集群了。

默认情况下，Virtualbox 使用 NAT 网络类型来隔离所创建的虚拟机，因此我们需要使用端口转发来从外部访问所需的虚拟机。

```
$ vagrant statusCurrent machine states:k8s-1                     running (virtualbox) # usually master node
k8s-2                     running (virtualbox)
k8s-3                     running (virtualbox)
```

和列出 Virtualbox 实例

```
$ vboxmanage list vms"kubespray_k8s-1_1599909087609_21866" {30ba6c6e-6a1d-4647-8f67-9f3f8276d660}
"kubespray_k8s-2_1599909125366_77977" {f618bfd4-fe82-4437-8e57-2bce2bb9e2b1}
"kubespray_k8s-3_1599909169592_4503" {86275e53-025f-4021-85a8-2ff23a0099cb}
```

因此，我们想要做的是，公开 id 为"*30 ba 6 c6e-6a1d-4647–8f 67–9f3f 8276d 660*"的 k8s-1 机器。对于 NAT 网络，我们需要使用从主机到客户机的端口转发，这样做是可能的:

```
vboxmanage controlvm 30ba6c6e-6a1d-4647–8f67–9f3f8276d660 natpf1 "master-node,tcp,,6443,,6443"
```

此命令正在为 k8s-1 主节点创建端口转发规则。“主节点”是您想要的任意名称，TCP 是协议，在两个逗号之间我们不指定任何 IP(做两次)，并且对于主机/客户机我们分配相同的端口(在本例中是 6443)。

一旦启动，该命令将通过端口转发暴露我们的集群。

Kubespray 可以配置很多东西，比如网络管理解决方案、入口控制器、节点数量等...我推荐查看以下链接的官方页面[https://kubespray.io/](https://kubespray.io/)

希望有帮助！

干杯:)
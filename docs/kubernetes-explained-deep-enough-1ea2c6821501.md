# Kubernetes 解释得够深刻了

> 原文：<https://itnext.io/kubernetes-explained-deep-enough-1ea2c6821501?source=collection_archive---------3----------------------->

![](img/f9d2a37b6445b6fd075eb2c918fd0b14.png)

图片由[皮克斯拜](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1332816)的 Gerd Altmann 提供

## 第 1 部分:导言

第 1 部分:简介——这个博客

[第二部分:Kubernetes 解释得够深刻:存储](https://piotrzan.medium.com/kubernetes-explained-deep-enough-storage-eb16a66483c2)

[第三部分:Kubernetes 解释得足够深入:部署](https://piotrzan.medium.com/kubernetes-explained-deep-enough-deployments-371755fbe2a3)

[第四部分:Kubernetes 解释得够深刻:配置](https://piotrzan.medium.com/kubernetes-explained-deep-enough-configuration-cd4a9d1d8dcd)

[第 5 部分:服务可发现性、DNS、集群通信](https://piotrzan.medium.com/kubernetes-explained-deep-enough-services-1a0647499616)

关于本博客的网络版以及关于 docker、Kubernetes 和 Docker 认证的更多信息，请访问:

 [## Docker 认证助理准备指南

### 描述

dcaguide.net](https://dcaguide.net/#/) 

# 为什么是另一个 Kubernetes 系列？

随着 Kubernetes 越来越受欢迎，优秀的在线资源和学习材料也越来越多。许多可用的信息要么是为绝对的初学者设计的，要么是对某个特定主题的深入研究。

我的目标是以实用的方式撰写 Kubernetes 主题，如存储、部署、服务等，并为每个人提供练习场景。这个想法是专注于核心功能，充分理解它并跟着练习。如果你已经看了一些教程，可能已经创建了 pod 或 deployment，并准备好进入下一个级别，那么这个系列就是为你准备的。

它也可以用来帮助准备 Kubernetes ( [CKA](https://medium.com/faun/preparation-and-resources-for-cka-exam-ca868fc678c9) 、 [CKAD](https://piotrzan.medium.com/preparation-and-resources-for-ckad-exam-ea1b2e8888e3) 、CKS)或 Docker 认证( [DCA](/a-comprehensive-guide-to-docker-certified-associate-exam-d7f25b6374ca) )。

希望在完成本系列后，您将:

*   对 Kubernetes 的不同方面有更深入的了解
*   了解它们各自解决了什么问题
*   知道如何在实际场景中实现它

如果你是 Kubernetes 的新手，但仍然想了解这个系列，我强烈推荐你去看看

*   [Collabnix Github — Kubelabs](https://github.com/collabnix/kubelabs) 获取优秀的 Kubernetes 教程和练习。
*   [Kubernetes 初学者教程【4 小时完整课程】](https://www.youtube.com/watch?v=X48VuDVv0do&ab_channel=TechWorldwithNana)来自 TechWorld 与娜娜

# Kubernetes 沙盒

免费玩 Kubernetes 有很多不同的选择:

*   **本地**安装用 [Minikube](https://minikube.sigs.k8s.io/docs/) 或 [MicroK8s](https://microk8s.io/)
*   **本地**安装[流浪汉](https://www.vagrantup.com/)和 [VirtualBox](https://www.virtualbox.org/)
*   在 Windows、Mac 或 Linux 上使用 [Docker 桌面](https://www.docker.com/products/docker-desktop)进行本地安装
*   **远程**集群与[片甲](https://www.katacoda.com/)
*   **远程**集群，可在任何公共云提供商获得免费积分，其中 3 个最受欢迎:Azure [AKS*](https://docs.microsoft.com/en-us/azure/aks/) 、谷歌云 [GKE](https://cloud.google.com/kubernetes-engine/) 、AWS [EKS](https://aws.amazon.com/eks/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc&eks-blogs.sort-by=item.additionalFields.createdDate&eks-blogs.sort-order=desc)
*   **远程**集群或本地安装有 [LXC 容器](https://linuxcontainers.org/)
*   **远程**集群 [PWK —与 Kubernetes 一起玩*](https://labs.play-with-k8s.com/)

在大多数示例中，我们将使用 PWK，因为不需要在本地安装任何东西，而且我们最终得到的环境足够强大，可以通过所有示例。一些示例将需要云提供商集群，这些将在 AKS 上完成。

按照本指南[中的说明](https://github.com/collabnix/kubelabs/blob/master/kube101.md)设置 3 节点集群。1 个主节点和 2 个工作节点

*   `git clone [https://github.com/collabnix/kubelabs](https://github.com/collabnix/kubelabs)`
*   `cd kubelabs`
*   `sh bootstrap.sh`
*   通过执行引导过程结束时输出的命令，将节点添加到群集

> PWK 有时没有响应，因此您需要关闭会话，稍后再试
> 
> 指南要求设置 5 个节点，但对于我们的目的，3 个就足够了(1 个主节点，2 个工作节点)
> 
> 万一 PWK 停机或没有反应，我建议安装 Docker 桌面

# 设置集群可视化工具

一旦集群准备就绪，让我们设置一些工具:

*   更好的`kubectl`:这是 `[kubectl CLI](/portable-kubernetes-management-with-kubectl-in-docker-cb861a2c3c02)`的[包装器，可以用这个命令安装:](/portable-kubernetes-management-with-kubectl-in-docker-cb861a2c3c02)

```
*# .kube/config is a symlink to /etc/kubernetes/admin.conf*
*# running this container as root is only for testing purposes!*docker run --network=host --name=kubectl-host -v /etc/kubernetes/admin.conf:/root/.kube/config --rm -it piotrzan/kubectl-comp:zsh
```

*   [Octant](https://octant.dev/) 是一款 VMWare 开源集群可视化工具，在浏览器中运行，因此无需本地安装。

所有这些工具将使我们能够更容易地在集群中移动，并帮助我们可视化和学习。

# 练习文件

练习和示例代码位于[单独的存储库](https://github.com/Piotr1215/dca-exercises)中。您可以克隆它并直接从命令行工作，或者使用`kubectl`远程定位您想要部署的文件或文件夹。

# 结构

该系列中的每个帖子都将遵循相同的核心结构:

1.  它是如何工作的？
2.  它解决什么问题？
3.  如何实施？

# 摘要

敬请关注本系列的第一部分，我们将讨论 Kubernetes 存储，特别是卷、持久卷和持久卷声明！
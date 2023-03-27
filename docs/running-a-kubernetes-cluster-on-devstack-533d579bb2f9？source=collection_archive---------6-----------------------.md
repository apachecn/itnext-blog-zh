# 在 Devstack 上运行 Kubernetes 集群

> 原文：<https://itnext.io/running-a-kubernetes-cluster-on-devstack-533d579bb2f9?source=collection_archive---------6----------------------->

2020 年 12 月 30 日更新:该系列的第四部已经出版。

Kubernetes 似乎已经成为管理容器环境的事实上的标准。虽然像 [minikube](https://minikube.sigs.k8s.io/docs/) 这样的便利工具让我们能够在家庭实验室中研究 Kubernetes，但更现实的 Kubernetes 集群运行在云上。

这是记录我如何设法在 OpenStack 云上建立 Kubernetes 集群的系列文章中的第一篇。如果您选择跟随，您将使用 [**Devstack**](https://docs.openstack.org/devstack/latest/) 创建一个单服务器云，这是在 OpenStack 的 CI 环境中生成测试平台的工具。

然后，您将探索在 Devstack 云上部署 Kubernetes 集群的两种方法。首先，使用 *kubeadm* 工具进行手动设置。为了将集群与云集成，您将配置 **OpenStack 云提供商**和 **Cinder CSI 插件**，这两个插件都可以在 [Kubernetes Github repo](https://github.com/kubernetes/cloud-provider-openstack) 上获得。

第二种方法是 OpenStack 的 [**Magnum 服务**](https://docs.openstack.org/magnum/latest/) ，使用 OpenStack 标准创建 Docker Swarm、Mesos 或 Kubernetes 集群。

目录:

1.  [建立开发堆栈云](https://berndbausch.medium.com/a-single-server-devstack-cloud-as-a-kubernetes-platform-cd30e28e405a)
2.  [手动设置 Kubernetes 集群](https://berndbausch.medium.com/a-kubernetes-cluster-with-openstack-cloud-provider-and-cinder-plugin-b3f058ce898c)
3.  [在集群上运行应用](https://berndbausch.medium.com/kubernetes-on-devstack-part-3-running-applications-on-the-cluster-f15cf4762822)
4.  [使用 OpenStack Magnum 建立 Kubernetes 集群](https://berndbausch.medium.com/using-openstack-magnum-to-create-a-kubernetes-cluster-af8ec3cfbda5)
5.  在 Magnum 管理的 Kubernetes 集群上部署应用程序(尚未发布)。

要遵循这些说明，您需要一台功能适中的 PC、工作站或服务器。我使用了一个虚拟 Devstack 服务器，有 15GB 的内存和 6 个 vCPUs，运行 Ubuntu 18.04。这被证明是一个非常舒适的环境。合适的硬件(就我而言，是一台惠普 Z420 工作站)可以以大约 300 美元的价格买到。或者，每周预算 50 美元来租用服务器。
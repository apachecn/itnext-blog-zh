# 不到 3 分钟的 Kube 工具:Heptio Ark

> 原文：<https://itnext.io/kubetool-in-under-5-min-1-ebcf32fa9e81?source=collection_archive---------2----------------------->

通过这篇文章，我想开始一系列的帖子，目的是在 3 分钟内介绍 Kubernetes 开源工具。K8S 周围的生态系统变得越来越复杂，很难跟踪每天出现的伟大项目。

这不再是借口了！在每篇文章中，我将提供一个在集群中自动部署该工具的导航图(无论是我的还是其他作者的),以及一个快速采用该工具的简要指南。

我只有 3 分钟的时间，所以让我们从第一个帖子开始: [Heptio Ark](https://github.com/heptio/ark)

![](img/4267889b826aa3077e43c2ad95d82655.png)

Ark 是一款开源软件，由 Heptio 创建，用于轻松管理整个 Kubernetes 集群的备份和恢复。这包括资源定义和持久性卷。

GitHub 中提供了一个舵图，只需一个命令就可以部署它:【https://github.com/nachomillangarcia/helm-chart-ark。您还需要在您的计算机上安装命令行工具。对于 Linux 用户:

```
wget https://github.com/heptio/ark/releases/download/v0.8.1/ark-v0.8.1-linux-amd64.tar.gz
tar -xvzf ark-v0.8.1-linux-amd64.tar.gz 
sudo mv ark /usr/local/bin/
```

查看[发布页面](https://github.com/heptio/ark/releases)，获取 OSX 和 Windows 的二进制文件

Ark 由一个控制器(服务器)和几个代表备份、恢复和时间表的自定义资源定义组成。

备份可以存储在多个后端(S3、GC 存储、Azure 存储……)。它们只是一个. tar.gz 文件，包含集群中每个资源的 JSONs(或者只包含您在创建备份时选择的资源)。

最好的一点来自它还能在 AWS、GC 和 Azure 中创建持久卷的快照。只是魔术，没有其他工具需要有你所有的数据安全！

让我们看看如何创建备份:

```
ark backup create my-backup-all-cluster
```

这将为群集上的每个资源创建一个，包括永久卷。

您可以限制某些名称空间、资源类型或标签。要知道 **Ark 不允许通过名字**选择，只允许通过标签选择，所以现在是开始标记一切的好时机；)

```
ark backup create my-backup-hello --selector app=hello
```

这将只备份标有 app=hello 的组件。有关选项的完整列表，请查看命令帮助。

最后，要恢复备份，只需创建一个恢复对象:

```
ark restore create  --from-backup my-backup-hello
```

需要注意的是**恢复永远不会覆盖任何内容**，这样做没有任何风险。

还有许多其他命令和选项，用于列出和检查备份和恢复、创建计划、检查日志等。但是我会让你自己去发现它们:[https://heptio.github.io/ark/v0.8.1/](https://heptio.github.io/ark/v0.8.1/)

如果您之前不知道 Heptio Ark，或者您是否喜欢它，请告诉我！

[ignaciomillan.com](https://ignaciomillan.com)
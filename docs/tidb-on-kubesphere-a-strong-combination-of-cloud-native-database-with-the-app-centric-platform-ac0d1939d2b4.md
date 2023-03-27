# KubeSphere 上的 TiDB:云原生数据库与以应用为中心的平台的强强联合

> 原文：<https://itnext.io/tidb-on-kubesphere-a-strong-combination-of-cloud-native-database-with-the-app-centric-platform-ac0d1939d2b4?source=collection_archive---------1----------------------->

![](img/239474906f7f38e4394950133d8f8c45.png)

KubeSphere 上的 TiDB

在 Kubernetes 已经成为构建跨越多个容器的应用程序服务的事实标准的世界里，运行云原生分布式数据库代表了使用 Kubernetes 体验的一个重要部分。在这方面， [TiDB](https://docs.pingcap.com/tidb/dev/overview) ，一个支持混合事务和分析处理(HTAP)工作负载的云原生开源 NewSQL 数据库，很好地满足了这些需求。它的架构适合 Kubernetes，并且兼容 MySQL。TiDB 还具有水平可伸缩性、强一致性和高可用性。

![](img/2c9747334b09154cdbf2446191443cc1.png)

除了 TiDB，我还在使用 KubeSphere，这是一个开源的分布式操作系统，以 Kubernetes 为内核管理云原生应用。它为无缝集成第三方应用程序提供了一个即插即用的架构，以促进其生态系统。KubeSphere 可以在任何地方运行，因为它是高度可插拔的，无需侵入 Kubernetes。

![](img/1f82ed5396e87ac8145c6c0332a8bcc9.png)

通过将 TiDB 与 KubeSphere 相结合，我们可以拥有 Kubernetes 支持的 TiDB 集群，并使用开发人员友好的向导 web UI 来管理集群。在这篇文章中，我将演示如何在 KubeSphere 上从头开始部署 TiDB。

# 准备环境

1.  对于那些不熟悉 Kubernetes 和 KubeSphere，并且正在寻找快速发现该平台的人来说，一体化模式是您入门的最佳选择。它的特点是快速部署和无麻烦的配置安装，KubeSphere 和 Kubernetes 都在您的机器上提供。我为 quickstart 安装了一个单节点 KubeSphere 集群，你可以按照这个指南[在 Linux](https://kubesphere.io/docs/quick-start/all-in-one-on-linux/) 上进行一体化安装以了解更多细节。确保您在安装 KubeSphere 时在配置文件中启用了应用商店。

> 除了在 Linux 机器上安装 KubeSphere 之外，还可以直接在现有的 Kubernetes 集群上部署它。参见 Kubernetes 上的 [Minimal KubeSphere 以供参考。](https://kubesphere.io/docs/quick-start/minimal-kubesphere-on-k8s/)

2.群集将在大约 15 分钟内启动并运行。在本例中，当集群准备就绪时，使用默认帐户和密码(`admin/P@88w0rd`)登录 KubeSphere 的 web 控制台。以下是集群**概述**页面:

![](img/a8a1104f104360f2ececfcf186339e8a.png)

3.使用右下角工具箱中的内置 **Web Kubectl** 执行以下命令来安装 TiDB 操作员 CRD:

`kubectl apply -f https://raw.githubusercontent.com/pingcap/tidb-operator/v1.1.6/manifests/crd.yaml`

4.您可以看到预期的输出如下:

```
customresourcedefinition.apiextensions.k8s.io/tidbclusters.pingcap.com created customresourcedefinition.apiextensions.k8s.io/backups.pingcap.com created customresourcedefinition.apiextensions.k8s.io/restores.pingcap.com created customresourcedefinition.apiextensions.k8s.io/backupschedules.pingcap.com created customresourcedefinition.apiextensions.k8s.io/tidbmonitors.pingcap.com created customresourcedefinition.apiextensions.k8s.io/tidbinitializers.pingcap.com created customresourcedefinition.apiextensions.k8s.io/tidbclusterautoscalers.pingcap.com createdcustomresourcedefinition.apiextensions.k8s.io/tidbclusters.pingcap.com created customresourcedefinition.apiextensions.k8s.io/backups.pingcap.com created customresourcedefinition.apiextensions.k8s.io/restores.pingcap.com created customresourcedefinition.apiextensions.k8s.io/backupschedules.pingcap.com created customresourcedefinition.apiextensions.k8s.io/tidbmonitors.pingcap.com created customresourcedefinition.apiextensions.k8s.io/tidbinitializers.pingcap.com created customresourcedefinition.apiextensions.k8s.io/tidbclusterautoscalers.pingcap.com created
```

5.现在，让我们回到列出了所有工作区的**访问控制**页面。在我继续之前，首先我需要创建一个新的工作区(例如`dev-workspace`)。

在工作区中，不同的用户拥有不同的权限来执行项目中的各种任务。通常，一个部门范围的项目需要一个多租户系统，这样每个人都负责自己的部分。出于演示的目的，我在这个例子中使用账户`admin`。你可以[查看 KubeSphere 的官方文档](https://kubesphere.io/docs/quick-start/create-workspace-and-project/)了解更多关于多租户系统如何工作的信息。

![](img/49d7eff74092be213b828f838db04676.png)

6.转到一个工作区，你可以看到 KubeSphere 提供了两种方法来添加舵图。本文只谈第一个，就是在 KubeSphere 中添加一个 app repository。或者，你也可以上传你自己的应用模板，提交到 KubeSphere 应用商店，这将在我的下一篇博客中谈到。在这里，按照下图在 **App Repos** 中添加 ping cap([https://charts.pingcap.org](https://charts.pingcap.org/))的舵库。

![](img/70dddb7da53c3018f3940d1a6cfc74ac.png)![](img/60f2394b67ffdebdfdf8b407dad0867c.png)

# 部署 TiDB 操作员

[TiDB Operator](https://github.com/pingcap/tidb-operator) 是 Kubernetes 中 TiDB 集群的自动操作系统。它为 TiDB 提供了完整的管理生命周期，包括部署、升级、扩展、备份、故障转移和配置更改。借助 TiDB Operator，TiDB 可以在部署在公共或私有云上的 Kubernetes 集群中无缝运行。您需要首先在 KubeSphere 上部署 TiDB Operator。

1.  就像我上面提到的，我们需要首先创建一个项目(即名称空间)来运行 TiDB 操作符。

![](img/d8e6445b8a4aa6344bdf1914423f8397.png)

2.创建项目后，导航到**应用**并点击**部署新应用**。

![](img/133d2fed2f7302e4375ccb4d7ab3d01d.png)

3.从应用模板中选择**。**

![](img/faf93692b1b8701da3159f1d097fa5d6.png)

4.切换到存储多个舵图的 PingCAP 存储库。本文只演示了如何部署 TiDB 操作符和 TiDB 集群。您还可以根据需要部署其他工具。

![](img/8cf9b2af976ed3b0ea09314b65d99e68.png)

5.点击 tidb-operator，选择**图表文件**。您可以直接从控制台查看配置或下载默认的`values.yaml`文件。从右边的下拉菜单中，您还可以选择要安装的版本。

![](img/50002bbab946b6fa46c55379d22c55be.png)

6.确认你的应用名称、版本和部署位置。

![](img/33964f604e610e74a1a74d6debb464ad.png)

7.您可以在这一步编辑`values.yaml`文件，或者直接点击**使用默认配置部署**。

![](img/bf6bf5640434d36c7783e31d16c3e436.png)

8.在**应用程序**中，等待 TiDB 操作员启动并运行。

![](img/cbc9e3d1dd9b918dacbda6865b8035da.png)

9.在**工作负载**中，您可以看到为 TiDB 操作员创建的两个**部署**。

![](img/bcb24473f0e0476e7578b7ff95cd5a9b.png)

# 部署 TiDB 集群

部署 TiDB 集群的过程类似于部署 TiDB Operator。

1.  同样从 PingCAP 存储库中，选择 tidb-cluster。

![](img/52cb0f9fad2c8fcadea854fa22c206c9.png)

2.切换到**图表文件**并下载 **values.yaml** 文件。

![](img/61a50069bbd5e36e2fb9c8c0dfdcce51.png)

3.一些 TiDB 组件需要持久卷。对此，QingCloud 平台为用户提供了以下存储类。

![](img/5cc0cce7d5c426c410bf4e3ed54be31f.png)![](img/e7ca92c4d37dc64db8050f05109e7fa7.png)

4.当我通过 QingCloud Kubernetes Engine (QKE)安装 KubeSphere 时，所有这些存储组件都是默认自动部署的。QingCloud CSI 插件实现了支持 [CSI](https://github.com/container-storage-interface/) 的容器编制器(CO)和 QingCloud 的存储之间的接口。如果对 QingCloud CSI 感兴趣，可以看看他们的 [GitHub 资源库](https://github.com/yunify/qingcloud-csi)。通过用`values.yaml`中的`csi-standard`替换`storageClassName`字段的默认值`local-storage`，在此选择 csi-standard。在下载的文件中，可以直接全部替换，复制粘贴到`values.yaml`文件中。

![](img/badf10778d8c2d75a15e10c8e748edf2.png)

**注意:**只有字段`storageClassName`被改变以提供外部永久存储。如果您想要将每个 TiDB 组件(如 [TiKV](https://docs.pingcap.com/tidb/dev/tidb-architecture#tikv-server) 和[布局驱动程序](https://docs.pingcap.com/tidb/dev/tidb-architecture#placement-driver-pd-server))部署到各个节点，请指定字段`nodeAffinity`。

5.点击**部署**，可以在列表中看到两个应用，如下图所示:

![](img/38a5e77fc2640d79ffa2227bae2f6ef6.png)

# 查看 TiDB 集群状态

现在我们已经准备好了我们的应用程序，我们可能需要更多地关注可观察性。KubeSphere 通过仪表盘上可用的不同指标，让用户直观地了解应用在整个生命周期中的表现。

1.  验证所有 TiDB 集群部署都已启动并在**工作负载**中运行。

![](img/64220bdeb8375ceb94c2518905fa6e91.png)

2.TiDB、TiKV 和 PD 都是有状态的应用，可以在 **StatefulSets** 中找到。请注意，TiKV 和 TiDB 将自动创建，可能需要一段时间才会显示在列表中。

![](img/b49a85e6a9142a99def0eb164d5a3609.png)

3.单击单个状态集以显示其详细信息页面。此页面以折线图显示一段时间内的指标。以下是 TiDB 指标的一个示例:

![](img/60c878939f00162122c5d8f9688f84e2.png)

4.查看 TiKV 负载:

![](img/4437cd53f3bc0b08cfc6394c6dd105a6.png)

5.还列出了相关的 pod。如您所见，您的 TiDB 集群包含三个 PD Pods、两个 TiDB Pods 和三个 TiKV Pods。

![](img/557c871c6e21acc841096cefcb63fc40.png)

6.转到**存储**部分，您可以看到 TiKV 和 PD 正在使用持久存储。

![](img/1c7444fe693b001a7e7290541a8a25e7.png)

7.还会监控卷的使用情况。下面是 TiKV 的一个例子:

![](img/2c862a08bf02aec53297036c07e0cdd4.png)

8.在**概述**页面上，您可以看到当前项目中的资源使用列表。

![](img/b1e36d559c81c9ea237f5724b71dfd9f.png)

# 正在访问 TiDB 群集

这些刚刚创建的服务可以很容易地访问，因为 KubeSphere 会告诉您服务是如何在哪个端口上公开的。

1.  在**服务**中，可以看到所有服务的详细信息。

![](img/20ba51cfcfe1433cb5ae3ef6fe368507.png)

2.由于服务类型设置为`NodePort`，您可以通过集群外的节点 IP 地址访问它。

3.下面是一个通过 MySQL 客户端连接到数据库的测试。

> [root @ k8s-master 1 ~]# docker run-it—RM MySQL bash
> 
> 【root @ 0 D7 cf 9d 2173 e:/# MySQL-h 192 . 168 . 1 . 102-P 32682-u root
> 欢迎来到 MySQL monitor。命令以结尾；或者\g.
> 您的 MySQL 连接 id 为 201
> 服务器版本:5 . 7 . 25-tid b-v 4 . 0 . 6 TiDB Server(Apache License 2.0)社区版、MySQL 5.7 兼容
> 
> 版权所有 2000、2020、Oracle 和/或其附属公司。保留所有权利。
> 
> 甲骨文是甲骨文公司和/或其附属公司的注册商标。其他名称可能是其各自所有者的商标。
> 
> 键入‘help；’或“\h”寻求帮助。键入' \c '清除当前的输入语句。
> MySQL>显示数据库；
> +————+
> |数据库|
> +—————+
> |信息 _ 模式|
> |度量 _ 模式|
> |性能 _ 模式|
> | mysql |
> |测试|
> +————+
> 集合中的 5 行(0.01 秒)
> 
> mysql 【T36

4.此外，TiDB 集成了 Prometheus 和 Grafana 来监控数据库集群的性能。从上面我们可以看到，Grafana 是通过`NodePort`曝光的。在 QingCloud 平台的安全组中配置必要的端口转发规则并打开其端口后，您可以访问 Grafana UI 查看指标。

![](img/2ee7ede25daaf281421c150b4df99796.png)

# 摘要

我希望你们都成功地部署了 TiDB。TiDB 和 KubeSphere 都是针对云原生应用程序的强大工具，所以实际上，我无法在本文中展示它们的所有方面。例如，应用程序部署功能可以为像我这样的云原生爱好者提供很多东西。我将发布另一篇文章，介绍如何通过将舵图上传到 KubeSphere 应用商店来部署 TiDB。

如果您有任何问题，请随时通过 [Slack](https://join.slack.com/t/kubesphere/shared_invite/enQtNTE3MDIxNzUxNzQ0LTZkNTdkYWNiYTVkMTM5ZThhODY1MjAyZmVlYWEwZmQ3ODQ1NmM1MGVkNWEzZTRhNzk0MzM5MmY4NDc3ZWVhMjE) 或 [GitHub](https://github.com/kubesphere) 联系我们。

# 参考

https://github.com/kubesphere/kubesphere**KubeSphere GitHub**:[](https://github.com/kubesphere/kubesphere)

**TiDB GitHub**:[https://github.com/pingcap/TiDB](https://github.com/pingcap/TiDB)

**TiDB 操作员文档**:[https://docs . ping cap . com/tid b-in-kubernetes/stable/tid b-Operator-overview](https://docs.pingcap.com/tidb-in-kubernetes/stable/tidb-operator-overview)

**KubeSphere 简介**:[https://kubesphere.io/docs/introduction/what-is-kubesphere/](https://kubesphere.io/docs/introduction/what-is-kubesphere/)

**KubeSphere 文档**:[https://kubesphere.io/docs/](https://kubesphere.io/docs/)
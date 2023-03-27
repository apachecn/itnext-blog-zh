# 为 Anthos 迁移—应用程序迁移和现代化

> 原文：<https://itnext.io/migrate-for-anthos-application-migration-and-modernization-78113264b07a?source=collection_archive---------1----------------------->

![](img/fbdf548d1afa19ebfaec9885a5e5b1b7.png)

通过 Migrate for Anthos 实现应用程序现代化

迁移涉及将运行在物理服务器上的虚拟机/传统应用程序提升和转移到公共云平台，这提供了许多好处，包括增强的灵活性、与数据中心相比显著降低的开销以及标准的使用和管理环境。更新应用程序的第一步是将应用程序分解成一组容器映像，这些映像包括运行应用程序的一部分所需的一切:代码、运行时、系统工具、系统库和设置。有了 Kubernetes，所有这些容器都可以托管在一个“托管”集群上，所有底层硬件、操作系统/内核和集群 SRE(站点可靠性工程)都由云提供商负责。

[Migrate for Anthos](https://cloud.google.com/migrate/anthos/docs/getting-started) 解决了迁移和现代化问题，提供了一种近乎实时的解决方案，可利用现有虚拟机并将其作为 Kubernetes 托管的 pod，其中包含与在 Kubernetes 集群中执行应用程序相关的所有价值。这个框架最大限度地减少了移动和转换现有应用程序到 Anthos GKE 和 Google Kubernetes 引擎上的容器所需的人工工作。用户可以从在本地、计算引擎或其他云中运行的虚拟机或物理服务器中选择工作负载。

为了对虚拟机进行容器化 *Migrate for Anthos* 从源虚拟机创建包装容器映像，将操作系统替换为 GKE 支持的操作系统，通过解析 fstab 来分析文件系统，然后使用 Migrate for Anthos 容器存储接口(CSI)驱动程序装载持久卷，该驱动程序从源虚拟机传输数据。

![](img/1c3e6e2f7a4ac3c2bca4f639ee4e1e18.png)

虚拟机到容器的转换

*Migrate for Anthos* 将源虚拟机转换为在 GKE 运行的系统容器。与应用程序容器相比，系统容器在一个容器中运行多个进程和应用程序。虽然应用程序没有转换为真正的微服务架构，但容器化 monolith 应用程序有一些基本优势，包括可移植性、更好的打包和容器化。在操作系统级别虚拟化应用程序，多个容器直接在操作系统内核上运行，使它们比虚拟机更加轻量级和灵活，为开发人员提供逻辑上与其他应用程序隔离的操作系统沙盒视图。

该工具提供了一个易于使用的界面，使用声明性规范来规划、执行和管理迁移。所有控制器都在一个处理集群(GKE 集群)上运行，用户可以使用 CRD 管理迁移的所有方面。

*Migrate for Anthos* 与 Migrate for Compute Engine 一起使用(在云到云迁移场景中，当 SourceProvider 是 Google Compute Engine 时不需要),后者使用流技术，不需要迁移完整的虚拟机存储来在 GKE 运行容器，这加快了整个迁移过程。以前的 Velostrata 这种云迁移技术是去年由谷歌收购的，以增强 Kubernetes，它通过提供两种重要功能来实现这一点-将现有虚拟机转换为 Pods (Kubernetes 应用程序)和流式传输本地虚拟机和物理机，以在 GCE 实例中生成克隆。

# 迁移—步骤序列

![](img/02199e230729f90d8aeec59cfee709c2.png)

为 Anthos 迁移—步骤顺序

# Google 计算引擎(虚拟机)到 Google Kubernetes 引擎(Kubernetes 部署)

下面的示例工作流显示了使用 Migrate for Anthos 将在 GCE (Google Compute Engine)上作为虚拟机运行的示例 Apache Tomcat 应用程序迁移到在 GKE (Google Kubernetes Engine)上运行的 Kubernetes 部署的步骤顺序和组件说明。

此处的迁移源是 GCE，不需要迁移计算引擎(以前为 Velostrata)。

![](img/04afff47b6a3891ed30dea3d5c47a459.png)

将 GCE 上的虚拟机迁移到 GKE Pod

迁移架构包括一个 GKE 集群，用于托管迁移所需的各种控制器，被称为“处理集群”。一旦迁移完成，所有工件都存储在 GCS (Google 云存储)中，生成的容器映像被推送到 GCR (Google 容器注册中心)。

可以使用 cloud shell 中的[Migrate for Anthos CLI](https://cloud.google.com/migrate/anthos/docs/migctl-reference)(MIG CTL)或使用 Anthos portal(迁移到容器)来执行迁移(设置和管理迁移)。

![](img/9c7b7d28dacd43581f45ed7abf8e38f7.png)

迁移到容器— Anthos 门户

将虚拟机迁移到容器，然后将其部署到 GKE 上的步骤顺序如下所示:

![](img/c04376baecf640cd92af960792e23c08.png)

将 GCE 上的虚拟机迁移到 GKE Pod

从 marketplace 部署的示例 Tomcat 应用程序用于试用，该应用程序被部署到单个计算引擎实例。

![](img/06051ae1176598637ea118e3e2978712.png)

作为 GCE 实例运行的示例 Tomcat 应用程序

如下图所示，当使用 External_IP 访问时，该应用程序完全可以显示默认网页。

![](img/25e9cbceb36a79277407159b6cf167d1.png)

作为 GCE 实例运行的示例 Tomcat 应用程序

# 为 Anthos 设置迁移

创建了一个名为“处理集群”的 GKE 集群，它将托管 Anthos 控制器、CRD 和执行迁移所需的其他组件的所有迁移。

![](img/f413bc1de30aa84eded53c06c4e6d250.png)

迁移处理集群— GKE

一旦 GKE 群集可用，用户就可以使用“MIG CTL”(Migrate for Anthos CLI)来引导所需的基础架构。

![](img/cbb55804d6a49f63cd8bdf15691aa772.png)

引导—处理集群— migctl

如下图所示，‘*MIG CTL setup install*’创建一个准入验证器、csi-vlsdisk-controller、csi-vlsdisk-node、v2k-controller-manager、v2k-generic-csi-controller、v2k-generic-csi-node。除此之外，还创建了一个 cert-deploy 作业，用于创建 webhook 身份验证所需的证书，以及一个 controllers-upgrade 作业，用于将控制器升级到最新版本。

![](img/45cf25f04e98dd0936dd1904c859b622.png)

正在处理群集-为 Anthos 迁移-组件

![](img/b0adbf011625e4a5a71f6d7eeb588374.png)

正在处理群集-为 Anthos 迁移-组件

作为引导的一部分，多个 CRD 安装在群集上。每个迁移单元/步骤/配置都作为自定义资源保留在集群中。

![](img/bd1b0c3101b39b1cad949fb796e66162.png)

处理集群—为安索斯迁移— CRD 的

![](img/b7e15ec7e3abf6d3a49043130d376487.png)

处理集群—为安索斯迁移— CRD 的

作为 bootstrap 的一部分，GCS (Google Cloud Storage)中会创建一个存储桶，一旦迁移完成，它就会存储工件。

![](img/80f176df91fb90e0b3b22d248078d5e3.png)

工件的 GCS 桶

# 配置源提供者和机器

迁移要求用户定义实例的迁移源。支持的来源包括 VMware、AWS、Azure 或计算引擎。如果从 CE (Google 计算引擎)迁移，则不需要迁移计算引擎，当迁移在其他云(包括内部部署(VMware)上运行的虚拟机(VM)时，需要迁移计算引擎。

SourceProvider 用于迁移配置。可以定制迁移配置:选择从容器化中排除的区域、选择要提取为永久卷的区域、工件存储位置信息、存储设置等。，下面显示的是 SourceProvider(针对 CE)和迁移 CRD:

![](img/e6b529ca30b8cd58d1ceb320b420a109.png)

SourceProvider and Migration — CRD

用户可以使用“migctl”来配置 SourceProvider，群集可以有多个 SourceProvider 对象，这些对象可以在不同的迁移配置中使用。

![](img/8772718aa2918ae55df67c995f23eba9.png)

配置 SourceProvider — migctl

![](img/1ea2b0cdf51aefd5b3c5e934c00986cb.png)

控制器—源提供者配置

如上所示，控制器管理器监视 Anthos 迁移相关对象，并在集群上实现相同的操作。

CRD SourceProvider 保存上面提供的信息，这里 SourceProvider 是计算引擎。

![](img/c2165d9919da009bd54331f8addec22b.png)

来源提供商-CRD

# 创建迁移计划

通过从门户开始迁移或使用 CLI 来启动虚拟机的迁移。这导致迁移计划文件的创建。这是作为 Kubernetes 自定义资源定义(CRD)实现的，并与其他资源(如 Kubernetes PersistentVolumeClaim)一起包含在迁移计划文件中。

![](img/a96cd245117772b4a1cef3f0cf430808.png)

为 Anthos 迁移—迁移计划

![](img/764da4c6a6c94fc5496e1b16198e25e2.png)

为 Anthos 迁移—迁移计划

迁移计划是作为自定义资源创建的，可以使用 Anthos portal 或编辑集群中的迁移对象来编辑。

![](img/9f30bf744fd7dd84925922bcd976bf7f.png)

为 Anthos 迁移—迁移计划

可以使用意向标志根据工作负载的性质定制迁移计划。该标志的值决定了您的迁移计划的内容，进而指导迁移。

迁移模式中支持三种意图。映像:用于无状态工作负载，映像和数据:用于有状态工作负载，其中应用程序和用户模式系统被提取到容器映像，数据被提取到持久卷，数据:用于有状态工作负载，其中只有迁移计划的数据部分将被提取到持久卷。

*   指定要从迁移中排除的内容(意图:任何):

默认情况下，Migrate for Anthos 将排除与 GKE 环境无关的典型虚拟机内容。“过滤器”字段值列出了应该从迁移中排除的路径，这些路径不会成为容器映像或数据卷的一部分。

*   设置数据卷的处理(意图:数据或图像和数据):

dataVolumes 字段指定迁移过程中要包含在数据卷中的文件夹列表。对于以数据或图像和数据为目的的迁移，必须对此进行编辑。

*   设置迁移的名称和命名空间(意图:任何):

使用元数据字段，用户可以指定迁移的名称和命名空间。默认情况下，Migrate for Anthos 使用迁移规范中指定的值。

*   设置容器图像的名称(Intent: Image 或 ImageAndData):

将根据从源平台复制的虚拟机文件和目录创建的映像的名称。这个映像将不能在 GKE 运行，因为它还没有适应在那里的部署。

*   为部署工件设置云存储位置(意图:任何):

Migrate for Anthos 使用云存储来存储与迁移相关的文件，可以自定义存储桶和文件夹名称。

此步骤还创建初始磁盘快照/副本，并将其存储在永久卷中，同时无缝创建所需的 PV 和 PVC。

![](img/7c3e7c64f74f0fbb9e17dd4a5fbf6ce9.png)

PV 和 PVC —初始快照

![](img/00af5837fa119705f5c31764244b0778.png)

PV 和 PVC —初始快照

![](img/d558726779486f123389d63bbc0092a8.png)

PV 和 PVC —初始快照

# 迁移计划执行

此阶段执行迁移计划，并执行以下任务:

*   将代表虚拟机的文件和目录作为映像复制到容器映像注册表中。
*   Migrate for Anthos 创建了两个映像:一个可运行的映像，用于部署到另一个集群；一个不可运行的映像层，可用于在将来更新容器映像。所有的图像和图层都被推送到 GCR。
*   生成可用于将虚拟机部署到 Kubernetes 集群的配置 YAML 文件。

![](img/956bb9453dac56cc01a76abc2a038671.png)

为 Anthos 迁移—迁移计划执行

控制器创建一个生成工件并将图像上传到 GCR 的作业:

![](img/b07e7f2171091848b4e722ba450410f7.png)

为 Anthos 迁移—控制器—生成工件

![](img/2f2fd7c25406e3603450eba7e84deb6f.png)

为 Anthos 迁移-作业-生成工件

迁移完成后，迁移细节—工件部分将列出所有资源 URL。Generate artifacts 创建 docker 文件、用于部署的容器映像(即用映像)、容器映像基础层(不可运行的映像层)、Kubernetes 部署规范、迁移计划和一个包含所有工件链接的文件。

![](img/9c8168cb4d5e826452672d3f81bf4ce4.png)

为 Anthos 迁移—工件链接

上面生成的所有工件都存储在 GCS(谷歌云存储)中:

![](img/95205ca1b0805a71f221ce90f0803914.png)

GCS 中的工件

为工件和图像层创建单独的存储桶:

![](img/8a424dd2fe2cc7542e87da2d65f62a1e.png)

GCS 中的工件

![](img/9a29db4d0229351bd23293f968a01e25.png)

GCS 中的工件

存储桶由 PVC 的请求存储(StorageClass: <gcs>)创建，相应的持久卷在集群中创建。</gcs>

![](img/f4d9a027957d29c727ac86bce63bfaa1.png)

用于存储的永久卷

生成的映像被推送到 GCR (Google 容器注册中心),并且在部署规范中进行了同样的配置。

![](img/62b91db2a934943a4b5daa524d872af6.png)

推送到 GCR 的图片

![](img/443278453f2210496bfb84ce9254d283.png)

推送到 GCR 的图片

生成的 Dockerfile 文件:

![](img/8f14be630a676dba5e0179fbbcbb61bd.png)

生成的 Dockerfile 文件

包含所有工件链接的文件:

![](img/aeff404707f090d5380af05b6fbed47f.png)

带有工件链接的生成文件

生成的部署规范:

![](img/ea02b7b7bb830b8faf006e3d63e6aa1d.png)

生成的 Kubernetes 部署规范

生成的部署文件可用于将迁移后的应用程序部署为 Kubernetes 部署。

![](img/1b02ec89fef16db71a690b9147921505.png)

部署到 Kubernetes —使用生成的 Kubernetes 部署规范

![](img/89c5bd3993d465fc7494822bd9966f57.png)

在 Kubernetes 上运行 Tomcat 应用程序

部署规范将使用推送到 GCR 的映像进行配置:

![](img/243448e5adfadca7a0b646a0720e9743.png)

Tomcat 部署规范，图片来源于 GCR

如下所示，该应用程序功能齐全，并作为 Kubernetes pod 运行。

![](img/c2334f3961287a4b3ed6580650d70297.png)

功能 Tomcat 在 Kubernetes 上的应用

用户可以将 GKE 的所有功能和其他支持操作、监控和网络功能在 GCE 上与迁移的应用程序一起使用。

![](img/75a4fe0cdae0a04f47acc0a4fc801afa.png)

从 GKE 管理 Tomcat 应用程序

# 从其他云和内部(VMware)虚拟机到 Google Kubernetes 引擎(云到云迁移)

在这种情况下，Migrate for Anthos 使用 Migrate for Compute Manager 对云或本地虚拟机和物理机进行流式处理，以在 GCE 实例中生成克隆。

由于上面讨论的另一个场景是在 GCP 内部，所以不需要这个实体，在这里，来自外部云提供商的虚拟机被迁移(克隆)到 GCE，并且一旦虚拟机在 GCE 上可用，就按照相同的步骤将虚拟机迁移到容器。迁移计算引擎和源云之间的通信将通过 VPN 进行。

![](img/2a9884529c1cd36b1252c2599664fb1f.png)

将外部云平台上的虚拟机迁移到 GKE Pod

除了为计算管理器设置 VPN 和配置迁移之外，VM ->容器迁移涉及与 CE-source provider 相同的步骤。

![](img/41994023ed1eeb99bb4c088f18c7cf78.png)

云到云迁移—步骤序列

针对计算引擎的迁移(以前称为 Velostrata)在 marketplace 中可用，并作为托管实例进行部署。

![](img/9abe4ea9cb492e10480c0ae9a779980c.png)

GCP —为计算引擎迁移

部署 Migration Manager 需要用户启用站点到站点 VPN 和所需的防火墙，以实现管理器所在的 VPC 和另一个云上的源虚拟机的 VPC 之间的通信。

![](img/ceb2081073e20f8697ed21b540f87eea.png)

为计算引擎迁移—先决条件

如下图所示，在 GCP 和 AWS 之间创建了一个站点到站点 VPN。对 Azure 来说也是如此:

![](img/8be89f9b38779407f76b64a5ca0dd153.png)

站点到站点 VPN — AWS-GCP

如下所示，GCP 使用基于云 VPN 或云互联连接的云路由器，通过使用边界网关协议(BGP)为您的 VPC 网络提供动态路由。

![](img/2b2e97ac07ea078c73137890514f1a1c.png)

站点到站点 VPN — AWS-GCP —云路由器

AWS 站点到站点 VPN:

![](img/3bc8b43b5ef25a2645bb88a61b7b5509.png)

站点到站点 VPN — AWS-GCP

管理器需要特定的许可来处理关闭、打开等情况。，这可以通过使用附加到用户的策略并在配置管理器时提供用户密钥来实现。AWS 的一个例子如下:

![](img/a3fdae68405d9878fbd6513c669bb93e.png)

AWS 上计算引擎迁移的 IAM 权限

一旦部署了迁移管理器，用户就可以使用门户/UI 对其进行配置。用户应该向源云和目标云提供身份验证。对于 GCP，它使用 service-account-json 文件，对于 AWS，它使用 AcessKey 和 SecurityKey。

![](img/fd19e8a0b65166772c4321e63d92fae5.png)

配置-为计算引擎迁移

配置了一个云扩展，在 GCP(目标)上创建所需的虚拟机。云扩展使用双节点主动/被动配置实现高可用性；每个节点处理自己的工作负载，同时为另一个节点提供备份。

![](img/98016d07d30f7f63543e01dc42f017c9.png)

配置-为计算引擎迁移

操作手册的配置定义了要迁移的虚拟机、它们的顺序以及其他配置参数。

![](img/387b5202c34c0571d0b7eb9d4ab5bf67.png)

配置-为计算引擎迁移

migrate for Anthos service provider 规范应配置管理器信息以启用迁移，在这种情况下，有两个步骤，第一步是从其他云提供商迁移虚拟机(技术上创建一个克隆作为 GCE 实例),第二步是对其进行容器化。从其他云和本地迁移到计算引擎的规范如下所示:

![](img/324ff4f1cb9e1045317ad450dc3bba82.png)

配置—为计算引擎迁移— SourceProvider

*******************************************************************

加快现代平台的迁移和采用，使企业能够跨现有和新开发的应用程序使用统一的策略、管理和技能集来更高效地运营。

当工作负载升级到在托管 Kubernetes 集群(如 GKE)上运行的容器时，IT 部门可以消除虚拟机的操作系统级维护和安全补丁，并实现大规模自动化。迁移到 GKE 的传统应用程序可以轻松升级为现代功能，并且可以使用大量可用/维护的附加组件进行集成，而无需重写应用程序代码。
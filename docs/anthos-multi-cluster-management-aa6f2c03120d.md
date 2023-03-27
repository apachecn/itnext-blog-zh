# Anthos —多集群管理

> 原文：<https://itnext.io/anthos-multi-cluster-management-aa6f2c03120d?source=collection_archive---------1----------------------->

![](img/76f240a35c5d0a48d6dc5e6890c46e6e.png)

Kubernetes —混合、多云、多集群管理— Anthos

Anthos 是一个混合云和工作负载管理的产品和服务组合，运行在 Google Kubernetes 引擎(GKE)上，除了 Google 云平台(GCP)，用户还可以管理运行在 AWS、Azure、Oracle 等第三方云上的工作负载。和内部(私有)集群。这意味着现在用户可以享受他们喜欢的云来满足他们的应用程序部署和管理需求。管理员和开发人员不需要学习新云带来的所有新 API 和环境功能，只需要学习谷歌的即可。

跨公共云和本地云的架构一致性简化了部署，更重要的是，客户不需要做任何额外的工作来实施混合云。一旦 Anthos 本地集群被部署并连接到 Google 计算平台，混合云就可以运行了。截至目前，GKE 本地部署作为基于 VMware vSphere 的虚拟设备运行。对 Hyper-V 和 KVM 等其他虚拟机管理程序的支持正在进行中，但 Anthos connect 有助于连接任何 Kubernetes 集群，无论它位于何处以及是如何创建的。

Anthos 多云架构由一组运行在谷歌云平台(GCP)上的核心软件组件组成。Anthos 支持多种部署目标，用户可以在不同的云平台上配置和管理 GKE 集群，如 AWS、Azure、GCP、GCP 本地等。这些技术上称为托管集群，其中 Anthos 拥有通过控制平面启动的 GKE 集群的生命周期。

![](img/9632eb63b4e52669a8b0bbb196761087.png)

Anthos —组件

谷歌云平台控制台为管理部署在多个位置的 Kubernetes 集群提供了一个单一的控制平面——在谷歌公共云上，在本地数据中心上，或其他云提供商，如 AWS、Azure 等。，用户可以使用这种多集群管理功能来管理在不同环境中运行的不同/异构 Kubernetes 集群，而不管 Kubernetes 集群是如何部署的。这些组件共同提供了在不同的云中协调容器工作负载所需的所有服务，并提供了通用策略框架、集中式配置管理、安全性、服务管理和容器市场访问。

![](img/a7190e33704b55ea387d285a84a7b69a.png)

元控制平面

随着 Anthos 的引入，Google 云平台还支持从单一位置管理在任何地方运行的 Kubernetes 集群——在 Google cloud、本地数据中心或其他云提供商(如 Amazon AWS)上。此外，多集群功能还支持跨云和本地环境以及在不同环境中运行的工作负载的配置管理。

多集群管理的核心组件是 GKE 连接中心(简称 Connect)、谷歌云平台控制台和 Anthos 配置管理。

# 谷歌云连接

Connect 允许用户将本地 Kubernetes 集群以及在其他公共云上运行的 Kubernetes 集群与 Google 云平台连接起来。Connect 在 Kubernetes 集群和谷歌云平台项目之间使用加密连接，使授权用户能够登录集群，访问有关其资源、项目和集群的详细信息，并管理集群基础设施和工作负载，无论它们是运行在谷歌的硬件上还是其他地方。

注册远程集群时，会将 GKE 连接代理部署到集群，该代理管理与 GCP 上各种 API 端点的连接。该集群不需要公共 IP，只需要一组 googleapis 的可达性。连接代理使用从 Kubernetes 集群到 GCP 的经过身份验证和加密的连接。Connect agent 使用 VPC 服务控制来确保 GCP 是用户私有云的延伸，并且可以穿越 NAT 和防火墙。所有用户与集群的交互都可以在 Kubernetes 审计日志中看到。

![](img/c867fca98a839282a31d9fde53cbcd2c.png)

连接代理和 GCP 连接

所有必需的 API 都可以在各自的项目中启用，用户可以从 GCP 控制台获得 API 交互的详细指标:

![](img/17bf04f23f26e109bde338166a8a99a6.png)

用于启用 Anthos 的 API—Hub & Connect

![](img/62e8afa35b75b11927f00cdde9467d71.png)

用于启用 Anthos 的 API—Hub & Connect

# 将外部 Kubernetes 集群注册到 GKE 连接

集群注册需要目标集群的 Kubeconfig 和 gcloud 命令行以及所需的 RBAC，以便 GKE 能够访问集群。

可以使用命令行(gcloud cli)或 GKE 控制台完成集群注册，如下所示:

![](img/47e9aa7529c9d1332eaf60ff46579065.png)

聚类注册

集群注册部署连接代理部署以及访问 GCP 项目和连接集线器所需的机密，连接集线器是从注册请求中提供的 GCP 服务帐户导出的。

![](img/7b01f8af0db6c97efffc1d5d2786838b.png)

成员集群上的 GKE 连接代理

![](img/f3f6cd990c16ab90db848994379141bc.png)

成员集群上的 GKE 连接代理部署

与 GKE 连接代理关联的所有对象:

![](img/d4a4bdbdbe930187b6997419fd45e360.png)

GKE 连接代理-对象

通过注册生成的秘密使 GCP 能够通信:

![](img/7df6efb42f8bdc71c36c1f144988b9e8.png)

GKE 连接代理-秘密

![](img/e613f6bfba78bef34f495f3158a46e01.png)

GKE 连接代理-秘密

一旦建立了到 GCP 的身份验证，连接代理就会打开一个隧道来连接所需的 API:

![](img/d5034e032c8202a3bb8981e35449e4c5.png)

GKE 连接代理—日志

群集上的成员身份状态存储为“Kind:Membership ”,其中包含 gkehub 连接信息:

![](img/decfaa1758cf0f5dd8cbc2907a52b411.png)

会员自定义资源

可以使用 prometheus-to-sd 将所有 GKE 连接代理指标导出到 Stackdriver，并且可以从控制台的“monitoring”页面查看这些指标。部署了一个 prometheus-to-sd pod，它从 at 代理收集指标并将它们推送到 Stackdriver:

![](img/6db38fa6f028a48a17a2ce43108fb6c3.png)

普罗米修斯——斯塔克德瑞弗

度量浏览器:

![](img/45d17d899e1696598d464f3a31815591.png)

谷歌云——指标浏览器

![](img/4dd2fc021fb4e68dc0792c5395723435.png)

谷歌云——指标浏览器

集群注册完成后，可以从控制台的“Anthos”部分或“GKE”部分查看集群:

![](img/c7a19d5a45e87c2a3aafa252e0311f62.png)

Anthos 控制台—注册的集群

![](img/0bd7f2a82c33404bfeb08a1f4b062113.png)

Anthos 控制台—注册的集群

gcloud 命令行可用于更新、列出和描述成员资格:

![](img/7302a424b3792f6a083437243e9c144f.png)

命令行—注册的集群

用户可以使用“集群角色”和相关的密码登录到注册的集群。根据所需的管理功能，用户可以使用只读角色或集群管理员角色。

![](img/ecbf945e986b7db1b635eb67eea7b38d.png)

访问注册的集群

![](img/c11818d00d1a751df541743943c4b007.png)

服务帐户—令牌

用户可以使用三种身份验证过程之一登录到已注册的集群:

![](img/681efbf65d4d8f506effbb1a21a5fbd9.png)

访问注册的集群

一旦用户登录到集群，就可以访问集群的信息:

![](img/4dd6002de59ad681574fbad7727d0c91.png)

Anthos —控制台—注册的集群信息

# 查看和管理集群资源

可以从控制台的“Kubernetes 引擎”部分访问所有注册和登录的集群信息。

![](img/75fefa4a27083d3502935933f45b4c49.png)

Kubernetes 引擎—查看注册的集群资源

所有工作负载都可以从门户访问，在多个集群的情况下，过滤器可以对工作负载和其他资源进行分类。

![](img/972042123501f22bd94e5acf5770ad23.png)

Kubernetes 引擎—查看注册的集群资源

![](img/a5aff6a29c448d2996c6a213edf516a3.png)

Kubernetes 引擎—查看注册的集群资源

可以从控制台编辑任何资源，连接代理将在成员集群上执行更改。

![](img/3d9fca82d56cc2832798b85520061ef7.png)

Kubernetes 引擎—编辑注册的集群资源

可以从对象浏览器访问已注册集群的所有资源对象:

![](img/83bc0414f64c2bf3c1c55870c827566b.png)

Kubernetes 引擎—对象浏览器

# 试用—注册 EKS、AK 和本地集群

EKS 和 AKS 集群部署在个人用户帐户上。使用上面讨论的注册过程来注册集群。注册后，连接代理将部署在集群上。

AWS 上的 EKS 集群:

![](img/5d89bfd6af685af9adab8a9bc6190781.png)

EKS 集群—自动气象站

![](img/074d39378f50769156e3b033b65bb62d.png)

EKS 集群—自动气象站

Azure 上的 AKS 集群:

![](img/03053046cea4d0cb2963306f8d5f3227.png)

AKS 集群

![](img/f3481007fba1faa4fc28a80a71d19961.png)

AKS 集群上的 GKE 连接代理

![](img/d592257f107840d297252c9745474f38.png)

AKS 集群

如下所示，所有三个 Kubernetes 集群都已注册并标记为“类型:外部”:

![](img/9283a43918a73d3a8621986f6c273a63.png)

多个群集(EKS、AKS 和本地)—注册的群集

# 从 Google Marketplace 在外部集群上部署应用程序

GCP 容器市场提供了对开放源码和商业容器应用程序映像的大型生态系统的访问，这些映像可以部署在使用 connect 注册到 GCP 的任何地方运行的 GKE 集群上。有了 marketplace，客户可以将预先创建的容器用于常见的应用程序，如数据库或 web 服务器，而不必自己创建它们。Anthos 上支持的应用程序目录标有“*与 Anthos* 配合使用，使用户能够配置和选择现有集群来部署应用程序。

下面选择了一个 Anthos 支持的 Nginx 应用程序示例:

![](img/5f0c97ffff86b714cdbbaf0ed631bdfd.png)

示例 Nginx 部署——Google 云市场——在 Anthos 上得到支持

面向应用程序部署的群集需要访问“gcr.io”映像注册表来提取容器映像。这可以通过在项目中创建一个服务帐户并在 Kubernetes 中提供相同的“imagePullSecrets”来实现:

![](img/4bba21d94334946b00487b47d9ebed74.png)

Google 容器注册表—访问

如下所示，部署目录包括集群选择字段，其中列出了所有已注册的集群。根据用于登录集群的权限级别，集群分为合格和不合格。例如，本地群集没有足够的权限，因为所使用的登录名使用的是只读角色。

![](img/2e49dda2e75937cac5924d48235179a1.png)

Nginx 部署示例—集群选择

部署名称空间的选择:

![](img/97c28fbec9308f5d8d0e136e5282dd7f.png)

Nginx 部署示例—名称空间选择

可以通过选择如下所示的特定参数来启用指标，参数的选择会注入一个 sidecar prometheus-to-sd 容器。

![](img/9616847deccd7b44708673d66dc17cbf.png)

Nginx 部署示例——Google 云市场

可以从控制台查看部署步骤:

![](img/ca56281c66c133aace620f92ea3a7f2e.png)

Nginx 部署示例——Google 云市场

eks 集群上的连接代理接收关于应用程序部署的信息:

![](img/9dfe168aaf0025455310d90eec785223.png)

GKE 连接代理—日志

运行所需作业的 nginx-deployer 容器，用于在集群上部署应用程序:

![](img/b26415977025268aa75057a85b45ad52.png)

Nginx 部署部署者——谷歌云市场

![](img/2cf673de34141c206a36f51cc307694f.png)

Nginx 部署部署者——谷歌云市场

应用程序部署成功完成后，可以从“应用程序”控制面板查看和管理。

![](img/d27a18e7129fb89b457e8301fe78e30e.png)

Nginx 部署示例——Google 云市场

![](img/3b3db1cf87510d2c85309afc27dfc437.png)

Nginx 部署示例——Google 云市场

如下所示，nginx 应用程序已部署，存储部分显示了由 AWS 上的“ebs”支持的 pvc。

![](img/e7c321c5f33571ead690eee15cf860fc.png)

持久卷— EKS EBS

# Anthos 配置管理— GitOps

这是 Anthos 的核心组件之一。在多个位置部署和管理 GKE 集群时，很难使所有集群在配置、安全策略(RBAC)、资源配置、名称空间等方面保持同步。随着人们开始使用这些集群并开始进行配置更改，随着时间的推移，用户将遇到“配置漂移”，这导致当相同的应用程序部署在不同的位置时，不同的集群表现不同。Anthos 配置管理使用[配置即代码](https://landing.google.com/sre/sre-book/chapters/release-engineering/#configuration-management-vJsYUr)方法，通过描述性模板(在存储库中作为代码维护)实现集中式配置管理。这样可以很容易地确保您在集群中看到一致的行为，并且任何偏差都可以通过将更改恢复到上一次已知的良好状态来轻松纠正。

如下所示，ACM-Operator 安装在所有三个集群上:

![](img/fa4cb41f89ce5c363fae20a3d4d4864d.png)

Anthos 配置管理

包含多个名称空间和一些群集角色清单的配置示例报告:

![](img/f49dcf57c3ea46c3971fc8fd2bb54b6b.png)

Git Repo —配置源

配置管理操作员与 CRD 一起部署在群集上—配置管理:

![](img/9493b67d574c9697db22d3b396f27c23.png)

配置管理操作员—日志

![](img/e9df77e5dd16d114eebbaaa9a5ec0558.png)

配置管理—自定义资源

创建一个 ConfigManagement 对象，它包含 syncRepo:带有配置的 Repo 的信息，syncBranch: Git Branch，secretType:如果 repo 是全球可读的，则可以将其设置为 none，如果不相同，则类型可以是 SSH、Key 等。，policyDir:包含所有配置的子目录。

![](img/47662899e669f79a01480ee2b3d6807b.png)

配置管理—规格/配置

一旦创建了上述配置，操作员就按如下所示处理请求:

![](img/a9ede2c7acc87d33bc96310829802669.png)

配置管理操作员日志

创建了三个部署来导入、同步和监控通过 ACM 部署的工作负载。

![](img/60fbcb08e6cda3dc5dc827adf0725c03.png)

对象— Anthos 配置管理

![](img/f36d9cf63e7dc761412abdbf48e7c496.png)

对象— Anthos 配置管理

具有 git-importer、monitor 和 syncer 的 EKS 集群—从 GCP 控制台查看:

![](img/5da21cc18b45a63cb3d0c8d4ab94692c.png)

对象— Anthos 配置管理

git-importer 从指定的目录和 git-repo 导入所有配置:

![](img/50ef52769716355834005cb84a863224.png)

ACM — Git 导入程序组件

syncer 定期将集群中的对象状态与 git 配置进行比较，并与 git 源保持同步:

![](img/119a2e22fe5f458e70a5028aeccba151.png)

ACM —同步组件

如下所示，所有对象都是使用 git-repo 中的配置在所有三个集群上创建的，并将被同步以匹配状态。用户不能编辑由 ACM 创建的对象，因为除非从受管对象中删除了特定的注释(如下所述),否则更改将在定期从 repo 中同步后恢复。

![](img/a5f7561a1bb582c32e4c316c840a1e97.png)

创建的对象— Anthos 配置管理

用户可以选择使用“ *nomos* ”命令行工具，该工具可以安装在本地，用于与配置管理操作员进行交互，并可以在提交报告之前检查配置的语法。

![](img/25b8c59bc8d4111330d973ab51f6a72b.png)

Nomos CLI—配置管理操作员

如果集群中的某个对象有注释“【configmanagement.gke.io/managed:】启用”，则该对象由 Anthos 配置管理进行管理。如果对象根本没有标签“config management . gke . io/managed ”,或者如果它被设置为除 enabled 之外的任何值，则该对象为非托管对象。

当 Anthos 配置管理管理一个集群对象时，它会监视该对象和 repo 中影响该对象的配置集，并确保它们是同步的。用户可以管理现有对象，也可以停止管理当前正在管理的对象，而无需删除该对象。

下面的流程图描述了导致对象成为托管或非托管对象的一些情况:

![](img/a942fb6002f123d8a5cb89a40493046c.png)

托管和非托管对象

Google 发展了 Anthos，通过将遗留应用程序容器化来实现应用程序的现代化。它增强了内部部署和云基础架构的整体安全性。Anthos 完全可以管理混合云。用户甚至可以将 Anthos 与其他 Kubernetes 管理工具和平台相结合。例如，如果用户正在使用 Kubernetes 集群，这些集群是通过使用 Kubeadm 自主开发的自动化部署的，这意味着他们正在使用开源的 Kubernetes (or)在 EKS 或 AKS 上托管他们的 Kubernetes 工作负载，用户仍然可以向 Anthos 注册该集群，Anthos 将在更高的级别上工作，允许进行工作负载分配、CSM、策略分配和多集群管理。
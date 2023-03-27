# Azure Arc —支持 Arc 的服务器 Kubernetes 和 Arc

> 原文：<https://itnext.io/azure-arc-arc-enabled-kubernetes-arc-for-servers-2ebcd87094af?source=collection_archive---------3----------------------->

![](img/f25f9b7829bc2a1c28020009b5330846.png)

面向混合云管理的 Azure Arc 随时随地管理一切

混合搭配云是一个新术语，随着时间的推移，这个术语越来越流行。在这种方法中，用户可以使用来自多个云提供商的服务，其中来自一个提供商的服务在竞争的云提供商之上提供(或者控制一个提供商与其他提供商的服务)，使组织能够以不同的方式一起使用来自不同提供商的服务。

Google Anthos 现已全面上市，Google 继续将其作为混合和多云产品进行推广。AWS 推出了混合云产品 Outposts。 [Azure Arc](https://venturebeat.com/2019/11/04/microsoft-unveils-azure-arc-stack-edge-and-new-virtual-machine-services/) (预览版)是最新发布的微软 Azure 版本的统一管理平台，可简化跨内部、边缘和多云的复杂分布式环境的操作。Anthos 和 Azure Arc 在本质上都是相似的；每个服务都利用 Kubernetes 作为运行托管数据服务的基础。此外，这两种服务都允许注册外部集群，并通过同一个控制平面提供管理，它们还允许用户跨多个集群部署应用程序。Anthos 已经公开宣传其在其他公共云上运行的能力(AWS 上的 GKE)。然而，关键的区别在于，使用 Azure Arc，客户可以在混合环境中混合搭配物理服务器、虚拟机和 Kubernetes 集群。

Azure Arc (Preview)旨在跨任何基础设施扩展 Azure 管理。这可能意味着在多个云中运行的工作负载，如 Azure、AWS 和 Google，在 Azure 堆栈或其他硬件中运行的工作负载，以及在边缘运行的服务。考虑当前跨组织在云中和内部运行的所有服务——Kubernetes 集群、数据服务、Windows 和 Linux 服务器等。Azure Arc 的主要区别在于传统的基于虚拟机的工作负载和现代的容器化工作负载之间的平衡，这两种工作负载在混合和多云环境的相同上下文中运行。

![](img/ee24b405aa7dc9e58447ca71ed926ac9.png)

来源:微软 Azure Arc

# 天蓝色弧形启用 Kubernetes

Azure Arc enabled Kubernetes(preview)允许用户将在本地或任何其他云提供商上运行的 Kubernetes 集群与 Azure 连接起来，以获得统一的管理体验。Arc 为跨多个位置和平台部署的所有 Kubernetes 集群的用户提供了单一平台操作模型。Arc 为集群提供了 Azure 管理功能——释放 Azure 特性，如 Azure 策略、Azure 监视器和 Azure 资源图。

通过将外部现有的 Kubernetes 集群附加到 Azure，用户可以像控制任何其他内部 Azure 资源一样使用所有 Azure 功能来控制外部集群。

![](img/2dca1b48d7090a2a24751babacdf10f3.png)

来源:微软—支持 Azure Arc 的 Kubernetes

# 聚类注册

用户可以将 Azure CLI 与启用了 Azure Arc 的 Kubernetes CLI 扩展(connectedk8s)或门户一起使用，在门户中可以自动生成包含所有必需 CLI 步骤的脚本。Arc 注册使用 Helm-3 通过 connectedk8s 扩展装载所有集群代理。

来自成员集群的 Azure Arc 代理安全地建立到各种 Azure 端点的出站连接(例如:[https://management.azure.com](https://management.azure.com)，[https://eastus.dp.kubernetesconfiguration.azure.com](https://eastus.dp.kubernetesconfiguration.azure.com)等)。)并且不需要 Public_IP(需要出站权限—端口 443 上的 TCP(https)和端口 9418 上的 TCP(Git))。

# 使用 Azure Arc 门户注册集群:

![](img/fa3372388b7216317c894a028cc9cbf3.png)

电弧注册一个集群

![](img/943867907e035f89b64944baa472d807.png)

电弧注册一个集群

用户可以选择一个特定的订阅和一个资源组，在该资源组下确定群集的范围。需要一个唯一的集群名称和一个区域(目前预览版仅支持美国东部和西欧):

![](img/a35ee0b217f0ea41fd0c34beb0b07ed0.png)

电弧注册一个集群

下一步是使用上面提供的输入(subscription_id、cluster_name、location)生成一个脚本(带有 connectedk8s 扩展的 Azure CLI 步骤),该脚本可以在一个环境中运行，在该环境中可以使用 kubeconfig 和 helm 访问群集:

![](img/251d10fab34b58a646be98654e3ae2ff.png)

电弧注册一个集群

该脚本向 Azure 请求动态身份验证(与用户使用 az cli 进行身份验证的方式相同— az 登录)，身份验证完成后，将为 TLS 通信生成代理证书，并启动 helm 安装以安装所有必需的代理和其他对象:

![](img/9674c27914daf3bc256a32234da7aad5.png)

电弧注册一个集群

用户可以使用注册门户的验证选项卡来验证连接性:

![](img/e5a7c655ee7dc8e19b2c822582f3474a.png)

电弧注册一个集群

注册完成后，可以从 Azure Arc 门户访问集群信息，注册的/外部集群可以利用本机 Azure Kubernetes 集群可用的所有功能。

![](img/010a9ee44c38c83bf177586b13c54324.png)

电弧注册一个集群

# 安装的组件/代理

使用 helm 的 connectedk8s 扩展安装多个代理作为部署，以及所需的机密和配置映射到群集中的“azure-arc”命名空间。

![](img/6294731689ab4f3afeab437cb9cc1113.png)

Arc —集群代理

![](img/3c9228a769a98c8ecc9912cb75f789d5.png)

Arc —集群代理— Kubernetes 部署

使用组件用来建立连接的令牌和其他身份验证信息创建的配置映射和机密:

![](img/b8e4fcf689561c3e212afa4a578be0f5.png)

Arc 配置图和机密

在群集“AzureClusterIdentityRequest”和“ConnectedCluster”中创建了两个 CRD:

![](img/e4902f1364dcfa69fa90488c3e0ba611.png)

弧形-连接集群-CRD

![](img/ef70a079f0268aab6cd4df7f56b45a64.png)

arc-AzureClusterIdentityRequest-CRD

“ConnectedCluster”用于定期将外部群集状态同步到 Azure，而“AzureClusterIdentityRequest”为群集上运行的所有代理提供身份管理。

![](img/788e3d97e04235ed16b9df50b4bcdd07.png)

电弧— CRD 的

helm 创建了一个名为“azure-clusterconfig”的配置映射，其中包含必要的信息，如 subscription_id、tenant_id、azure_resource_group、azure_region 等。上面创建的所有组件将使用它来建立到 azure_management 和其他相应端点的安全连接。

![](img/35cd4c161280329f6fee35fb34958975.png)

Arc —集群配置

“kube-rbac-proxy”和“fluent-bit”边车容器与操作员舱一起部署，以提供授权/安全连接和记录。

![](img/f40d57bc9e2f732d0d9ed9e71742a05b.png)

Arc — kube_rbac_proxy 和流畅位边车容器

kube-RBAC-为 TLS 连接生成自签名证书的代理容器:

![](img/dff2be9319aba9b7e1576b016d177623.png)

Arc — kube_rbac_proxy 边车容器

从其他运营商代理 pod 收集指标的 fluent-bit 容器:

![](img/061777b93bee5c6ede5c12b3ec36aae8.png)

Arc — fluent_bit 边车容器

群集上的配置代理安全地连接到“kubernetisconfiguration . azure . com ”,从而实现从 Azure 到外部群集的主要连接。其他代理，如集群身份运营商，资源同步和控制器管理器，定期连接到管理端点，以执行必要的操作。度量代理轮询来自所有组件的监控信息，并将这些信息推送到 Azure。

群集中的代理向 Azure 端点发出出站调用:

![](img/1928a6a552d8644455823a64dca0c4e1.png)

Arc —集群代理

群集中的配置代理轮询群集中的本地 CRD 更改，并将其发送到 Azure:

![](img/438d7be748bf241cf5481383bcea9fca.png)

Arc —集群代理

显示本地群集信息轮询的配置代理日志:

![](img/85bc536311f0c56906e50cf6c8d1b113.png)

Arc —配置代理日志

度量代理从其他代理收集度量并将它们发布到 Azure:

![](img/d1cab0e1adc4513fa5f93199e8bf4c00.png)

Arc 指标-代理

![](img/1fa275a03b73daeb473e31482cd26a1d.png)

Arc —指标—代理日志

显示基于角色的身份管理的集群身份操作员日志:

![](img/79489bdb6d485fc68d6c21b9e3bfcbe0.png)

Arc 集群身份-操作员日志

显示群集元数据与 Azure 端点同步的群集元数据操作员日志:

![](img/9443fcd58df8cc2de33f2a3f7f5db5d3.png)

Arc 集群-元数据-操作员日志

显示特定 CRD 同步的资源同步代理日志:

![](img/3775f243954e146ed1f5a2c9cf5f04a8.png)

Arc —资源同步代理日志

# 多云和多集群注册

用户可以从任何云提供商添加 Kubernetes 集群，并从统一的仪表板管理它们。所有注册的集群都可以利用关键功能，包括库存和组织、治理和配置、集成开发和管理功能以及统一的工具体验。

尝试注册运行在 GCP 上的 GKE 集群和运行在 AWS 上的 EKS 集群。

GKE 集群:

![](img/d95de91737dc353a51ab98fe6c71da04.png)

GCP GKE 集群

注册后，所有代理都部署到 azure-arc 名称空间。以下是从 GCP 的 GKE 控制台看到的画面:

![](img/30ef4994d1be79890d554a28c9b8cad4.png)

集群代理— Azure Arc — GKE 门户

Azure Arc 门户上的 GKE 集群:

![](img/97666106475597d368cd4fea251e6249.png)

Azure Arc —注册的 GKE 集群

从 Azure Arc 访问的所有 GKE 集群工作负载—洞察:

![](img/e85c705ede8c63ba4480792dd6a0d9fe.png)

Azure Arc —注册的 GKE 集群—工作负载

AWS 上的 EKS 集群:

![](img/ce95f6a914cf7249c8a0188fb168f3ac.png)

AWS 上的 EKS 集群

Azure Arc 门户上的 EKS 集群:

![](img/782e027bc0cb559a436dffc82a06947e.png)

Azure Arc —注册的 EKS 集群

从 Azure Arc 访问的所有 EKS 集群工作负载—洞察:

![](img/fa30c44e3183e1f1443bca1abb7aa387.png)

Azure Arc —注册的 EKS 集群—工作负载

具有多个集群的 Azure Arc 仪表板:

![](img/bee3580fbf3c044580db62d5d6d874e6.png)

Azure Arc —多集群控制面板

用户可以使用资源图浏览器查询所有连接的集群和其他容器资源:

![](img/17119f16cd33f3dfc813bb2b1ac97050.png)

Azure Arc — Azure 资源图表浏览器

![](img/dac2e31730a19b74157f6c9c05659313.png)

Azure Arc — Azure 资源图表浏览器

# 监控和日志记录— Azure Monitor

Azure Monitor for Containers on Azure Arc-enabled Kubernetes 为用户提供了与 Azure Kubernetes 服务(AKS)监控类似的功能，例如:通过从 Kubernetes 中可用的控制器、节点和容器中收集内存和处理器指标的性能可见性，通过工作簿和 Azure 门户中的可视化，为故障排除问题发出警报和查询历史数据，收集 Prometheus 指标的功能等。

可以使用 [PowerShell 或 Bash 脚本](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-enable-arc-enabled-clusters)为 Kubernetes 的一个或多个现有部署启用 Azure Monitor for containers。该脚本创建一个日志分析工作区(用户可以选择支持区域中的现有工作区)，helm 用于引导 kube-system 命名空间中的监控代理(omsagents as Daemonsets)。

![](img/0ef88d30aa4c65dbbc7c2bef0f5a0c47.png)

天蓝色电弧—监控代理

![](img/447ee6849a15bbd5e4393abcbb89c021.png)

天蓝色电弧—监控代理

监控和日志记录代理(fluentd)的所有配置都以配置图的形式提供:

![](img/dc2150409b340e6c6e9027e79ffe77a5.png)

Azure Arc —监控代理—配置

所有的 omsagent 处理来自集群的日志，并将它们发送到 Azure Application Insights(Azure Monitor 的功能，一个可扩展的应用性能管理(APM)服务):

![](img/6db81c9cc7ee14f96f0e6364e5e41996.png)

天蓝色电弧—监控代理

集群指标:

![](img/1bb7767870f8667efbe0a7c343ee08b1.png)

Azure Arc —集群指标

集群运行状况:

![](img/2059867d8221e6dc93d96f957b2f7783.png)

Azure Arc —集群运行状况

节点运行状况:

![](img/561f48268e861997e16f7981131f3ec4.png)

Azure Arc —节点运行状况

节点级工作负载分布:

![](img/f2655d43962c076d70365270370cea0c.png)

Azure Arc —工作负载视图

容器级别信息:

![](img/016f93dae616493f50f2bde302838d2e.png)

Azure Arc —工作负载视图

![](img/c791201d77cfd51041a5d27c2f686726.png)

Azure Arc —工作负载视图

日志门户提供大量内置查询来导出主机级和容器级日志:

![](img/b34efee5f6495c05b69e3fd7c26b7fbe.png)

天蓝色弧光—记录

![](img/2143cc50942cff9197eede3ac70f42de.png)

天蓝色弧光—记录

![](img/65a041e6b8b60cf14003f4b9fa71bf72.png)

天蓝色弧光—记录

如果注册了多个集群，monitoring dashboard 的单个窗格会提供所有已注册集群的摘要:

![](img/6635dd4c0ad53e6a4848b81c69c9229d.png)

Azure Arc —多集群监控仪表板

# 策略实施— Azure 策略

Azure Policy 有助于强制执行组织标准并评估大规模合规性。Azure Policy 通过将这些资源的属性与业务规则进行比较来评估连接的 Kubernetes 群集中的资源。

Azure Policy 扩展了[看门人](https://github.com/open-policy-agent/gatekeeper) v3，一个用于[开放策略代理](https://www.openpolicyagent.org/) (OPA)的准入控制器 webhook，以集中、一致的方式在您的集群上应用大规模实施和保护。

策略门户包括用户可以使用的内置定义，或者用户可以使用[资源管理器模板](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-tutorial-create-encrypted-storage-accounts)参考定义自己的自定义定义。

![](img/25f31d3054b01bc1cf7c06c0d972b5c1.png)

Azure Arc —策略

策略可以在资源组级别应用/限定范围，用户可以使用排除从组中忽略任何资源。下面的策略示例在资源组中的所有三个已注册集群上实现了一个标记规则，其中所有资源都应指定强制标签:

![](img/bc67c65e70f2360bbf27dc2183afd0a6.png)

Azure Arc —策略

参数包括为策略要执行的操作定义的一组属性/变量(这些是 ARM 模板中的变量):

![](img/3d7fdc021fce4984604d063a69137fbe.png)

Azure Arc —策略

补救参数使用户能够强制执行创建的策略，以对当前现有的资源进行操作。默认情况下，如果未选择补救，则仅在新创建的资源上实施该策略。

![](img/a20881de5e0bf55438d823a5d09e232d.png)

Azure Arc —策略

ARM 模板是基于选定的策略自动创建的:

![](img/38c1009e96d953b9a8cb7644eefaefbe.png)

azure Arc-ARM-策略

定义后，将在所有集群上评估该策略，并生成包含所有非投诉资源的报告:

![](img/6ab560a2888a1db2fbacdb21cde9555c.png)

Azure Arc —策略

![](img/65fcc82d068405a0f6229f9232528ec7.png)

Azure Arc —策略

# 已连接集群的 GitOps

有了 GitOps，集群管理员可以集中管理各种重复的任务，比如创建名称空间、RBAC、注册表机密等等。

Arc 使用了 [Flux](https://fluxcd.io/) ，这是一个来自 Weaveworks 的开源 GitOps 部署工具，目前是[云本地计算沙箱项目](https://www.cncf.io/sandbox-projects/)的一部分。代理监视 Git repo 的更改并应用它们。该代理还会定期确保集群状态与 Git repo 中声明的状态相匹配，并在发生任何非托管更改时将集群返回到所需状态。

![](img/46d706f24fc1c485e01a99a469ef5a84.png)

天蓝色电弧— GitOps —通量

用户可以有多种方式使用 Git 作为管理 Flux 应用程序的资源，用户也可以使用 Azure 策略来执行每个“微软。kubernetes/connected clusters ' resource 或 Git-Ops enabled 'Microsoft。container service/managed clusters 的资源具有特定的“Microsoft。kubernetisconfiguration/sourcecontrolconfiguration '应用于它。

在集群中运行的支持 Azure Arc 的 Kubernetes 配置代理负责监视新的或更新的 sourceControlConfiguration 资源，并自动协调添加、更新或删除 Git repo 链接。

# 使用来自 Git 的集群配置(Kubernetes 清单)的 GitOps

用户可以使用门户或 Azure CLI 扩展“*k8s 配置*”中的“配置”在集群上执行 GitOps。

下面的配置中使用了一个示例 repo，它由多个文件夹组成，命名空间为:cluster-config、team-a、team-b、Deployment:cluster-config/azure-vote-config map:team-a/endpoints。

使用 Arc 集群门户配置 GitOps:

在下面名为 gitops-deploy 的配置中，指示代理在 gitops-deploy 名称空间中部署操作员，并授予操作员 cluster-admin 权限。Helm 操作符在此配置中被禁用，因为 Git 源由 Kubernetes 清单组成。

![](img/183d70ae54ce6e5e1fb36cc16975d336.png)

天蓝色的弧形——吉托普斯

使用 CLI 配置上述相同的规范:

![](img/47c12db26e4cf29185c19b4836786638.png)

天蓝色的弧形——吉托普斯

集群中的 config-agent 每 30 秒轮询 Azure 一次新的或更新的源代码控制配置。这是配置代理获取新的或更新的配置所需的最长时间。

![](img/2b22752f96d080e58996565f8288e7e4.png)

Azure Arc —配置代理日志

此配置在“git-deploy”名称空间中部署了一个操作符(flux)和 memcached(Flux 的一个依赖项，它缓存容器映像以加快速度),如下所示:

![](img/a1644ac8fa2fd8d173a17d4e8efb4d68.png)

天蓝色电弧— GitOps —通量

从 Arc Insights 控制台部署操作员视图:

![](img/b6dcedb1ccd4f6bd53ff0dc41c2cdddc.png)

天蓝色电弧— GitOps —通量

显示来自 Git 配置的名称空间和其他对象的处理的操作员日志:

![](img/7e1953044e824175a1431169648431c0.png)

天蓝色电弧— GitOps —通量

# 使用舵的 GitOps 带通量的舵操作器

舵操作员提供了 Flux 的扩展，可以自动发布舵图。一个图表发布通过一个名为 HelmRelease 的 Kubernetes 定制资源来描述。Flux 将这些资源从 git 同步到集群，而 Helm 操作员确保按照资源中指定的方式发布 Helm 图表。

![](img/6b4a2effaf6b576366e0bdbe828db1a0.png)

azure Arc-GitOps-Helm 操作员

示例 repo 包含两个目录，一个包含掌舵图，另一个包含发布配置。在 releases/prod 目录中，azure-vote-app.yaml 包含 HelmRelease 配置。

```
├── charts
│   └── azure-vote
│       ├── Chart.yaml
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── deployment.yaml
│       │   └── service.yaml
│       └── values.yaml
└── releases
    └── prod

        |_ azure-vote-app.yaml
```

使用 Arc 集群门户配置 gitops:

在名为 azure-voting-app 的配置中，指示代理在 prod 名称空间中部署操作员，并授予操作员 cluster-admin 权限。资源库 URL 中提供了源 Git-URL。在此配置中启用了 helm 操作符，因为 Git 源包含了与 helm 操作符一起工作的 Helm 配置。

![](img/75125064f640c405a0400bf377278c71.png)

天蓝色弧线— GitOps —头盔

使用 CLI 配置上述相同的规范:

![](img/f42e1fd296e98a2f825e6c84367703c4.png)

天蓝色弧线— GitOps —头盔

此配置在“azure-voting-app”命名空间中部署一个操作符(flux)和 memcached(Flux 的一个依赖项，它缓存容器图像以加快速度),并应用部署配置来创建投票应用部署，如下所示:

![](img/f5d3c305108aa0bbb10b976f2283a2eb.png)

天蓝色弧线— GitOps —头盔

显示舵图处理的操作员日志:

![](img/a118741362058229aecb81c43a61735b.png)

天蓝色弧线— GitOps —头盔

Arc Insights 控制台中的已部署组件视图:

![](img/8552354ca29baf05155682a89dcb99e0.png)

天蓝色弧线— GitOps —头盔

# 使用 Azure 策略的 GitOps

用户可以使用策略提供上述步骤中陈述的所有配置，以大规模启用 GitOps。

![](img/5e80625d65a2ac03accfa4e54e0b34ca.png)

Azure Arc — GitOps —策略

# 使用 Azure Arc for servers 管理其他云平台上的 Kubernetes 节点

Azure Arc 不仅支持外部 Kubernetes 集群注册，还支持任何外部运行的机器。这为用户提供了额外的好处，他们可以控制托管 Kubernetes 集群的基础架构(例如:EC2、GCP 虚拟机、本地服务器等。).

Azure Arc for servers (preview)允许用户管理 Azure 外部托管在其他云提供商或企业网络内部的 Windows 和 Linux 机器，类似于管理原生 Azure 虚拟机的方式。当混合机器连接到 Azure 时，它就变成了连接的机器，并被视为 Azure 中的资源。

![](img/1dbb07861cdd4912c32fa3a09be8f99a.png)

Azure Arc —服务器

例如，将本地集群中的一个 Kubernetes 节点添加到 Azure，步骤如下所示。

与 Kubernetes 集群类似，Arc 登录页面包括添加服务器:

![](img/865d346045fe65f212ad2bdff41afbd9.png)

Azure Arc —服务器注册

用户可以使用交互式脚本或以防万一来添加机器群用户可以提供服务主体——使用 Azure Active Directory [服务主体](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals)将机器连接到 Azure Arc for servers，而不是使用个人特权身份[交互式连接机器](https://docs.microsoft.com/en-us/azure/azure-arc/servers/onboard-portal)。

![](img/c4117589618562ec8d552a6a50dcd5a5.png)

Azure Arc —服务器注册

如果需要，用户应该配置资源组、区域、操作系统(Linux/Windows)和代理。

![](img/84915717a2dd11dc32a7902688d8bc7c.png)

Azure Arc —服务器注册

创建一个安装混合代理的脚本，该脚本可以在目标服务器上运行。使用' *azcmanagement* '完成服务器注册，这些包适用于 Linux 和 Windows。

![](img/3edfba878432c4fe57ffa7b284100011.png)

Azure Arc —服务器注册

注册脚本请求对 Azure 进行动态身份验证(与用户在使用 az cli 时进行身份验证的方式相同— az 登录)。

![](img/cfed235637853b6043f59676dcbcb4f2.png)

Azure Arc —服务器注册

注册后，用户可以从 Azure Arc -Machine 门户访问外部机器信息，如下所示，左边的工具栏显示了任何其他 Azure VM 可以从目录中使用的所有功能。

![](img/0f4d4c267d3d2c6d239270f848dd333f.png)

Azure Arc —服务器注册—门户

用户可以使用 Azure 策略、更新管理、变更跟踪等所有功能。可以在 Azure VM 上执行来管理外部机器。

用户可以使用扩展从目录中安装日志记录和其他代理，如下所示:

![](img/8871f6c749e34609863197f319fc1a25.png)

天蓝色弧线-延伸

例如，用户可以使用“Linux 定制脚本扩展”来跨云提供商或内部数据中心的多个虚拟机/服务器运行脚本。

![](img/62d8b9a10579004cb5ef1ecf3eeeb0b6.png)

天蓝色弧线-延伸

扩展自动创建所需的 ARM 模板所需的:

![](img/be3225bd082af9d612ce3eb0b2fd0fcc.png)

天蓝色弧线-延伸

随着这个平台将每个系统都引入 Azure Arc，在不牺牲可见性和访问的情况下，基于清晰的关注点划分为团队成员设置清晰的角色和职责变得非常容易。Azure Arc 的另一大好处是大规模的服务器管理。当您将本地服务器连接到 Azure 时，它将成为一种资源，并作为资源组的一部分进行管理。对于在 Azure 上有大量投资的客户来说，这提供了一个受欢迎的好处，即扩展他们熟悉的管理模型到其他环境。客户可以在混合环境中轻松混合搭配物理服务器、虚拟机和 Kubernetes 集群。

微软也是首批将托管数据服务引入混合云的公司之一。由于这些数据库服务被打包成容器并运行在 Kubernetes 之上，从集中的 Azure 控制平面管理它们变得非常有效。Azure Arc 是最近发布的，仍处于预览阶段，Arc 有很大的空间成为混合环境的革命性管理平台，并在竞争中为其他玩家提供激烈的竞争。
# SuperEdge，OpenYurt —将本地 Kubernetes 扩展到 Edge

> 原文：<https://itnext.io/superedge-openyurt-extending-native-kubernetes-to-edge-cc59094f92c?source=collection_archive---------4----------------------->

![](img/6f25af77d62122c655c46cda8f430d9c.png)

将云原生功能扩展到边缘—云边缘隧道、边缘自治、应用群管理

企业需要一个强大的管理层来有效管理提供动态编排和自动化的边缘流程。然而，整个边缘架构由不同的硬件和软件资源组成。edge 生态系统包含分布在多个地区的异构硬件，并根据需求进行定位。据领先的全球研究和咨询组织称，边缘体系结构具有分层结构。

![](img/bf45aedef8f038962e5cb3965b42b2eb.png)

Gartner 和 IDC —边缘拓扑结构

无论层次结构中的硬件是什么，Kubernetes 都可以提供一个竞争激烈的基础架构，能够无缝管理在不同计算资源上运行的各种工作负载，提供一组可以集中管理但在边缘环境中执行的通用服务。Kubernetes 可以编排和调度资源，因为它们从云反弹到边缘，反之亦然。

有效的边缘计算模型应该解决延迟和带宽的限制，使用统一的管理系统部署到异构环境，设备群管理，边缘站点的网络自治等。设计集中管理的边缘拓扑有相当大的复杂性:

![](img/dba4e556074b7e055c6e71d81f3427c7.png)

边缘—复杂性

带有特定工具的 Kubernetes 可以有效地解决大多数问题。将边缘节点作为 Kubernetes 节点进行管理，而不是在每个站点托管一个具有控制平面的成熟集群，这提供了一些优势，如通过仅部署工作组件来最小化边缘站点资源，不必使用任何其他联合机制来集中控制边缘站点，以与本地 Kubernetes 节点相同的方式控制边缘站点等。但是，使用当前的上游设计(每个节点上的 Kubelet 将在内存中缓存信息)将节点移动到边缘有很大的局限性，并且主要依赖于节点和托管在不稳定的公共互联网上的云上的主节点之间的连接。

[OpenYurt](https://openyurt.io/en-us/) (阿里巴巴第一个面向边缘计算的开源云原生项目，CNCF [沙盒项目](https://www.cncf.io/sandbox-projects/))和 [SuperEdge](https://github.com/superedge/superedge) (由腾讯与英特尔、VMware、虎牙、Cambricon、Captialonline 和美团牵头)是两个基于原生 Kubernetes 构建的同源项目，旨在通过将云原生能力扩展到边缘端来无缝支持边缘计算。简而言之，使用户能够管理在边缘基础架构中运行的应用，就像它们在云基础架构中运行一样。这两个项目都为云边缘隧道、边缘自治、智能和统一应用管理、大规模应用交付等提供了必要的控制器、组件和定制资源。

# 操作和维护隧道

> **云上的 kube-apiserver 和边缘上的 kubelet 之间的隧道**

在应用部署、运行和维护的过程中，用户经常需要获取应用日志或者直接登录正在运行的应用进行调试。这通常涉及 ku bectl——日志、执行、编辑、修补、获取、描述操作。

从控制平面(apiserver)到节点有两条主要的通信路径。第一个是从 apiserver 到在集群中每个节点上运行的 kubelet 进程。第二种是通过 apiserver 的代理功能从 apiserver 到任何节点、pod 或服务。在 kubectl 请求的一般设置中，kubelet 将作为服务器端，负责处理 kube-apiserver (KAS)转发的请求，这需要 KAS 和 kubelet 之间的网络路径，以允许 KAS 主动访问 kubelet。

![](img/3d185d105586f213d6ae7a734c3ea897.png)

kube-API server-kube let 通信

然而，在边缘节点场景中，边缘节点通常位于本地专用网络中。尽管这确保了边缘节点的安全性，但这可能会给位于云控制节点中的 KAS 直接访问位于边缘节点的 kubelet 带来问题。因此，为了支持来自云节点的边缘应用的操作和维护，请求必须被代理到边缘节点，以解决云不能直接访问边缘节点的问题。

# 露天蒙古包—蒙古包隧道

[Yurt-tunnel](https://github.com/alibaba/openyurt/blob/master/docs/tutorial/yurt-tunnel.md) 由云中的 [TunnelServer](https://pkg.go.dev/github.com/alibaba/openyurt@v0.0.0-20200917165522-b447d2ebcca1/cmd/yurt-tunnel-server?tab=overview) 和运行在各个边缘节点上的 [TunnelAgent](https://pkg.go.dev/github.com/alibaba/openyurt@v0.0.0-20201022001436-cdc6acf7c8ec/cmd/yurt-tunnel-agent) 组成。TunnelServer 通过反向代理与运行在各个边缘节点的 TunnelAgent 守护进程建立连接，在公有云的控制平面和边缘节点之间建立安全的网络访问。

![](img/190d1844139795d6dbcc0393a142dae6.png)

露天蒙古包—蒙古包隧道

Yurt-tunnel 使用上游项目[API server-network-proxy(ANP](https://github.com/kubernetes-sigs/apiserver-network-proxy))来实现服务器和代理之间的通信。ANP 是 1.16 中的 alpha 特性，用来寻址 [EgressSelector](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/server/egressselector/egress_selector.go) ，旨在实现 Kubernetes 集群组件的跨内网通信(例如主组件位于控制 VPC，其他组件如 kubelet 位于用户 VPC)。ANP 的一个核心设计思想是使用 gRPC 封装 KAS 的所有外部 HTTP 请求。这里选择 gRPC 主要是因为它支持流和清晰的接口规范。此外，强类型客户端和服务器可以有效地减少运行时错误并提高系统稳定性。

与原生 TCP 连接相比，ANP 不仅可以提供更高的传输稳定性，还可以大大减少公共网络流量。考虑到边缘集群中数万个节点的规模。

# 超级边缘— TunnelCloud 和 TunnelEdge

SuperEdge 支持自建隧道(目前支持 TCP、HTTP、HTTPS)，解决不同网络环境下的云边缘连接问题。隧道由[隧道-云](https://github.com/superedge/superedge/blob/main/docs/components/tunnel.md)和[隧道-边缘](https://github.com/superedge/superedge/blob/main/docs/components/tunnel.md)组成，能够保持持久的云-边缘网络连接。SuperEdge 使用 [tunnel-dns](https://github.com/superedge/superedge/blob/main/pkg/edgeadm/constant/manifests/tunnel-coredns.go) (coredns)组件将发往边缘节点的特定请求转发给 tunnel-cloud 服务。

![](img/5bf1e9766e3c19dcbd968d450c3c3d8d.png)

超级边缘—隧道边缘和隧道云

tunnel-cloud 负责维护与边缘节点 tunnel-edge 的网络隧道，tunnel-edge 负责建立与 cloud edge 集群 tunnel-cloud 的网络隧道，接受 API 请求，并转发给边缘节点组件(kubelet)。

一旦 tunnel-edge 与 tunnel-cloud 建立 grpc 连接，tunnel-cloud 会将其 podIp 和 tunnel-edge 所在节点的 nodeName 的映射写入 DNS。如果 grpc 连接断开，tunnel-cloud 将从映射中删除 podIp 和节点名称。当 APIServer 或其他云应用访问 kubelet 或边缘节点上的其他应用时，tunnel-dns 通过 dns 劫持将请求转发到 tunnel-cloud Pod(通过用 tunnel-coredns 的集群 Ip 替换 kube-API server 的 DNS 名称服务器，将主机中的节点名称解析为 tunnel-cloud 的 podIp)。

# 边缘节点自治

> **在网络断开和其他中断情况下处理边缘节点上的服务(本地自治)**

在 Kubernetes 扩展到边缘计算场景中，边缘节点将通过公网和云端连接到控制平面。考虑到公网的不稳定性和成本等因素，边缘要求边缘业务能够在断开状态或弱网状态下继续运行。

在传统的 Kubernetes 系统架构中，代理(kubelet)中的容器信息保存在内存中。当网络断开时，无法从云中获取业务数据。如果节点或 kubelet 重新启动，服务容器将无法恢复。Kubernetes 的节点生命周期控制器根据节点的租约和节点状态更新时间决定是否驱逐节点上的 pod 或设置 taint，并将节点就绪状态设置为 unknown。

需要边缘节点中的代理连续轮询云主机上的 KAS，并在节点上本地缓存信息，其中在网络中断的情况下，该代理可以用作影子 apiserver，这还使得边缘节点上的 pod 能够从代理检索信息，而不是到达控制平面，这显著减少了公共网络上的往返。

当节点被标记为未就绪或未知时，与启用自治的节点相比，通常的流程如下:

![](img/39bef30f3df79414f56f218b55dfc81b.png)

节点管理和自治

# OpenYurt — YurtHub

[YurtHub](https://www.alibabacloud.com/blog/deep-dive-into-openyurt-yurthub-extended-capabilities_597190) 是一个具有数据缓存功能的透明网关，边缘节点上的 kubelet 通过 YurtHub 与云端通信。它充当节点上的临时配置中心，在网络连接中断时，继续为节点上的所有设备和客户服务提供数据配置服务。

YurtHub 提供了对大量原生 Kubernetes API 的支持，并可以在节点和边缘单元的维度上提供“影子 Apiserver”能力，这在网络链路较弱的边缘计算场景中尤其有价值。当到云的网络连接被断开并且节点或 kubelet 被重启时，服务容器的数据被从 YurtHub 获得以确保边缘自治。

![](img/d32552526c666d755c6dec121cdd467e.png)

OpenYurt — YurtHub

YurtHub 与其他资源一起将网络配置资源(如服务)缓存在本地存储中，通过提供网络中断前对象的状态和配置信息，使服务在本地访问时不受影响。当 YurtHub 接管边缘节点和云之间的流量时，它也可以接管节点证书管理。

YurtHub 可以从系统组件和服务容器接管云访问流量，并在云上实现基于节点的节流。在这种情况下，当单个节点对云的并发请求数超过 250 时，YurtHub 会拒绝新的请求。

[Yurt-controller-manager](https://www.alibabacloud.com/blog/introducing-main-feature-of-openyurt-local-autonomy_596322) 组件为不同的边缘计算用例提供了节点控制器、单元控制器(待发布)等少量控制器。用户可以用“【openyurt.io/autonomy:】真”标记边缘节点，其中处于自治模式的节点中的 pod 将不会被逐出 API 服务器，即使节点心跳丢失(从控制平面断开)。nodelifecycleController 组件管理自治并控制分离舱的驱逐。

# 超级— lite-apiserver

[lite-apiserver](https://github.com/superedge/superedge/blob/main/docs/components/lite-apiserver.md) 是运行在边缘节点上的 kube-apiserver 的轻量级版本。它充当从边缘节点上的所有组件和 pod 到云 apiserver 的请求的代理，lite-apiserver 的请求缓存控制器缓存响应，以在云-边缘网络断开的情况下实现边缘自治。边缘节点上的 Kubelet 通过 lite-apiserver 与云端通信。

超级边缘提供 L3 级边缘自治。当边缘节点不稳定或与云网络断开连接时，边缘节点仍然可以正常运行，不会影响已经部署的边缘服务。

![](img/b8204dc0b39b5d607a4387c31e899bd1.png)

超级— Lite-Apiserver

SuperEdge 提供[端到端分布式健康检查](https://github.com/superedge/superedge/blob/main/docs/components/edge-health.md)能力。每个边缘节点都将部署一个边缘健康代理。同一边缘群集中的边缘节点将对彼此执行健康检查，并对节点的状态进行投票。这样，即使云边缘网络出现问题，只要边缘节点之间的连接正常，节点就不会被驱逐。每个边缘节点默认运行 [edge-health](https://github.com/superedge/superedge/blob/main/docs/components/edge-health.md) ，并引入分布式健康检查，实现对每个节点组的独立健康检查，允许每个节点检测和监控同一组内的节点，消除不稳定的云到边缘网络造成的误报。

整个设计避免了云侧网络不稳定导致的迁移和重建，保证了边缘站点服务的稳定性。

# 跨异构边缘节点的应用群管理

> **智能和拓扑感知的应用程序调度和维护**

在边缘场景中，工作者节点的分布相对分散。同一物理位置可能只有一个工作节点，也可能有多个工作节点。边缘节点通常具有区域性、地带性、硬件特定性或其他逻辑分组特征(例如相同的 CPU 架构、相同的运营商、云提供商)。

从边缘节点的角度来看，需要将节点分成不同的组/池，以表示同一组功能。这种分组可以通过 Kubernetes 添加相同标签、注释和污点的本地方式来实现。OpenYurt 和 SuperEdge 都提供了一个健壮的应用程序群管理系统，用户可以添加一个逻辑来根据需求将应用程序部署到特定的站点。

# 超级边缘—应用网格控制器

[应用网格控制器](https://github.com/superedge/superedge/blob/main/pkg/application-grid-controller/controller/service/controller.go)负责管理构成[部署网格](https://github.com/superedge/superedge/blob/main/pkg/application-grid-controller/controller/common/deploymentgrid-crd.go)和[服务网格](https://github.com/superedge/superedge/blob/main/pkg/application-grid-controller/controller/common/servicegrid-crd.go)的服务组，这两个服务网格是由 SuperEdge 实现的，用于管理跨边缘节点的应用部署。

SuperEdge 使用户能够使用节点单元和节点组将节点隔离到一个池中，其中节点单元是根据标签区分节点的基本对象，节点组可以将多个节点单元分组。这些节点组在 ServiceGroup 中使用，它提供了两个 CR:部署网格和服务网格。

![](img/507cb31bc49d03df5dfdff305ca6c818.png)

超级—应用程序管理

ServiceGroup 提供的部署网格(deployment-grid)和服务网格(service-grid)可用于部署和管理边缘应用，控制服务流程，确保每个区域的服务数量和灾难恢复。

DeploymentGrid 的格式类似于部署的格式。“< deployment template >”字段是 Kubernetes 原始部署对象的模板字段，具体增加的是*gridunikke*y 字段，表示上面创建的节点组标签的键值。例如，如果用户使用' region:us-** '创建一个节点组，那么 gridUniqKey 的值将是 region。

![](img/4498d495493cca7b647755005efdd0ee.png)

超级边缘—部署网格和服务网格

ServiceGrid 的格式类似于服务的格式。‘< service template >’字段是 Kubernetes 原始服务的模板字段。在 ServiceGrid 规范中，可以使用*gridunikke*y 将特定的节点组分组为服务下的端点。例如，如果用户选择键的值为“region ”,那么服务端点仅构成部署在由区域标签区分的特定节点组中的 pod。当服务尝试使用 ServiceGrid 从节点组中的特定节点创建的服务的 ClusterIP 访问应用程序时，流量将仅分发到同一节点组中的端点。

ServiceGrid 使用户能够创建本地域，避免跨区域访问服务，控制对特定服务的访问，控制连接阈值，还便于更轻松地发布和管理边缘集群服务。

如果在部署完 DeploymentGrid 和 ServiceGrid 资源后，有新的 NodeUnit 添加到集群中，系统将自动为新的 NodeUnit 创建相应的部署和服务。

# open Yurt-Yurt-App-Manager

Yurt-App-Manager 是 Kubernetes 的标准扩展。它可以与 Kubernetes 结合使用，提供两个定制资源: [NodePool](https://github.com/alibaba/openyurt/blob/54daca0801a8f645c2832390076fac5e0bb70455/pkg/yurtappmanager/client/clientset/versioned/typed/apps/v1alpha1/nodepool.go) 和 [UnitedDeployment](https://github.com/alibaba/openyurt/blob/54daca0801a8f645c2832390076fac5e0bb70455/pkg/yurtappmanager/client/clientset/versioned/typed/apps/v1alpha1/uniteddeployment.go) 。

节点池是一个池、一个组或一个节点单元。使用标签对主机进行分类和管理，将具有相同属性的工作节点分组。节点池根据不同边缘区域的相似标签对节点进行分组。当一个节点被添加到节点池时，它继承了节点池规范中定义的注释、标签和污点，并且将添加一个标记'*apps.openyurt.io/nodepool:<节点池名称> '* 。

![](img/70f6e4e048787581588d3a8bf93c215f.png)

OpenYurt —节点池

UnitedDeployment 控制器提供了一个模板来定义应用程序，并根据节点池标签匹配不同的区域。每个 UnitedDeployment 下每个区域中的工作负载称为池。目前，池支持两种类型的工作负载:状态设置和部署工作负载。控制器基于 UnitedDeployment 的池配置创建子工作负载资源对象。

![](img/f5293ce20c35ebb7ced6a58a2a7131eb.png)

OpenYurt —统一部署

UnitedDeployment 使 k8s 本机部署、Statefulset 和 Daemonset 资源能够根据指定的节点池进行智能部署。通过 UnitedDeployment 实例，可以自动维护多个部署或状态集资源，并且它还可以具有不同的配置，如副本。用户可以对特定的 UD 对象规范进行更改，以管理跨多个边缘站点的应用程序配置，而无需更改多个传统部署或状态集对象。

# 试验

# 集群拓扑

该拓扑包括一个运行在具有 public_ip 的公共云提供商上的主节点和三个连接到家庭网络(私有)的边缘节点(udoo-x86 板)。SuperEdge 和 OpenYurt 使用相同的集群，因为它们都是本地 Kubernetes 扩展，用户可以通过拆除控制平面和边缘组件来重新使用集群。

![](img/922e579063ebfb4fd560a1d6904a9553.png)

公共云大师

![](img/103238be6733f6bd35220eb2af382a43.png)

私有边缘上的节点和云主节点

如下图所示，public-cloud-master 在 public_ip 上服务 Kubernetes apiserver，所有边缘节点都在私有网络中。

![](img/1a8300e7719b66ee77df8baa84cade60.png)

专用网络上的节点和公共网络上的主节点

# OpenYurt

OpenYurt 在云端和边缘的组件如下图所示。

![](img/4b1cbebb006cc865293125f51a918c4e.png)

OpenYurt —建筑

用户可以使用 [Yurtctl](https://github.com/alibaba/openyurt/blob/master/cmd/yurtctl/yurtctl.go) ，这是一个集中的安装和管理工具，可以快速地将任何 Kubernetes 集群转换为支持 OpenYurt 的集群(或者可以手动安装所有组件)。

![](img/ef9e6bca260a8f4585911df34d1fa5a1.png)

OpenYurt 依靠一个节点标签'【openyurt.io/is-edge-worker】T4'来安装组件。

![](img/886abf3cc81548b94c1673d42637edae.png)

OpenYurt —标识边缘节点的节点标签

控制器管理器、应用管理器和隧道服务器部署在云上，所有边缘节点都部署有隧道代理和 YurtHub。

![](img/7cb0371a6b556aa6c1e06329c44bbfaf.png)

OpenYurt —隧道服务器和隧道代理

公共云主机上的隧道服务器服务作为节点端口公开，以实现与边缘节点上所有隧道代理的连接。

![](img/20f7d66e5d89d0364094086ff0727e8e.png)

open yurt-TunnelServer 服务-节点端口

显示节点信息的示例应用程序部署在一个边缘节点上。

![](img/6ed97b35e49c592fe12caf3fd7173b6a.png)

边缘节点上的示例应用程序

可以使用 Service:ClusterIP 从本地网络访问该服务，如下所示。

![](img/d64f844a69d8a05f49b6694694bf63bc.png)

由于在边缘节点上还没有安装 OpenYurt 组件(没有隧道代理)，当从 cloud master 访问时，像 exec 和 logs 这样的 Kubectl 操作会失败。

![](img/b0fd519edb06fae543f5b8d121d47af8.png)

失败—访问边缘节点上的应用程序

一旦节点被标注了 OpenYurt 所需的标签，隧道代理就被安装在边缘节点上。

![](img/d0ced2b6eff51a5a8dcb8342ada62ee1.png)

OpenYurt —隧道

云主机上的隧道服务器接受并批准来自验证证书身份的边缘节点上的隧道代理的隧道。用户可以为反向代理连接生成特定的证书，也可以使用 apiserver 和 kubelet 证书。

![](img/df02856ed4545b68f41ce48e73d0ebec.png)

OpenYurt —隧道服务器

一旦隧道代理成功连接到云主机上的隧道服务器，就可以从云主机上执行对边缘节点对象的所有 Kubectl 管理操作，其中边缘节点位于完全隔离的专用网络中。

![](img/c5302aff7da0a3a4dc9ec7c55800a146.png)

通过隧道访问边缘节点上的应用

边缘节点部署有 YurtHub 组件，该组件持续缓存来自云主机的集群状态信息。所有从库伯莱到 KAS 的通信都将通过该枢纽转发。

![](img/856e2924cfe47172e508b77b1eef1703.png)

边缘节点上的 YurtHub

云主机上的 yurt-controller-manager 维护已注册节点的列表，还通过 OpenYurt CRD 管理与自治和应用程序管理相关的其他功能。

![](img/7d7c568ae3752227b58edbb89df8df82.png)

云主机上的蒙古包-控制器-管理器

OpenYurt 在集群中安装两个定制资源:NodePool 和 UnitedDeployment，用于应用程序部署和管理。

![](img/25bbc3a9c4113c224810824e82d2e9ec.png)

CRD—open yurt 应用管理

创建节点池资源，其中节点池使用标签“apps-pool-r”对节点进行分组。

![](img/4ef25016b80518493af175f0d81a7179.png)

OpenYurt —节点池

要分组到节点池下的节点可以通过标记节点添加到池中。

![](img/9b9f5e5b0886e71f37415076643f45a4.png)

OpenYurt —节点池—节点标签

节点池 CR 的状态部分显示了属于池的所有节点，如下所示。在示例配置中，节点池仅包括边缘节点 1 和边缘节点 3。

![](img/60ab6e68efd08bb04d5b194f4f4d35af.png)

OpenYurt —节点池

云主机上的 yurt-app-manager 维护集群中创建的所有节点池和联合部署的信息，并管理资源。

![](img/47454c7811a9fd6b3a8399d1acc49193.png)

OpenYurt —云主机上的 Yurt-App-Manager

使用 workloadTemplate 创建 UnitedDeployment 资源，以便在上面创建的节点池上部署一个示例 Nginx 应用程序，如下所示。

![](img/baaaf732f92cbd0fcf6c829b46d0049e.png)

OpenYurt —统一部署

![](img/a5b887c5807c01dffb69ee93640e873a.png)

OpenYurt —统一部署控制器

UD 创建所需的 Kubernetes 部署，并将 pod 放置在节点上，这些节点是上面节点池的一部分。

![](img/1f04ac6ba862620cdc3900f9fc024fb3.png)

OpenYurt —统一部署

如下所示，pod(副本:3)仅放置在边缘节点-1 和边缘节点-3 上，它们是“*节点池:应用池-r* ”的一部分。

![](img/f32d21126f68dbd5fd4da3d91276aed5.png)

OpenYurt —统一部署

UD 资源创建的所有部署都可以使用 UD 进行集中管理，例如编辑 UD 对象中的副本数量和映像，如下所示。这同样适用于由 UD 创建的部署的所有吊舱部分。

![](img/195fc8db25f936fc98d6386ab0c62f19.png)

OpenYurt —统一部署

网络中断场景是通过断开边缘节点 3 来创建的，如下所示，当节点转换到未就绪状态时，应用荚没有被驱逐，边缘节点中的 YurtHub 组件继续提供必要的网络信息，使得服务对于本地网络中的应用/用户可用。

![](img/bb0a0511e67d9188841d7c7bd4cbc189.png)

OpenYurt —边缘节点自治

# 超边缘

云边和云边上的 SuperEdge 的组件如下图所示。

![](img/9d04295c3f32860ebddef5ace7e73421.png)

超边缘——建筑

可以使用 [edgeadm](https://github.com/superedge/superedge/blob/main/docs/installation/install_via_edgeadm.md) CLI 提供 SuperEdge(先决条件是使用 Kubeadm 引导集群),或者用户可以手动生成所需的证书并使用清单部署组件。

![](img/4c79ffedce677b95c5ea180d546007b0.png)

超边—组件

SuperEdge 在所有边缘节点上部署 tunnel-edge，在云主节点上部署 tunnel-cloud。

![](img/1dce073f8250610bb58efb2aad912d23.png)

超级边缘—隧道边缘和隧道云

安全连接将用于隧道、tunnel-edge-conf 和 tunnel-cloud-conf 配置映射保存配置和证书信息。

![](img/5e0077d3cc9b7f7b6fd64e4cc15325b1.png)

超级边缘—隧道边缘和隧道云

Kubelet 和 apiserver 针对隧道边缘和隧道云的认证:

![](img/67592a222abe6a6801b6e1e847e7c8bb.png)

TunnelEdge 和 TunnelCloud 证书

其他配置映射包括 tunnel-coredns 配置，它用于将代理从 kube-apiserver 隧道传输到边缘节点。隧道节点配置包括所有注册的边缘节点，隧道云使用这些节点来跟踪和管理隧道边缘站点的连接性。

![](img/14a7c4dc58c64f6ee060f608f778bc32.png)

超边缘-隧道-核心 dns

当隧道建立后，部署在边缘节点上的应用程序可以使用 Kubectl 操作进行管理，如下所示。

![](img/51f22158935b3f2ee6632c5ed15ee8eb.png)

超边缘—访问边缘节点上的应用程序

SuperEdge 安装了两个定制资源:用于应用程序部署和管理的 DeploymentGrid 和 ServiceGrid。这两种资源都是命名空间范围的。

![](img/bffceee5de0483778edcf77c1c346cb6.png)

超级 CRD 的应用管理

为了测试应用程序管理功能，节点 1 和节点 2 标有“*地区 1:美国西部*”，节点 3 标有“*地区 1:美国东部*”。

![](img/5d67df374aaa948ca5a9c100d46ffd41.png)

超边-节点标签

示例DeploymentGrid 资源中的 *gridUniqKey* 被映射到上面节点标签中使用的键“region1”。

![](img/d18c6edf023a5319ea8c5734ff3ee651.png)

超级边缘—部署网格

DeploymentGrid 创建一个 Kubernetes 部署，每个区域有两个副本，如下所示。

![](img/7fd6b12a83bfadccf30e1877803a6917.png)

超级边缘—部署网格

使用相同的 *gridUniqKey:region1* 创建一个示例 ServiceGrid 资源，如下所示。

![](img/bf2151599bc81b5ce2ac22d6e3e7ea9e.png)

超级边缘—服务网格

ServiceGrid 创建了一个 Kubernetes 服务对象，它映射了在两个区域(3 个节点)中运行的所有四个 pods(端点)。

![](img/9760aa09ee5d7a00b94e773fe35fa8ce.png)

超级边缘—服务网格

当使用来自边缘节点 1 或边缘节点 2 的集群 IP 访问服务时，请求仅由“区域:美国西部”中的 pod(节点组中节点 1 和节点 2 上的 pod)提供服务，类似地，来自边缘节点 3 的流量将仅由“区域:美国东部”中的 pod 提供服务。

![](img/5fba115349445d0bab14823cc7955aba.png)

超级边缘—服务网格

网络中断场景是通过断开边缘节点 2 而创建的，如下所示，当节点转换到 NotReady 状态时，应用荚没有被驱逐，边缘节点中的 lite-apiserver 组件继续提供必要的网络信息，使得服务对于本地网络中的应用/用户可用。

![](img/aaf38b3cef9cad7a28e732290c267b27.png)

超边——边节点自治

默认情况下，edge-health 在每个边缘节点上运行，以准确确定边缘节点的实际运行状态和边缘准入:通过提供来自分布在所有边缘工作节点上的边缘健康服务的实时健康检查状态来帮助 Kubernetes 控制器识别节点的真实状态。

![](img/754fb329df4f32813daf17432bed8341.png)

超边——边节点自治

[**【KubeEdge】**](https://gokulchandrapr.medium.com/kubeedge-extending-kubernetes-to-edge-dcfedd91f5f9)**，**最初来自华为——CNCF 主持的第一个基于 Kubernetes 的边缘计算项目，与上述项目也有重叠的特点，主要区别在于这些项目是 Kubernetes 原生的，不会侵入 Kubernetes 源代码，所以理论上不需要考虑与 Kubernetes 版本和生态的兼容性。除了特定于边缘的功能，这些项目还提供了边缘自治和拓扑感知的应用管理功能。

Gartner 预测，到 2025 年，超过 75%的企业生成数据将在数据中心或云之外创建和处理。这些项目可以极大地解决大量的 edge 用例，并支持 Kubernetes 在 edge 中的使用。这些项目仍处于开发模式，来自社区的贡献可以极大地提高采用和使用。
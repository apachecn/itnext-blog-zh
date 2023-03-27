# Anthos 的入口—多集群入口和全局服务负载平衡

> 原文：<https://itnext.io/ingress-for-anthos-multi-cluster-ingress-and-global-service-load-balancing-c56c57b97e82?source=collection_archive---------1----------------------->

![](img/e432484d75668f59c58437b47f4364c4.png)

地理感知多集群入口 Anthos 的入口

Anthos 的入口是一个谷歌云托管的多集群入口控制器，用于 Anthos GKE 集群。Ingress for Anthos 支持跨集群和跨区域部署共享负载平衡资源，使用户能够为运行在多集群和多区域拓扑中的应用程序使用带有任播 IP 的同一负载平衡器。

简而言之，这允许用户将位于不同区域的多个 GKE 集群置于一个负载平衡器之下。它是外部 HTTP(S)负载平衡器的控制器，通过使用[网络端点组](https://cloud.google.com/load-balancing/docs/negs) (NEGs)对外部 HTTP(S)负载平衡器进行编程，为来自互联网的流量提供跨一个或多个集群的入口。neg 对于容器本地负载平衡很有用，其中每个容器都可以表示为负载平衡器的端点。

利用 GCP 的 100 多个存在点和全球网络，Ingress for Anthos 利用 GCLB 以及在世界各地运行的多个 Kubernetes 引擎集群，使用单个任播 IP 地址为来自最近集群的流量提供服务。

![](img/7144c70d08ec68f9a241097568e9c58d.png)

基于地理感知的邻近选播路由

# 利用任播实现全局负载平衡

Ingress for Anthos 在[高级层](https://cloud.google.com/network-tiers/docs/overview)中创建了一个外部 HTTP(S)负载平衡器，它使用一个作为任播 IP 公布的全球外部 IP 地址，并可以根据邻近程度智能地将用户的请求路由到最近的后端实例组或 NEG(网元组)。与使用基于 DNS 的负载平衡的多个地址相比，专用任播地址意味着任何地方的客户端都可以连接到同一个 IP 地址，同时仍然可以尽快进入谷歌的网络，并连接到谷歌网络边缘的负载平衡器，流量进入该网络，从而最大限度地缩短客户端和前线负载平衡器之间的网络距离。

任播支持使用相同的 IP 地址，并且在世界各地不同的地理区域请求具有相同内容的后端提供了可能的最短响应时间。任播根据边界网关协议(BGP)路径将数据包定向到地理上最近的后端。例如，如果用户在北美、欧洲和亚洲设置了实例组/neg，并将它们连接到负载平衡器的后端服务，则世界各地的用户请求会自动发送到离用户最近的虚拟机/pod，前提是这些虚拟机/pod 通过了运行状况检查并具有足够的容量(由平衡模式定义)。

任播与 BGP 协议相关联，BGP 协议确保路由器的所有邻居都知道可以通过该路由器到达的网络以及到这些网络的拓扑距离。有了任播，系统每次都会一致地选择最短路径。如果一个节点出现故障，将确定下一个最短的路由，并在无需更改 IP 地址的情况下重定向流量。

![](img/ef1be963e983c1e312bf6e0e24793622.png)

任播路由

任播提高了吞吐量，最大限度地减少了世界各地客户的延迟。此外，如果添加了[云 CDN](https://cloud.google.com/cdn/docs/) ，则可以在这些边缘位置启用缓存。全球任播 IP 地址使用户能够无缝地更改或添加用于部署应用实例的区域，并根据需要增加容量。除了延迟优化之外，这种方法的另一个最大优势是高可用性，如果最近的后端都不正常，或者如果最近的实例组/NEG 达到容量，而另一个实例组/NEG 没有达到容量，负载平衡器会自动将请求发送到下一个具有容量的最近区域。

# 冷土豆路由到谷歌流行

来自客户的请求被路由到[冷土豆](https://wikipedia.org/wiki/Hot-potato_and_cold-potato_routing)谷歌 PoP，这意味着互联网流量去最近的 PoP，并尽快到达谷歌主干。

冷土豆技术与热土豆(流量被发送到最近的交换点的对等点)相反，在使用内部网络/主干将数据包传送到对等点之前，尽可能长时间地传送客户的流量。虽然 Cold Potato 增加了总体运营成本，但它提供了一些优势，如使流量更长时间地处于网络管理员的控制之下，使用自定义软件堆栈应用复杂的规则，允许配置良好的网络的运营商向他们的客户提供更高质量的服务。

![](img/abf6c70cffdefc45106436f61a4a8833.png)

冷土豆 vs 热土豆路由

谷歌 PoPs 或 GFE 的([谷歌前端](https://cloud.google.com/security/security-design#google_front_end_service))是软件定义的分布式系统，位于谷歌存在点(PoPs)，与其他系统和控制平面一起执行全局负载平衡。任何选择对外发布自己的内部服务都使用 GFE 作为智能反向代理前端。该前端提供其公共 DNS 名称的公共 IP 托管、拒绝服务(DoS)保护和 TLS 终止。

# 具有 NEG(网络端点组)的容器本地负载平衡

NEG 是 Google 云负载平衡器使用的 IP 地址列表，NEG 中的 IP 地址可以是虚拟机的主要或次要 IP 地址，这意味着它们可以是 Pod IPs。在 GKE VPC 本地集群中(Pod IP 地址可在集群的 VPC 网络和通过 VPC 网络对等连接到它的其他 VPC 网络中本地路由)，GCP VPC 网络知道别名 IP，因此路由由 VPC 负责，无需任何额外的路由配置。

neg 对于容器本地负载平衡很有用，其中每个容器都可以表示为负载平衡器的端点。这实现了容器本地负载平衡，将流量从 Google Cloud 负载平衡器直接发送到 Pods。

在旧的 GKE 负载平衡方法中，传统的实例组曾经是后端，它们组成了工作负载运行的实际 Kubernetes 节点。

![](img/5aec6b7f405579aaa864757b9b3fa6e6.png)

负载平衡器—后端—实例组

具有 Kubernetes 节点的实例组:

![](img/91cf1bccc73b049de1688ce0bf4fbcd7.png)

实例组— Kubernetes 节点

用户可以使用一个名为 [kubemci](https://github.com/GoogleCloudPlatform/k8s-multicluster-ingress) 的工具(一个配置 Kubernetes ingress 以跨多个 Kubernetes 集群负载平衡流量的工具)来创建一个跨集群分布的负载平衡器，并关联一个任播 IP。

![](img/cbc82ee265501e9e13991d2dc26ad674.png)

带任播 IP 的 LB

这种方法现在已被否决，而 Ingress for Anthos 是部署多集群 Ingress 的推荐方法。

如果没有容器本地负载平衡，负载平衡器会将数据包分发到每个节点(Kubernetes 节点),每个节点中的 iptables 会将数据包进一步分发到每个 pod/容器，这增加了额外的跳数。使用容器本地负载平衡，优势是更好的负载分布、减少额外的跳数以减少延迟和更好的健康检查。

Anthos 的 Ingress 是一个云托管的多集群 ingress 控制器，它使用[网络端点组](https://cloud.google.com/load-balancing/docs/negs) (NEGs)对外部 HTTP(S)负载平衡器进行编程。这项服务由 Google 托管，支持跨集群和跨地区部署共享负载平衡资源。控制器部署计算引擎负载平衡器资源，并在用户创建 MultiClusterIngress 资源时跨集群将适当的 pod 配置为后端。neg 用于动态跟踪 Pod 端点，因此 Google 负载平衡器拥有一组健康的后端。

![](img/1c86f73d5fdb3063f6f7349bedeeb508.png)

具有 NEGs 的容器本地负载平衡

在 GKE 上，可以通过向 Kubernetes 服务(*cloud.google.com/neg*)添加注释来自动创建和管理 NEGs。当 NEGs 与 Anthos Ingress 一起使用时，Ingress 控制器有助于创建 L7 负载平衡器的所有方面。这包括创建虚拟 IP 地址、转发规则、健康检查、防火墙规则等。

# 试验

在三个不同的地区创建了三个 GKE 集群，并且所有这三个集群都注册到了 Anthos Hub。us-west2-a 中的集群被选为配置集群，配置集群是多集群资源——multiclustergress 和 MultiClusterService——的集中控制点。这些多集群资源存在于单个逻辑 API 中，并且可以从单个逻辑 API 进行访问，以保持所有集群之间的一致性。入口控制器监视配置集群并协调负载平衡基础设施。

![](img/a651897b98288ced49167f77ee004b70.png)

多集群和多区域拓扑

入口控制器是一个全局分布式控制平面，作为集群外部的服务运行。Environ 提供了一种统一的方式来查看和管理多个集群及其工作负载，作为 Anthos 的一部分。集群使用项目的 environ 通过 Connect 向 Anthos 注册，ingress 使用 environ 的概念来说明如何跨不同的集群应用 Ingress。注册到环境的集群对 Ingress 变得可见，因此它们可以用作 Ingress 的后端。注册成为中心成员的 GKE 集群成为概念上被称为 T2 环境 T3 的一部分。

注册到 environ 的集群称为成员集群，在上面的拓扑中，所有三个集群都是成员集群(配置集群可以用集群选择器隔离)。环境中的成员集群包括 Ingress 知道的所有后端。

MCI(multiclusteringress)和 MCS (MultiClusterService)是自定义资源(CRD ),它们是入口和服务资源的多群集等价物。这些是在配置集群上配置的，MCS 在成员集群中创建所需的派生服务(映射到端点的实际服务)。

![](img/fa32b70f29aa93d2d203422f80c6b4a1.png)

多群集回归和多群集服务

MultiClusterIngress 资源包含后端服务列表(与此 MCI 部署在同一命名空间和配置群集中的 MCS 的名称)。MultiClusterIngress (MCI)资源在许多方面的行为与核心入口资源相同。两者在定义主机、路径、协议终端和后端方面都有相同的规范。

多集群服务(MCS)是 Ingress for Anthos 使用的自定义资源，是跨多个集群的服务的逻辑表示。MCS 与核心服务类型相似，但本质上不同。MCS 仅存在于配置集群中，并在目标集群中生成派生服务。MCS 不像 ClusterIP、LoadBalancer 或 NodePort 服务那样路由任何东西。

![](img/77c7f558753229b0d953c28dcf642670.png)

多群集回归和多群集服务

一个 MCI 资源可以有一个默认后端(MCS)和指定多个后端的规则列表。MCI 通过将流量发送到后端中指定的 MCS 资源，将流量与规则中指定的主机上的 VIP 进行匹配，所有其他不匹配的流量将被发送到默认后端 MCS。

用户可以在 MCS 配置上使用“*集群*有选择地应用入口规则，其中派生服务仅在指定列表上创建。如果未指定 MultiClusterService 的“clusters”部分，或者未列出任何集群，则它将被解释为默认的“all”集群。该特征提供了如下优点:隔离配置集群以防止 MCS 在它们之间进行选择，路由到仅存在于集群/区域的子集内的应用后端，等等。，以有助于应用迁移的蓝绿色方式控制集群之间的流量，便于不同集群使用单个 L7 VIP。

![](img/7d2bcb4ef13615dfbf4fd054cd3280dc.png)

多集群入口和多集群服务—集群选择器和入口规则

以下是跨多个集群为 Anthos 配置入口的工作流中的步骤:

![](img/d02a5ebc5cfac4ca31cc2fa1ca2f272e.png)

工作流—为 Anthos 配置入口

应在项目中启用多集群入口 API:

![](img/d619451e4e79d45d6e7e5138f978eb15.png)

多聚类回归 API

如下所示，在不同的区域创建了三个 GKE 集群，并将其添加到中心。选择 Cluster1 作为配置集群，并在集线器级别启用多集群入口功能。

![](img/e0369527ebddfec4ac9775987ecf1106.png)

启用多集群入口功能

使用 Connect 添加到集线器的所有集群，因为所有这些集群都是 GKE 管理的集群，类型将它们指定为 GKE:

![](img/791403002c5e926eadc84c7661f13aa8.png)

集线器中的集群—使用 Connect 注册

从 Anthos 门户启用入口功能:

![](img/ced21795c59edcf6d8f8e05649801570.png)

启用多集群入口功能— Anthos 门户

显示配置集群信息和所有其他已注册集群(成员)的入口详细信息页面:

![](img/3b7d5a974209460b3e4644516c6b285f.png)

集群成员和配置成员

在所有三个集群上都部署了一个演示应用程序“zone-ingress ”,该应用程序在被访问时只打印运行它的数据中心的名称。

在跨集群部署应用程序时，用户必须考虑一个称为“ [*名称空间一致性*](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-for-anthos#namespace_sameness) *”、*的概念，这是环境所具有的一个特征。这假设在集群中具有相同名称和相同命名空间的资源被认为是同一资源的实例。实际上，这意味着从 Anthos 入口的角度来看，跨不同集群的标签为“app: zoneprinter”的“zoneprinter”命名空间中的 pod 都被视为同一个应用程序后端池的一部分。这对不同的开发团队如何在一组集群中操作有影响。

在此场景中，名为 zone-ingress 的部署部署在“zoneprinter”命名空间中的所有三个集群上:

![](img/0a542606ab2528b3bbd75f68050e8d3e.png)

部署在所有集群上的演示应用程序

MCI (MuliClusterIngress)和 MCS (MultiClusterService)在配置群集上创建:

![](img/77c13db3bcf9b7b19666aa6db46f8d7f.png)

在配置群集中创建了 MultiClusterIngress 和 MultiClusterService

![](img/0c3edc60eadeb48b16c7cf768f026220.png)

在配置群集中创建了 MultiClusterIngress 和 MultiClusterService

由于我们没有在 MCS 配置中提供任何“集群”,因此将在包括配置集群在内的所有三个集群上创建一个派生服务:

![](img/de6209d7d78aac056d7707cdd067a88d.png)

MCS 在所有成员集群上创建的派生服务

在成员集群(本例中为所有三个集群)中创建的派生服务使用 NEGs(各自网元组的映射)、multiclusterservice-parent(在配置集群上创建的 MCS)和端口信息进行注释，如下所示:

![](img/7bea1d2444833d023c609d30a0e433a4.png)

MCS 在成员群集上创建的派生服务

这一步在计算引擎中创建 NEGs，开始注册和管理服务端点。

![](img/4dafa6c644d6f44eb902a6fe8bca47de.png)

负附件

控制器会自动创建负号。neg 映射所有三个集群中的“区域入口”pod:

![](img/5c79325986031c0c5d1d848782aaa156.png)

网络元素组

如下所示，NEG 中的网络端点映射到集群 1 上的*区域入口*的 pod_ip:

![](img/0b19db49a95848a18f9ee5988c9ae190.png)

指向 Pod_IP 的网络元素组

![](img/843696deeb698c5672a575e4a104b577.png)

指向 Pod_IP 的网络元素组

正在配置群集上创建 MCI 资源:

![](img/7f550629006f5d0774cb67bf943f8528.png)

正在配置群集上创建 MCI 资源

此步骤部署计算引擎外部负载平衡器资源，并通过单个负载平衡器 VIP 跨集群公开端点。

MultiClusterIngress 资源“状态”字段显示后端服务:MCS、防火墙、转发规则、健康检查和单个群集的 neg。VIP 是映射到负载平衡器的任播 IPV4 地址。

![](img/9a72451ae539cdaba420c160f40144cb.png)

具有后端服务、防火墙、转发规则、健康检查和否定的 MCI 资源

负载平衡器后端将 NEGs 映射为后端，这是在创建 MCI 对象时由入口控制器自动创建的:

![](img/9acf0283259f49d9c97003f6d2b942b1.png)

以负数作为后端的负载平衡器

neg 被配置为负载平衡器的后端。

![](img/9e2d0e62132047182c08bf7df521e785.png)

以负数作为后端的负载平衡器

前端映射到单个 IPV4 任播地址。

![](img/7c37fc6b48b6ade4775f37300cfa3178.png)

具有任播 IPV4 地址的负载平衡器

转发规则被配置为将流量从 LB 发送到入口资源。

![](img/9edc34185b83ed169cf9bd5c44f8665a.png)

转发规则

# 从不同地理区域访问演示服务(Zoneprinter)

在 GKE 集群所处的各个分区中创建了三个虚拟机实例，以复制来自不同区域的访问。

![](img/37af69b5d04f10da1d0655d884770530.png)

测试实例—测试来自不同区域的访问

从 US-West 使用 LB 的任播 IP 访问 Zoneprinter，从集群 1-US-West 2-a 中运行的 Zoneprinter pod 返回响应。

![](img/9d4f77b29bc14ee6a390e6aeff4c9ea5.png)

从美国西部进入

从 US-East 使用 LB 的任播 IP 访问 Zoneprinter，从在 cluster 2-US-East 4-a 中运行的 Zoneprinter pod 返回响应。

![](img/9f2cb1f3076f8b945d07bac5fdb1c189.png)

从美国东部进入

使用来自 EU-西部的 LB 的任播 IP 访问 Zoneprinter，返回来自在集群 3-欧洲-西部 2-b 中运行的 Zoneprinter pod 的响应

![](img/ba2e8deb996c00b9e3bc55610f0b4e47.png)

从 EU-西进入

使用网页性能工具访问应用程序显示，美国和欧洲的响应时间相同。

来自弗吉尼亚:

![](img/feece0c88ce02271108f708198eb849b.png)

从弗吉尼亚访问(美国东部)

来自伦敦:

![](img/0b8641395d191b444dee3246135dd9ea.png)

从伦敦访问(欧盟西部)

比较 LB 的任播 IP 和在传统 EIP(弹性 IP)上运行的应用程序的 *traceroute* 结果，显示出响应时间和下一跳数的巨大差异。

从不同区域到 LB 的任播 IP 的*跟踪路由*显示出最小的跳数和较低的响应时间。这基本上证明了基于邻近性的路由选择在最近的集群上运行的应用。

![](img/841e9c6818c2f3fa4502d6e4ab0a68b2.png)

Traceroute —来自不同地区的任播 IP

从纽约到伦敦 EIP 上的应用程序的 *traceroute* 显示了跳数和响应时间的显著增加。

![](img/bd784de5e6cca71498cfbd6c2adaf91f.png)

Traceroute —从纽约到伦敦的 EIP

# 监控和指标

负载平衡器详细信息部分提供后端运行状况、利用率和请求指标。

![](img/a60278da67f85efb490206f155797afd.png)

LB —监控

显示请求、流量和服务后端的地图。

![](img/7901c38f3f51b58245c4c68c87888a0f.png)

LB —监控

# 其他功能

Anthos 的 Ingress 支持其他功能，例如:

*   支持 HTTPS:库本内特家族[的秘密](https://cloud.google.com/kubernetes-engine/docs/concepts/secret)支持 HTTPS。在启用 HTTPS 支持之前，用户必须创建一个静态 IP 地址。这个静态 IP 允许 HTTP 和 HTTPS 共享同一个 IP 地址。可以使用 MCI 资源中的“tls”参数来提供该秘密。
*   后端配置支持:[后端配置 CRD](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-for-anthos#backendconfig_support) 允许您定制计算引擎后端服务资源上的设置。

# 有云甲的晶片

谷歌云盔甲部署在谷歌网络的边缘，并与全球负载平衡基础设施紧密耦合。Cloud Armor 可以使用 Google Cloud 提供的自定义或预配置规则来实现基于地理的访问控制、预配置 WAF 规则和自定义 L7 过滤策略。Google Cloud Armor 提供了预配置的复杂 web 应用防火墙(WAF)规则，这些规则带有几十个根据开源行业标准编译的*签名*。

用户可以制定安全和 SSL 策略，这些策略可以应用于目标，这里的目标是负载平衡器。

![](img/d2946d70d880af03e69767cbf6676725.png)

云装甲—安全策略

![](img/85c34353ad8dd59c5d472b9fa017e04f.png)

云甲——增加 LB 作为目标

![](img/22f85938b323ca9bb72564068c0e0b33.png)

云装甲— SSL 策略

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

除了基于邻近度的路由，多集群入口还简化了其他操作，如蓝/绿集群升级、灾难恢复、基于复杂健康检查的自动故障转移、在 Kubernetes 上运行高可用性(HA)应用、降低 DDoS 攻击风险等。并且具有许多其他优势，所有这些优势都有可能转化为显著的竞争优势。
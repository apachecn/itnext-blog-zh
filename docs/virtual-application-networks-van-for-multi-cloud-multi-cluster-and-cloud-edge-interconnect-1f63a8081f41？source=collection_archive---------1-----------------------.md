# 面向多云、多集群和云边缘互连的虚拟应用网络(VAN)

> 原文：<https://itnext.io/virtual-application-networks-van-for-multi-cloud-multi-cluster-and-cloud-edge-interconnect-1f63a8081f41?source=collection_archive---------1----------------------->

![](img/2961ddc1c91f380efe0223dfdebd184d.png)

虚拟应用程序网络—连接跨云、集群和网络的分布式应用程序

现代软件开发正从客户机/服务器向服务无处不在的更灵活的架构发展。在多个位置部署或复制应用有多种原因:地理上分散的应用用于增强性能和可用性、维护合规性、互联车辆、5G 的本地突破、远程边缘站点等。这种需求使得面向应用的多云和多集群连接成为云计算的必然趋势。

VPN 技术，如 OpenVPN、Strongswan、Wireguard 等。被广泛用于在站点之间建立安全的站点到站点的连接，尽管 VPN 在促进远程站点之间的连接方面是一致的，但也存在一些问题和复杂性。一些重要注意事项:VPN 会混淆网络路径，增加网络延迟，通过 VPN 隧道监控网络路径是一项挑战，随着新站点的增加，要管理的站点到站点连接的数量呈指数级增长—扩展站点变得不切实际，云原生应用程序基于高级调度框架更改拓扑，连接必须自动检测拓扑，最后，VPN 的配置和管理非常复杂。

虚拟应用网络(VAN)可在云原生应用环境中实现分布式软件组件之间的高级通信功能，而无需处理复杂的第 3 层网络控制，如 VPN、防火墙规则和访问策略。

![](img/42733781351eb3ac9b7bde4a4f7401e1.png)

虚拟应用网络

VAN 是覆盖在基于 TCP/IP 的互联网上的应用层网络。货车可以小到一台笔记本电脑，也可以大到跨越时区和大陆的全球网络。一个 VAN 可以由一个没有特权的开发人员快速地建立起来，它的拓扑结构可以很容易地添加和修改。使用现有协议(如 HTTP、gRPC、AMQP 或任何运行在 TCP 或 UDP 上的协议)编写的应用程序无需修改就可以在 VAN 上运行。VAN 寻址使用任意字符串直接指向端点，而不依赖于底层主机和端口信息。

Skupper 项目是为 Kubernetes 构建的一个免费的开源实现。Skupper 可以通过简单的配置实现不同集群中运行的应用程序之间的连接，而不必担心特权提升、集群访问或管理员访问(用户只需要访问他被允许使用的特定名称空间)。Skupper 可以满足各种各样的用例，其中一些如下所示。

![](img/8262777a12ac286b9525d65ef3ee6eb8.png)

用例—货车

**VAN 地址是一个 DNS 风格的名称，引用一个进程**

传统的 TCP/IP 寻址使用“*主机:端口*对来指代端点，传统的 TCP/IP 互联网在将数据从一个点移动到另一个点时快速可靠，但它不太适合支持云原生软件系统的现代需求。在容器和无服务器计算的现代环境中，端口主机的概念增加了额外的开销，并且无法扩展。在提供容器的多个开发人员或团队之间协调端口分配是非常困难的，并且会使用户面临超出他们控制的集群级问题。

VAN 寻址遵循 DNS 风格命名，其中地址是一个名称。VAN 根据 VAN 地址(简称为“名称”)路由应用流量，而不考虑托管应用的系统所使用的底层 IP 地址。VAN 地址引用正在运行的进程，而 IP 地址引用运行该进程的主机。

![](img/b508e8fd3376ab13d610c2a435aab9a6.png)

TCP/IP 与 VAN 寻址

**货车地址是多路访问的**

IP 寻址主要是单播，每个地址代表一台主机。IP 确实有多播寻址的概念，其中 IPV4 和 IPV6 支持多播，IPV6 支持任播，但是在局域网范围之外的实践中并不使用。VAN 寻址是*任播*或*组播*。它建立了这样一种思想，即多个软件组件可以使用相同的地址连接到网络上，以实现多播传输或负载平衡。VAN 多播和任播不受组件位置的限制，因此提供网络范围的多播传递和任播负载平衡。

**安全的站点到站点连接**

在 VAN 上运行的应用程序受益于网络固有的安全机制。VAN 中的所有站点间连接都通过专用的证书颁发机构使用相互 TLS(传输层安全性)锁定。来自外部世界的访问只发生在开发者希望提供对门户或 web 前端的公共访问的地方。服务不会意外地暴露在互联网上。

# VAN 路由器— Apache Qpid 调度路由器

通过在将成为分布式应用命名空间的一部分的每个站点中部署一个 VAN 路由器(调度路由器),并通过在成对的站点之间建立安全连接，来建立一个 VAN。一个站点可以是数据中心中的一个 LAN、一台主机或 Kubernetes 集群中的一个名称空间。

![](img/69a537e45df623f270e9cdcd4a21456e.png)

VAN 路由器连通性

Skupper 使用 [Apache Qpid 调度路由器](http://qpid.apache.org/components/dispatch-router/)作为它的 VAN 路由器。该路由器基于 AMQP 协议，是轻量级和无状态的。除了基本路由器之外，Skupper 还提供工具来简化网络设置，并提供从 HTTP、gRPC 和 TCP 等常用协议到 VAN 网络的映射。

![](img/d3f894f5c4b41924a04ae2a9afa5bba6.png)

Apache Qpid 调度路由器

调度路由器是一个轻量级的 AMQP 消息路由器，用于构建可扩展的、可用的和高性能的消息网络。调度路由器灵活地在任何支持 AMQP 的端点之间路由消息，包括客户端、服务器和消息代理。AMQP 除了是一个开放的标准协议，允许系统之间的消息传递互操作性之外，它还是一个通用的网络协议，在服务和 API 交付方面比 HTTP 有真正的优势。

![](img/c1eec57d8bdf846a034c983768f72d26.png)

计算最低成本路径-LSRP

路由器网络使用类似于 OSPF 或 IS-IS 的链路状态路由协议来检测拓扑，并计算每对站点之间的最低成本路径。路由协议会快速自动地对拓扑结构的变化做出反应。这用于提供对故障的恢复能力，并且还使得在需要扩展或改变时从网络中添加位置和移除位置变得容易。

在下面的拓扑示例中，专用网络 A 和专用网络 B 是具有两个不同专用网络的两个不同站点。客户端(C1)位于专用网络 B 中，后端(B2)位于专用网络 A 中，通过调度路由客户端 C1 可以使用专用网络 B 中的路由器和公共网络之间建立的连接来访问专用网络 A 中的队列、主题和其他 AMQP 服务。

![](img/6ec152166793d6af39c467c3af9b00d2.png)

Qpid 调度路由器连接

例如，路由器 Ra 和 Rp 可以被配置为将代理 B2 和代理 B1 中的队列暴露给网络。然后，客户端 C1 可以建立到本地路由器 Rb 的连接，打开到“b1.event-queue”和“b2.event-queue”的订阅链接，并接收存储在代理 B1 & B2 中的那个队列上的消息。

在这种情况下，B1、B2 和 C1 都没有以任何方式被修改，也不需要知道它们之间存在消息路由器网络的事实。

# Skupper —组件

Skupper 的控制平面组件包括服务控制器和路由器。这两个组件都被部署为感兴趣的名称空间中的部署。路由器组件是实际的 Apache Qpid 调度路由器，服务控制器促进了公开服务的同步。

![](img/49df5ba4c875be2a8fb38a4febabbdf6.png)

Skupper —组件

该路由器有助于跨站点连接到其他路由器，并依赖于网桥配置(配置连接器时将被编程的 configmap)来检索任何已配置的连接器。使用生成的令牌建立连接，并且使用“*set-context-current-namespace*”对令牌进行命名空间限定。

![](img/9fa4b4387177f02d917789463ea1650a.png)

Skupper 路由器

服务控制器跟踪公开服务的信息。名称空间中的 s cupper-services config map 在用户配置时用暴露服务的列表进行编程。

![](img/b3dc3e30133f7aa307cce6e3245436ce.png)

Skupper 服务控制器

所有与路由器相关的配置都作为配置映射提供给路由器部署。skupper-site 包括站点名称和其他全局配置参数。s kupper-内部和 s kupper-由控制器编程的服务当用户在远程站点公开可用的服务时，这可以使用 s kupper CLI(s kupper expose deployment)或向部署提供注释(*skupper.io/proxy 和*)来配置。

![](img/fb03f1bb003e012ff33d22c4043188c2.png)

Skupper 配置映射

![](img/afbaa6dbe5af2f6b4adbf5c45693eea8.png)

Skupper —站点配置

所有到 VAN 中其他路由器的站点间连接都使用 mutual TLS(传输层安全性)锁定，并提供一个私有的专用认证机构作为秘密。

![](img/33d097aac3085e156a8941f6582b73f0.png)

秘密

在公共云设置中，skupper-controller(路由器)服务被公开为具有公共连接的服务类型负载平衡器，因为私有站点对等需要它。在私有/边缘设置中，唯一的要求是具有出口互联网连接。

![](img/5ec607071eac7f68d74a0020fa9a0088.png)

Skupper —服务

skupper-controller 提供了两个控制台，一个是 Apache Qpid 调度控制台，另一个是 skupper 控制台(这两个控制台都可以用 skupper-site configmap 激活，或者用' *skupper init* '传递参数)。Apache Qpid 调度控制台提供与路由器和相关连接相关的大量信息，以及具体的监控和模式信息。

![](img/3efd72651558e64d447fab72c28e4a43.png)

Qpid —控制台

![](img/83197ddccfed07e2c65ce9d3f1e8fe71.png)

Qpid —控制台

“连接”部分提供了与网络中其它路由器建立的所有路由器-路由器连接、本地连接和基于 AMQP 的连接的列表。拓扑部分提供了网络中路由器和消息流信息的图形视图。

![](img/eb9c3f86a854a0f8fe87d32660082d1b.png)

Qpid —控制台

![](img/01c27da3584b0a1e88bcf409603c7f7d.png)

Qpid —控制台—拓扑

Skupper console 提供了关于连接站点的简单而丰富的信息，以及部署和服务的详细图形视图。

![](img/794b133ac8be83d052d87f6ea33ac181.png)

Skupper —控制台

# 尝试—在公共云和边缘站点之间创建一个具有私有连接的 VAN

下面的测试拓扑包括一个 GKE 集群、EKS 集群和两个使用 Kubeadm 引导的私有集群，Skupper 只需要一个 Kubernetes 集群，不依赖于平台/发行版或供应程序。

在下面的拓扑中，有四个跨云平台和网络的 K8 集群。GKE 和 EKS 包含 *skupper-controller* 作为负载均衡器公开，路由器彼此对等。另外两个专用集群(没有来自互联网的公共连接)与 GKE 集群上的路由器对等。专用集群可以位于不同的隔离网络中，在下面的示例中，尽管两个专用集群位于同一个网络中，但它们并不相互对等(模拟相互之间没有连接的远程站点)。

![](img/4201a0b7ec9d6752fb9f144c7116b800.png)

测试拓扑

# 1.连接 GKE 和 EKS

GKE 集群:

![](img/b9cfe3de33ae4f5e38293d386a0c8cd9.png)

GKE 集群

EKS 集群:

![](img/f382b353f99a17ceb9eda67f14ba49d1.png)

EKS 集群

GKE 的 Skupper 路由器和服务控制器:

![](img/7a841c33633bb48f576bcbc3adc9a472.png)

GKE 的 Skupper 组件

EKS 的 Skupper 路由器和服务控制器:

![](img/4452f9c76d9674c3ab2c5705e01d4cb6.png)

EKS 的 Skupper 组件

Skupper 控制器在 GKE 上显示为'*类型:外部负载平衡器'*:

![](img/5f678f8321523df37f92aed4b2a9b6e1.png)

Skupper 控制器服务 LB

使用命名空间范围的连接令牌对连接路由器进行身份验证。这可以使用 s kupper CLI(s kupper connection-token)或通过创建一个将产生连接令牌*的资源——‘skupper.io/type:连接令牌请求’*来生成。在这种情况下，在 GKE 上创建一个连接令牌，并在 EKS 用于建立连接。

![](img/f9be29d230e708111dd956deb229f758.png)

Skupper —连接令牌

一旦连接建立，基础设施 AMQP 消息流将在调度路由器之间开始。用户可以使用 QPID 控制台收集关于拓扑、日志和流量信息的各种信息。

EKS 凝视着 GKE:

![](img/83d18fa38b4a8ac4479d6c4489df8233.png)

Qpid 调度路由器—拓扑

![](img/b8293754d21940972c3ae4253844486b.png)

Qpid 调度路由器—控制台

EKS 和 GKE 之间的消息流可视化:

![](img/280f8160d21e4f246016f100ed33eba1.png)

Qpid 调度路由器—可视化

EKS 上的调度路由器连接到 GKE 上作为负载平衡器公开的 skupper-controller 服务:

![](img/8dc5cfe016642b26e1569b071d340bbd.png)

Qpid 调度路由器—控制台

具有连接站点信息的 Skupper 控制台门户:

![](img/587a4020f42331393abc6cadd725c0f5.png)

Skupper —控制台

# 2.连接边缘西部和边缘东部到 GKE

edge-west 和 edge-east 集群是在连接到家庭网络的 Udoo X86 上运行的私有集群。

如下所示，edge-west 路由器使用公共服务端点与 GKE 上的路由器对等。在这种情况下，GKE 连接令牌用于创建 edge-west 和 gkeuseast 之间的连接。

![](img/89baad51ac878717cde70ca0ed2de114.png)

公开服务—货车

为每个连接器创建 SSL 配置文件，边缘站点中的路由器与 GKE 上的路由器建立安全的相互 TLS 连接:

![](img/7d8992a0020d3714599ce34c3fc2b819.png)

公开服务—货车

连接站点的 Skupper 控制台视图:

![](img/9688ff5191679aeb0f96cbf06721f598.png)

Skupper 连接的站点拓扑

![](img/980a3defe04d7252c9404b8b27963c8e.png)

Skupper 连接的站点拓扑

![](img/000063615c2072294f4dc7cac3202f06.png)

Skupper 连接的站点拓扑

QPID 拓扑视图:

![](img/2477c8ae42f46b0f6093809dbb82b17c.png)

Qpid 路由器—连接的站点拓扑

![](img/cab5c0755ddb1ed4813a85005bab9a60.png)

Qpid 路由器—连接

Qpid 控制台—流量指标:

![](img/219204137724117b34a1629f8f4ff7dd.png)

Qpid 路由器—连接

类似地，第四个集群/站点 edge-east 也与 gkeuseast 对等，如下所示:

![](img/1ef34475ff33755b26d57dceabc714ce.png)

Skupper 连接的站点拓扑

![](img/0d1d3b5c6d44c5afb59e5b53a8082193.png)

Qpid 路由器—连接的站点拓扑

该拓扑现在在两个集群之间有一个 VAN 连接，具有到 Skupper 服务的公共连接，以及两个没有来自互联网的入口连接的私有集群。没有打开特定的端口，而是在路由器之间建立了 TLS 连接。

![](img/af21da8b9948be40b860407178bfb4f8.png)

Skupper —消息流监控

# 尝试—在分布于上述四个相连集群的单个 Hipster 服务之间建立 VAN 连接

[潮人商店](https://github.com/GoogleCloudPlatform/microservices-demo)由许多用不同语言编写的微服务组成，这些微服务通过 gRPC 相互对话。这个应用程序部署拓扑包括服务，这些服务是位于不同平台的不同集群上的不同名称空间中的应用程序(前端、后端和数据库)的一部分。从技术上来说，这意味着他们没有办法交流，除非他们暴露在公共互联网上。

![](img/b42fe2ff83dd4d9c833ba7c96d50e97f.png)

跨集群的样本 Hipster 应用程序

将 Skupper 引入每个名称空间有助于创建一个虚拟应用程序网络，该网络可以连接不同集群中的服务。应用程序网络上公开的任何服务在所有连接的名称空间中都表示为本地服务。当前端向所有其他后端实体发送请求时，Skupper 将请求转发到后端运行的名称空间，并将响应路由回前端。

![](img/a0a68b340d9be4b49dff9d27927495aa.png)

Skupper 路由器-服务交互

Skupper 使用户能够配置协议(tcp、http 或 http2。默认“tcp”)与端口一起用于通信。在上面的例子中，服务与数据库的连接是“tcp ”,因此用户可以通过用的和的标注服务对象来配置具体的协议

前端服务部署在 *gkeuseast* (GKE)集群上，没有必要向 Skupper 公开此服务，因为这是面向用户的，将使用 GKE 提供的负载平衡器 IP (public_ip)进行访问。

![](img/08fe642394c1fbe0c457faa4e05851ed.png)

Fronetend 应用程序— GKE

服务:购物车，结帐，货币和 redis-cart 部署在 EKS

![](img/6f00b7fd97a33ad8e116970e444bc5ee.png)

服务— EKS

服务:产品目录和建议部署在 edge-west 上

![](img/b2e8bc9e2432678f667f645c877ede50.png)

服务—边缘 1

服务:emailservice、paymentservice 和 shippingservice 部署在 edge-east 上

![](img/d9b8fe331bc7ac7d99e9a543baa47eff.png)

服务— Edge2

如下图所示，hipster 应用程序不起作用，因为不同集群中的其他服务还没有暴露给 Skupper。

![](img/c4efe9a8be68f02fe15a58c0725aa1a8.png)

潮人—非功能性应用

服务产品目录和建议在 edge-west 上向 VAN 公开。

![](img/9bdadd0fe30264ac8b1c9c4e3863af3e.png)

Skupper —公开服务

一旦服务被公开，服务就向 skupper-router 发送请求，然后这些请求被转发给网络中的其他路由器，如下所示:

![](img/7e222e67f50a8901bf19161ab6a491aa.png)

Skupper —公开服务

控制器添加 skupper-internal config map*bridges . JSON 和* skupper-services configmap 文件，其中包含特定路由器所在命名空间中服务的连接器、相应地址、协议和端口信息，skupper-router 和服务控制器将使用这些信息来配置连接。

![](img/fd1898c11f8d1556a5039fc073f49776.png)

Skupper —公开服务

![](img/ca4612881a30cad380cf1fcec15e9cb0.png)

Skupper —公开服务

Skupper 路由器公开带有端口和站点 ID 信息(路由器 id)的服务:

![](img/334bd9177dd16284077d225879d480be.png)

Skupper —公开服务

Skupper 服务控制器维护所有公开服务的信息:

![](img/bc55e6ee190e1029386d0fb4f74ad0f1.png)

Skupper —公开服务

服务:广告、购物车、收银台、货币都在 EKS 集群上展示。

![](img/0fd7542602ff78087a2810c4c900fea4.png)

Skupper —公开服务

服务:电子邮件、支付和运输都在 edge-east 上公开。

![](img/1ceb6c77a5da5911f801e9a046a6049a.png)

Skupper —公开服务

默认情况下，所有的服务都用协议:tcp 进行了注释，例如，上面的拓扑结构包括多个通过“http”进行通信的服务和 cart 服务，cart 服务是一个需要“tcp”连接的 redis 数据库。可以根据所需的连接性向单个服务添加注释。

![](img/3cc1df2336c26d52be99abca4780be77.png)

Skupper 注释

![](img/6453cd90b11748d896ab2b34c3279a42.png)

Skupper 注释

一旦所有的服务都暴露在 VAN 中，应用程序就可以运行，并且现在可以与位于不同平台上的不同集群中的所有其他集成服务进行通信。使用 VAN，私有集群中的服务可以被位于公共集群和其他远程私有集群中的其他服务访问。

![](img/e809c3b797c2b2cc6d0d8985b468c9d4.png)

潮人—功能性应用

“铜控制台部署”选项卡显示成员站点中的所有部署:

![](img/36e8371553e9a12049552c64fe90dfb3.png)

Skupper —部署控制台

带有服务连接信息的部署和站点的图形视图:

![](img/fcbf0d1d1a66288fcdea72da8bf9493c.png)

Skupper —部署控制台

监控服务之间的流量:

![](img/c2834254c9b276cf1ad837725f217443.png)

Skupper —部署控制台

如上图所示‘10 . 48 . 2 . 6’是位于 GKE 集群的前端服务的 pod_ip。借助 VAN，pod 可以与不同集群和网络空间中的其他 pod(服务)进行通信，而无需任何站点到站点的 VPN 连接。

![](img/5d8c599b19c1904c159c3720788c5c53.png)

前端— Pod_IP

Qpid 控制台提供了有关服务连接的详细信息:

![](img/359747709d528b540d9d31193dd6c901.png)

Qpid 路由器—控制台

Skupper —服务的图形视图，包含运行状况、流量和协议信息:

![](img/40001314f8c9fce8b6ad707b3ece296d.png)

Skupper —服务控制台

![](img/18e269755d6c8f26cd28eb2e574c892d.png)

Skupper —服务控制台

Skupper 控制台—站点和服务之间消息流的图形视图:

![](img/8c13422127641c2754adbddbba1b0d6f.png)

Skupper —消息流可视化

Qpid 控制台提供流量水平监控和实时流量:

![](img/f5f8471fcbb2b60c0c2c9bbe79bba908.png)

Qpid 路由器—消息流可视化

拓扑中的冗余路径提供了针对站点或连接故障的弹性。即使多个专用网络有重叠的 CIDR 子网，并且没有中央控制器或单点故障，它们也可以参与进来。例如，在使用的拓扑中，可以从两个边缘群集到 EKS 群集建立连接以实现冗余，如下所示:

![](img/5cc9a539258139a6b0e8c6bf3856743b.png)

Skupper —连接的站点拓扑

两个边缘集群的拓扑与 EKS 和 GKE 的路由器对等:

![](img/8ddaac30f822e54c76a9ccecd82b228e.png)

Qpid 路由器—连接的站点拓扑

越来越多的应用程序和框架在设计上不再直接绑定到任何云，也不再局限于部署在单个集群中。有了 VAN，云原生应用程序的开发人员就不必考虑运行应用程序服务所涉及的主机或端口。VAN 可以用更简单的方法和配置满足各种复杂的用例。
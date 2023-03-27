# 开放服务网格——微软基于 SMI 的开源服务网格实现

> 原文：<https://itnext.io/open-service-mesh-microsofts-smi-based-open-source-service-mesh-implementation-bcf41fd30732?source=collection_archive---------1----------------------->

![](img/5374879cd4541f545716794152b9b796.png)

OSM-SMI 兼容开源服务网格

微软的[开放服务网格](https://openservicemesh.io/blog/introducing-open-service-mesh/)是一个符合 SMI 的轻量级服务网格，作为开源项目运行。在包括 HashiCorp、Solo.io 和 bubbly 在内的服务网格合作伙伴的支持下，微软去年推出了服务网格接口，目标是通过提供一套规范标准，帮助最终用户和软件供应商处理服务网格技术带来的无数选择。OSM 可以被视为 SMI 的参考实现，它建立在现有的服务网格组件和概念之上。

开放服务网格数据平面在架构上基于特使代理，并实现了 [go-control-plane xDS v3 API](https://github.com/envoyproxy/go-control-plane) 。然而，尽管默认情况下 OSM 附带了 Envoy，但使用标准接口允许它与其他反向代理集成(与 xDS 兼容)。

SMI 跟随现有 Kubernetes 资源的脚步，如 Ingress 和 Network Policy，它们也没有提供一个实现，在该实现中，与 Kubernetes 交互所需的接口便于提供商插入他们的产品。相反，SMI 规范定义了一组公共 API，允许网格提供者交付他们自己的实现。这意味着网格提供者既可以直接使用 SMI APIs，也可以构建操作符将 SMI 转换为本地 API。

![](img/c18e7db3c32ed28f074f1ee535511ded.png)

SMI 实施

通过 OSM，用户可以在 Kubernetes 上使用 SMI 和 Envoy，并获得简化的服务网格实现。SMI 生态系统已经有多个提供商，如 Istio、Linkerd、Consul Connect、now Open Service Mesh 等。其中一些已经使用适配器(Istio，Consul Connect)和其他(OSM，Linkerd 等)实现了 SMI 兼容性。)直接使用 SMI APIs。

OSM 的实现非常类似于 Linkerd，它也直接使用 SMI APIs，不需要任何像 Istio 这样的适配器，但是一个关键的区别是，OSM 使用 Envoy 作为其代理和通信总线，而 Linkerd 使用 linkerd2-proxy(基于 rust 比 Envoy 轻)。

# 架构和组件

OSM 控制平面包括四个核心组件。所有这四个组件被实现为单个控制器实体(Kubernetes pod/deployment)，与旧版本的 Istio(其中有 4 个控制平面组件)相比，这在重量上要轻得多(Istio-1.6 引入的 istiod 将所有控制平面组件统一到一个二进制文件中)。

![](img/8101984578b8423ad43859b6cf3b5d67.png)

OSM 建筑—组件

OSM 数据平面—默认情况下使用 Envoy 作为反向代理，类似于大多数其他服务网格提供商(在这种情况下，Linkerd 是唯一的，它使用 Rust 编写的超轻透明代理)。虽然默认情况下 OSM 配备了 Envoy，但设计利用了[接口](https://github.com/openservicemesh/osm/blob/main/DESIGN.md#interfaces)(Go 中的接口类型是一种定义。它定义并描述了一些其他类型必须拥有的确切方法)，这些方法支持与任何 xDS 兼容的反向代理集成。所有代理的动态配置由 OSM 控制器使用 [Envoy xDS go-control-plane](https://github.com/envoyproxy/go-control-plane) 来处理。

网状规范是现有 SMI 规范(流量规范、流量目标、流量分割和流量度量)的包装。服务网格接口实现为提供者使用 SMI 规范提供了两种不同的方法。第一，使用适配器:需要一个单独的实体将 SMI APIs 转换成特定于提供者的配置。Istio 和 Consul Connect 使用适配器。第二种方法是直接使用 SMI API 和控制器实现，不需要额外的层。开放服务网格、Linkerd、Maesh、Meshery 等。使用这种方法。

网状目录是 OSM 的核心组成部分。MC 与收集各个输出的所有其他组件进行交互，并提供一个聚合结构，该结构随后被转换为代理配置。代理配置通过代理控制平面发送给所有监听/注册的代理。

![](img/add97875f9d3b380b96349fbadf23ac0.png)

OSM 建筑—组件

证书管理器实体为各个代理提供唯一的 X.509 证书。Kubernetes 服务对象是 SMI 规范中用于实施流量策略的核心实体。所有流量策略都由目标服务的代理实施，一个服务下的所有端点都由一个代理增强。将为每个服务提供服务证书，该证书将用于服务代理控制平面 mTLS 连接。

组件代理面向网格服务进程(在 Kubernetes 或 VM 上运行的容器或二进制文件)，所有代理建立到代理控制平面(OSM 控制器)的 mTLS gRPC 连接，并且使用 Envoy xDS go-control-plane 实现的控制器使用 xDS 协议缓冲区发送配置更新。

![](img/5cd4fc2a0d5592543a8ffdd5ab999a58.png)

OSM 建筑—组件

OSM 依靠 [SMI spec](https://github.com/servicemeshinterface/smi-spec) 来引用将参与服务网格的服务。OSM 提供了开箱即用的所有必要组件，以部署跨多个计算平台的完整服务网络。

# SMI 规范—概述

服务网格接口(SMI)规范包括四个 API 对象:流量规范、流量目标、流量分割和流量度量。每个 API 对象都被扩展为 Kubernetes CRD(自定义资源定义)。

![](img/1053dbbb59044b8b71a5d5f23bf0878e.png)

SMI —流量规格和流量目标

![](img/763cb0a728a4a8ea342ac4b16d54e01f.png)

SMI —流量分割和流量指标

下面的文章提供了关于 SMI 规范和 Istio SMI 实现的详细解释:

> https://it next . io/SMI-flexibility-and-inter operability-with-the-service-mesh-on-kubernetes-8 CD 88 a 4d 8238？source = friends _ link&sk = 33e 20531 be 24 d8e 0600443938 bb 35 f 17

# OSM 控制飞机— Kubernetes

OSM 命令行界面(CLI)使用此舵图将 OSM 控制计划安装到 Kubernetes 中。在幕后，“osm”使用 Helm 库在控制平面命名空间中创建一个 Helm“release”对象。头盔“释放”名称是网格名称(该名称用于标记启用边车注射的名称空间)。“helm”CLI 也可用于检查安装的 Kubernetes 清单的更多细节。

整个生态系统由一个控制器控制，其他组件构成 CRD 的监测和遥测系统(普罗米修斯、格拉法纳和齐普金)。所有组件都部署在 osm-system 命名空间中。

![](img/f9b6a26bbf70de75531e6748647712ce.png)

OSM —控制平面

OSM 控制器和相关部件:

![](img/d58e78dbf7b806fcc4b8777aeebf17dc.png)

OSM —控制平面

Open Service Mesh controller 初始化所有需要的 informers(client-go 包附带了一个子包，使获取事件变得容易:k8s.io/client-go/tools/cache.它允许用户轻松地添加当某些事件到来时将被调用的函数。它还允许用户轻松地将所有对象存储在内存(称为存储)中，以监视 Kubernetes 对象当前状态的创建/更改。

![](img/d2024cdebe520872c8ed3ccf8ff62a48.png)

OSM —主计长

开放服务网格控制器初始化与 SMI 组件(TrafficTarget、TrafficSplit、TrafficSpec 等)相关的通知器。).这使控制器能够识别和处理集群上的所有 SMI 规范。由于逻辑嵌入在控制器中，因此不需要单独的适配器(如 Istio)来转换 SMI 规范。

![](img/ac8748cb64148d519bcf8977dec710cf.png)

OSM —主计长

开放式服务网格使用 mTLS 对 pod 之间的数据以及特使和服务身份进行加密。证书由 OSM 控制平面通过 SDS(秘密发现服务)协议创建并分发给每个特使代理。开放服务网格支持 3 种颁发证书的方法:使用内部 OSM 包，称为[特勒索](https://github.com/openservicemesh/osm/tree/main/pkg/certificate/providers/tresor)(这是首次安装时的默认设置)，使用[哈希公司保险库](https://github.com/openservicemesh/osm/tree/main/pkg/certificate/providers/vault)和 [Azure 密钥保险库](https://github.com/openservicemesh/osm/tree/main/pkg/certificate/providers/keyvault)。

OSM 生态系统需要两种类型的证书:

1.  用于 Envoy 代理连接到 xDS 控制面板的证书—标识连接到 xDS 的代理和 pod。
2.  用于服务到服务通信的证书(一个代理连接到另一个代理)—标识彼此连接的服务。

![](img/3f5fc482bdec0502ffbaaba8c5580fc6.png)

OSM —主计长

![](img/e4a171506ad5a56e178fecf63fc14896.png)

OSM —主计长

Prometheus 部署与控制器一起创建，以持续轮询与 OSM 相关的指标—控制平面和数据平面(服务-服务、pod-服务)。它有助于收集使用特定的 *trace_id* 对服务架构中的延迟问题进行故障排除所需的计时数据。

![](img/4e6293385f457e571039411051c16822.png)

OSM——普罗米修斯

Zipkin 部署在 osm-system 名称空间中创建，可用于分布式跟踪。

![](img/1d3e9748e501aabe52fc0b89a624ab28.png)

OSM —齐普金

Grafana 部署与 Prometheus 一起创建，以可视化 Prometheus 收集的指标。OSM 项目提供存储在配置图(osm-grafana-config)中的默认仪表板。

![](img/eb93634af3c431e89c6b9743659ca097.png)

OSM —格拉法纳

所需的配置映射在 osm-system 命名空间中创建，其中包括全局 osm 配置、grafana 配置以及仪表板定义、数据源和 prometheus 配置。

![](img/f9e7d3f8234334ad8ae39302708e7821.png)

OSM —系统配置图

配置映射‘osm-config’存储 osm 控制器使用的全局配置。

![](img/1ffac6e8daca18ac58d962aaa00ed0ac.png)

OSM —全球配置

所需的机密在 osm-system 命名空间中创建。这些包括变异 webhook 所需的证书(用于边车注射)。helm secret 存储已部署的 OSM 版本的发布信息。

![](img/e3e7acdc29071cf782b2fb395f9888e4.png)

OSM——系统机密

所有 SMI CRD 都是作为控制面板引导程序的一部分创建的，这些都是 SMI 专用 CRD，其中没有与 OSM 相关的特定配置，控制器实现使 OSM 能够转换和处理这些配置。

![](img/5178f535a8597f34b38eb43f41895f0a.png)

OSM——史密斯·CRD

除了上述 CRD 的实验性*背压/速率限制*外，还创建了 CRD，这是 SMI 的实验性扩展。这使用户能够在下游服务过载或服务代理本身出现性能或容量问题的情况下，动态响应和控制入站流量。一旦它成为 SMI 的一部分，就会被删除。

![](img/506d27b7f81be7af9f3b6146ed9766b3.png)

SMI —实验 CRD —背压

# 代理边车—注射

使用所需的证书创建一个*变异 Webhook* 以与 Kube-API 交互，这将监视带有标签“*openservicemesh.io/monitored-by:<mesh_name>*”的名称空间，mesh _ name 是指使用“ *— mesh-name* ”或 helm 发布名称配置的网格名称。

![](img/9976a9876064ecc6e025f4f80e88250d.png)

OSM——改变边车注射的网钩

OSM 控制器使用 informers 来派生带有标签的所有名称空间，并向名称空间中的所有 pod 注入一个 sidecar。

![](img/e7b3355665772da5bdfa0ad070c1ff82.png)

OSM——边车注射的命名空间标签

控制器接收关于名称空间的信息，并将有效负载注入到具有初始化容器规范和配置代理容器的 osm-init 作业容器的所有 pod:

![](img/56211c79b136557ea7e9178767134c8f.png)

OSM —控制器日志

![](img/c677616ebef46ebef77fd8336f5b6d11.png)

OSM —控制器日志

带有代理边车的样品容器:

![](img/09bbb44e888f9cb4622ecf26690dae07.png)

OSM——特使边车

OSM 控制器—特使使用 ADS(聚合发现服务)来发现所有代理注入的服务。ADS 为所有资源类型的端点(此处为代理边车)、集群、路由、监听器和聚合流启用发现服务。

![](img/89e034c6640d63ff86de420f67bacbba.png)

OSM —特使边车—发现

# 演示应用程序

OSM 官方文档为用户提供了一个演示应用程序(类似于 Istio 的 bookinfo 应用程序)。这使用户能够测试所有可以使用 OSM 实现的功能。该应用程序包括书店(v1 和 v2)，购书者和偷书者，所有的服务都可以使用各自的用户界面访问。

![](img/754bbdc5b33a672ecdd8490bdfd27966.png)

OSM —演示应用程序

每个应用程序都部署在一个单独的名称空间中，名称空间被标记为 sidecar 注入。

![](img/0f38829a8002f542affa07fe3b43c7d9.png)

OSM —演示应用程序

跨名称空间的示例应用程序堆栈:

![](img/8914904f7b66f9f31062511dd42be10e.png)

OSM —演示应用程序

所有应用程序命名空间都将有一个关联的服务和服务帐户，用于配置流量规范。例如:

![](img/b1b46626e928cc19fd3da374efd6e6df.png)

OSM —演示应用程序

所有的应用程序都注入了代理 sidecar:

![](img/81ba2a07f4b8bca7558c943ab7bc998a.png)

OSM —演示应用程序

OSM 控制器—发现所有单个服务的 ADS(聚合发现服务)。

![](img/0fdec0335fed6dfcf0cd25a6b2d32a26.png)

OSM —控制器—聚合发现日志

OSM 控制器— SDS(秘密发现服务),为用于 mTLS 的所有个人服务提供所需的证书。

![](img/6a29fb2e199c966bff3b1298a28f5775.png)

OSM —控制器—秘密发现日志

OSM 使用 Envoy 的 go-control-plane xDS v3 API 来动态配置代理配置。

![](img/b32a780d4ae789e085f4238a9baa8c4f.png)

OSM —动态代理配置

# 出口

在 OSM 环境中，通过全局设置(使用 osm-system 命名空间中的 osm-config 配置映射进行配置)来启用出口。该设置在打开或关闭之间切换，并影响网格中的所有服务。安装 OSM 时，默认情况下会启用出口。

该配置要求指定网格 CIDR 范围。网状 CIDR 范围是与集群中配置的 pod 和服务 CIDR 相对应的 CIDR 范围的列表。出口需要网状 CIDR 范围，以防止在集群内指定的任何流量作为出口流量逸出，并能够对出口流量实施网状流量策略。所有网状网络内的流量都基于 L7 流量策略进行路由，出口流量的路由方式不同，不受网状网络内流量策略的限制。

![](img/70fdeacc82d6b9235b31dd4e8dd09aea.png)

OSM —启用出口

例如，在下面的示例应用程序中，服务 *bookbuyer* 和 *bookthief* 不断向 github.com 执行 GET 请求。

![](img/10357d5a5bf1e7aec7db4e64671d7f81.png)

OSM —演示应用程序—与 Github 的出站连接

![](img/59a739e7c737e6d4b44290bcd7f6b27c.png)

OSM —演示应用程序—与 Github 的出站连接

启用出口功能并配置 mesh_cidr_ranges 后，两种服务都可以到达 github.com，如下所示。

![](img/4f1240b41c758a960e1e68230dc061e8.png)

OSM —演示应用程序—与 Github 的成功出站连接

![](img/ac085878b3545df62c3dc17bba03b241.png)

OSM —演示应用程序—与 Github 的成功出站连接

如下所示，将出口功能设置为 false 会阻止所有发往网状网络外部的流量。

![](img/fda81587baae9960cccb3a19258d7ddf.png)

OSM —禁用出口

服务*买书人*和*偷书人*在出口关闭的情况下无法到达 github.com:

![](img/68651df9aaa9040a061be0dc031cd74d.png)

OSM——演示应用程序——拒绝与 Github 的出站连接

![](img/2487a5efecf818426419e576227c0442.png)

OSM——演示应用程序——拒绝与 Github 的出站连接

# 进入

服务可以使用 Kubernetes Ingress 和入口控制器在集群外部公开 HTTP 或 HTTPS 路由。一旦入口资源被配置为向集群内的服务公开集群外的 HTTP 路由，OSM 将在 pod 上配置 sidecar 代理，以允许基于 Kubernetes 入口资源定义的入口路由规则的服务的入口流量。

默认情况下，当入口资源与属于网格的后端服务一起应用时，OSM 将 HTTP 配置为服务的后端协议。OSM 的 osm-config ConfigMap 中的网状范围配置设置支持将后端协议的入口配置为 HTTPS。可以通过更新 osm-config 配置图来启用 HTTPS 入口。OSM 今天有官方[文档](https://github.com/openservicemesh/osm/blob/main/docs/patterns/ingress.md)提供集成 Nginx 入口控制器、Azure 应用网关入口和 Gloo API 网关等解决方案的步骤。

例如，使用 Nginx 入口控制器的入口实施需要以下配置:

部署在配置有 OSM 根证书的集群中的 Nginx 入口控制器，能够在使用 HTTPS 入口时验证后端服务器提供的证书(根证书存储在 osm-system 命名空间中的“osm-ca-bundle”秘密中)。

![](img/0ae5d0e552dd2782305aef91c3a93a2c.png)

OSM — Nginx 入口控制器

入口资源(HTTPS)配置示例如下:

![](img/255cf8d71c5c70206616487d3aedf5b3.png)

OSM —入口定义

一旦创建了入口资源，OSM 就会注入一个 sidecar，并将在 pod 上配置 sidecar 代理，以允许基于入口路由规则的服务的入口流量。

![](img/04ae1dcfc5fcde3d71315b2e85c4c632.png)

OSM —入口定义

入口控制器的工作方式(更新/同步)与其在传统 Kubernetes 环境中的工作方式相同。

![](img/7183d836f3fcec9a53e7dc0ad9266f18.png)

Nginx 入口控制器——入口配置

# 流量访问控制(允许/拒绝流量)

流量访问控制有三个概念，每个概念都是通过 TrafficTarget 使用三个字段定义的:

1.  Source:流量的来源，反映在特定的 Pod 列表中(由选择器选择/分组)。目前，只有使用 ServiceAccount 的选择器支持它。目前不支持资源选择(如指定部署、指定服务)。
2.  目的地:流量的目标也反映在特定的 Pod 列表中(由选择器选择/分组)。目前，它还只支持使用带有 ServiceAccount 的选择器。
3.  Route:流量规范(HTTPRouteGroup，TCPRoute)，用于区分目的地提供的不同接入方式。

第一个场景是配置一个 TrafficTarget 和 HTTPRouteGroup，以支持 Bookbuyer 和 V1 书店之间的通信。

![](img/8967a09ff29b92f0701347a85fdf0722.png)

OSM —演示应用—交通接入

当没有定义访问策略时，计数器不会增加，因为默认情况下，没有策略的服务之间的所有流量都会被丢弃。

![](img/80264ea5d8665c7ae7edc8c1f6e49930.png)

OSM —演示应用—交通接入

由于上述场景中只有一个版本的书店(v1 ),所有流量都发往 V1，因此流量拆分被配置为将所有流量发送到 V1 书店，如下所示:

![](img/956ed9a16cbc217f5142cbee9bebc691.png)

OSM —交通分流

在 TrafficTargets 中，进行了一项配置，以启用从 *ServiceAccount:bookbuyer* 到*service account:book store-v1*的连接，使用源和目的地如下所示:

![](img/55cf061d7c4b873886a44d6e559c24c2.png)

OSM —演示应用程序

HTTPRouteGroups 包含应该由 bookstore-v1 提供服务的两个匹配路径/路由:

![](img/906afe5b5b92a91f67f1abde9b34c6cf.png)

OSM —演示应用程序

使用上面的配置，Bookbuyer 服务可以访问 Bookstore-v1service，并且同样可以通过增加计数器来推断。由于 *ServiceAccount:bookthief* 不包含在上述来源中，因此不增加 Bookthief 计数器:

![](img/894008d1220931cc9379028a1ba31099.png)

OSM —演示应用程序

Bookthief 服务无法访问书店-v1:

![](img/481d5f8c08db98a226532b8364f34ace.png)

OSM —演示应用程序—偷书贼被拒绝进入书店

在上面的配置中将 bookthief 服务帐户添加到源中，可以使服务访问 bookstore-v1。

![](img/b0039032a7c05b515c66e2ef824a2a01.png)

OSM —演示应用程序—偷书贼被允许进入书店

![](img/f22779cdda189e453d4f21b2742409c4.png)

OSM —演示应用—交通接入

通过上面的配置，购书者和偷书者都可以访问书店，这导致所有三个服务的计数器增加，如下所示:

![](img/f476799c54165c3f8e9761c5c36eb501.png)

OSM —演示应用—交通接入

Bookthief 服务日志显示对已配置的 HTTPRouteGroups 的请求的 200 个响应:

![](img/0a146a50cc7e4c6ef0cd7575de823596.png)

OSM —演示应用程序—偷书贼被允许进入书店

# 流量分流

流量分流资源用于实现同一应用程序不同版本的流量百分比分流。

流量分割包括三个概念:根服务:流量起始点；后端服务:根服务的子集；权重:控制流向每个后端服务的流量。

在下面的场景中，引入了新版本的服务书店——book store-v2，并且完成了配置，使得两个服务 bookbuyer 和 bookthief 可以访问服务书店，并且书店中的流量由版本 v1 和 v2 平均(50%)提供服务。

![](img/96c478e70d8a3c999afaf0c836fd3593.png)

OSM —演示应用程序—流量分流

一个新的部署添加到命名空间书店，具有关联的服务和服务帐户

![](img/25b3cfed911b5876b0aa960c11f65c3b.png)

OSM —演示应用程序—流量分流

![](img/fbc1ab109c7e76bb09d90f266f241ce1.png)

OSM —演示应用程序—流量分流

如下所示，计数器没有增加，因为所有的流量都被配置到书店-v1。

![](img/2c56f3572b6de430a5ad8c66e39a2259.png)

OSM —演示应用程序—流量分流

为每个服务配置 50%的权重*书店-v1* 和*书店-v2* :

![](img/8cf5165a2e7d6112a6bce3e596bf79f1.png)

OSM —演示应用程序—流量分流

如下所示，随着流量在不同版本之间分流，计数器开始增加。

![](img/818b5b7876d6fba4a0cf4c213f725c28.png)

OSM —演示应用程序—流量分流

从 Zipkin 可以很容易地验证/监控这种拆分，如下所示，使用*服务名:bookbuyer* 和 *spanName:bookstore* 可以看到请求被分布在版本 v1 和版本 v2 *中。*

![](img/1a5b27fff7431b1fa3169628b6d02faa.png)

OSM —齐普金—验证流量分流

依赖关系图显示了在目标服务(书店-v1 和 v2)之间分布的源服务流量(来自购书者和偷书者):

![](img/1a06b071584cd7b81cf146751250bae8.png)

OSM —齐普金—验证流量分流

![](img/266e7c198eee7c02aa5aca86d947b6af.png)

OSM —齐普金—验证流量分流

# 监测和追踪

OSM 预装了普罗米修斯、格拉法纳和齐普金。开放服务网格(OSM)为网格内通信的所有服务生成详细的指标。这些指标提供了对网格中服务行为的洞察，帮助用户排除故障、维护和分析他们的应用程序。

Prometheus 有助于收集网格中所有服务的一致流量指标和统计数据:

![](img/8a031e82c4a9c3f8a9ed2d7fd2f41f07.png)

OSM——普罗米修斯

Grafana 可视化普罗米修斯后端时间序列数据库。OSM 提供预配置的控制面板(服务-服务、工作负载-工作负载、服务-工作负载)。

![](img/86d996e8e552cf6e60df70f6d129eb0f.png)

OSM —格拉法纳

![](img/6c0b87dd9748814b9c8c730eca7c2d86.png)

OSM —格拉法纳

APM 和跟踪的 Zipkin:

![](img/09f88e7aa608a49fbb3c97cbd74f513b.png)

OSM —追踪—齐普金

![](img/dbe16b359b14566081084d3df175814e.png)

OSM —追踪—齐普金

使用 SMI 规范提供了更容易地迁移到另一个 SMI 配置的服务网格的承诺，并且还消除了从网格提供商迁移时对重大配置改变的需要。到目前为止，SMI 不适合高级配置(如断路、基于位置的负载平衡等)的情况。是必需的，但是如果进一步开发 SMI 来使用 advances Envoy xDS APIs，所有这些都是可能的。

微软正计划将开放服务网格的控制权移交给 CNCF，这将有助于它在更广泛的开源社区中得分，谷歌决定不与 Istio 合作(将 Istio 移交给[开放使用公共资源](https://openusage.org/))激怒了开源社区。
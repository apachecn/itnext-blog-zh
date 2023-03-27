# 利用微软的 Dapr(分布式应用运行时)简化 Kubernetes 上的微服务

> 原文：<https://itnext.io/simplifying-microservices-on-kubernetes-with-microsofts-dapr-distributed-application-runtime-9aece5723484?source=collection_archive---------0----------------------->

![](img/7338828d79502a956820a34dd6f217b7.png)

用于在云和边缘上构建微服务应用的开源运行时

如今，Kubernetes 和微服务几乎是同义词，微服务将应用程序的功能分成可独立部署的块。开发人员在为边缘或云开发微服务时经常遇到的大多数问题都围绕着事件驱动的需求。他们需要管理诸如事件和响应触发器之类的事情，这需要应用程序代码之外的额外逻辑。多个微服务之间的通信目前要求使用 gRPC、API、pub/sub 等。此外，开发人员还必须进行“服务发现”和“状态管理”。这两个实例涉及几个参数。此外，根据它是无状态还是有状态应用程序，开发人员必须使用不同的 SDK 和编程模型。

Dapr (分布式应用程序运行时)使用一个 side-car 模式，类似于服务网格架构中的 side-car 实现，其中本地代理用于路由请求。Dapr 中的 Side-cars 用于在运行时与编译时集成微服务构建模块。Dapr 将其 API 公开为 sidecar 架构，作为容器或进程，不需要应用程序代码包含任何 Dapr 运行时代码。这带来了 Dapr 与通过标准 HTTP/gRPC 接口操作的现有和遗留代码集成的优势。这有助于企业开发人员体验微服务开发的好处，而不必重写他们的应用程序。

Dapr 的一个关键特性是它完全不受平台和语言的限制，这意味着它可以在任何平台、任何 Kubernetes 部署、云中或内部甚至任何物联网设备上运行。

Dapr 为 [Go](https://github.com/dapr/go-sdk) 、 [Java](https://github.com/dapr/java-sdk) 、 [JavaScript](https://github.com/dapr/js-sdk) 、[提供语言特定的 SDK。NET](https://github.com/dapr/dotnet-sdk) 和 [Python](https://github.com/dapr/python-sdk) 。开发人员可以通过类型化的语言 API 来公开 Dapr 构建块中的众多功能，如保存状态、发布事件或创建参与者，而不是为每个服务实现 http/gRPC API，这有助于开发人员使用他们选择的语言开发无状态&有状态函数和参与者的组合。

# Dapr — Kubernetes

Dapr 控制平面组件部署为 Kubernetes 部署。可以使用 Dapr-CLI 在 Kubernetes 集群上轻松初始化 Dapr，也可以使用 Helm 进行部署。

![](img/d2fa0e475f3347cda6469882f1bb7b63.png)

dapr——Kubernetes 上的组件

Dapr 控制平面由三部分组成。 *dapr-sidecar-injector* 和 *dapr-operator* 服务提供一流的集成，在与服务相同的 pod 中将 dapr 作为 sidecar 容器启动，并提供 dapr 组件更新的通知。

*   dapr-operator:监视来自 kube-apiserver 的服务(kubernetes pods)的创建、删除和更新事件，并带有注释“dapr.io/enabled: true”。

![](img/86ed96e2a6e6c32ef620c8e02095de21.png)

dapr-运营商— Kubernetes

*   dapr-sidecar-injector:用注释“dapr.io/enabled: true”将 Dapr sidecar 注入到所有服务中。

![](img/f74ef8e648100267e6f630a997e5ca84.png)

dapr-边车注射器 Kubernetes

*   dapr-placement:连接所有注入到 Kubernetes pods 的 dapr 边车，并添加 IP 地址来路由通信。

![](img/a827ba06379cd4c8a05697268ba81a9b.png)

dapr-安置 Kubernetes

Dapr 使用许多不同的州存储 Redis、CosmosDB、DynamoDB、Cassandra 等提供站点管理。保持和检索状态。状态存储是可插拔的，可以包括 Redis、Azure Cosmos 等。在上面的部署中，具有持久卷的 Redis 被用作状态存储。

![](img/c969070b7c1c7448800586cd3592fc1a.png)

Redis —国营商店— Dapr

# Dapr —概念

[Dapr 概念](https://github.com/dapr/docs/tree/master/concepts)

![](img/fb797787b0d809e0395b1022c8f0cafc.png)

Dapr —概念

# Dapr —规格

# 服务调用

使用户能够使用唯一的 id 调用其他应用程序。通过命名标识符促进应用程序之间的交互，并将服务发现的负担放在 Dapr 运行时上。

![](img/9e46f569771f82fbe7341eb4908db3cd.png)

Dapr —服务调用

# 粘合剂

促进应用程序的双向绑定功能，以便与不同的云/本地服务或系统进行交互。开发人员可以使用 Dapr API 调用输出绑定，并让 Dapr 运行时用输入绑定触发应用程序。

输入绑定:

当来自外部/其他资源的事件发生时触发应用程序。

![](img/a27cb95fe5e654cba003801de531953f.png)

Dapr —输入绑定

输出绑定:

调用外部资源。

![](img/ea407c1684a197442777513f412c19f1.png)

Dapr —输出绑定

# 公共订阅和广播

便于将有效负载发布给正在收听某个主题的多个消费者。Dapr 保证这个端点至少有一次语义。

![](img/557db8f8efa9300d8ad94067867d50a4.png)

Dapr —发布订阅

# 状态管理

允许开发人员通过 API 保存和检索状态。Dapr 有一个可插拔的架构，允许绑定到大量的云/本地状态存储。

![](img/673c7bbfef22a878fd9be4fa7f2edba5.png)

Dapr —现场管理

# 演员

Dapr 具有原生、跨平台和跨语言的虚拟演员能力。除了特定于语言的 Dapr SDKs，开发人员还可以使用下面的 API 端点来调用 actor。

![](img/c60514587cb2ca50257ee2db2116252b.png)

Dapr —演员

# 试验

从 Dapr 示例中选取一个示例计算器应用程序。这里的计算器应用程序是一个多语言程序，其中每个操作——加法、减法、乘法和除法使用不同的编程语言——Go Mux、Python Flask、Node Express。Net 核心，并作为单独的微服务部署在 Kubernetes 上，一起用于一个更大的计算器应用程序。用 React 编写的带有服务器和客户端的前端应用程序使用 Dapr 调用每个服务。每个 Kubernetes pod/应用程序/服务由两个容器组成:一个应用程序容器和一个 Dapr sidecar 容器。

![](img/2d3ff3a11fd26773420e2c4b9f183645.png)

Dapr —示例计算器应用程序

如上所示，每个微服务都服务于不同的操作。redis 集群用于管理应用程序的状态。

# 应用代码

如下所示，每个应用程序都是用不同的语言/框架编写的。该逻辑只涉及一个服务端点，没有任何依赖代码，以便与其他应用程序或 Dapr 一起工作。更简单地说，每个微服务逻辑没有额外的代码来通过 RPC 或 API 与其他逻辑进行通信。

![](img/754ba9e5d4c84820a7aea476cede1af5.png)

Dapr —计算器应用

![](img/ef5c4d1e95d3296e55ecddb9f7f2c0b4.png)

Dapr —计算器应用

基于 React 的前端充当所有其他应用程序的前端服务器和客户端，如下所示。Routes ('calculate/add '，'/calculate/subtract '，'/calculate/divide '，'/calculate/multiply ')将来自 React 客户端的请求转发到所有启用了 dapr 的服务(启用 Dapr 将在下一节介绍)。

Dapr sidecar 服务于本地主机:<daprport>。所有 Dapr 服务都是通过调用:</daprport>

```
/v1.0/invoke/<DAPR_ID>/method/<SERVICE'S_ROUTE>
```

![](img/248c6bcb1225413362be5b7a12ba6ba6.png)

Dapr —计算器—反应前端

# Kubernetes 部署—计算器应用程序

上面提到的每个应用程序都被部署为一个 Kubernetes pod(技术上来说是一个微服务),带有特定的 Dapr 注释，如下所示:

![](img/394d6e9e20828e2b4e7489d41f9a5c87.png)

Dapr — Kubernetes 计算器应用

当上述应用程序部署在 Kubernetes 上时，监听来自 Kube-API 服务器的事件的 Dapr-Operator 获得关于启用 Dapr 的应用程序的更新，并且 Dapr-Sidecar-Injector 将一个 Dapr sidecar 容器注入到带有注释“dapr.io/enabled: true”的所有 pod 中。

![](img/0ab488104eb1a268483f0b59b35843c4.png)

Dapr —计算器 Kubernetes 部署

![](img/33e01c53946aa132f08a744963660bef.png)

Dapr —操作员

![](img/8aecb6cc5aceb57c805e1db130d069de.png)

Dapr —边车注射器

如下图所示，上面部署的所有 Kubernetes pods 包括两个容器“app”和“daprd”。

![](img/ae87dbaed4f167b2653b6d8b0eab1514.png)

Daprd — Kubernetes 应用程序

所有应用程序单元都有一个到 redis-master 和 dapr-placement 以及 redis 的出站连接，用于状态管理。所有基于 Dapr 的连接信息都由 sidecar (Dapr)容器配置为 Application-Kubernetes Pod spec 中的环境变量，这使得不同的微服务应用程序能够通过 Dapr 相互发现并识别通道以相互通信，而无需任何额外的代码来执行发现和服务调用。

![](img/1cce19d53e680eb46dbfe29a08af7b11.png)

Dapr —容器环境变量

Dapr 放置组件识别所有 Kubernetes pods/微服务应用程序的 IP 地址信息，并基于前端计算器应用程序执行的操作实现相互之间的无缝 API 路由。

![](img/07ec6059675fae0bd813916bc67c425b.png)

Dapr —放置

使用基于 React 的前端应用程序执行不同的数学运算，React 应用程序应根据所选的运算到达特定的微服务。

![](img/628d5b823fbe997a5c4739050a461be3.png)

Dapr —前端反应应用—计算器

添加“3+3”，此操作无缝调用基于 Go Mux 的加法应用程序。从上面的应用程序代码中可以看出，没有额外的逻辑来调用服务。Dapr 使用路径和 dapr-id 根据操作选择的路径无缝调用不同的服务。

![](img/540d0ffa5b8ac0003cf10f3449068ce3.png)

计算器—加法应用

乘法“3*3”，此操作无缝调用基于 Python Flask 的乘法应用程序。

![](img/f08a5f43d3eb9b6e025b9ba974e202eb.png)

计算器—乘法应用

划分“3%3”时，此操作无缝调用基于节点的划分应用程序。

![](img/b8b46d13e055bc622006c356984e8d59.png)

计算器—除法应用

如上所述，来自前端的操作被无缝地引导到不同的微服务，应用程序代码中没有任何额外的逻辑。对于来自前端计算器应用程序的每个事务，使用 Dapr 将状态无缝地持久存储到 Redis。

![](img/3add18da23c7b8ec89f66023979094ee.png)

Dapr —状态持久性

处于持续状态的 total、next 和 operation 反映了计算器需要操作的三种状态。通过持久化这些，用户可以刷新页面或取下前端窗格，并仍然可以跳回到原来的位置。对唯一服务的每个交易也被记录在“调用多个服务”等状态中。

例如，完成了“10+10”的加法运算，页面被刷新。如下所示，状态通过从状态存储(redis)中检索的事务信息进行了重新组合:

![](img/b15cef84eb486b7465d90b09eca49d83.png)

Dapr —状态持久性

应用程序窗格中的所有 Dapr sidecars 都连接到 Redis 以保持状态。

![](img/f4a3729f9effee8e2fe4097632ffbd49.png)

Dapr — Redis StateStore —状态持久性

下面列出了上述示例和 dapr 控制平面组件中涉及的所有 Kubernetes 部署和服务:

![](img/91273d35b70274e10ea2d8b780121237.png)

dapr—Kubernetes 上的示例计算器应用程序

![](img/57a4d54e309cf5de62bee167713b1d0e.png)

dapr—Kubernetes 上的示例计算器应用程序

上图展示了如何将 Dapr 用作简化分布式系统开发的编程模型。Dapr 方便用户将它作为一个可插拔的架构来使用，以提供诸如多语言编程、服务发现和简化的状态管理等功能。这使得开发人员能够使用最适合工作或特定开发团队的语言来构建他们想要的任何服务。

如上所述，当用户从前端服务器调用相应的操作时，不需要他们居住的任何 IP 地址，也不需要使用任何语言/版本来对他们进行编程。相反，它通过名称调用本地 dapr side-car，dapr side-car 知道如何调用服务上的方法，利用平台的服务发现机制，在本例中是 Kubernetes DNS 解析。

例如，如果用户从前端应用程序调用“添加”操作，通过 Dapr URLs 的服务如下所示:

![](img/8bcbcb488076dc6415013f16bef2ff3d.png)

Dapr —操作模式

Dapr 使用户能够使用一致的 URL 语法调用服务端点，利用托管平台的服务发现功能来解析端点位置。除了服务调用，Dapr side-car 还提供无缝状态管理。应用程序使用嵌入式状态管理的唯一要求是对其 Dapr sidecar 的状态端点进行 GET 或 POST。客户端通过简单地发布 JSON 键值对来持久化状态，如下所示:

![](img/31e57187c08f3e28bcc3a9feb77ee84d.png)

Dapr —操作模式

除了服务调用和状态管理之外，Dapr 今天还提供了一组广泛的构建模块，如:服务间的发布和订阅消息、事件驱动的资源绑定、虚拟角色、服务间的分布式跟踪等。Dapr 本身就是用 Go 写的。它不依赖于 K8s，可以在其他环境中运行，尽管 Windows 巨人在这个项目中确实考虑了 K8s。

Dapr 的构建模块为开发人员提供了业内公认的最佳实践，每个构建模块都是独立的；其中用户可以在应用程序中使用它们中的一个、一些或全部。Dapr 抽象了微服务应用开发中涉及的许多复杂性，使开发人员只需关注应用逻辑。微软表示，“随着这一转变，微服务架构已成为构建云原生应用的标准，预计到 2022 年，90%的新应用将采用微服务架构。”。

总之，增加微服务的采用以解决边缘和云上应用和服务开发的爆炸式增长，超敏捷应用技术、部署速度和规模可以利用 Dapr 来简化基于微服务的应用开发。
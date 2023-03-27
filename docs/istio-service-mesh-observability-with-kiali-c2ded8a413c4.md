# 基于 Kiali 的 Istio 服务网格可观测性

> 原文：<https://itnext.io/istio-service-mesh-observability-with-kiali-c2ded8a413c4?source=collection_archive---------5----------------------->

![](img/40d370b0cdc5cf78fa4d963e456822f0.png)

用 Kiali 可视化 Istio 服务网格模式

Kiali 是一个开源项目，它与 [Istio](https://istio.io/) 合作来可视化服务网状拓扑。Kiali 包括在粒度级别映射流量、虚拟服务、断路器、延迟和请求率的功能。它提供了对 Istio 服务网格中微服务的行为/模式/动作的洞察。Kiali 还包括 Jaeger Tracing 以提供分布式跟踪，从而提供一个统一的界面来监控和管理服务通信。

Istio 在部署的一些棘手方面提供了一定程度的间接性。Istio as a service mesh 提供了保护服务间通信的模式，如使用电路制动、重试、超时等的容错。，其中路由决策是在网格级完成的，这消除了在平台级执行所有这些操作的用户。

在一个非常高的层面上，Istio 为用户提供了一个服务级网络依赖注入的机会，而 Kiali 提供了一个强大的 web 框架来可视化复杂的服务网状架构，通过功能指标来轻松收集有关策略、延迟、响应时间、服务和接收的请求的信息，从而更容易制定具体的规则集。

# 布局

Kiali 可以很容易地[部署到 Kubernetes 上](https://www.kiali.io/gettingstarted/),所有组件都应该部署在 istio-system 名称空间中，其中安装了 istio 控制平面组件。Kiali 会自动发现集群上按照名称空间隔离的所有工作负载、服务和 Istio 规则。

![](img/c93ceeb4d6471124c882dfd9127ff7de.png)

Kubernetes 上的 Kiali 部署

# Hello-World 示例

![](img/eb9bf453537559c9c8dbaaa9f53d6090.png)

带有 Istio 边车的 Helloworld 应用程序示例

在新的 Istio 和 Kubernetes 中，istio-proxy 容器被自动注入到 pods 中，其中名称空间被标记为“istio-injection=enabled ”,消除了显式 kubectl inject 的使用。

![](img/231e1fdbc33384d2c04b174eecfcbfbb.png)

Helloworld 应用程序示例

上述示例有两个 helloworld 部署 V1 和 V2，分别带有一个服务、一个 istio-gateway(在接收传入或传出的网格边缘运行的负载平衡器)和一个虚拟服务(当主机被寻址时应用的一组流量路由规则)。Kiali 发现了上面创建的对象和 Istio 对象:

![](img/7be6fe394b108cd2ac0d3d252268a44e.png)

Kiali — Helloworld 应用程序有两个版本:v1 和 v2

用户可以选择图形布局，如下所示。

![](img/bc01fecfd6b9c0292d283990901f838d.png)

Kiali —选择图表布局

工作负载和服务选项卡显示相应的服务->工作负载。

![](img/b4a0c437219dc5d36bd0332429b81277.png)

Kiali —服务和工作负载视图

Istio 配置选项卡显示相应服务的规则/策略配置。

![](img/053b3bffb332414b782e1571b7ca3b4e.png)

Kiali —可视化规则配置

“分布式跟踪”选项卡打开 Jaeger UI。

![](img/a9d798431a2e89429a01841574e14606.png)

Kiali —内置 Jaeger 追踪功能

用户可以选择用动画来可视化实时交通，并可以使用徽章(断路器、虚拟服务、边车)来过滤特定信息。

![](img/18faeb1c5cb7a27dd543e132a5568d46.png)

Kiali —实时交通可视化

如上图所示，地图显示了从网关到服务(v1 和 v2)的流量。左侧的选项卡提供了有关每秒请求数、服务的请求数的信息，以及上述信息的微型图表表示。用户可以选择特定的端点来过滤特定于服务的信息，如下所示:

![](img/2ccc4e53b4b6e25565893ce913fa5042.png)

Kiali —基于端点的可视化

分布式跟踪提供了流和交互的跟踪视图，以及依赖图(DAG)。

![](img/b6f5d0b1b11851fe71bb352d23fbec00.png)

用 Jaeger 进行分布式跟踪

![](img/5e05dff5a7c47fd322edf4641c2b0022.png)

使用 Jaeger 进行分布式跟踪—图形视图

# 使用 Kiali 可视化流量控制和金丝雀流量

金丝雀部署和受控流是服务网格可以实现的重要功能。这些主要用于企业用例中的日常场景。下面的示例显示了一个简单的场景，其中有两个服务 canary 和 ga，规则规定在它们之间分配 60%-40%的流量。

![](img/0b063fe1a3bb91d5fbbd8fb1cda15bc9.png)

金丝雀配置示例

![](img/2507a0b53b7dd47c44ab14637d3113b5.png)

基于权重的虚拟服务配置

Kiali 提供了一个清晰的地图视图，显示服务之间的实时流量，并带有图例信息，显示到达特定服务的流量百分比。

![](img/644ffcd7ff1086239823bd4922d02794.png)

Kiali —规则关联可视化

如上图所示，显示了一个图例，说明特定的部署有一个与之关联的规则。

# 用 Kiali 可视化断路器

断路器是一种规则，它可以基于许多条件(如连接和请求限制)允许与服务中不同端点集的不同连接。下面的示例是 Istio 提供的一个经典的 httpbin 示例，其中创建了一个 traffic_policy 来阻止连接和一个排出百分比。创建一个示例客户机(fortio)来与 httpbin 交互。

![](img/fb38f03310d56122e767609d878386a3.png)

面向 Http 的应用程序示例

![](img/bcf42d99093a20dc0d1c5e6c6820a5bc.png)

带断路器的示例目的地规则

根据上述规则,‘max connections:1’和‘http 1 maxpendingrequests:1’规定，如果一个以上的连接和请求同时命中 httpbin，则 istio-proxy 为进一步的请求打开电路，这导致失败。

有 2 个并发连接和 20 个请求队列:

![](img/08e71a2c756b50e1ccef57e76a9e823e.png)

测试断路配置

![](img/06069efe8c3c6c721058dc20d87aad39.png)

Kiali —可视化断路器

如上所示，大多数请求都成功了。Kiali map 在 httpbin 上显示一个符号，表示已经配置了一个规则。

![](img/29fbd8e286e49a4af597e2a6ed640ab7.png)

Kiali —可视化服务的特定流量

有 12 个并发连接和 20 个请求队列:

![](img/351836bcd4565d634f20637488aaec9d.png)

测试断路配置

![](img/fbe277263f05cd81af305f2383e4d6cc.png)

Kiali —可视化断路器

如上所示，近 50%的请求成功了，其他的则失败了。Kiali 显示上述故障范围超过 50%。“服务”选项卡还会发出有关连接降级的警报，如下所示:

![](img/77a9df0e0db3ebc28469e281152f329d.png)

Kiali —可视化每项服务的错误率

分布式跟踪提供了请求跟踪的深入视图，如下所示:

![](img/b14f39ec42f350630b802e407f5eaee6.png)

Kiali —使用 Jaeger 进行应用追踪

![](img/1934f13738bc3966bf88c3ad6c6e8604.png)

Kiali —使用 Jaeger 进行应用程序跟踪—图形表示

# 可视化多层 Bookinfo 应用程序

上面的例子展示了基于单一服务的模式。Kiali 在监控复杂的应用程序模式中扮演着重要的角色。以下 Bookinfo 示例由多层服务架构(产品、评论、评级和详细信息)组成。

![](img/3c4396db2866e1cc03fd1105389bc39d.png)

使用 Bookinfo 应用程序的示例插图

![](img/b61e75c06994edbac259a1821371500a.png)

Bookinfo 在 Kubernetes 上的应用

Kiali 发现了上面创建的所有对象。

![](img/e9c937dfb15f64629c479e74b23a396e.png)

Kiali —自动发现对象

![](img/3c174c612381e01d61a7ebc9429d3aac.png)

Kiali —自动发现对象

在没有任何 Istio 政策和规则的情况下，流程模式如下所示:

![](img/1d86afaae0bc247a0245f3fad61409d8.png)

Kiali —可视化，无需执行任何 Istio 规则

![](img/6579b013d7d2ac6f7a2e1681a8568d31.png)

Kiali —可视化，无需执行任何 Istio 规则

Jaeger 的分布式跟踪提供了应用程序中所有流的全面跟踪图，如下所示:

![](img/f4aa345c81cf74f33bbe4c9df2841349.png)

在 Kiali UI 中嵌入 Jaeger 进行应用级跟踪

![](img/463598bda6c29ab4875feefd43d4898f.png)

在 Kiali UI 中嵌入 Jaeger 进行应用级跟踪

Istio Grafana 仪表板视图:

![](img/df45e818b66d1a8f040eeda7e945902c.png)

Istio Metrics — Grafana 仪表板

# 使用 Kiali 可视化延迟

监控基于微服务的应用的一个主要方面是弹性测试。Istio 使用户能够注入延迟故障来测试弹性、达到服务所需的时间和响应时间。Kiali 的交通动画可以清楚地呈现流量和延误统计。使用与评级服务相关联的 istio-config(目的地规则)引入 7 秒的延迟来测试弹性。Bookinfo 应用程序是一个多层应用程序

![](img/cba3161ed6b1361c76c882d84dd54a41.png)

延迟的 Istio 虚拟服务

如上所述，7 秒的固定延迟与相应子集(服务中子集的名称)的评级相关联。目标规则是使用 istio-config 创建的，如下所示:

![](img/411af405cba4de52bdab4fb90fd47f56.png)

示例目标规则配置

![](img/c6665fea6e542b885ed5d51b4c2f3e88.png)

有延迟的虚拟服务配置示例

将目的地规则应用于服务后，用户可以从 Kiali UI 直观地看到流程，如下所示:

![](img/0b12b3648acf91bb2b68517d2859a20d.png)

Kiali —目的地规则可视化

如上所示，评级服务有一个图例，显示虚拟服务是相关联的。开始使用网关访问服务会显示如下所示的动画:

![](img/1785c9a655ae305fb236beccce65bc8b.png)

Kiali —延迟可视化

上面的图清楚地捕捉到了到达评级服务的诱导延迟。可以使用门户中提供的分布式跟踪清楚地跟踪流，如下所示:

![](img/d42361f39001fa54c11b1e0f3586926a.png)

显示延迟的应用程序跟踪

# 用 Kiali 可视化故障

应用程序中的错误服务会对应用程序的可用性和性能产生重大影响。使用上面的 bookinfo 示例，引入了一个 http-fault 来限制对 reviews 服务的 50%的成功可达性，配置如下所示:

![](img/e6e5e003c31cda02385d8563b283eda7.png)

带有故障注入的 Istio Virtualservice

所有三个版本的评论都有特定的权重。

![](img/68c722d78f1e7872c8be3c988b32585d.png)

基于 Istio 目标规则的权重分配

![](img/588d43a444e90021340c31f101a0b095.png)

权重分配—三个版本

查看 Kiali 上的图表:

![](img/e44fa89715a1c9ccdb9bfd6bcb0f3746.png)

Kiali —传出请求可视化

如上所述，reviews 和 reviews-v2 附有规则。根据配置，只有 50%的成功传出请求来自 reviews 服务。

![](img/e56dbe48bd1a30ae5b90f447e889a9b1.png)

Kiali —传出请求可视化

# 使用 Kiali 可视化中止

服务中完全无功能的端点会影响应用程序的高可用性。在上面的例子中，只有 50%的响应使用 http-fault 配置被阻塞。下面的配置注入了 100%的故障，使得细节端点完全不可用，以复制服务的不可用性。

![](img/01f79e55d3a9d227bb8914ffa2969746.png)

带有中止配置的 Istio Virtualservice

![](img/53f6069d0c0719da6370e68379cea476.png)

Istio 虚拟服务配置

![](img/aa6a15bf69a82e65681db86d9502ae16.png)

Kiali — Http 故障可视化(图例)

如上所示，详细信息有一个关联的 http-fault 规则，这使得详细信息端点完全不可用。

![](img/beee11c8f9b0e56bb9b297248494c4a6.png)

Kiali — Http 故障可视化

上面的图显示了到达细节服务的 100%失败，这是通信中的中止。

![](img/177090f9331f0696a1844f86e07940ac.png)

示例应用程序—中止

下面的 Istio Grafana 仪表板视图显示，详细信息服务完全不可访问:

![](img/e982e20a4ba00757e53a5439b4c2e3db.png)

Istio Metrics — Grafana 仪表板

随着用户开始将其应用从整体架构过渡到微服务，可见性对于部署的微服务来说变得至关重要，不仅仅是简单的指标。Istio with Mixer 可以提供具体的功能，如流量管理、可观察性、策略实施、服务身份和安全性，Kiali 等工具有助于在实时动态拓扑视图中理解复杂的服务网格交互，并在管理复杂的多层现实世界应用程序中发挥重要作用，使制定适合应用程序环境的特定策略变得更加容易。
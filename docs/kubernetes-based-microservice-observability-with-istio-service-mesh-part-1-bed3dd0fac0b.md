# 基于 Kubernetes 的 Istio 服务网格的微服务可观测性:第 1 部分

> 原文：<https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b?source=collection_archive---------0----------------------->

## 使用 Jaeger、Prometheus、Grafana 和 Kiali 在 Google Cloud 的 GKE 和 Istio 服务网格上观察分布式系统

在这篇由两部分组成的文章中，我们将探索 Istio 服务网络中的一组可观察性工具。这些工具包括耶格、基阿利、普罗米修斯和格拉夫纳。为了协助我们的探索，我们将在 Google 云平台上为 Google Kubernetes 引擎部署一个基于 Go 的微服务参考平台。

![](img/1f56c4e3637481e3951b87635d4dfcbf.png)

# 什么是可观测性？

与区块链、无服务器、人工智能和人工智能、聊天机器人、网络安全和服务网格类似，可观察性是目前 IT 行业的热门词汇。根据维基百科，可观察性是一种衡量系统内部状态从其外部输出的知识中推断出来的程度。日志、度量和跟踪通常被称为可观察性的三大支柱。这些是我们可以观察到的系统的外部输出。

辛迪·斯里德哈兰所著的《T2 分布式系统可观测性》一书在第四章(T5)中很好地详述了“可观测性的三大支柱”。在继续之前，我推荐阅读这个免费的在线摘录。关于可观察性的第二个重要信息来源是 [honeycomb.io](https://www.honeycomb.io/) ，这是一个生产系统可观察性工具的开发者，由著名的行业思想领袖 [Charity Majors](https://twitter.com/mipsytipsy) 领导。honeycomb.io 网站包括关于可观测性的文章、博客文章、白皮书和播客。

随着现代分布式系统变得越来越复杂，观察这些系统的能力需要同样现代的工具，这些工具是在考虑到这种复杂程度的情况下设计的。传统的日志记录和监控系统往往难以应对当今的混合和多云、基于多语言、事件驱动、基于容器和无服务器、可无限扩展的短暂计算平台。

像 [Istio Service Mesh](https://istio.io/) 这样的工具试图通过提供与几个同类最佳的开源遥测工具的本机集成来解决可观测性挑战。Istio 的集成包括用于分布式跟踪的 [Jaeger](https://www.jaegertracing.io/) ，用于分布式系统可视化的 [Kiali](https://www.kiali.io/) ，以及用于指标收集、监控和警报的 [Prometheus](https://prometheus.io/) 和 [Grafana](https://grafana.com/) 。结合云平台-原生监控和日志记录服务，如谷歌云平台(GCP)上的[stack driver](https://cloud.google.com/monitoring/)for[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/)(GKE)，我们拥有一个完整的现代分布式应用的可观测性平台。

# 参考微服务平台

为了展示与最新版本的 Istio Service Mesh 集成的可观察性工具，我们将在 GCP 的 GKE 部署一个用 Go 编写的参考微服务平台。我开发了参考平台来演示 API 管理、服务网格、可观察性、DevOps 和[混沌工程](https://principlesofchaos.org/)等概念。该平台由(14)个组件组成，包括(8) [基于 Go 的](https://golang.org/)微服务，一般标记为服务 A —服务 H，(1)角度 7，[基于类型脚本的](https://en.wikipedia.org/wiki/TypeScript)前端，(4) MongoDB 数据库，以及(1)用于基于事件队列的通信的 RabbitMQ 队列。该平台及其所有源代码都是免费和开源的。

参考平台旨在生成基于 HTTP 的服务到服务、基于 TCP 的服务到数据库(MongoDB)和基于 TCP 的服务到队列到服务(RabbitMQ) IPC(进程间通信)。服务 A 调用服务 B 和服务 C，服务 B 调用服务 D 和服务 E，服务 D 在 RabbitMQ 队列上产生一条消息，由服务 F 消费并写入 MongoDB，以此类推。当系统被部署到运行 Istio 服务网格的 Kubernetes 集群时，可以使用 Istio 的可观察性工具来观察这些分布式通信。

## 服务响应

在参考平台上，每个上游服务通过返回一个小的信息性 JSON 负载(在源代码中称为*问候*)来响应来自下游服务的请求。

![](img/98d795290d27f625eccecea038b33eb8.png)

响应在服务调用链中聚合，导致一系列服务响应返回到边缘服务，并到达基于角度的 UI，在最终用户的 web 浏览器中运行。响应聚合功能仅用于确认服务对服务通信、Istio 组件和遥测工具是否正常工作。

![](img/93765905cb936f4253efe73505d53641.png)

每个 Go 微服务包含一个`/ping`和`/health`端点。`/health`端点可用于配置 Kubernetes 活跃度和就绪性探测器。此外，边缘服务，服务 A，使用`access-control-allow-origin: *`响应头被配置用于[跨来源资源共享](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (CORS)。CORS 允许运行在最终用户的 web 浏览器中的 Angular UI 将服务称为一个`/ping`端点，它驻留在与 UI 不同的子域中。下面显示的是[服务 A](https://github.com/garystafford/golang-srv-demo/blob/master/service-a/main.go) 的 Go 源代码。

对于这个演示，MongoDB 数据库将在 GCP 的服务之外托管在 [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) 上，这是一个基于云的 MongoDB 即服务平台。类似地，RabbitMQ 队列将托管在 [CloudAMQP](https://www.cloudamqp.com/) 上，这是一个 RabbitMQ 即服务的基于云的平台。我在以前的几篇文章中使用过这两个 SaaS 提供者。使用外部服务将有助于我们理解 Istio 及其可观测性工具如何收集遥测数据，以便在 Kubernetes 集群和外部系统之间进行通信。

下面显示的是[服务 F](https://github.com/garystafford/golang-srv-demo/blob/master/service-f/main.go) 的 Go 源代码，该服务使用来自 RabbitMQ 队列的消息，由[服务 D](https://github.com/garystafford/golang-srv-demo/blob/master/service-D/main.go) 放置在那里，并将消息写入 MongoDB。

# 源代码

这篇文章的所有源代码都可以在 GitHub 的两个项目中找到。基于 Go 的微服务源代码、所有 Kubernetes 资源和所有部署脚本都位于[k8s-istio-observe-back end](https://github.com/garystafford/k8s-istio-observe-backend)项目存储库中。基于 Angular UI [类型脚本的](https://en.wikipedia.org/wiki/TypeScript)源代码位于[k8s-istio-observe-frontend](https://github.com/garystafford/k8s-istio-observe-frontend)项目库中。对于这个演示，您应该不需要克隆 Angular UI 项目。

```
git clone --branch master --single-branch --depth 1 --no-tags \
https://github.com/garystafford/k8s-istio-observe-backend.git
```

用于 Go 服务和 UI 的 Kubernetes `Deployment`资源文件中引用的 Docker 图像都可以在 [Docker Hub](https://hub.docker.com/u/garystafford/) 上获得。Go 微服务 Docker 映像是使用 DockerHub 上的官方 [Golang Alpine](https://hub.docker.com/_/golang) 基础映像构建的，包含 Go 版本 1.12.0。使用 Alpine 映像来编译 Go 源代码可以确保容器尽可能小，并包含最小的攻击面。

# 系统需求

要跟进这篇文章，您需要最新版本的`gcloud` CLI(最低。版本。239.0.0)、Google Cloud SDK 的一部分、 [Helm](https://helm.sh/) 和 just releases[Istio 1 . 1 . 3](https://github.com/istio/istio/releases/tag/1.1.3)(2019 年 4 月 15 日)安装并配置在本地或您的构建机器上。

![](img/10b4f673e2708eb8759bbfdf19fb13a5.png)

# 设置和安装

要将微服务平台部署到 GKE，我们将按以下顺序进行。

1.  创建 MongoDB Atlas 数据库集群；
2.  创建 CloudAMQP RabbitMQ 集群；
3.  为您自己的环境修改 Kubernetes 资源和脚本；
4.  在 GCP 创建 GKE 集群；
5.  使用 Helm 将 Istio 1.1.3 部署到 GKE 集群；
6.  为平台的公开资源创建 DNS 记录；
7.  将基于 Go 的微服务、Angular UI 以及相关资源部署到 GKE；
8.  对平台进行测试和故障排除；
9.  观察第二部分的结果！

# MongoDB Atlas 集群

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas) 是一个完全托管的 MongoDB-as-a-Service，可以在 AWS、Azure 和 GCP 上使用。Atlas 是一款成熟的 SaaS 产品，提供高可用性、有保证的正常运行时间 SLA、弹性可伸缩性、跨区域复制、企业级安全性、LDAP 集成、BI 连接器等等。

MongoDB Atlas 目前提供四种[定价方案](https://www.mongodb.com/cloud/atlas/pricing)，免费、基本、专业和企业。计划范围从最小的 M0 大小的 MongoDB 集群，具有共享 RAM 和 512 MB 存储，到大规模的 M400 MongoDB 集群，具有 488 GB 的 RAM 和 3 TB 的存储。

在这篇文章中，我在 GCP 的美国中部(爱荷华州)创建了一个 M2 大小的 MongoDB 集群，这个演示使用了一个用户数据库帐户。该账户将用于连接 GKE 上运行的八个基于 Go 的微服务中的四个。

![](img/fcaacf3d986452d4b54dc1df4f9fd327.png)

最初，我从 M0 规模的集群开始，但是计算资源不足以支持来自基于 Go 的微服务的大量调用。我建议至少有一个 M2 大小或更大的集群。

# CloudAMQP RabbitMQ 集群

[CloudAMQP](https://www.cloudamqp.com/) 在所有主要的云和应用平台上提供完全托管的 RabbitMQ 集群。RabbitMQ 将为我们基于 Go 的微服务的一部分支持一个解耦的、最终一致的、基于消息的架构。在本文中，我在 GCP 的美国中部(爱荷华州)地区创建了一个 RabbitMQ 集群，与我们的 GKE 集群和 MongoDB Atlas 集群相同。我选择了 RabbitMQ 的最小配置免费版本。CloudAMQP 还提供健壮的多节点 RabbitMQ 集群供生产使用。

# 修改配置

在 GitHub 项目的 Kubernetes 资源文件和 Bash 部署脚本中，您需要更改一些配置设置。

## MongoDB Atlas 的 Istio ServiceEntry

修改 Istio `ServiceEntry`，[external-mesh-MongoDB-Atlas . YAML](https://github.com/garystafford/golang-srv-demo/blob/master/resources/other/external-mesh-mongodb-atlas.yaml)文件，添加你的 MongoDB Atlas 主机地址。该文件允许流量从 GKE 上的四个微服务流出到外部 MongoDB Atlas 集群。

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: mongodb-atlas-external-mesh
spec:
  hosts:
 **- {{ your_host_goes_here }}**
  ports:
  - name: mongo
    number: 27017
    protocol: MONGO
  location: MESH_EXTERNAL
  resolution: NONE
```

## istio service entry for cloud amqp rabbit MQ

修改 Istio `ServiceEntry`，[external-mesh-cloudamqp . YAML](https://github.com/garystafford/golang-srv-demo/blob/master/resources/other/external-mesh-cloudamqp.yaml)文件，添加您的 CloudAMQP 主机地址。该文件允许从两个微服务到 CloudAMQP 集群的出口流量。

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cloudamqp-external-mesh
spec:
  hosts:
 **- {{ your_host_goes_here }}**
  ports:
  - name: rabbitmq
    number: 5672
    protocol: TCP
  location: MESH_EXTERNAL
  resolution: NONE
```

## Istio 网关和虚拟服务资源

您可以使用多种策略通过 Istio 将流量路由到 GKE 集群。我对这篇文章使用了一个域名，`example-api.com`，和四个子域名。一组子域用于 Angular UI，在`dev`名称空间(`ui.dev.example-api.com`)和`test`名称空间(`ui.test.example-api.com`)中。另一组子域用于 edge API 微服务，服务 A，UI 调用(`api.dev.example-api.com`和`api.test.example-api.com`)。基于 URL，流量被路由到特定的 Kubernetes `Service`资源。

根据 [Istio](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Gateway) 的说法，`Gateway`描述了一种在网格边缘运行的负载平衡器，接收传入或传出的 HTTP/TCP 连接。修改 Istio 入口`Gateway`，在`hosts`部分插入您自己的域或子域。这些是端口 80 上允许进入网状网络的主机。

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: demo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
 **- ui.dev.example-api.com**
 **- ui.test.example-api.com**
 **- api.dev.example-api.com**
 **- api.test.example-api.com**
```

根据 [Istio](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#VirtualService) ，一个`VirtualService`定义了当主机被寻址时要应用的一组流量路由规则。一个`VirtualService`被绑定到一个`Gateway`以控制到达特定主机和端口的流量的转发。修改项目的四个 Istio `VirtualServices`，插入你自己的域或子域。下面是四个`VirtualServices`之一的例子，在 [istio-gateway.yaml](https://github.com/garystafford/golang-srv-demo/blob/master/resources/other/istio-gateway.yaml) 文件中。

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: angular-ui-dev
spec:
  hosts:
 **- ui.dev.example-api.com**
  gateways:
  - demo-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        port:
          number: 80
        host: angular-ui.dev.svc.cluster.local
```

## 库伯内特的秘密

该项目包含一个 Kubernetes `Secret`， [go-srv-demo.yaml](https://github.com/garystafford/golang-srv-demo/blob/master/resources/secrets/go-srv-demo.yaml) ，有两个值。一个用于 MongoDB Atlas 连接字符串，一个用于 CloudAMQP 连接字符串。记住 Kubernetes `Secret`值需要`base64`编码。

```
apiVersion: v1
kind: Secret
metadata:
  name: go-srv-config
type: Opaque
data:
 **mongodb.conn: {{ your_base64_encoded_secret }}**
 **rabbitmq.conn: {{ your_base64_encoded_secret }}**
```

在 Linux 和 Mac 上，您可以使用`base64`程序对连接字符串进行编码。

```
> echo -n "mongodb+srv://username:password@atlas-cluster.gcp.mongodb.net/test?retryWrites=true" | base64
**bW9uZ29kYitzcnY6Ly91c2VybmFtZTpwYXNzd29yZEBhdGxhcy1jbHVzdGVyLmdjcC5tb25nb2RiLm5ldC90ZXN0P3JldHJ5V3JpdGVzPXRydWU=**

> echo -n "amqp://username:password@rmq.cloudamqp.com/cluster" | base64
**YW1xcDovL3VzZXJuYW1lOnBhc3N3b3JkQHJtcS5jbG91ZGFtcXAuY29tL2NsdXN0ZXI=**
```

## Bash 脚本变量

bash 脚本， [part3_create_gke_cluster.sh，](https://github.com/garystafford/golang-srv-demo/blob/master/part3_create_gke_cluster.sh)包含一系列环境变量。至少，您需要更改所有脚本中的`PROJECT`变量，以匹配您的 GCP 项目名称。

```
# Constants - CHANGE ME!
**readonly PROJECT='{{ your_gcp_project_goes_here }}'**
readonly CLUSTER='go-srv-demo-cluster'
readonly REGION='us-central1'
readonly MASTER_AUTH_NETS='72.231.208.0/24'
readonly GKE_VERSION='1.12.6-gke.10'
readonly MACHINE_TYPE='n1-standard-2'
```

bash 脚本 [part4_install_istio.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part4_install_istio.sh) 包含了`ISTIO_HOME`变量。该值应该对应于 Istio 1.1.3 的本地路径。在我的本地 Mac 上，该值如下所示。

```
readonly ISTIO_HOME='/Applications/istio-1.1.3'
```

# 部署 GKE 集群

接下来，使用包含的 bash 脚本[part 3 _ create _ gke _ cluster . sh](https://github.com/garystafford/golang-srv-demo/blob/master/part3_create_gke_cluster.sh)部署 GKE 集群。这将创建一个区域性、多区域、3 节点 GKE 集群，使用本文发布时的最新版本 GKE 1 . 12 . 6-gke . 10。该集群将部署到与 MongoDB Atlas 和 CloudAMQP 集群相同的区域，即 GCP 的美国中部 1(爱荷华州)区域。对于 SaaS 提供商和主要云提供商而言，规划云资源的驻留位置对于最大限度地降低网络 I/O 密集型应用的延迟至关重要。

![](img/efe4027f218e300a4cfd341232dfe541.png)

# 使用头盔部署 Istio

GKE 集群和相关基础设施就绪后，部署 Istio。在这篇文章中，我选择了用头盔安装 Istio，这是 Istio 推荐的。要使用 Helm 部署 Istio，请使用附带的 bash 脚本， [part4_install_istio.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part4_install_istio.sh) 。

![](img/26e12e35996fc17795da5c3b526f9759.png)

该脚本使用本地 Istio 1.1.3 `install/kubernetes/helm/istio`目录中的舵图安装 Istio，该目录是作为本演示的一个要求而安装的。Istio 安装脚本使用`--set`标志覆盖 [Istio 舵图表](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio)中的几个默认值。Istio 图表的 [GitHub 项目](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio#configuration)中详细列出了可用的配置值列表。这些选项启用了 Istio 的可观察性特性，我们将在第二部分中探讨这些特性。特征包括基亚利、格拉法纳、普罗米修斯和耶格。

```
helm install ${ISTIO_HOME}/install/kubernetes/helm/istio-init \
--name istio-init \
--namespace istio-system

helm install ${ISTIO_HOME}/install/kubernetes/helm/istio \
--name istio \
--namespace istio-system \
**--set prometheus.enabled=true \
--set grafana.enabled=true \
--set kiali.enabled=true \
--set tracing.enabled=true** 
kubectl apply --namespace istio-system \
-f ./resources/secrets/kiali.yaml
```

下面，我们看到在集群上运行的 Istio 相关工作负载，包括可观察性工具。

![](img/7ccec2bdb6eb62f276258e8ff1ad6bdd.png)

下面，我们看到相应的 Istio 相关的`Service`资源在集群上运行。

![](img/52ac269717cf7656f085052ca551adee.png)

# 修改 DNS 记录

我们将使用 DNS，而不是使用 IP 地址将流量路由到 GKE 集群及其应用程序。如前所述，我为帖子选择了一个域，`example-api.com`，和四个子域。一组子域用于 Angular UI，位于`dev`名称空间和`test`名称空间中。另一组子域用于边缘微服务，即 API 调用的服务 A。基于 URL，流量被路由到特定的 Kubernetes `Service`资源。

部署 GKE 集群和 Istio 触发了一个 [Google 负载平衡器](https://cloud.google.com/load-balancing/)、四个 IP 地址和所有必需的防火墙规则的创建。与转发规则相关联的四个 IP 地址之一(如下所示)将与负载平衡器的前端相关联。

![](img/2f7162e140190519135655438de130ac.png)

下面，我们看到了新的负载平衡器，它具有前端 IP 地址和由三个 GKE 集群工作节点组成的后端虚拟机池。如上所示，每个节点被分配一个 IP 地址。

![](img/b3024846681a233132fb9dbfd937a0bb.png)

如下图所示，使用 [Google Cloud DNS](https://cloud.google.com/dns/) ，我已经创建了四个子域，并将负载平衡器前端的 IP 地址分配给了所有四个子域。到这些地址的入口流量将通过 Istio 入口`Gateway`和四个 Istio `VirtualServices`路由到适当的 Kubernetes `Service`资源。使用您选择的 DNS 管理工具来创建四个 A 类型的 [DNS 记录](https://en.wikipedia.org/wiki/List_of_DNS_record_types)。

![](img/5914e1466edb9e0aafe69000c17d8caa.png)

# 部署参考平台

接下来，将八个基于 Go 的微服务、Angular UI 以及相关的 Kubernetes 和 Istio 资源部署到 GKE 集群。要部署平台，请使用附带的 bash 部署脚本，[part5a _ deploy _ resources . sh](https://github.com/garystafford/golang-srv-demo/blob/master/part5a_deploy_resources.sh)。如果出现任何故障，并且您希望删除现有资源并重新部署，而不破坏 GKE 集群或 Istio，您可以使用[part5b _ delete _ resources . sh](https://github.com/garystafford/golang-srv-demo/blob/master/part5b_delete_resources.sh)删除脚本。

![](img/983da0fec5cfaebf0f92f03cfd362fff.png)

deploy 脚本部署所有资源两个 Kubernetes 名称空间，`dev`和`test`。这将允许我们看到在使用可观察性工具时如何区分名称空间。

下面，我们看到了刚刚部署的 Istio 相关资源。它们包括 T2 资源、四个 T3 资源和两个 T4 资源。

![](img/723fdd451d0d0ec4ab933fcfd8fb2eb0.png)

下面，我们看到平台的工作负载(Kubernetes `Deployment`资源)在集群上运行。这里我们看到每个工作负载有两个 pod，总共 18 个 pod，在`dev`名称空间中运行。每个 Pod 都包含已部署的微服务或 UI 组件，以及 Istio 的 [Envoy 代理](https://istio.io/docs/concepts/what-is-istio/#envoy)的副本。

![](img/978e3cfc1de2737fe98b0a9f1088576e.png)

下面，我们看到相应的 Kubernetes `Service`资源在`dev`名称空间中运行。

![](img/61275bb0245041f5aa561dce287cd40c.png)

下面是在`test`名称空间中运行的`Deployment`资源的类似视图。同样，每个部署都有两个 Pod，每个 Pod 包含已部署的微服务或 UI 组件，以及 Istio 的 [Envoy 代理](https://istio.io/docs/concepts/what-is-istio/#envoy)的副本。

![](img/ce2db55a57bb5340b97107a78babe12e.png)

# 测试平台

我们确实希望确保平台的八个基于 Go 的微服务和 Angular UI 正常工作，相互通信，并与外部 MongoDB Atlas 和 CloudAMQP RabbitMQ 集群通信。测试集群最简单的方法是在 web 浏览器中查看 Angular UI。

![](img/93765905cb936f4253efe73505d53641.png)

UI 要求您输入服务 API 的边缘服务)的主机域。因为您不能使用我的子域，并且 JavaScript 代码在您的 web 浏览器本地运行，所以此选项允许您提供自己的主机域。这与您插入到 UI 的两个 Istio `VirtualService`中的域是相同的。这个域将您的 API 调用路由到在`dev`名称空间`service-a.dev.svc.cluster.local`或`test`名称空间`service-a.test.svc.cluster.local`中运行的 Kubernetes 服务的 FQDN(完全限定的域名)。

![](img/cd1c426af5ab1ef3feec1e3fc33fd0af.png)

您也可以使用性能测试工具对平台进行负载测试。在平台加载之前，许多问题都不会出现。我最近开始使用[嘿](https://github.com/rakyll/hey)，一种现代负载生成器工具，作为 Apache Bench ( `ab`)的替代品。与`ab`不同的是，`hey`支持 HTTP/2 端点，这是在 GKE 用 Istio 测试平台所需要的。下面，我将直接从[谷歌云端外壳](https://cloud.google.com/shell/)运行`hey`。该工具正在模拟 25 个并发用户，向服务 a 生成总共 1000 个基于 HTTP/2 的 GET 请求。

![](img/c08af31b868eaa209a05a0aeabfd3525.png)

# 解决纷争

如果由于某种原因 UI 无法显示，或者从 UI 到 API 的调用失败，并且假设所有 Kubernetes 和 Istio 资源都在 GKE 集群上运行(全部为绿色)，最常见的解释通常是对以下资源的错误配置:

1.  您的四条云 DNS 记录不正确。它们没有指向负载平衡器的前端 IP 地址；
2.  您没有用正确的子域配置四个库本内特`VirtualService`资源；
3.  您的基于 Go 的微服务无法到达外部 MongoDB Atlas 和 CloudAMQP RabbitMQ 集群。很可能是 Kubernetes `Secret`构造不正确，或者两个`ServiceEntry`资源包含了这些外部集群的错误主机信息；

我建议通过直接调用服务 API 的边缘服务)并使用 cURL 或邮差来开始故障诊断。您应该会看到一个类似于下面的 JSON 响应负载。这表明问题在于用户界面，而不是应用编程接口。

![](img/533d6bf0faf0b9c9bf4960f92c18c5bf.png)

接下来，确认为服务 D、服务 F、服务 G 和服务 h 创建了四个 MongoDB 数据库。此外，确认正在将新文档写入数据库的集合中。

![](img/ec2cd4cb270456d73117781e4989261f.png)

接下来，使用 CloudAMQP RabbitMQ 管理控制台确认新的 rabbtmq 队列已创建。服务 D 产生消息，服务 F 从队列中消费这些消息。

![](img/ff022322f8ade525398a41cfa922dc65.png)

最后，查看 Stackdriver 日志，看看是否有任何明显的错误。

![](img/1bc88433684f8420d323b4fab1437e8b.png)

# 第二部分

在本文的[第二部分](https://medium.com/@GaryStafford/kubernetes-based-microservice-observability-with-istio-service-mesh-part-2-f25c4b474a65)中，我们将探索每一种可观测性工具，看看它们如何帮助我们管理我们的 GKE 集群和集群中运行的参考平台。

![](img/8815aaf37c1ba81ffcffb5f09ffbfc06.png)

由于集群只需要几分钟就可以完全创建和部署资源，如果您想要拆除 GKE 集群，请运行 [part6_tear_down.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part6_tear_down.sh) 脚本。

![](img/8c979a3206cc4e4e7e8d6a953ff194e0.png)

*本文表达的所有观点都是我个人的，不一定代表我现在或过去的雇主或他们的客户的观点。*
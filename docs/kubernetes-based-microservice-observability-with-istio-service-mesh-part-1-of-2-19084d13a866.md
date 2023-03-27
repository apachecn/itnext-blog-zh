# 基于 Kubernetes 的 Istio 服务网格的微服务可观测性:第 1 部分，共 2 部分

> 原文：<https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-of-2-19084d13a866?source=collection_archive---------2----------------------->

## 通过 Istio 服务网格在亚马逊 EKS 上使用 Jaeger、Prometheus、Grafana、Kiali 和 Fluent Bit 观察分布式系统

这篇由两部分组成的文章探索了一组流行的开源可观察性工具，这些工具很容易与 Istio 服务网格集成。虽然这些工具不是 Istio 的一部分，但它们对于充分利用 Istio 的可观察性特性是必不可少的。这些工具包括用于分布式事务监控的 **Jaeger** 和 **Zipkin** ，用于指标收集和警报的 **Prometheus** ，用于指标查询、可视化和警报的 **Grafana** ，以及用于 Istio 整体可观察性和管理的 **Kiali** 。我们将通过添加用于日志处理和聚合的 **Fluent Bit** 来完善工具集。我们将使用这些工具来观察一个部署到**亚马逊弹性 Kubernetes 服务(亚马逊 EKS)** 集群的分布式、RESTful、基于微服务的参考应用平台。该平台运行在 EKS 上，将使用亚马逊文档数据库作为持久数据存储，并使用亚马逊 MQ 来交换信息。

![](img/37c8eeadc51933d4f622774230efb9ee.png)

显示参考应用平台的 Kiali 管理控制台

## Post 的以前版本

这篇文章的前几个版本展示了基于 Istio 的可观察性，包括 HTTP 和 gRPC，以及 Google Cloud 的 GKE 和 Azure 的 AKS 上的协议缓冲区，可以在这里找到:

[](/istio-observability-with-go-grpc-and-protocol-buffers-based-microservices-d09e34c1255a) [## 基于 Go、gRPC 和协议缓冲区的微服务在 Google Kubernetes 引擎(GKE)上的 Istio 可观测性

### 检查使用 Istio 的可观察性工具来监控使用协议缓冲区、gRPC 和…的基于 Go 的微服务

itnext.io](/istio-observability-with-go-grpc-and-protocol-buffers-based-microservices-d09e34c1255a) [](/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b) [## 基于 Kubernetes 的微服务可观测性与谷歌 Kubernetes 引擎(GKE)上的 Istio 服务网格:第 1 部分

### 在这篇由两部分组成的文章中，我们将探索最新版本的 Istio……

itnext.io](/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b) [](/azure-kubernetes-service-aks-observability-with-istio-service-mesh-4eb28da0f764) [## Azure Kubernetes 服务(AKS)与 Istio 服务网格的可观察性

### 使用 Istio 服务网格探索 Azure Kubernetes 服务(AKS)的可观察性

itnext.io](/azure-kubernetes-service-aks-observability-with-istio-service-mesh-4eb28da0f764) 

# 可观察性

与量子计算、大数据、人工智能、机器学习、5G 类似，可观测性是目前 IT 行业的热门流行语。根据维基百科的说法，可观测性是一种衡量系统内部状态从其外部输出中推断出来的程度。辛迪·斯里德哈兰所著的《T2 分布式系统可观测性》一书在第四章中描述了[可观测性的三大支柱](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html):“*日志、度量和跟踪通常被称为可观测性的三大支柱。虽然简单地访问日志、度量和跟踪并不一定使系统更容易被观察到，但是这些是强大的工具，如果理解得好，可以释放构建更好的系统的能力。*”

> 日志、度量和跟踪通常被称为可观察性的三大支柱。
> —辛迪·斯里达哈兰

Honeycomb 是生产系统可观察性工具的开发者。honeycomb.io 网站包括关于可观察性的文章、博客、白皮书和播客。根据 Honeycomb 的说法，“*当一个系统是可理解的时候，可观察性就实现了——这对于复杂的系统来说是很难的，在复杂的系统中，大多数问题是许多事情同时失败的集合。*

随着现代分布式系统变得越来越复杂，观察这些系统的能力同样需要考虑到这种复杂程度而设计的现代工具。不幸的是，传统的日志和监控工具难以适应当今多语言、分布式、事件驱动、短暂、容器化和无服务器的应用环境。Istio 服务网试图通过提供与几种流行的开源遥测工具的轻松集成来解决可观测性挑战。Istio 的集成包括用于分布式跟踪的 [Jaeger](https://www.jaegertracing.io/) 和 [Zipkin](https://zipkin.io/) ，用于 Istio 服务网格的整体可观察性和管理的 [Kiali](https://www.kiali.io/) ，以及用于指标收集、监控和警报的 [Prometheus](https://prometheus.io/) 和 [Grafana](https://grafana.com/) 。结合云原生监控和日志记录工具，如 [Fluent Bit](https://fluentbit.io/) 和[亚马逊 cloud watch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html)，您就拥有了一个完整的可观测性平台，用于运行在[亚马逊弹性 Kubernetes 服务](https://aws.amazon.com/eks/)(亚马逊 EKS)上的现代分布式应用。

> 传统的日志记录和监控工具难以应对当今多语言、分布式、事件驱动、短暂、容器化和无服务器的应用环境。

# 源代码

这篇文章的所有源代码都可以在 GitHub 的两个项目中找到。基于 Go 的微服务源代码和 Kubernetes 资源位于[k8s-istio-observe-back end](https://github.com/garystafford/k8s-istio-observe-backend/tree/2021-istio)项目库中。基于 Angular UI [类型脚本的](https://en.wikipedia.org/wiki/TypeScript)源代码位于[k8s-istio-observe-frontend](https://github.com/garystafford/k8s-istio-observe-frontend/tree/2021-istio)项目库中。对于这个演示，您不需要克隆 Angular UI 项目。

[](https://github.com/garystafford/k8s-istio-observe-backend) [## garystafter/k8s-istio-observe-back end

### 两篇博文的源代码，基于 Kubernetes 的带有 Istio 服务网格的微服务可观察性。请参见…

github.com](https://github.com/garystafford/k8s-istio-observe-backend) [](https://github.com/garystafford/k8s-istio-observe-frontend) [## garystafter/k8s-istio-observe-frontend

### 两篇博文的源代码，基于 Kubernetes 的带有 Istio 服务网格的微服务可观察性。请参见…

github.com](https://github.com/garystafford/k8s-istio-observe-frontend) 

该演示对两个 GitHub 项目都使用了`2021-istio`分支。使用以下命令克隆项目:

```
git clone --branch 2021-istio --single-branch \
  [https://github.com/garystafford/k8s-istio-observe-backend.git](https://github.com/garystafford/k8s-istio-observe-backend.git)*# optional - not needed for demonstration* git clone --branch 2021-istio --single-branch \
  [https://github.com/garystafford/k8s-istio-observe-frontend.git](https://github.com/garystafford/k8s-istio-observe-frontend.git)
```

用于 Go 服务和 UI 的 Kubernetes `Deployment`资源文件中引用的 Docker 图像都可以在 [Docker Hub](https://hub.docker.com/u/garystafford/) 上获得。Go 微服务 Docker 镜像是使用 DockerHub 上的官方 [Golang Alpine](https://hub.docker.com/_/golang) 镜像构建的，包含 [Go 版本 1.16.4](https://golang.org/doc/go1.16) 。使用 Alpine 映像和多阶段 Docker 构建方法来编译 Go 源代码，可以确保容器尽可能小，并最大限度地减少容器的潜在攻击面。

# 参考应用平台

为了展示 Istio 的可观察性工具，我们将在 AWS 上为 EKS 部署一个参考应用平台，该平台用 Go 和 TypeScript 编写。我开发了参考应用平台来演示不同的 Kubernetes 平台，如 EKS、GKE 和 AKS，以及服务网格、API 管理、可观察性、DevOps 和[混沌工程](https://principlesofchaos.org/)等概念。该平台包括一个后端，包含八个基于 [Go 的](https://golang.org/)微服务，一般标记为服务 A-服务 H，一个基于 Angular 12 [类型脚本的](https://en.wikipedia.org/wiki/TypeScript)前端 UI，四个 [MongoDB](https://www.mongodb.com/) 数据库，以及一个 [RabbitMQ](https://www.rabbitmq.com/) 消息队列。该平台及其所有源代码都在 GitHub 上开源。

![](img/4b55840eff21834568e18353928f0c44.png)

参考应用平台的基于角度的用户界面

参考应用程序平台旨在生成基于 HTTP 的同步服务到服务 IPC(进程间通信)、基于 TCP 的异步服务到队列到服务通信以及基于 TCP 的服务到数据库通信。比如服务 A 调用服务 B 和服务 C；服务 B 调用服务 D 和服务 E；服务 D 在 RabbitMQ 队列上生成一条消息，服务 F 使用这条消息并将其写入 MongoDB，依此类推。当系统部署到运行 Istio 服务网格的 Kubernetes 集群时，可以使用 Istio 的可观察性工具观察分布式服务通信。

![](img/ff8086d3129c8136e44f2292c3172883.png)

参考应用平台的高层架构

# 服务响应

每个 Go 微服务包含一个`/greeting`、`/health`和`/metrics`端点。服务的`/health`端点用于配置 [Kubernetes 活性、就绪和启动探测器](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)。`/metrics`端点暴露了普罗米修斯废弃的指标。最后，当调用它们的`/greeting`端点时，上游服务通过返回一个小的信息性 JSON 有效负载——一个*问候*来响应来自下游服务的请求。

```
{
    "id": "1f077127-2f9f-4a90-ad88-da52327c2620",
    "service": "Service C",
 **"message": "Konnichiwa (こんにちは), from Service C!",**    "created": "2021-06-04T04:34:02.901726709Z",
    "hostname": "service-c-6d5cc8fdfd-stsq9"
}
```

响应在整个服务调用链中聚合，导致一系列服务响应返回到边缘服务(服务 A ),随后返回到在最终用户的 web 浏览器中运行的平台 UI。

```
[
    {
        "id": "a9afab6a-3e2a-41a6-aec7-7257d2904076",
        "service": "Service D",
 **"message": "Shalom (שָׁלוֹם), from Service D!",**        "created": "2021-06-04T14:28:32.695151047Z",
        "hostname": "service-d-565c775894-vdsjx"
    },
    {
        "id": "6d4cc38a-b069-482c-ace5-65f0c2d82713",
        "service": "Service G",
 **"message": "Ahlan (أهلا), from Service G!",**        "created": "2021-06-04T14:28:32.814550521Z",
        "hostname": "service-g-5b846ff479-znpcb"
    },
    {
        "id": "988757e3-29d2-4f53-87bf-e4ff6fbbb105",
        "service": "Service H",
 **"message": "Nǐ hǎo (你好), from Service H!",**        "created": "2021-06-04T14:28:32.947406463Z",
        "hostname": "service-h-76cb7c8d66-lkr26"
    },
    {
        "id": "966b0bfa-0b63-4e21-96a1-22a76e78f9cd",
        "service": "Service E",
 **"message": "Bonjour, from Service E!",**        "created": "2021-06-04T14:28:33.007881464Z",
        "hostname": "service-e-594d4754fc-pr7tc"
    },
    {
        "id": "c612a228-704f-4562-90c5-33357b12ff8d",
        "service": "Service B",
 **"message": "Namasté (नमस्ते), from Service B!",**        "created": "2021-06-04T14:28:33.015985983Z",
        "hostname": "service-b-697b78cf54-4lk8s"
    },
    {
        "id": "b621bd8a-02ee-4f9b-ac1a-7d91ddad85f5",
        "service": "Service C",
 **"message": "Konnichiwa (こんにちは), from Service C!",**        "created": "2021-06-04T14:28:33.042001406Z",
        "hostname": "service-c-7fd4dd5947-5wcgs"
    },
    {
        "id": "52eac1fa-4d0c-42b4-984b-b65e70afd98a",
        "service": "Service A",
 **"message": "Hello, from Service A!",**        "created": "2021-06-04T14:28:33.093380628Z",
        "hostname": "service-a-6f776d798f-5l5dz"
    }
]
```

## 克-奥二氏分级量表

平台的后端边缘服务，服务 A，使用`access-control-allow-origin`响应头为[跨源资源共享](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (CORS)进行配置。CORS 配置允许在最终用户的 web 浏览器中运行的 Angular UI 调用服务 A 的`/greeting`端点，该端点可能驻留在与 UI 不同的主机中。下面显示的是服务 a 的 Go 源代码。注意第 33 行和第 189 行使用的`ALLOWED_ORIGINS`环境变量，它允许您配置服务的`Deployment`资源所允许的来源。

# MongoDB 和 RabbitMQ 即服务

使用外部服务将有助于我们理解 Istio 及其可观测性工具如何收集遥测数据，以便在 Kubernetes 上的参考应用平台和外部系统之间进行通信。

## 亚马逊文档 b

对于这个演示，参考应用平台的 MongoDB 数据库将在 EKS 之外托管在 [Amazon DocumentDB](https://aws.amazon.com/documentdb/) ( *与 MongoDB 兼容*)上。根据 AWS 的说法，Amazon DocumentDB 是一个专门为大规模 JSON 数据管理而构建的数据库服务，完全由 AWS 管理并与 AWS 集成，是企业级的，具有高耐用性。

## 亚马逊 MQ

类似地，参考应用平台的 RabbitMQ 队列将在 EKS 之外的亚马逊 MQ 上托管。AWS MQ 是 Apache ActiveMQ 和 RabbitMQ 的托管消息代理服务，使得在 AWS 上设置和操作消息代理变得很容易。Amazon MQ 通过为您管理消息代理的供应、设置和维护来减少您的操作责任。对于 RabbitMQ，Amazon MQ 提供了对 RabbitMQ web 控制台的访问。控制台允许我们监控和管理 RabbitMQ。

![](img/c86ed02293b3ec9cd6f3f2ec0488ecec.png)

RabbitMQ Web 控制台显示参考平台的问候队列

下面显示的是服务 f 的 Go 源代码。该服务使用 RabbitMQ 队列中的消息，由服务 D 放在那里，并将消息写入 MongoDB。服务使用 Sean Treadway 的 [Go RabbitMQ 客户端库](https://github.com/streadway/amqp)和 MongoDB 的 [MongoDB Go 驱动程序](https://github.com/mongodb/mongo-go-driver)进行连接。

# 先决条件

这个职位将承担 AWS EKS，Kubernetes 和 Istio 的基本知识水平。此外，本文假设您已经安装了最新版本的 [AWS CLI](https://aws.amazon.com/cli/) v2、 [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/) 、weaver works’[eks CTL](https://eksctl.io/)、 [Docker](https://docs.docker.com/docker-for-mac/install/) 、 [Istio](https://istio.io/latest/docs/setup/getting-started/#download) 和 [yq](https://github.com/mikefarah/yq) 。这意味着`aws`、`kubectl`、`eksctl`、`istioctl`、`docker`和`yq`命令工具都可以从终端获得。

![](img/51a3f3216e16e2f07c5f53e0230db67e.png)

## 亚马逊 EKS 的 CLI

Weaveworks 是一个简单的 CLI 工具，用于在 EKS 上创建和管理集群——亚马逊为 EC2 提供的托管 Kubernetes 服务。它是用 Go 写的，用的是 CloudFormation。

## Istio 的 CLI

Istio 配置命令行实用程序`istioctl`旨在帮助调试和诊断 Istio 网格。

# 设置和安装

尽管你可能更喜欢使用像 [Helm](https://helm.sh/) 或[Kustomize](https://kustomize.io/)with[Flux](https://www.weave.works/oss/flux/)或 [Agro CD](https://argoproj.github.io/argo-cd/) 这样的工具来将微服务平台部署到 Kubernetes，在这篇文章中，我们将使用`kubectl`来简化事情。如果你真的喜欢 Helm，在 GitHub 库中有一个测试版的图表[可以安装`dev`名称空间资源。使用这个图表，你可以忽略这篇文章中引用`dev`名称空间中的资源的任何说明。参见图表的](https://github.com/garystafford/k8s-istio-observe-backend/blob/2021-istio/helm/ref-app/)[自述文件](https://github.com/garystafford/k8s-istio-observe-backend/blob/2021-istio/helm/ref-app/README.md)获取进一步说明。

我们将大致按照以下顺序进行:

1.  为 ALB 创建 TLS 证书和 Route53 托管区域记录；
2.  创建一个 Amazon DocumentDB 数据库集群；
3.  创建一个 Amazon MQ RabbitMQ 消息代理；
4.  创建 EKS 集群；
5.  为您自己的环境修改 Kubernetes 资源；
6.  部署 AWS 应用程序负载平衡器(ALB)和相关资源；
7.  将 Istio 部署到 EKS 集群；
8.  将 Fluent Bit 部署到 EKS 集群；
9.  将参考平台部署到 EKS；
10.  对平台进行测试和故障排除；
11.  观察第二部分的结果；

# 亚马逊文档 b

如前所述，MongoDB 数据库将托管在 EKS 之外的亚马逊 document db(T20)上，与 MongoDB 兼容。创建 DocumentDB 集群。为了演示的简单性和经济性，我建议创建一个单一的`db.r5.large`节点集群。您将使用提供的`mongodb://`连接字符串从微服务连接到 Amazon DocumentDB。

亚马逊 DocumentDB 集群部署在亚马逊虚拟私有云(亚马逊 VPC)内。如果您在 VPC 而不是 EKS 安装 DocumentDB，您将需要确保 EKS VPC 可以访问 DocumentDB VPC。根据 DocumentDB [文档](https://docs.aws.amazon.com/documentdb/latest/developerguide/connect-from-outside-a-vpc.html)，部署在同一个亚马逊 VPC 中的亚马逊 EC2 实例或其他 AWS 服务可以直接访问 DocumentDB 集群。此外，Amazon DocumentDB 可以通过 EC2 实例或其他 AWS 服务从同一 AWS 区域或其他区域的不同 VPC 通过 [VPC 对等](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)进行访问。

![](img/7e2c5bcca7a8385c3807edb307f52d90.png)

# 亚马逊 MQ

类似地，RabbitMQ 队列将在 EKS 之外的亚马逊 MQ 上托管。创建一个 Amazon MQ RabbitMQ 代理。为了确保演示的简单性和可负担性，我推荐使用单个`mq.m5.large`实例代理。代理正在运行 RabbitMQ 引擎，并禁用了 TLS。您将使用 [AMQP](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) (高级消息队列协议)从微服务连接到 Amazon MQ。Amazon MQ 提供了一个`amqps://`端点。`amqps` URI 方案用于指示客户端与服务器建立安全连接。您可以从 Amazon MQ 提供的 RabbitMQ web 控制台管理和观察 RabbitMQ。

![](img/473bab79b2b94e6ef2e3916997dbb701.png)

# 修改 Kubernetes 资源

您需要更改 GitHub 项目的 Kubernetes 资源文件中的几个配置设置，以匹配您的环境。

## 文档数据库的 Istio ServiceEntry

修改 Istio `ServiceEntry`资源`external-mesh-document-db.yaml`，添加您的 DocumentDB 主机地址。这些命令假设您的帐户中只有一个经纪人。

```
export DOCDB_ENDPOINT=$(aws docdb describe-db-clusters --query 'DBClusters[0].Endpoint' | sed -e 's/^"//' -e 's/"$//')yq e '.spec.hosts[0] = strenv(DOCDB_ENDPOINT)' -i ./resources/dev/istio/external-mesh-document-db.yaml
```

该文件允许流量从 EKS 上的微服务流出到 DocumentDB 集群。

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: docdb-external-mesh
spec:
  hosts:
 **- {{ your_document_db_hostname }}**  ports:
  - name: mongo
    number: 27017
    protocol: MONGO
  location: MESH_EXTERNAL
  resolution: NONE
```

## 亚马逊 MQ 的 Istio ServiceEntry

修改 Istio `ServiceEntry`资源`external-mesh-amazon-mq.yaml`，添加您的 Amazon MQ 主机地址。这些命令假设您的帐户中只有一个经纪人。

```
export BROKER_ID=$(aws mq list-brokers --query 'BrokerSummaries[0].BrokerId' | sed -e 's/^"//' -e 's/"$//')export BROKER_HOSTNAME=$(aws mq describe-broker --broker-id $BROKER_ID --query 'BrokerInstances[0].ConsoleURL' | sed 's/https:\/\///g' | sed -e 's/^"//' -e 's/"$//')yq e '.spec.hosts[0] = strenv(BROKER_HOSTNAME)' -i ./resources/dev/istio/external-mesh-amazon-mq.yaml
```

该文件允许流量从 EKS 上的微服务流出到 Amazon MQ RabbitMQ 代理。

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: amazon-mq-external-mesh
spec:
  hosts:
 **- {{ your_amazon_mq_hostname }}**  ports:
  - name: rabbitmq
    number: 5671
    protocol: TCP
  location: MESH_EXTERNAL
  resolution: NONE
```

## Istio 网关

您可以使用多种策略通过 Istio 将流量路由到 EKS 集群。在这个演示中，我使用了一个 [AWS 应用程序负载平衡器](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) (ALB)。我已经将一个主机名`observe-ui.example-api.com`映射到运行在 EKS 上的 Angular UI 应用程序。后端基于微服务的 API，特别是边缘服务 Service A，被映射到第二个主机名`observe-api.example-api.com`。

![](img/7199ae3ccdd3c6b1ba733c4d073509e2.png)

根据[Istio](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Gateway),`Gateway`描述了在网格边缘运行的负载平衡器，接收传入或传出的 HTTP/TCP 连接。修改 Istio 入口`Gateway`资源`gateway.yaml`。将您自己的 DNS 条目插入到`hosts`部分。

```
yq e '.spec.servers[0].hosts[0] = "**{{ your_ui_hostname }}**"' -i ./resources/dev/istio/gateway.yamlyq e '.spec.servers[0].hosts[1] = "**{{ your_api_hostname }}**"' -i ./resources/dev/istio/gateway.yaml
```

这些是唯一允许通过端口 80 进入网状网络的主机。

```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: istio-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
    - port:
        number: 80
        name: ui
        protocol: HTTP
      hosts:
 **- {{ your_ui_hostname }}
        - {{ your_api_hostname }}**
```

## Istio 虚拟服务

根据 [Istio](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#VirtualService) ，当主机被寻址时，`VirtualService`定义一组要应用的流量路由规则。一个`VirtualService`绑定到一个`Gateway`来控制到达特定主机和端口的流量的转发。修改项目的两个 Istio `VirtualServices`资源，`virtualservices.yaml`。从 Istio 网关插入相应的 DNS 条目。

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: angular-ui
spec:
  hosts:
 **- {{ your_ui_hostname }}**  gateways:
    - istio-gateway
  http:
    - match:
        - uri:
            prefix: /
      route:
        - destination:
            host: angular-ui.dev.svc.cluster.local            subset: v1
            port:
              number: 80
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: service-a
spec:
  hosts:
 **- {{ your_api_hostname }}**  gateways:
    - istio-gateway
  http:
    - match:
        - uri:
            prefix: /api
      route:
        - destination:
            host: service-a.dev.svc.cluster.local
            subset: v1
            port:
              number: 8080
```

## 库伯内特的秘密

根据 Kubernetes 项目，Kubernetes Secrets 可以让你存储和管理敏感信息，比如密码、OAuth 令牌和 SSH 密钥。将机密信息存储在 Secret 中比将其逐字放入 Pod 定义或容器映像中更安全、更灵活。

该项目包含两个库伯内特斯`Opaque`型`Secret`资源，`go-srv-demo.yaml`。`Secret`包含几个您想要保护的任意用户定义的数据。数据包括微服务使用的完整 DocumentDB `mongodb://`连接字符串和 Amazon MQ `amqps://`连接字符串。您将使用`Secret`来保护整个连接字符串，包括主机名、端口、用户名和密码。该数据还包括 DocumentDB 主机、用户名和密码，以及使用基本身份验证登录 Mongo Express 的任意用户名和密码。

您必须使用`base64`对您的秘密值进行编码。在 Linux 和 Mac 上，可以使用`base64`程序对连接字符串进行编码。

```
echo -n '{{ your_secret_to_encode }}' | base64# e.g., echo -n 'amqps://username:password@hostname.mq.us-east-1.amazonaws.com:5671/' | base64
```

将`base64`编码值添加到`Secret`资源中。

```
apiVersion: v1
kind: Secret
metadata:
  name: go-srv-config
  namespace: dev
type: Opaque
data:
 **mongodb.conn: {{ your_base64_encoded_secret }}
  rabbitmq.conn: {{ your_base64_encoded_secret }}**
```

Mongo Express:

```
apiVersion: v1
kind: Secret
metadata:
  name: mongo-express-config
  namespace: mongo-express
type: Opaque
data:
 **me.basicauth.username: {{ your_base64_encoded_secret }}
  me.basicauth.password: {{ your_base64_encoded_secret }}
  mongodb.host: {{ your_base64_encoded_secret }}
  mongodb.username: {{ your_base64_encoded_secret }}
  mongodb.password: {{ your_base64_encoded_secret }}**
```

## AWS 负载平衡器控制器

该项目包含一个[自定义资源定义](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CRD)和关联资源`aws-load-balancer-controller-v220-all.yaml`。这些资源使用 [AWS 负载平衡器控制器](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) v2.2.0、`aws-load-balancer-controller`配置 AWS 应用负载平衡器(ALB)。AWS 负载平衡器控制器为 Kubernetes 集群管理 AWS 弹性负载平衡器(ELB)。当您创建 Kubernetes `Ingress`时，控制器会提供一个 AWS ALB。

修改第 797 行，以包含您自己的集群的名称。在整个演示过程中，我一直使用集群名称`istio-observe-demo`。

```
spec:
      containers:
      - args:
 **- --cluster-name=istio-observe-demo**        - --ingress-class=alb
        image: amazon/aws-alb-ingress-controller:v2.2.0
        livenessProbe:
          failureThreshold: 2
          httpGet:
            path: /healthz
            port: 61779
            scheme: HTTP
```

## EKS 集群配置

该项目包含一个`eksctl` `ClusterConfig`资源`cluster.yaml`。`ClusterConfig`定义了亚马逊 EKS 集群的配置以及网络、安全和其他相关资源。在本演示中，`eksctl`将创建一个 VPC 和相关的 AWS 资源，作为集群创建的一部分，而不是预先存在的亚马逊虚拟私有云(亚马逊 VPC)。修改文件以匹配您的 AWS 区域、期望的 EKS 集群名称和 Kubernetes 版本(第 4–6 行)。对于演示，我使用的是最新的 Kubernetes 1.20 版本。

# 设置环境变量

在您的终端中修改和设置以下环境变量。我将对作为演示一部分的所有演示 AWS 资源使用`us-east-1`。它们应该匹配上面的`eksctl` `ClusterConfig`资源。

```
export AWS_ACCOUNT=$(aws sts get-caller-identity --output text --query 'Account')
export EKS_REGION="us-east-1"
export CLUSTER_NAME="istio-observe-demo"
```

## Istio 主页

设置您的`ISTIO_HOME`目录。我已经安装了最新的 [Istio 1.10.0](https://istio.io/latest/news/releases/1.10.x/announcing-1.10/) ，并且在我的[Oh My Zsh](https://ohmyz.sh/)文件中设置了`ISTIO_HOME`环境变量。我还在我的`PATH`环境变量中设置了 Istio 的`bin/`子目录。`bin/`子目录包含`istioctl`可执行文件。

```
echo $ISTIO_HOME*/Applications/Istio/istio-1.10.0*where istioctl*/Applications/Istio/istio-1.10.0/bin/istioctl*istioctl version

*client version: 1.10.0
control plane version: 1.10.0
data plane version: 1.10.0 (4 proxies)*
```

# 创建 EKS 集群

使用前面修改的`cluster.yaml`文件，将 EKS 集群部署到 AWS 上的新 VPC。

```
eksctl create cluster -f ./resources/other/cluster.yaml
```

这一步使用 CloudFormation 部署大量资源。完整的 EKS 资源调配过程可能需要 15–20 分钟才能完成。

![](img/74fb2d594553977de8209ec8874d1a92.png)

对于完整的演示，`eksctl`将在您的 AWS 环境中部署总共四个 CloudFormation 堆栈。

![](img/12e8c364a0f504ccc305d92f166abbce.png)

完成后，配置`kubectl`以便可以连接到亚马逊 EKS 集群。

```
aws eks --region ${EKS_REGION} update-kubeconfig \
    --name ${CLUSTER_NAME}
```

使用以下命令确认集群创建成功:

```
kubectl cluster-infoeksctl utils describe-stacks \
  --region ${EKS_REGION} --cluster ${CLUSTER_NAME}
```

![](img/d10c8d06e3a84114ef0864bee40fa5de.png)

使用 EKS 管理控制台查看新集群的详细信息。

![](img/f84d9d4d66b38889003d5ee77067ffe0.png)

EKS 集群管理控制台的配置>计算选项卡

本演示中的 EKS 集群是使用单个亚马逊 EKS [托管节点组](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)、`managed-ng-1`创建的。受管节点组包含三个`m5.large` EC2 实例。EKS 集群的组成可以在`eksctl`、`cluster.yaml`的`ClusterConfig`资源中修改。

![](img/80037e0ffd2898c31681c59eb21ee0e3.png)

EKS 集群管理控制台的“概览”选项卡

# 部署 AWS 负载平衡器控制器

使用您之前修改的`aws-load-balancer-controller-v220-all.yaml`文件，部署 AWS 负载平衡器控制器 v2.2.0。请仔细阅读 [AWS 负载平衡器控制器](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)说明，以了解该资源是如何配置并与 EKS 集成的。

```
curl -o resources/aws/iam-policy.json \
  [https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.0/docs/install/iam_policy.json](https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.0/docs/install/iam_policy.json)aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy220 \
    --policy-document file://resources/aws/iam-policy.jsoneksctl create iamserviceaccount \
  --region ${EKS_REGION} \
  --cluster ${CLUSTER_NAME} \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::${AWS_ACCOUNT}:policy/AWSLoadBalancerControllerIAMPolicy220 \
  --override-existing-serviceaccounts \
  --approvekubectl apply --validate=false \
  -f [https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml](https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml)kubectl apply -f ./resources/other/aws-load-balancer-controller-v220-all.yaml
```

要确认`aws-load-balancer-controller`已部署就绪，运行以下命令:

```
kubectl get deployment -n kube-system aws-load-balancer-controller*NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller* ***1/1*** *1            1           55s*
```

## AWS 负载平衡器控制器策略

有一个 [OpenID 连接提供者 URL](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) 与 EKS 集群相关联。要对服务帐户使用 IAM 角色，集群中必须存在 IAM OIDC 提供程序。从 EKS 管理控制台的详细信息选项卡中获取 URL。

![](img/c7cd65dc57c83c58f1482f53d56ef267.png)

您也可以使用以下 AWS CLI 命令获取 URL:

```
aws eks describe-cluster --name ${CLUSTER_NAME}aws iam list-open-id-connect-providers
```

该项目包含一个策略文档`trust-eks-policy.json`。通过添加上面找到的 OpenID Connect 信息来修改策略文档。AWS [为集群创建 IAM OIDC 提供者](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)文档中也包含相关说明。

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Principal":{
            "Federated":" **{{ your_openid_connect_arn }}**"
         },
         "Action":"sts:AssumeRoleWithWebIdentity",
         "Condition":{
            "StringEquals":{
               "oidc.eks.us-east-1.amazonaws.com/id/**{{ your_open_id_connect_id }}**:sub":"system:serviceaccount:kube-system:alb-ingress-controller"
            }
         }
      }
   ]
}
```

创建并附加 AWS 负载平衡器控制器 IAM 策略和角色。

```
aws iam create-role \
  --role-name eks-alb-ingress-controller-eks-istio-observe-demo \
  --assume-role-policy-document file://resources/aws/trust-eks-policy.jsonaws iam attach-role-policy \
  --role-name eks-alb-ingress-controller-eks-istio-observe-demo \
  --policy-arn="arn:aws:iam::${AWS_ACCOUNT}:policy/AWSLoadBalancerControllerIAMPolicy220"aws iam attach-role-policy \
  --role-name eks-alb-ingress-controller-eks-istio-observe-demo \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicyaws iam attach-role-policy \
  --role-name eks-alb-ingress-controller-eks-istio-observe-demo \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
```

# 创建名称空间

Kubernetes 支持由同一个物理集群支持的多个虚拟集群。这些虚拟集群被称为[名称空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)。`dev`名称空间将容纳本次演示的参考应用平台 Angular UI 前端和 Go 微服务后端。这个名称空间代表我们的参考应用程序平台在 EKS 上的开发环境。第二个名称空间`mongo-express`将在本文稍后用于部署 Mongo Express。

```
kubectl apply -f ./resources/other/namespaces.yaml
```

## 启用自动边车注射

要利用 Istio 的功能，网格中的单元必须运行 Istio 边车代理。通过在名称空间上设置`istio-injection=enabled` `label`并启用注入 webhook，在该名称空间中创建的任何新 pod 将自动添加一个 Istio sidecar 代理。为[自动边车注入](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/#automatic-sidecar-injection)标记`dev`命名空间确保了我们的参考应用平台——UI 和微服务——将 Istio 边车代理自动注入到它们的 [pods](https://kubernetes.io/docs/concepts/workloads/pods/) 中。

```
kubectl label namespace dev istio-injection=enabled
```

# 部署秘密资源

在适当的`dev`和`mongo-express`名称空间中创建 DocumentDB 和 Amazon MQ 秘密。

```
kubectl apply -f ./resources/dev/secrets/secrets.yamlkubectl apply -f ./resources/mongo-express/secrets/secrets.yaml
```

# 安装 Istio 配置描述文件

Istio 带有几个内置配置[配置文件](https://istio.io/latest/docs/setup/additional-setup/config-profiles/)。这些配置文件提供了对 Istio 控制平面和 Istio 数据平面侧柜的定制。

```
istioctl profile list*Istio configuration profiles:
  default
  demo
  empty
  external
  minimal
  openshift
  preview
  remote*
```

对于这个演示，使用`default`概要文件，它安装 Istio 核心、`istiod`、`istio-ingressgateway`和`istio-egressgateway`。

```
istioctl install --set profile=demo -y*✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete*
```

# 在 Dev 命名空间中部署 Istio 资源

Istio `[Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/)`描述了在网格边缘运行的负载平衡器，接收输入或输出的 HTTP/TCP 连接。Istio `[VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/)`定义了当主机被寻址时要应用的一组流量路由规则。最后，Istio `[DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/)`定义了在路由发生后应用于某个服务的流量的策略。你需要部署一个 Istio `[Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/)`和一套`[VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/)`。您还需要部署一组`[DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/)`资源。使用以下命令创建 Istio `[Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/)`、两个`VirtualService`、九个`[DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/)`和两个`ServiceEntry`资源:

```
kubectl apply -f ./resources/dev/istio/
```

# 部署 Istio 遥测插件

Istio 项目包括与 Istio 集成的各种遥测插件的示例部署。附加组件包括 Jaeger、Kiali、Prometheus 和 Grafana，以及可选的 Zipkin。虽然这些应用程序不是 Istio 的一部分，但它们对于充分利用 Istio 的可观察性功能至关重要。根据 Istio 项目，这些部署旨在快速启动并运行以用于演示目的，并针对这种情况进行了优化。因此，这些部署不适用于生产。参见 [GitHub 项目](https://github.com/istio/istio/tree/master/samples/addons)了解更多关于集成每个插件的生产级版本的信息。

使用默认方法安装附加组件。然后，用项目中包含的修改后的配置更新 Prometheus。在`prometheus.yaml`文件中修改的 Kubernetes `ConfigMap`增加了配置来抓取参考平台的`/api/metrics`端点。

```
kubectl apply -f $ISTIO_HOME/samples/addonskubectl apply -f ./resources/istio-system/prometheus.yaml
```

在 EKS 管理控制台的“workloads”选项卡中，您应该会看到命名空间中有七个工作负载，每个工作负载都有一个 pod 正在运行。工作负载包括 Grafana、Jaeger、Kiali 和 Prometheus。还包括之前安装的 Istio 配置`demo`配置文件的`istiod`、`istio-ingressgateway`和`istio-egressgateway`。

![](img/8831953259ecb50fc5b9bf0d11559eb3.png)

# 部署 Kubernetes Web UI(仪表板)

[Kubernetes Web UI](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) (仪表板)是一个基于 Web 的 Kubernetes 用户界面。您可以使用 Dashboard 将容器化的应用程序部署到 Kubernetes 集群，对容器化的应用程序进行故障排除，并管理集群资源。此外，您可以使用仪表板来了解集群上运行的应用程序的概况，以及创建或修改单个 Kubernetes 资源。

![](img/682ec29dfd73632b36756a27b01c2a56.png)

要部署仪表板，请遵循[教程中概述的步骤:部署 Kubernetes 仪表板(web UI)](https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html) 。

```
kubectl apply -f [https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml)kubectl apply -f ./resources/kube-system/eks-admin-service-account.yaml
```

每个服务帐户都有一个密码，该密码带有用于登录仪表板的有效不记名令牌。使用以下命令检索与`eks-admin`帐户相关联的令牌。

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```

在单独的终端窗口中启动`kubectl proxy`。

```
kubectl proxy
```

使用`eks-admin`帐户的令牌登录到位于以下 URL 的 Kubernetes 仪表板:

```
[http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login)
```

![](img/5c70079047edfe9bd8ca80cd9d693b3a.png)

# 部署 Mongo Express

Mongo Express 是一个基于 web 的 MongoDB 管理接口，用 Node.js、Express 和 Bootstrap3 编写。将 Mongo Express 安装到 EKS 集群上的`mongo-express`名称空间中，以管理 DocumentDB 集群。

```
kubectl apply -f ./resources/mongo-express/services/mongo-express.yaml
```

使用以下两个命令获取任何 Kubernetes 工作节点的外部 IP 地址和 Mongo Express 的节点端口:

```
kubectl get nodes -o wide |  awk {'print $1" " $2 " " $7'} | column -tkubectl get service/mongo-express -n mongo-express
```

为了确保对 Mongo Express 的安全访问，在您的 VPC 的安全组中创建一个入站规则，只允许您的 IP 地址(“我的 IP”选项)访问在上面获得的节点端口上运行的 Mongo Express。

![](img/6be1185430629d46170310b973acca2d.png)

在单独的终端窗口中启动`kubectl proxy`。

```
kubectl proxy
```

使用任何 Kubernetes 工作节点和当前节点端口的外部 IP 地址来访问 Mongo Express。Mongo Express 将要求您输入您在之前使用基本身份验证创建的 Kubernetes Secret 中编码的用户名和密码。一旦您部署了参考应用程序平台，在本文的后面，您将会看到四个数据库:`service-c`、`service-f`、`service-g`和`service-h`。您通常在自己的 MongoDB 安装中看到的典型操作数据库在 UI 中是不可用的，因为 DocumentDB 是一个托管服务。

![](img/8cb2e821bd9796e707c999f1e3818373.png)

Mongo Express UI 显示了四个参考平台的数据库

# 修改和部署 ALB 入口

该项目包含一个 ALB `Ingress`资源`alb-ingress.yaml`。先前安装的 AWS 负载平衡器控制器被配置为限制 ALB 入口控制器控制的入口。通过设置`--ingress-class=alb`参数，它将控制器的范围限制为使用匹配的`kubernetes.io/ingress.class: alb`注释。这在同一个集群中运行多个入口控制器时尤其有用。

ALB `Ingress`资源`alb-ingress.yaml`需要在部署之前进行修改。首先，更新`[alb.ingress.kubernetes.io/healthcheck-port](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#health-check)`注释。端口值来自于`istio-ingressgateway`的`status-port`，它作为 Istio `demo`配置文件的一部分安装。要从`istio-ingressgateway`获得`status-port`，运行以下命令:

```
export STATUS_PORT=$(kubectl -n istio-system get svc istio-ingressgateway \
  -o jsonpath='{.spec.ports[?(@.name=="status-port")].nodePort}')yq e '.metadata.annotations."alb.ingress.kubernetes.io/healthcheck-port" = env(STATUS_PORT)' -i ./resources/istio-system/alb-ingress.yaml
```

接下来，将与`[external-dns.alpha.kubernetes.io/hostname](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/integrations/external_dns/)`注释中列出的域相关联的 SSL/TLS ( [传输层安全性](https://en.wikipedia.org/wiki/Transport_Layer_Security))证书的 ARN 插入到 ALB `Ingress`资源`alb-ingress.yaml`中。运行以下命令将 TLS 证书的 ARN 插入到`[alb.ingress.kubernetes.io/certificate-arn](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/tasks/ssl_redirect/)` [](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/tasks/ssl_redirect/)注释中。该命令假设您的 SSL/TLS 证书已向 [AWS 证书管理器](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html) (ACM)注册。

```
export ALB_CERT=$(aws acm list-certificates --certificate-statuses ISSUED \
  | jq -r '.CertificateSummaryList[] | select(.DomainName=="*.example-api.com") | .CertificateArn')yq e '.metadata.annotations."alb.ingress.kubernetes.io/certificate-arn" = strenv(ALB_CERT)' -i ./resources/istio-system/alb-ingress.yaml
```

`[alb.ingress.kubernetes.io/actions.ssl-redirect](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/tasks/ssl_redirect/)`注释将所有 HTTP 流量重定向到 HTTPS。TLS 证书用于 HTTPS 流量。然后，ALB 终止 ALB 上的 HTTPS 流量，并将未加密的流量转发到端口 80 上的 EKS 集群。

最后，用您的平台的 UI 和 API 主机名的公共分隔列表更新`[external-dns.alpha.kubernetes.io/hostname](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/integrations/external_dns/)`注释。

```
yq e '.metadata.annotations."external-dns.alpha.kubernetes.io/hostname" = "**api.example.com, ui.example.com"**' -i ./resources/istio-system/alb-ingress.yaml
```

下面是完整的 ALB `Ingress`资源，`alb-ingress.yaml`。

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: istio-system
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev
 **alb.ingress.kubernetes.io/healthcheck-port: '{{ your_status_port }}'**    alb.ingress.kubernetes.io/healthcheck-path: /healthz/ready
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
 **external-dns.alpha.kubernetes.io/hostname: "{{ your_ui_hostname, your_api_hostname }}"
    alb.ingress.kubernetes.io/certificate-arn: "{{ your_ssl_tls_cert_arn }}"**    alb.ingress.kubernetes.io/load-balancer-attributes: routing.http2.enabled=true,idle_timeout.timeout_seconds=30
  labels:
    app: reference-app
spec:
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: ssl-redirect
                port:
                  name: use-annotation
          - pathType: Prefix
            path: /
            backend:
              service:
                name: istio-ingressgateway
                port:
                  number: 80
          - pathType: Prefix
            path: /api
            backend:
              service:
                name: istio-ingressgateway
                port:
                  number: 80
```

要部署 ALB `Ingress`资源`alb-ingress.yaml`，请运行以下命令:

```
kubectl apply -f ./resources/istio-system/alb-ingress.yaml
```

要确认 AWS 负载平衡器控制器和 ingresses ALB 入口控制器控件的配置，请运行以下命令:

```
kubectl describe ingress.networking.k8s.io --all-namespaces
```

任何错误配置都应该在`Events`部分显示为错误。

![](img/511f7ad7968e6beef9099ba1e611a53a.png)

运行以下命令应该会显示与端口 80 相关联的 ALB 的公共 DNS 地址。

```
kubectl -n istio-system get ingress*NAME           CLASS    HOSTS   ADDRESS                                                                   PORTS   AGE
demo-ingress   <none>   *       k8s-istiosys-demoingr-*...*us-east-1.elb.amazonaws.com   80      23m*
```

使用 EC2 负载平衡器管理控制台查看新 ALB 的详细信息。

![](img/4c02edae740fafe9de72dfc99a2169d2.png)

# 部署流畅位和云观察代理

根据 AWS 最近的一篇博客文章，[Fluent Bit Integration in CloudWatch Container Insights for EKS](https://aws.amazon.com/blogs/containers/fluent-bit-integration-in-cloudwatch-container-insights-for-eks/)， [Fluent Bit](https://fluentbit.io/) 是一个开源的多平台日志处理器和转发器，允许您从不同的来源收集数据和日志，并将其统一发送到不同的目的地，包括 cloud watch 日志。Fluent Bit 还完全兼容 [Docker](https://www.docker.com/) 和 [Kubernetes](https://kubernetes.io/) 环境。使用新推出的 Fluent Bit `DaemonSet`，您可以将容器日志从您的 EKS 集群发送到 CloudWatch logs 进行日志存储和分析。亚马逊 EKS 集群上的 CloudWatch 代理`DaemonSet`将向 CloudWatch 发送指标，而 Fluent Bit `DaemonSet`将向 CloudWatch 日志发送日志。

为了安装 Fluent Bit 和 CloudWatch 代理，我使用了 AWS 文档中概述的步骤:[亚马逊 EKS 和 Kubernetes 上 Container Insights 的快速启动设置](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html)。我建议查看该文档以获得详细的安装说明。

```
kubectl apply -f [https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml](https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml)ClusterName=${CLUSTER_NAME}
RegionName=${EKS_REGION}
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
kubectl create configmap fluent-bit-cluster-info \
--from-literal=cluster.name=${ClusterName} \
--from-literal=http.server=${FluentBitHttpServer} \
--from-literal=http.port=${FluentBitHttpPort} \
--from-literal=read.head=${FluentBitReadFromHead} \
--from-literal=read.tail=${FluentBitReadFromTail} \
--from-literal=logs.region=${RegionName} -n amazon-cloudwatchkubectl apply -f [https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml](https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml)kubectl get pods -n amazon-cloudwatchDASHBOARD_NAME=istio_observe_demo
REGION_NAME=${EKS_REGION}
CLUSTER_NAME=${CLUSTER_NAME}curl [https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/service/cwagent-prometheus/sample_cloudwatch_dashboards/fluent-bit/cw_dashboard_fluent_bit.json](https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/service/cwagent-prometheus/sample_cloudwatch_dashboards/fluent-bit/cw_dashboard_fluent_bit.json) \
| sed "s/{{YOUR_AWS_REGION}}/${REGION_NAME}/g" \
| sed "s/{{YOUR_CLUSTER_NAME}}/${CLUSTER_NAME}/g" \
| xargs -0 aws cloudwatch put-dashboard --dashboard-name ${DASHBOARD_NAME} --dashboard-bodycurl [https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluentd/fluentd.yaml](https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluentd/fluentd.yaml) | kubectl delete -f -
kubectl delete configmap cluster-info -n amazon-cloudwatch
```

## 可选:Prometheus 的 CloudWatch 代理

或者，您也可以将带有 Prometheus monitoring 的 [CloudWatch 代理安装到您的亚马逊 EKS 集群中。如果您主要使用 CloudWatch 来监控您的 EKS 集群，我建议添加这个额外的指标收集功能。要使用 Prometheus monitoring `DaemonSet`配置和部署 CloudWatch 代理，请遵循文档](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-metrics.html)[在亚马逊 EKS 和 Kubernetes 集群上设置和配置 Prometheus 度量集合](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-install-EKS.html)。

为了节省时间，我修改了说明中讨论的`prometheus-eks.yaml`文件，并将其包含在项目中。要使用该文件，用以下命令替换教程 ( `kubectl apply -f prometheus-eks.yaml`)的[步骤 5 中的命令:](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup-configure.html#ContainerInsights-Prometheus-Setup-new-exporters)

```
kubectl apply -f ./resources/amazon-cloudwatch/prometheus-eks.yaml
```

如果您检查 EKS 集群的`amazon-cloudwatch`名称空间，您应该观察到两个`DaemonSet`正在运行:`cloudwatch-agent`和`fluent-bit`。如果你在 Prometheus 上安装了 CloudWatch 代理，你会多一个`Deployment`、`cwagent-prometheus`。每个`DaemonSet`向集群的每个工作节点部署一个 pod。

![](img/7a53ed115200bad779bf26d807386415.png)

一旦参考应用程序平台部署并运行，您应该能够在[Amazon cloud watch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html)控制台的地图视图中可视化应用程序。

![](img/9b2656cbb5664084fd7e9d81341f8935.png)

Amazon CloudWatch 容器洞察 UI

参考平台的集群日志也将在 Amazon CloudWatch 中提供。您应该能够访问每个应用程序组件的单独日志组。

![](img/e402b193b0a79e821b02baafd96899d6.png)

最后，还可以通过 Kubernetes 仪表板查看单个 pod 日志。微服务的日志详细级别默认设置为`info`。这个级别可以使用服务的 Kubernetes `Deployment`资源中的`LOG_LEVEL`环境变量来更改。

![](img/82ffd45340bfcf0768cce4ad11be0ec5.png)

# 部署 ServiceEntry 资源

使用 Istio `ServiceEntry` [配置](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/#controlled-access-to-external-services)，您可以从您的 Istio 集群中访问任何可公开访问的服务。Istio 代理可以配置为阻止任何没有在网格中定义 HTTP 服务或服务条目的主机。我们不会在演示中走到这个极端。然而，我们将配置`ServiceEntry`配置来监控参考平台的两个外部服务 DocumentDB 和 Amazon MQ 的出口流量。

确认`istio-egressgateway`正在运行，然后部署您之前修改的两个`ServiceEntry`资源。

```
kubectl get pod -l istio=egressgateway -n istio-system*NAME                                   READY   STATUS    RESTARTS   AGE
istio-egressgateway-585f7668fc-74qtf   1/1     Running   0          14h*kubectl apply -f ./resources/dev/istio/external-mesh-document-db-internal.yamlkubectl apply -f ./resources/dev/istio/external-mesh-amazon-mq-internal.yaml
```

# 部署参考应用程序平台

平台的每个组件在项目中都有一个文件，包含 Kubernetes `Service`和相应的`Deployment`资源。

使用以下命令将参考应用平台的前端 UI 和八个后端微服务部署到 EKS 集群:

```
kubectl apply -f ./resources/dev/services/ -n dev
```

从 EKS 管理控制台的“工作负载”选项卡中，您应该观察到每个参考应用程序平台组件的三个窗格都已启动并在`dev`名称空间中运行。

![](img/bd8b296b10a94ea902a564b8beb23220.png)

您还可以使用 Kubernetes 仪表板来确认成功部署到了`dev`名称空间。

![](img/ab7b882d860ae497478e23e988d568ea.png)

# 测试平台

您希望确保平台的基于 web 的 UI 可以通过 AWS 应用程序负载平衡器到达 EKS(通过 Istio)和 UI 的 FQDN(完全合格的域名)`angular-ui.dev.svc.cluster.local`。您希望确保平台的八个微服务相互通信，并与外部 DocumentDB 集群和 Amazon MQ RabbitMQ 代理通信。测试集群最简单的方法是在 web 浏览器中查看 Angular UI。比如我的`[https://observe-ui.example-api.com](https://observe-ui.example-api.com).)` [。](https://observe-ui.example-api.com).)

![](img/f0ba4f78f12b3cb7eacdf25066dedd7c.png)

参考应用平台的基于角度的用户界面

UI 要求您输入后端的主机名，即边缘服务 Service A。因为您希望使用自己的主机名，并且 UI 的 JavaScript 代码在您的 web 浏览器中本地运行，所以该选项允许您提供自己的主机名。这个主机名与您插入到服务 A 的 Istio `VirtualService`中的主机名相同。这个主机名将 API 调用路由到在`dev`名称空间`service-a.dev.svc.cluster.local`中运行的服务 A 的 FQDN。您应该观察到 UI 中显示了七个问候响应，除了 Service F。

也可以使用类似 [Postman](https://www.postman.com/) 这样的工具直接测试后端，使用和上面一样的后端主机名。

![](img/25bd247c4aa62452f2a24ac6851c16cf.png)

使用 Postman 对服务 A 的/greeting 端点发出请求

![](img/9a405525ab33c9413055d19e159e799d.png)

使用 Postman 对服务 A 的/request-echo 端点发出请求

## 使用 Hey 进行负载测试

您还可以使用性能测试工具对平台进行负载测试。许多问题直到平台被置于升高的负载下才会显现出来。我最近尝试了 [hey](https://github.com/rakyll/hey) ，一个基于 go 的现代负载生成器工具，作为 Apache Bench ( `ab`)的替代品，与`ab`不同，`hey`支持 HTTP/2 端点，这是在 EKS 用 Istio 测试平台所必需的。可以用[自制](https://brew.sh/)安装 hey。

```
brew install hey
```

使用 hey，您可以通过点击 API 主机名和`/api/greeting`端点来测试参考应用程序平台。下面的命令生成 1000 个请求，模拟 25 个并发用户，使用 [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) 。所有服务、RabbitMQ 代理和 DocumentDB 数据库之间都会产生流量。

```
hey -n 1000 -c 25 -h2 [{](https://observe-istio-eks.example-api.com){ your_api_hostname }}/api/greeting
```

结果显示，参考平台的 API 在大约 43 秒内成功响应了 1000 次`HTTP 200`，平均响应时间为 1.0430 秒。

![](img/51b7d2f729cb08555d3250d405dd2979.png)

要在较长时间内生成一致的流量水平，请尝试以下命令变体:

```
hey -n 25000 -c 25 -q 1 -h2 [{](https://observe-istio-eks.example-api.com){ your_api_hostname }}/api/greeting
```

该命令会产生大约 18 分钟的稳定流量，这使得在探索和排除可观测性工具的故障时更加方便。

![](img/1c05b61e07dbaaa0f161a8d77c86a076.png)

# 第二部分

在本帖的第二部分[中，我们将探索每个可观察性工具，看看它们如何帮助我们管理运行在 EKS 集群上的参考应用平台。](/kubernetes-based-microservice-observability-with-istio-service-mesh-part-2-of-2-ceec27575040)

![](img/ccd5177931f55550987feb62f0d528a9.png)

Jaeger UI 显示服务 A 的分布式跟踪

本博客代表我自己的观点，不代表我的雇主亚马逊网络服务公司(AWS)的观点。所有产品名称、徽标和品牌都是其各自所有者的财产。
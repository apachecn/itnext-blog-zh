# Argo:Kubernetes 的工作流引擎

> 原文：<https://itnext.io/argo-workflow-engine-for-kubernetes-7ae81eda1cc5?source=collection_archive---------1----------------------->

![](img/d584367f9e679cee42eb224f111df820.png)

Argo:Kubernetes 的工作流管理器

来自 [Applatix](https://applatix.com/) 的 Argo 是一个开源项目，它为 Kubernetes 提供容器原生工作流，将工作流中的每一步都实现为一个容器。Argo 使用户能够使用类似于传统 YAML 的自定义 DSL 来启动多步管道。该框架提供了复杂的循环、条件、依赖管理和 DAG 等。这有助于增加部署应用程序堆栈的灵活性以及配置和依赖性的灵活性。使用 Argo，用户可以定义依赖关系，以编程方式构建复杂的工作流，将任何步骤的输出作为输入链接到后续步骤的工件管理，并在易于阅读的用户界面中监控计划的作业。

阿尔戈 V2 是作为一个库伯内特 CRD(自定义资源定义)实现的。因此，Argo 工作流可以使用 kubectl 进行管理，并与其他 Kubernetes 服务(如卷、秘密和 RBAC)进行本地集成。新的 Argo 软件是轻量级的，安装不到一分钟，但提供完整的工作流程功能，包括参数替换、工件、夹具、循环和递归工作流程。

Argo 中的工作流自动化是由 YAML 模板驱动的(易于采用，因为 Kubernetes 主要使用相同的 DSL ),这些模板是使用 ADSL (Argo 领域特定语言)设计的。使用 ADSL 提供的每条指令都被视为一段代码，并与应用程序代码一起托管在 SCM(源代码管理)中。Argo 提供/支持六种不同的 YAML 构造:容器模板:根据需要创建单个容器和参数，工作流模板:定义作业，换句话说，短期运行应用程序运行到完成。工作流中的步骤可以是容器、策略模板:用于触发/调用作业或通知的规则、部署模板:创建长时间运行的应用程序、固定模板:在 Argo 之外粘合第三方资源、项目模板:可以在 Argo 目录中访问的工作流定义。

Argo 支持几种不同的方式来定义 Kubernetes 清单: [Ksonnet](https://ksonnet.io/) 应用程序、 [Helm](http://helm.io/) 图表和 YAML/json 清单的简单目录。

# 基本 Argo 工作流模板

下面的模板是一个简单的模板，其中定义了一个工作流来创建一个包含两个容器的 pod，其中一个容器包含 curl(仅包含 curl 的基于 Alpine 的映像)，另一个是 Nginx sidecar 容器(sidecar 作为第二个流程与服务一起运行，并提供“平台基础架构功能”)。这里 curl 是一个“主”容器，它轮询 nginx sidecar 容器，直到它准备好服务请求。

![](img/7a7b238ac6b3589abe6347aca9119146.png)

Argo 配置/工作流模板

*   上述工作流中的事件:

![](img/a3ea41a28ffb0a7a2a4ef574540bddf5.png)

上面工作流中的详细事件

![](img/4f806b75dccae36ec8a7d0d33b454705.png)

Nginx 边车执行卷曲操作

如上所示，工作流创建一个 pod，执行规范中定义的配置，并终止将 pod 状态标记为完成的容器。

![](img/d78b091c306bdcc03ae1dd439c72ddd1.png)

工作流生命周期示例

*   Argo 仪表板视图

![](img/260a2b9a2d827d3ff28ab478b9d4c57b.png)

Argo 仪表板工作流执行视图

# 带有条件的工作流模板

Argo 支持工作流执行中的条件模式。argo 提供的示例描述了如何在模板中使用“when ”,模板的执行依赖于从父模板接收的输出。

![](img/54014a7d1ac11a974ccffba0cdf8739a.png)

抛硬币示例

![](img/3087822b1f0ef7948c5108e0b57026c2.png)

抛硬币工作流程配置

上面的工作流运行一个随机整数脚本，其中常数 0 =正面，1 =反面，其中特定模板(正面或反面)的调用取决于“抛硬币”任务的输出。如上面的工作流图所示，抛硬币任务产生正面，其中仅执行正面模板。

![](img/821aaf95221b55afc3c87e04d1e93598.png)

Argo CLI 查询工作流

上述工作流创建了两个容器，一个用于执行 randomint python 脚本，另一个用于根据脚本结果执行 heads 模板。如下所示，heads 模板是根据 randomint 脚本的结果(result==heads)执行的。

![](img/57c945128dfccdbeb1ec0120a50e14d8.png)

抛硬币工作流执行

类似地，递归(递归调用的模板)可以添加到条件规范中。例如，下面的模板输出显示抛硬币模板被执行 n 次，直到结果是正面。

![](img/5875adc230bbbc5c419f01b1505357b2.png)

抛硬币=递归

# 带有循环和参数的工作流模板

Argo 便于在工作流模板中迭代一组输入，用户可以提供参数(例如:输入参数)。在下面的工作流中，使用两个输入参数“hello kubernetes”和“hello argo”执行 Whalesay 模板。

![](img/a71369dd5cccb9a81c79ea218ebb8fe9.png)

基于循环和参数的工作流配置

![](img/f8eef5a5c4499e0e417d3059249c9dd6.png)

Argo CLI 查询循环

![](img/bc8e05ab890a9c1580eea036dd5e35c4.png)

工作流程—回路配置

上面的模板包含一个模板，该模板将使用一个项目列表，并根据所提供的参数值的数量来运行任务。因此，这创建了两个 pod，一个执行使用参数“hello kubernetes”的工作流，另一个使用“hello argo ”,如下所示:

![](img/2ed5c5f2764b53a617ebbb56abfe618c.png)

具有循环配置的工作流执行

类似的复杂循环:遍历一组条目，动态生成条目列表。

# 具有由 DAG(有向非循环图)和步骤定义的依赖性的多步骤工作流模板

Argo 允许用户使用简单的 Python 对象 DAG(有向无环图)启动多步管道，并且还便于在工作流规范(嵌套工作流)中定义多个模板

![](img/d58b35f2db4fa2cb930f3588c21eb69c.png)

DAG 工作流执行模式

![](img/4bd4ce670240026c3c48713f78fdc1e0.png)

基于 DAG 依赖性的工作流配置

![](img/38f96746f251db692ca002292e1e1670.png)

DAG 执行

类似地，步骤可以用于定义多步骤工作流，下面的例子包括两个模板 hello-hello-hello 和 whalesay(嵌套工作流)。

*   使用步骤的多步骤工作流

![](img/a9617d0862d004f06c47e7f5195788bd.png)

基于配置的排序

![](img/7dc0bf809892cf499bb4d6cdfe9ab248.png)

模板配置

这里，基于步骤控制流程，基于虚线定义运行格式。

# 使用 Minio 进行工件管理并与 Argo 集成

工件是任何工作流(例如:CI-CD)的组成部分，工作流中的步骤生成工件，并且这些工件将被其他后续步骤消费。

这里使用的工件存储库是 Minio:带有亚马逊 S3 兼容 API 的开源对象存储服务器。

![](img/b3e41c764c26a3efa3f79c638672700f.png)

工厂管理

![](img/fad2e4fc84514f35d978bb001f9e494b.png)

工厂管理配置

上述工作流程由两个步骤组成。生成工件:使用 whalesay 模板和 2。消费工件:消费步骤 1 中创建的工件，并打印消息。按照配置中的定义，这两个任务按顺序运行。第一步中生成的 artifactory 存储在 Minio 中，通过在 workflow-configmap 中提供以下配置，artifactory 存储库可以与 Argo 集成。

![](img/0d51ea05f25010201e325806d4b58678.png)

Artifactory 管理的工作流配置

![](img/210762c9e7ed378f2913b405d777e2fe.png)

米尼奥

上面创建的 artifactory 存储在 my-bucket 的 Minio 中。“使用 artifactory”任务将根据提供的配置提取 artifactory。Minio 类似于 S3，并提供一个可共享的链接来使用 artifactory，如下所示。

![](img/0dfb1ce8e0824321f45a8015a1fc7088.png)

迷你桶视图

![](img/b6575740419698d41f30d706cd4dfbf1.png)

Minio 工件列表

# 使用 Argo 部署完整的应用程序堆栈(应用程序、数据库、监控和日志记录以及跟踪),以及长期运行的应用程序(部署和服务)

如上所述，Argo 支持各种结构，这些结构可用于部署简化的工作流，以创建具有定义良好的规则集的复杂应用程序堆栈。在下面的例子中，一个成熟的 web 应用程序(样本袜子商店应用程序)部署了日志记录(使用 Elastic Search、Kibana 和 FluentD)、监控(Prometheus 和 Grafana)、跟踪(Zipkin)。

Argo 可以使用各种 Kubernetes 配置(部署、服务、集群角色等。)使用传统的基于 YAML 的文件提供。在本例中，工作流目录包含所有基于 Argo 的工作流配置，这些配置使用 Kubernetes-spec 目录中的 Kubernetes 对象配置。

*   **Kubernetes Spec 目录内容:**

如下图所示，应用程序堆栈配置分为日志记录、监控、db (shock-shop-db)、app (sock-shop)和 zipkin (tracing)等单独的子目录，这些子目录构成了特定于服务的部署、服务、配置图和集群角色等。(基本上所有 Kubernetes 配置都需要部署 app)。例如，Zipkin 包含:Mysql 部署、Mysql 服务、Zipkin 部署、Zipkin 服务。

```
kubernetes-spec
├── logging
│   ├── elasticsearch.yml
│   ├── fluentd-crb.yml
│   ├── fluentd-daemon.yml
│   ├── fluentd-sa.yaml
│   └── kibana.yml
├── logging-cr
│   └── fluentd-cr.yml
├── monitoring
│   ├── grafana-dep.yaml
│   ├── grafana-svc.yaml
│   ├── prometheus-alertrules.yaml
│   ├── prometheus-configmap.yaml
│   ├── prometheus-crb.yml
│   ├── prometheus-dep.yaml
│   ├── prometheus-exporter-disk-usage-ds.yaml
│   ├── prometheus-exporter-kube-state-dep.yaml
│   ├── prometheus-exporter-kube-state-svc.yaml
│   ├── prometheus-sa.yml
│   └── prometheus-svc.yaml
├── monitoring-cfg
│   ├── grafana-configmap.yaml
│   └── grafana-import-dash-batch.yaml
├── monitoring-cr
│   └── prometheus-cr.yml
├── sock-shop-db
│   ├── carts-db-dep.yaml
│   ├── carts-db-svc.yaml
│   ├── catalogue-db-dep.yaml
│   ├── catalogue-db-svc.yaml
│   ├── orders-db-dep.yaml
│   ├── orders-db-svc.yaml
│   ├── session-db-dep.yaml
│   ├── session-db-svc.yaml
│   ├── user-db-dep.yaml
│   └── user-db-svc.yaml
├── sock-shop-persist-db
│   ├── carts-db-ss.yaml
│   ├── carts-db-svc.yaml
│   ├── catalogue-db-dep.yaml
│   ├── catalogue-db-svc.yaml
│   ├── orders-db-ss.yaml
│   ├── orders-db-svc.yaml
│   ├── session-db-dep.yaml
│   ├── session-db-svc.yaml
│   ├── user-db-ss.yaml
│   └── user-db-svc.yaml
├── sock-shop-usvc
│   ├── carts-dep.yaml
│   ├── carts-svc.yml
│   ├── front-end-dep.yaml
│   ├── front-end-svc.yaml
│   ├── orders-dep.yaml
│   ├── orders-svc.yaml
│   ├── queue-master-dep.yaml
│   ├── queue-master-svc.yaml
│   ├── rabbitmq-dep.yaml
│   ├── rabbitmq-svc.yaml
│   ├── shipping-dep.yaml
│   └── shipping-svc.yaml
├── sock-shop-zipkin-usvc
│   ├── catalogue-dep.yaml
│   ├── catalogue-svc.yaml
│   ├── payment-dep.yaml
│   ├── payment-svc.yaml
│   ├── user-dep.yaml
│   └── user-svc.yaml
└── zipkin
    ├── zipkin-cron-dep.yml
    ├── zipkin-dep.yaml
    ├── zipkin-mysql-dep.yaml
    ├── zipkin-mysql-svc.yaml
    └── zipkin-svc.yaml10 directories, 63 files
```

*   **工作流目录内容:**

工作流目录包含所有基于 Argo 的工作流配置，这些配置使用上述 Kubernetes 规范文件。

```
workflows/
├── logging-workflow.yaml
├── monitoring-workflow.yaml
├── sock-shop-persist-workflow.yaml
└── sock-shop-workflow.yaml0 directories, 4 files
```

请看上面的登录工作流示例:

![](img/e88b33457dca91c71b1fa6e667346ecc.png)

记录工作流配置

类似地，所有特定的工作流都使用特定的 Kubernetes 资源来无缝地部署应用程序，如下面的工作流所示。

*   **伐木 **

![](img/f38bc2421be7add108f7460ba6e1f50f.png)

记录工作流程

![](img/b0eb1b3a524337a2970c0e953d513d50.png)

Argo CLI 查询日志记录工作流

如上所示，登录工作流创建了 elastic search:deployment and service，Kibana: deployment and service，Fluentd: service account，daemonset。

*   ***监控***

![](img/758af186d47151a471362a4468d22c90.png)

监控工作流程

![](img/6d82c8b04f3e346b70857c5180eac112.png)

Argo CLI 查询监控工作流

如上所示，监控工作流为正在运行的 Prometheus 和 Grafana 堆栈创建所需的资源。

![](img/d3c5ca325653aa94d7169727abf5c7b8.png)

Argo 工作流引擎部署的 Kubernetes 资源

*   ***袜子店***

![](img/1d98a8a1d95bcc6a73346b46bad04d6e.png)

袜子店工作流程执行

如上所述，该工作流构成了一个多应用程序/模板工作流，其中 Zipkin 与具有依赖性需求的 sock-shop 一起部署。

![](img/0e644be329f756f696889c24757cd753.png)

袜子店 Argo 工作流执行

![](img/0a3126867b57e27940e7f2a0e090ff05.png)

Kubernetes 上部署的袜子店

![](img/2f33f37bf53ec3f1d29dbb7e10ef1318.png)

应用程序跟踪组件

有了它，一个成熟的应用程序与日志记录、监控和跟踪一起上线了。所有必需的服务都将基于所提供的配置来创建。

![](img/e83b627d88f584b058cb93caf93c6690.png)

Kubernetes 为上面创建的组件提供服务

*** ***前端(网店)***

![](img/70ef244476f929242945110369d511d8.png)

Shock-Shop 前端 Web 应用程序

![](img/c666573962769c83de16053d103ca4d6.png)

Shock-Shop 前端 Web 应用程序—目录

*** ***伐木(基巴纳)***

![](img/32109e3bfca9b90f627393bccd1471fb.png)

用于测井可视化的 Kibana

*** ***监控(普罗米修斯和格拉夫纳)***

![](img/8b8a77d988bbd92ac354eab1e471019a.png)

普罗米修斯进行监控

![](img/1c66fc1b8f51e46e753b9eac529396af.png)

在 Grafana 上可视化普罗米修斯导出的度量

![](img/0f46d936c5ee58a918956071ce5fae89.png)

在 Grafana 上可视化普罗米修斯导出的度量

*** ***描摹(Zipkin)***

![](img/d41fc3c10f13a2addcd47904392c642a.png)

使用 Zipkin 进行应用程序跟踪

Argo 通过将工作流引擎的丰富功能集与本机工件管理、准入控制、“fixtures”、对 DinD (Docker-in-Docker)的内置支持以及策略相结合，在诸如传统 CI/CD 管道、具有顺序和并行步骤及依赖性的复杂作业、分布式应用的编排部署以及基于策略架构触发基于时间/事件的工作流等场景中具有极大的重要性。
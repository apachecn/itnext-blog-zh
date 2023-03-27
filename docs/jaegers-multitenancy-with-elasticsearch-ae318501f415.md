# 耶格的多重租赁与弹性搜索

> 原文：<https://itnext.io/jaegers-multitenancy-with-elasticsearch-ae318501f415?source=collection_archive---------3----------------------->

我是 IBM 的一名应用架构师，致力于构建一个平台来加速高质量的持续交付，确保以现代灵活的方式实现开发运维并支持敏捷文化。

![](img/060b68c1c65e8fc8804d028b225f9e62.png)

作为规模的复杂性

在过去的一年里，由于复杂的分布式微服务环境中分布式事务监控和根本原因分析的需要，我们引入了 Jaeger 框架来帮助我们解决这个问题。由于我们的平台被多个租户使用，我们必须决定如何使用 Elasticsearch 作为后端来实现多租户 Jaeger。

这是一个关于如何使用 Elasticsearch 设置 Jaeger 以支持多个租户的实践练习。但是首先，你应该阅读下面这篇文章 [Jaeger 和多租户](https://medium.com/jaegertracing/jaeger-and-multitenancy-99dfa1d49dc0)，它讨论了 Jaeger 的各种多租户选项。

## 背景

我们正在构建和运行一个基于 Kubernetes 的平台，该平台允许我们的客户使用我们的平台构建和部署他们自己的应用程序，因此在追踪数据方面有特定的要求:

*   一个支持所有租户的弹性搜索实例，
*   每个租户的跟踪数据都必须单独持久化—具有不同的保留时间范围，
*   每个租户能够查看和查询自己的跟踪数据，
*   尽可能减少开发活动，从而重用现有的 Jaeger 功能。

## 解决方案

在从不同来源查阅了足够多的资料后，我决定采用以下解决方案，包括:

*   Elasticsearch 实例安装在租户的独立名称空间中——我们希望管理自己的 es 集群，
*   为每个租户安装 Jaeger 收集器，配置为使用具有特定租户名称/id 的 ES 集群，
*   安装 Jaeger 代理作为被跟踪服务的辅助程序，
*   为每个租户配置自己的 Jaeger 的弹性搜索索引清理设置，
*   一切都是用 0 开发工作建立起来的。

## 实施解决方案

好了，现在让我们跳到“代码”，看看我们如何配置上面的故事。对于这个练习，我已经使用 **Helm** 部署了所有组件，所以从这个角度来看代码片段。

安装 Elasticsearch 并没有什么特别的，因此你可以按照 Helm 提供的详细的在线文档来使用不同的 ES 图像风格。出于本练习的目的，我们假设已经在 *jaeger* 名称空间中安装了具有以下配置的 ES:

```
clusterName: "elasticsearch" 
nodeGroup: "master"
masterService: ""
roles:
  master: "true"
  ingest: "true"
  data: "true"
httpPort: 9200
transportPort: 9300
```

转到 Jaeger 的部分，首先我们需要告诉它与新安装的 Elasticsearch 集群一起工作。因为我们想为每个租户单独存储 ES 中的数据，所以我们必须为每个租户提供一个索引前缀，即租户的名称。

```
storage:
  type: **elasticsearch**
  elasticsearch:
    host: **elasticsearch-master.jaeger.svc.cluster.local**
    indexPrefix: **<TENANT_NAME>**
    extraEnv:
      # We need this one for the index cleaner (later in our story)
      - name: INDEX_PREFIX
        value: **<TENANT_NAME>**
```

我们之前说过，我们将为每个租户配备一个收集器。因此，我们将只启用收集器部署并禁用 Jaeger 代理组件。

```
agent:
  enabled: **false**
collector:
  enabled: **true**
  image: jaegertracing/jaeger-collector
  pullPolicy: IfNotPresent
```

我们还想为每个租户展示他们自己的 Jaeger UI，以便查看跟踪数据。

```
query:
  enabled: **true**
  image: jaegertracing/jaeger-query
  basePath: /ops/jaeger/**<TENANT_NAME>**
```

Jaeger 有一个自我 ES 索引清理组件，可以在我们的设置中为每个租户进行配置。我们只需要添加以下配置。由于这是我们的租户的相同配置和安装的一部分，因此它将清理在 env 变量 INDEX_PREFIX 下配置的 elasticsearch 索引(我们之前在设置存储类型时已经完成了这一步)。

```
esIndexCleaner:
  enabled: **true**
  image: jaegertracing/jaeger-es-index-cleaner
  schedule: "59 23 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  numberOfDays: 3
```

*如果您使用 helm 部署 Kubernetes 资源，您可以在安装时使用* `***--set key=value***` *提供您的租户名称，并在您的 Helm 图表文件中引用它。*

让我们使用 Helm 图表最后看一下为一个租户安装 Jaeger 组件所需的 yaml 配置。

```
storage:
  type: elasticsearch
  elasticsearch:
    host: elasticsearch-master.jaeger.svc.cluster.local
    indexPrefix: <TENANT_NAME>
    extraEnv:
      - name: INDEX_PREFIX
        value: <TENANT_NAME>
agent:
  enabled: false
collector:
  enabled: true
  image: jaegertracing/jaeger-collector
  pullPolicy: IfNotPresent
query:
  enabled: true
  image: jaegertracing/jaeger-query
  basePath: /ops/jaeger/<TENANT_NAME>
esIndexCleaner:
  enabled: true
  image: jaegertracing/jaeger-es-index-cleaner
  schedule: "59 23 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  numberOfDays: 3
```

剩下的唯一事情是将 Jaeger 的 **agent** 组件作为 sidecar 部署到我们的服务中，并确保将它链接到每个租户的适当收集器。这可以通过在您的部署中添加一个新容器来轻松完成，如这里的[所述](https://github.com/jaegertracing/jaeger-kubernetes#deploying-the-agent-as-sidecar)。

假设您的部署有一个名为`my-app`的应用程序在端口`8080`上运行一个来自`yourimagerepository/hello-my-image`的容器，您将需要添加额外的`jaeger-agent`容器，其中的参数需要指向正确的收集器。

*根据您的架构需求，您可以从每个 pod 部署一个代理或每个 pod 集部署一个代理甚至每个集群节点部署一个代理中受益。*

```
apiVersion: apps/v1
kind: Deployment
...
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/name: my-app
    spec:
      containers:
      - image: yourimagerepository/hello-my-image
        name: my-app-cntr
        ports:
        - containerPort: 8080
      - image: jaegertracing/jaeger-agent:1.17.0
        name: jaeger-agent
        resources:
          limits:
            cpu: 20m
            memory: 20Mi
        args: ["--reporter.grpc.host-port=jaeger-**<TENANT_NAME>**-collector.jaeger.svc.cluster.local:14250"]
```

这就是为多租户目的部署 Jaeger 和 Elasticsearch 的全部内容。为每个租户重复上述过程可能会很乏味且容易出错，因此考虑将所有这些更改移入一个舵图中，并将其作为附加组件部署到您的租户中。

## “做吧。或者不要。没有尝试。”

配置生产设置时要考虑的其他方面。

*   因为 Jaeger 的 UI 没有任何认证/授权机制，所以你需要保护它。然而，这是很容易的，因为耶格的头盔图内置了对入口资源的支持，并且有多种保护它们的方法。
*   检查 Elasticsearch 安全性以防止对您的 Elasticsearch 集群的未授权访问。
*   验证 Elasticsearch Index Cleaner 作业是否正在执行并删除了旧数据。您可以检查每个租户的 ES 索引是否被删除。

```
**kubectl logs jaeger-tenant1-es-index-cleaner-1596412740-29njp -n jaeger**
Removing tenant1-jaeger-service-2020-07-31
Removing tenant1-jaeger-span-2020-07-31**kubectl logs jaeger-tenant2-es-index-cleaner-1596412740-mdl8l -n jaeger**
Removing tenant2-jaeger-span-2020-07-31
Removing tenant2-jaeger-service-2020-07-31
```

*特别感谢*🍺*感谢我的同事* [*乔治·萨夫塔*](https://medium.com/@saftageorge) *为实现它所做的工作。*
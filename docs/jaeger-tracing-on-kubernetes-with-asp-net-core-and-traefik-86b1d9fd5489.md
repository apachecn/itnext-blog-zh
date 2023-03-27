# 用 ASP.NET 岩心和 Traefik 对 Kubernetes 进行 Jaeger 示踪

> 原文：<https://itnext.io/jaeger-tracing-on-kubernetes-with-asp-net-core-and-traefik-86b1d9fd5489?source=collection_archive---------3----------------------->

## 可能是艰难的方式

![](img/91503b49bb8b669e21eb0d6ccaceb0a4.png)

*这是我之前写的一篇* [*文章*](https://medium.com/imaginelearning/jaeger-tracing-on-kubernetes-with-net-core-8b5feddb6f2f) *的后续文章，其中使用了 Jaeger 一体化部署。本教程将演示一个生产就绪的 Jaeger 部署，以及一些改进。net 核心代码。与前一篇文章不同，我用 Elasticsearch 将 Jaeger 收集器、查询和代理分成单独的组件进行存储。我们还为*[*Traefik*](https://traefik.io/)*添加了对 Jaeger 跟踪的支持，以通过我们的边缘路由器跟踪轨迹。*

*值得一提的是，Jaeger 空间变化很快，在你的集群上设置 Jaeger 可能有更简单的方法，比如* [*这个*](https://medium.com/jaegertracing/jaeger-operator-for-kubernetes-b3128f3c4943) *。*

# 耶格特工

Jaeger 代理监听来自客户端的跨度，并将它们发送到 Jaeger 收集器。

代理 yaml 基本上与模板相同，只是添加了一些资源限制，并将配置嵌入到一个秘密中，如果您有敏感的配置数据，如 elasticsearch 密码，这就更好了。如果你正在考虑是使用边车还是 daemonset，[这篇](https://medium.com/jaegertracing/deployment-strategies-for-the-jaeger-agent-1d6f91796d09)是一篇比较两者区别的好博文。我们选择使用 daemonset，作为 daemonset，我们可以通过 Kubernetes downwards api 访问代理 ip，如下所示:

通过 env 变量访问代理/节点 ip。

这使得 pods 能够访问运行它们的节点 ip，即 jaeger 代理主机 ip。

# 耶格收集器

Jaeger 收集器从 Jaeger 代理中提取跟踪信息，并对其进行验证、索引、转换和存储。对于我们的存储，我们使用了现有的 Elasticsearch 集群。

同样，这类似于这里提供的收集器模板，除了我们添加了资源限制和请求(我们发现收集器不是非常资源密集型的)并从秘密中提取配置值。如果您的 Elasticsearch 实例是通过 TLS 访问的，那么您需要在 Jaeger 收集器上安装您的证书，并包含`--es.tls`标志。我们还挂载了一个采样策略 json 文件来支持远程采样。远程采样允许您直接从收集器动态控制不同服务和端点的采样策略。这意味着您可以更改跟踪吞吐量，而不必重新启动服务。

# 耶格查询

我们在这里做了一些小的改动，即把服务隐藏在入口后面，这样我们就可以很容易地在一个组织的主机名中公开我们的 UI。我们还通过基本认证保护了 Jaeger UI。类似于耶格收集器，我们需要`--es.tls`旗帜和安装证书。

配置 jaeger 使用 elasticsearch 需要什么？基本上，您需要告诉 Jaeger 查询和收集器在哪里查找证书，如下所示:

# 用 Jaeger 配置 Traefik

如果您想通过您的边缘路由器(在本例中为 Traefik)跟踪您的服务，您可以这样做。以下几行可以添加到您的`traefik.toml`文件中。我们将跟踪类型设置为速率限制，并对 1/1000 个请求进行采样。

Traefik 还需要知道 Jaeger 代理主机和端口。对我们来说，这些值存在于环境变量中(参见上面的 Jaeger Agent 部分)。这意味着我们不能在 toml 文件中设置`localagenthostport`或`samplingserverurl`配置值，而是必须将它们作为参数传递给 docker 映像。

设置好 Treafik 后，下一步是将其跟踪链接到您的服务。因为边缘路由器应该作为所有服务子跨度的父出现。

# Asp。网络核心代码

为了让其他开发者更容易使用 Jaeger，我建立了一些默认值和配置，可以在`startup.cs`的依赖注入中使用。为此，我首先定义了一些默认的 Jaeger 选项，如下所示:

此外，我实现了 IServiceCollection 扩展，简化了 Jaeger 的初始设置。

我们的默认设置包括一个`GuaranteedThroughputSampler`和一个配置了 jaeger 代理主机和端口的报告器。此外，添加`services.AddOpenTracing()`将能够自动跟踪我们的端点。然而，这意味着您的`/health`和`/metrics`端点也将生成踪迹。所以为了避免这些痕迹的膨胀，你可以通过指定忽略模式来忽略它们。现在我们已经有了易用的代码，我们可以在`startup.cs`中用下面几行简单的代码设置 Jaeger:

通过设置`options.SamplingRate`和`options.LowerBound`覆盖默认值。

感谢您的阅读，希望这能简化您使用 Jaeger tracing 的过程。

# 额外资源

*   将 Jaeger 部署到您的 Kubernetes 集群看起来是一种令人兴奋且可能更简单的方式
*   改进的 [Elasticsearch 和 Kibana](https://medium.com/jaegertracing/jaeger-elasticsearch-and-kibana-7ecb846137b6) 支持
*   [耶格部署](https://www.jaegertracing.io/docs/1.8/deployment/)
*   [https://github.com/jaegertracing/jaeger-kubernetes](https://github.com/jaegertracing/jaeger-kubernetes)
*   [https://github.com/jaegertracing/jaeger-client-csharp](https://github.com/jaegertracing/jaeger-client-csharp)
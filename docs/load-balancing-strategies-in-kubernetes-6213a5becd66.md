# Kubernetes 中的负载平衡策略

> 原文：<https://itnext.io/load-balancing-strategies-in-kubernetes-6213a5becd66?source=collection_archive---------0----------------------->

## L4 循环法、L7 循环法、环形散列法等等

负载平衡是在多个后端服务之间高效分配网络流量的过程，是最大化可扩展性和可用性的关键策略。在 Kubernetes 中，有多种选择可以对 pods 的外部流量进行负载平衡，每种选择都有不同的权衡。

# 使用 kube 代理的 L4 循环负载平衡

在一个典型的 Kubernetes 集群中，发送到 Kubernetes 服务的请求由一个名为`[kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)`的组件路由。有些令人困惑的是，`kube-proxy`不是传统意义上的代理，而是通过 iptables 规则为服务实现虚拟 IP 的过程。这种架构给路由增加了[额外的复杂性。每个请求都会引入少量的延迟，随着服务数量的增加，延迟也会增加。此外，`kube-proxy`在 L4 (TCP)路由，这不一定很适合今天以应用为中心的协议。例如，假设两个 gRPC 客户端连接到您的后端 pod。在 L4 负载平衡中，每个客户端将被发送到不同的后端 pod(循环)。即使一个客户端每分钟发送 1 个请求，而另一个客户端每秒发送 100 个请求，也是如此。](https://jvns.ca/blog/2017/10/10/operating-a-kubernetes-network/)

那么为什么要使用`kube-proxy`呢？一个字:简单。负载平衡的整个过程委托给 Kubernetes，这是默认策略。因此，无论您是通过 Ambassador 还是通过其他服务发送请求，您都要经历相同的负载平衡机制。

# L7 循环负载平衡

如果您使用 gRPC 或 HTTP/2 之类的多路复用保活协议，并且需要更公平的循环算法，该怎么办？你可以为 Kubernetes 使用 API 网关，比如 [Ambassador](https://www.getambassador.io/) ，它可以完全绕过`kube-proxy`，将流量直接路由到 Kubernetes pods。Ambassador 建立在 Envoy Proxy 之上，这是一个 L7 代理，因此每个 gRPC 请求在可用的 pods 之间进行负载平衡。

在这种方法中，您的负载平衡器使用 [Kubernetes 端点 API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/#endpoints-v1-core) 来跟踪 pod 的可用性。当对特定 Kubernetes 服务的请求被发送到您的负载平衡器时，负载平衡器会在映射到给定服务的 pods 之间循环交换请求。

# 环形散列

环哈希负载平衡策略不是在不同的 pod 之间轮换请求，而是使用哈希算法将来自给定客户端的所有请求发送到同一个 pod。环形散列方法既用于“粘性会话”(其中设置 cookie 以确保来自客户端的所有请求到达相同的 pod)，也用于“会话相似性”(依赖于客户端 IP 或客户端状态的某个其他部分)。

哈希方法对于维护每个客户端状态的服务(例如，购物车)非常有用。通过将相同的客户端路由到相同的 pod，给定客户端的状态不需要跨 pod 同步。此外，如果您在给定的 pod 上缓存客户端数据，缓存命中的概率也会增加。使用环哈希的缺点是，在不同的后端服务器之间平均分配负载可能更具挑战性，因为客户端工作负载可能不相等。此外，散列的计算成本增加了请求的延迟，尤其是在大规模的情况下。

# 磁力悬浮火车

和环哈希一样，maglev 也是一种[一致哈希算法](https://en.wikipedia.org/wiki/Consistent_hashing)。maglev 最初由 Google 开发，设计目的是在哈希表查找上比环形哈希算法更快，并最小化内存占用。环形哈希算法生成相当大的查找表，不适合您的 CPU 处理器缓存。对于微服务，Maglev 有一个相当昂贵的权衡:当一个节点出现故障时，生成查找表的成本相对较高。鉴于 Kubernetes 豆荚短暂的本质，这可能行不通。关于不同一致性散列算法的权衡的更多细节，[本文](https://medium.com/@dgryski/consistent-hashing-algorithmic-tradeoffs-ef6b8e2fcae8)详细介绍了用于负载平衡的一致性散列，以及一些基准。

# 了解更多信息

Kubernetes 中的网络实现比第一次出现时更复杂，也比许多工程师理解的更有限。Matt Klein 撰写了一篇内容丰富的博客文章"[现代网络负载平衡和代理介绍](https://blog.envoyproxy.io/introduction-to-modern-network-load-balancing-and-proxying-a57f6ff80236)"，为理解关键概念提供了坚实的基础。还有一系列额外的帖子解释了为什么组织选择使用第 7 层感知代理来负载平衡入口流量，如 [Bugsnag](https://blog.bugsnag.com/envoy/) 、 [Geckoboard](https://medium.com/geckoboard-under-the-hood/we-rolled-out-envoy-at-geckoboard-13c45b4eaddd) 和 [Twilio](https://www.twilio.com/blog/2017/10/http2-issues.html) 。

# 大使 API 网关

Ambassador 建立在 Envoy 代理之上，是一个开源 API 网关，支持 Kubernetes 上讨论的所有负载平衡方法。请访问[https://www . getambassador . io](https://www.getambassador.io/)或[加入我们的 Slack 频道](http://d6e.co/slack)了解更多信息。
# Kubernetes 上 Kafka Streams 应用程序的运行状况检查

> 原文：<https://itnext.io/health-checks-for-kafka-streams-application-on-kubernetes-e9c5e8c21b0d?source=collection_archive---------0----------------------->

这是一个由三部分组成的博客系列，涵盖了 Kafka Streams 在 Kubernetes 上的应用

*   第 1 部分— [如何在 Kubernetes 上运行 Kafka Streams 应用程序](/learn-how-to-develop-a-kafka-streams-application-for-data-processing-and-deploy-it-to-kubernetes-231d4cbb0688)
*   第 2 部分— [如何在 Kubernetes 上运行有状态的 Kafka Streams 应用程序](/how-to-use-kubernetes-to-deploy-stateful-kafka-streams-applications-872c77f03c3a)
*   第 3 部分—这篇博文

在之前的博客中，我们探索了如何使用 Kubernetes 原语运行由 Kafka Streams 支持的有状态流处理应用。权力越大，责任越大！如果你使用[交互式查询](https://kafka.apache.org/23/documentation/streams/developer-guide/interactive-queries#interactive-queries)和一个有状态的 Kafka Streams 应用程序，你需要注意这样一个事实:如果你的实例正在从 changelog 主题中重新平衡/恢复/调整它的本地状态，那么本地状态存储并不处于可查询状态。

您可以使用 Kubernetes [探测器](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#probe-v1-core)来配置您的应用程序并妥善处理这个问题。

# 就绪探测

Kubernetes 使用就绪探测器来检测容器何时准备好开始接受流量。将此应用到 Kafka Streams 应用程序将确保只有在满足某些(用户定义的)标准后，应用程序实例才会注册到 Kubernetes `Service`。如果一个实例由于重新平衡而不可用，它的 REST 端点将不会暴露给客户端(通过`Service`)进行交互查询。

> *此时，只有您的应用程序的部分状态可以通过通过准备就绪探测的实例进行查询*

容器启动后，Kubernetes 将在触发准备就绪探测前等待`initialDelaySeconds`中指定的持续时间，并在每隔`periodSeconds`后重复一次，直到成功或超时(`timeoutSeconds`)。

# 活性探针

即使在你的 Kafka Streams 应用程序(及其所有实例)启动并运行后，由于多种原因——成员加入、离开等，可能会有重新平衡。在这种情况下，您可能希望从交互式查询的角度检查实例是否处于就绪状态。您可以配置一个活跃度探测器，就像它的准备就绪对应物一样(甚至为它们使用相同的机制，例如`/health`端点),并且它的工作方式也是相同的(除了当探测器失败时会发生什么)

> *Kubernetes v1.16 增加了一个* `[*Startup Probe*](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#when-should-you-use-a-startup-probe)` *，可以用来检查容器进程是否已经启动。它可以与活跃度和就绪性探测器一起使用，但值得一提的是，如果进行了配置，它会禁用活跃度和就绪性检查，直到成功，确保这些探测器不会干扰应用程序的启动*

# 探针配置

要使这些探测器工作，您需要:

*   实施检查机制，以及
*   在 Kubernetes ( `StatefulSet`)清单中配置探测器

> *为了进行测试，您可以按照下面提到的步骤*更新 [*有状态 Kafka 流*](https://dev.to/azure/learn-about-the-kubernetes-components-required-to-run-stateful-kafka-streams-applications-44jl) *中的应用程序*

# 探测器的运行状况检查端点

Kubernetes 支持 HTTP 端点、TCP 套接字和任意命令执行作为健康检查探针。对于我们的 Kafka Streams 应用程序，就健康检查而言，通过 REST 端点公开状态存储状态信息非常有意义

```
@GET("health")
public Response health() {
    Response healthCheck = null;
    KafkaStreams.State streamsAppState = KafkaStreamsBootstrap.getKafkaStreamsState();
    if (streamsAppState.name().equals(RUNNING_STATE)) {
        healthCheck = Response.ok().build();
    } else {
        healthCheck = Response.status(Response.Status.SERVICE_UNAVAILABLE)
                .entity(streamsAppState.name())
                .build();
    }
    return healthCheck;
}
```

# 探测器配置清单

下面是就绪探测器的样子(同样适用于活动探测器)

```
readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 5
      timeoutSeconds: 2
      failureThreshold: 5
```

虽然这些调查将缓解问题，但在处理 Kafka Streams 应用程序(尤其是)时，您应该注意一些极端情况。涉及大量有状态计算的。

# 警告

您的 Kafka Streams 状态可能非常大，以至于它的重新平衡可能会超出就绪性和/或活性探测器指定的阈值/约束。以下是在这些情况下可能发生的情况:

*   就绪探测被破坏:在 Kubernetes 停止执行之前，该探测将考虑`failureThreshold`计数，并将 Pod 标记为`Unready`，从而使其不可用。
*   活性探测器被破坏:在这种情况下，Kubernetes 在`failureThreshold`后重启 Pod。因此，您的应用程序可能会陷入一个试图恢复状态的循环中，但是由于 Kubernetes 重新启动(由于探测)Pod，它会被中断(并且它必须重新开始！)

# 可能的解决方案

如果与 Kafka Streams 应用程序相关的状态存储在外部(例如，`PersistentVolume`由持久的云存储支持)，则可以在很大程度上避免冗长的重新平衡

> [*本博客*](/how-to-use-kubernetes-to-deploy-stateful-kafka-streams-applications-872c77f03c3a) *详细介绍了*

另一个关键点是测试您的应用程序，注意它的状态以及恢复/重新平衡/调整状态可能需要的时间。然后，您可以做出明智的决定并调整探头配置参数(`failureThreshold`、`initialDelaySeconds`等)。)相应地，这样您的应用程序就有足够的时间在您使用探针之前实际执行恢复过程！

您还可以配置您的 Kafka Streams 应用程序，使其拥有本地状态的备用副本(使用`num.standby.replicas`配置)，这些副本是状态的完全复制副本。当重新平衡被触发时，Kafka Streams 将尝试将任务分配给已经存在这种备用副本的实例，以最小化任务(重新)初始化成本。

您应该考虑在 Kubernetes 的未来版本中使用 Startup probe(一旦稳定下来)

暂时就这样了。我真的希望你喜欢这篇文章，并从中学到了一些东西。如果你做了，请喜欢并跟随。很高兴通过 [Twitter](https://twitter.com/abhi_tweeter) 获得反馈或发表评论！
# Kubernetes:就绪探测

> 原文：<https://itnext.io/kubernetes-readiness-probe-83f8a06d33d3?source=collection_archive---------5----------------------->

如果对这个特性有任何疑问，我写这篇文章是为了说明这不是一个可选的特性，而是一个必须实现的特性。好吧，一切都是可选的，但是如果您希望您的 Kubernetes 集群按照您想要的方式运行，那么您最好实现一个就绪探测器。最棒的是，这是一个相当容易添加到代码中的特性。对于准备就绪探测，我们需要将代码添加到 Pod YAML 文件和运行在 Docker 容器中的应用程序代码中。

![](img/c91a2004b335018f83c57b744f233a8b.png)

准备好了。

# 为什么我们要添加准备就绪探测器？

现在我们已经有了在必要时自动扩展的[单元，我们的集群将在它认为合适的时候添加单元。一旦添加了一个单元，群集就需要来自该单元的信息，该单元已准备好开始将流量路由到该单元。现在，如果您的微服务需要在准备就绪之前完成配置查找或其他一些过程，那么您需要让 Kubernetes 知道它应该何时开始将请求路由到 Pod。如果您跳过准备就绪探测，并且您的 Pod 立即获得路由到它的流量，并且您的 Pod 被确定为不健康，那么 Kubernetes 将继续重新启动您的 Pod。这是个可怕的消息。相反，我们可以做一些简单的事情，添加一个](https://medium.com/@jonbcampos/kubernetes-horizontal-pod-scaling-190e95c258f5)[就绪探测器](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-readiness-probes)。

*如果你没有通读甚至没有读过本系列* *的第一部分* [*，你可能会感到困惑，对代码在哪里或者之前做了什么有疑问。记住这里假设你正在使用*](https://medium.com/@jonbcampos/kubernetes-day-one-30a80b5dcb29)[*GCP*](https://cloud.google.com/)*和*[*GKE*](https://cloud.google.com/kubernetes-engine/)*。我将始终提供代码和如何测试代码是按预期工作。*

[](https://medium.com/@jonbcampos/kubernetes-day-one-30a80b5dcb29) [## Kubernetes:第一天

### 这是 Kubernetes 帖子的必选步骤之一。如果你对 Kubernetes 感兴趣，你可能已经读过 100 本了…

medium.com](https://medium.com/@jonbcampos/kubernetes-day-one-30a80b5dcb29) 

# 服务码

对于我的例子，代码很简单…非常简单。它基本上不做任何事情，接受开始返回 200 个响应。我相信您可以想办法调用其他服务，并在用 200 response 进行响应之前设置一个状态。

```
app.use('/readiness', function (req, res, next) {
    res.status(200).json({ ready: true });
});
```

从我之前提供的[申请代码](https://github.com/jonbcampos/kubernetes-series/blob/master/partone/app.js#L29-L31)中。

## 我必须使用 HTTP 请求吗？

不。在 Pod 定义中，您可以使用 HTTP 请求(可能是最简单的选项)，也可以设置 TCP 探测，甚至运行命令脚本来验证您的 Pod 正在运行。你有很多选择，我只喜欢最简单的一个。

# Pod 定义

对 Pod 定义的更改非常小，只需一会儿就能完成。我们只需要说明在哪里以及如何测试我们的吊舱的准备状态。我在代码中添加了注释，以显示所有可以定制探针的地方。

```
# the readiness probe details
**readinessProbe:**
  **httpGet:** # make an HTTP request
    **port: 8080** # port to use
    **path: /readiness** # endpoint to hit
    **scheme: HTTP** # or HTTPS
  **initialDelaySeconds: 3** # how long to wait before checking
  **periodSeconds: 3** # how long to wait between checks
  **successThreshold: 1** # how many successes to hit before accepting
  **failureThreshold: 1** # how many failures to accept before failing
  **timeoutSeconds: 1** # how long to wait for a response
```

根据我之前提供的[Pod/部署定义](https://github.com/jonbcampos/kubernetes-series/blob/master/partone/k8s/deployment.yaml#L29-L39)。

# 后续步骤

在我的第一篇文章中，我承诺过这些细节可能会很短。这是一篇我已经知道会很短但很重要的文章。如果你刚刚进入 Kubernetes，还没有看到准备就绪探针，或者也许这是一个你认为不重要的特性，它是重要的和必要的。在发布之前把这个添加到你的默认清单中——即使它不是 Kubernetes 提供的最性感的特性。

# 本系列的其他文章

[](https://medium.com/@jonbcampos/kubernetes-day-one-30a80b5dcb29) [## Kubernetes:第一天

### 这是 Kubernetes 帖子的必选步骤之一。如果你对 Kubernetes 感兴趣，你可能已经读过 100 本了…

medium.com](https://medium.com/@jonbcampos/kubernetes-day-one-30a80b5dcb29) [](https://medium.com/@jonbcampos/kubernetes-horizontal-pod-scaling-190e95c258f5) [## Kubernetes:水平 Pod 缩放

### 通过 Pod 自动扩展，您的 Kubernetes 集群可以监控现有 Pod 的负载，并确定我们是否需要更多…

medium.com](https://medium.com/@jonbcampos/kubernetes-horizontal-pod-scaling-190e95c258f5) [](https://medium.com/@jonbcampos/kubernetes-cluster-autoscaler-f1948a0f686d) [## Kubernetes:集群自动缩放

### 自动缩放是 Kubernetes 的一个巨大的(并且已经上市的)特性。当你的网站/应用程序/应用程序接口/项目变得越来越大时，洪水…

medium.com](https://medium.com/@jonbcampos/kubernetes-cluster-autoscaler-f1948a0f686d) 

[Jonathan Campos](http://jonbcampos.com/) 是一个狂热的开发者，喜欢学习新事物。我相信我们应该不断学习、成长和失败。我总是开发社区的支持者，并且总是愿意提供帮助。因此，如果你对这个故事有任何问题或意见，请在下面提出。在 [LinkedIn](https://www.linkedin.com/in/jonbcampos/) 或 [Twitter](https://twitter.com/jonbcampos) 上与我联系，并提及这个故事。
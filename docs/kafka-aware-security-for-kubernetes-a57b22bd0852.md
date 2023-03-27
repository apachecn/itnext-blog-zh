# 面向 Kubernetes 的 Kafka 感知安全性

> 原文：<https://itnext.io/kafka-aware-security-for-kubernetes-a57b22bd0852?source=collection_archive---------6----------------------->

![](img/be38a30d1ed222bfcbada7fed4507b80.png)

miosz klin owski 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

说到卡夫卡，细粒度的安全非常重要。通常，流经 Kafka 的数据来自许多组件，并且通常包含不应该被任何没有特权的人使用的信息。在 [Cilium](https://cilium.io/) 的帮助下，我们可以直接在 Linux 内核层实施 Kafka 感知安全策略，这意味着无需对应用程序代码或容器配置进行任何更改。

使用标准的 Kafka 设置，任何用户或应用程序都可以向任何主题编写任何消息，以及从任何主题读取数据。

保护 Kafka 的一种典型方法是为每个客户颁发 SSL 证书，并强制 Kafka 经纪人验证其有效性。另一个选择是使用 SASL，它有几种不同的应用方式(经典用户名/密码，Kerberos 等)，但是开始并不容易。这只是身份验证，向授权迈进—事情变得更加复杂(在更大范围内使用 kafka-acl 命令具有挑战性)。让我们看看怎样才能做到和卡夫卡完全不同，完全独立。

我们将使用一个名为 Cilium 的工具，它使用 eBPF 技术在一个完全不同的层面上保护 Kafka。首先说一下纤毛本身。一个很好的解释是在他们自己的[网站](https://cilium.io/):

> 现有的 Linux 网络安全机制(例如，iptables)仅在网络和传输层(即，IP 地址和端口)运行，并且缺乏对微服务层的可见性。
> 
> Cilium 为 Linux 容器框架带来了 API 感知的网络安全过滤，如 [**Docker**](https://www.docker.com/) 和 [**Kubernetes**](https://kubernetes.io/) 。使用名为 **eBPF** 的新 Linux 内核技术，Cilium 提供了一种简单有效的方法来定义和实施基于容器/pod 身份的网络层和应用层安全策略。

你需要了解的是，Cilium 是一个可以安装在 Kubernetes 上的工具，它使用 eBPF 来实现下一级或云原生网络过滤。那 eBPF 是什么？最佳解释:布兰登·格雷格:

> eBPF 对 Linux 的作用就像 JavaScript 对 HTML 的作用一样。(算是吧。)因此，与静态 HTML 网站不同，JavaScript 让你定义在鼠标点击等事件上运行的迷你程序，这些程序在浏览器中的安全虚拟机中运行。使用 eBPF，而不是一个固定的内核，您现在可以编写迷你程序，运行在发送/接收 TCP 包等事件上，这些事件运行在内核中的安全虚拟机上。实际上，eBPF 更像是运行 JavaScript 的 v8 虚拟机，而不是 JavaScript 本身。eBPF 是 Linux 内核的一部分。

那么，这一切如何转化为在 Kubernetes 上保护卡夫卡呢？通过使用 eBPF 技术，Cilium 可以直接与 Linux 内核对话，并对 pod 应用安全策略，如“pod X 只能从 Kafka topic Y 读取，pod Z 只能向 Kafka topic W 写入”。所有这些都是通过在我们的 Kubernetes 集群上应用简单的 yaml 定义来实现的。

![](img/de8a5b863e1b81ae40c23437fe87fdd1.png)

由[科林·阿姆斯特朗](https://unsplash.com/@brazofuerte?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

我们现在已经有足够的理论了，所以让我们进入正题。假设您在 Kubernetes 集群上运行 Kafka 集群，那么您需要做的第一件事当然是安装 Cilium。我不会在这里讨论它，因为按照[纤毛文件](https://docs.cilium.io/en/v1.7/gettingstarted/#installation)上的说明，它是非常简单的。如果您已经安装了 Cilium，就可以用一个(或多个)简单的 YAML 文件来保护您的 Kafka 集群了。正如我前面提到的“使用标准的 Kafka 设置，任何用户或应用程序都可以向任何主题编写任何消息，以及从任何主题读取数据”。所以如果你刚刚安装了卡夫卡，它是敞开的。假设我们想要一个只允许写两个特定主题的生产者，和两个只能读两个主题之一的消费者。这是一个非常简单的例子，只是为了向您展示从完全不安全的 Kafka 集群到完全安全的 Kafka 集群有多容易。要实现以上目标，我们唯一需要做的就是应用(*ku bectl apply-f make-my-Kafka-secure . yaml*)这个简单的 YAML 文件:

那么当我们这么做的时候会发生什么呢？Kubernetes 将创建一个新的 CiliumNetworkPolicy 资源，Cilium 将获取该资源，然后监控所有 Kafka pods(它将通过标签找到它们),并阻止它们执行除 YAML 文件的规则部分中指定的操作之外的任何操作:

让我们回到整个 YAML 的档案，并把它分解成碎片。

这很简单，对吧？作为任何 Kubernetes 资源，我们需要在元数据部分指定 apiVersion、描述、种类和至少名称。

接下来，在 spec 部分，我们需要告诉 Cilium 如何通过在“endpointSelector”中提供 Kafka broker 的 pod 标签来找到它。

然后针对该经纪人应用什么规则。在“ingress”>“from endpoint”中，我们指定了我们的客户端——也就是将连接到我们的 Kafka 集群的 pods(因此可以向/从 Kafka 生产或消费某些东西)。“toPorts”只是告诉 Kafka 监听哪个端口，然后在规则中，我们基本上告诉“fromEndpoints”中指定的 pod 将被允许做什么。以下是可能的规则参数的完整列表:

 [## 第 7 层示例- Cilium 1.7.0 文档

### 第 7 层策略建立了关于哪些端点可以相互通信的基本连接规则…

docs.cilium.io](http://docs.cilium.io/en/latest/policy/language/#kafka-beta) 

…对 Kubernetes 应用 Kafka 感知的安全策略就是这么简单。像“允许一个 pod 只在 Kafka 主题“topicA”上生产，在主题“topicB”上消费”这样的逻辑很容易用纤毛实现。

您还需要记住，无论是容器中的应用程序代码还是 pod 配置都没有更改，但是我们通过简单地应用 YAML 配置，获得了对 Kafka 设置的身份验证和授权的完全控制。最后但同样重要的是，当消费者(或生产者)试图做一些不被允许的事情时，它会收到一条真实的 Kafka 错误消息，比如:

> 提取相关 id 为 20 的元数据时出现警告错误:{ example-TOPIC = TOPIC _ AUTHORIZATION _ FAILED }(org . Apache . Kafka . clients . network client)

这里我们来了解一下 Cilium 是如何工作的——它了解群集内的整个流量，并可以相应地保护它。不仅限于卡夫卡，也理解卡夫卡。对于 HTTP 流量，您可以用几乎相同的方式使用它(想想像“pod A 只能在/info 端点上对 pod B 进行 GET 调用”这样的规则)。如果您对它的所有功能感兴趣，请在 Cilium 网站上阅读更多内容，或者等待我的下一篇文章；)

 [## 欢迎使用 Cilium 的文档！-纤毛 1.7.0 文档

### 文档分为以下几个部分:

纤毛。readthedocs.io](https://cilium.readthedocs.io/en/stable/)
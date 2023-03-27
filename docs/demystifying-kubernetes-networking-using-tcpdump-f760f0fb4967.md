# 使用 tcpdump 揭开 Kubernetes 网络的神秘面纱

> 原文：<https://itnext.io/demystifying-kubernetes-networking-using-tcpdump-f760f0fb4967?source=collection_archive---------1----------------------->

你有没有想过在 Kubernetes 环境中，数据包是如何到达目的地的？一个数据包在从它的 Kubernetes 节点“退出”之前要经过多少跳？像 [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) 这样的组件和像 [Istio](https://istio.io/) 和 [Linkerd](https://linkerd.io/) 这样的服务网格解决方案如何影响这个数据包的流量？

观察包的最好方法之一当然是 tcpdump。在本文中，我们将试图揭开 Kubernetes 和 Istio 网络行为的神秘面纱，并通过使用来自 tcpdump 的 TCP 流来了解这些工具在实践中是如何工作的。

在我们进一步讨论用例之前，描述一下我们的实验室环境是有意义的。我们的 Kubernetes 集群在版本`1.21`上，它在 iptables 模式下运行 kube-proxy，而 Istio 在版本`1.13.4`上。

**从吊舱内部观察**

对于我们的第一个实验，我们将有一个默认的 Nginx 安装，我们将使用它作为目标；另一个独立的 ubuntu pod 将是我们数据包的来源。我们的 Nginx Kubernetes 服务以`10.0.185.118`为 IP，Nginx pod `10.244.4.224`和 ubuntu pod 将会是`10.244.10.89`。

我们将从 ubuntu pod 向 Nginx Kubernetes 服务发出一个简单的 HTTP `HEAD`请求开始。在 ubuntu pod 中，我们将并行运行一个 tcpdump 会话，在该会话中，我们可以看到数据包飞过。

使用`curl`完成 HTTP 请求后，我们看到下面的 TCP 流:

从 Pod 内部取出的 Tcpdump

让我们深入了解一下我们在这里看到了什么。

前四个包是 UDP 包，它们是 DNS 查询和应答，由`curl`触发以获得我们想要的服务的 IP。接下来的三个包是标准的 [TCP 三次握手](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_establishment)，接下来的四个包是保存 HTTP 信息的实际包，最后三个包用于[TCP 连接终止](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_termination)。就 TCP 流而言，没有什么值得观察的。

**从 Kubernetes 节点内部观察**

但是，我们如何从服务的 IP ( `10.0.185.118`)到分配给特定 Nginx pod 的 IP(`10.244.4.224`)。在流中，我们没有看到任何 IP 属于 Nginx pod ( `10.244.4.224`)的数据包。

让我们试着深入一个层次，从 Kubernetes 节点层次观察事物。

从 Kubernetes 节点获取的 Tcpdump

第一个容易观察到的现象是，每个数据包似乎出现了四次。幸运的是，最新的 tcpdump 特性之一允许我们看到数据包来自的接口。如您所见，单个数据包在离开 Kubernetes 节点之前会经过四个接口。使用`ip a`查看我们的节点中的接口，我们看到几十个接口，其中大多数接口的名称中都有一个`veth*`前缀。

Kubernetes 节点接口

这些是大多数 Kubernetes [CNIs](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni) 为每个节点中调度的每个 pod 创建的虚拟接口。如果我们也使用`brctl show`查看我们节点中的桥，我们会看到所有的`veth*`虚拟接口都是`cbr0`桥的一部分。

Kubernetes 节点桥

该信息也可在`ip a`输出中获得，其中指定该虚拟接口的主接口是`cbr0`。如果我们在系统中看到使用`ip route`的现有路由，我们会看到`eth0`用于“退出”节点。
`eth0`和`enP41291s1`似乎也是成对的，因为我们看到`eth0`被声明为`enP41291s1`的主机。
所以现在我们知道了，为了让一个 pod 内的数据包离开节点，它从它的虚拟接口到网桥，然后到节点的接口`eth0`和`enP41291s1`。

第二个观察结果是，在前面的流中，我们从 pod 内部看到了服务的 IP 和端口。

带有服务 IP 的单个 TCP 数据包

此外，在下一个数据包中，我们看到目的 IP 和端口已被转换为 pod 的 IP 和端口。什么会导致这种情况发生？

带有 pod IP 的单个 TCP 数据包

如果你使用 Kubernetes 已经有一段时间了，你应该熟悉它的一个核心组件 kube-proxy。Kube-proxy 负责完成这项翻译工作。在我们的实验室中，kube-proxy 设置为使用 iptables，所以如果我们检查 Kubernetes 节点中的 iptables 规则，我们将看到下面的行。

kubernetes 服务的 iptables 规则

[iptables 模式下的 Kube-proxy](https://kubernetes.io/docs/concepts/services-networking/service/#ips-and-vips)确保我们始终拥有将服务 IP 转换为 pods IPs 的最新规则。

**Istio 观察**

如今，在 Kubernetes 上运行服务网格解决方案是很常见的事情。这些解决方案对我们的 TCP 流有什么影响吗？

让我想想…

在我们的实验室中，我们将使用 Istio，但 Linkerd 应该具有大致相同的行为。我们将再次使用 ubuntu pod，我们将在相同的 Kubernetes 名称空间中针对一个简单的 Nginx 服务，该服务启用了 Istio，因此其中的所有 pod 都有 Istio。
让我们从 ubuntu pod 使用`curl`启动一个 HTTP `HEAD`请求，并使用 tcpdump 从 Kubernetes 节点查看数据包流。

Istio 注入 pod 的 Kubernetes 节点内的 Tcpdump

基于上述流程，我们可以进行一些有趣的观察。

第一个观察结果是，我们没有看到以服务的 IP 为目的地的数据包。与前面来自 Kubernetes 节点的 TCP 流不同，这里我们看到 3 次握手部分中的第一个数据包将 pod 的 IP 作为目的地。因此，不知何故，我们跳过了 kube-proxy 提供的从服务 ip 到 Pod IP 的转换，如上所述。

如果我们知道 Istio 的一些关键特征，就可以解释这一点。第一个是 Istio 安装自己的 iptables 规则，以便在 pod 初始化期间借助`istio-init`容器拦截所有流量。它安装的 iptables 规则可以通过一个简单的`kubectl logs <pod-name> -c istio-init`找到。第二个特性是 Istio 拥有所有 Kubernetes 服务及其相关端点(pods IPs)的更新地图。任何人都可以使用`istioctl proxy-config endpoints <pod-name>`检查`istio-proxy`知道哪些端点。为了保持这个地图不断更新`istio-proxy`集装箱不断与 istiod pod 对话。如果您启动 tcpdump 并使用端口`15012` (Istio 的[配置端口](https://istio.io/latest/docs/ops/deployment/requirements/#ports-used-by-istio))过滤数据包，就可以很容易地观察到这一点。
知道了这些，我们就可以理解，当 Istio 截获一个以服务 IP 为目的 IP 的数据包时，它会直接使用一个 pod 的 IP 来启动连接。

第二个观察结果是，与没有任何 Istio 的流相比，我们看到了更多的数据包。这与 Istio 增加的 TLS 开销有关，因为[Istio pod 之间的所有通信都是通过 TLS](https://istio.io/latest/docs/concepts/security/#mutual-tls-authentication) 进行的。

我们观察到的最后一点是，我们没有看到关闭连接的`FIN` TCP 数据包。虽然`curl`已经退出并且连接应该已经关闭，但是我们似乎保持连接打开。

现在，我们最后的观察是有趣的，所以让我们尝试一些额外的东西来获得更多的数据。我们将在一分钟内触发多个请求，而不是只做一个请求。下面是我们在运行请求循环时看到的几个单独 curl 请求的 TCP 数据包。

具有多个 curl 请求的 Istio 注入 pod 的 Tcpdump 内部节点

同样，在我们的最后一个流中有一些有趣的观察。

第一个问题是，我们似乎没有为每个请求建立新的 TCP 连接。在一个`while true`循环中使用 bash 中的普通`curl`,人们可能会认为每次`curl`调用都会创建一个新的 TCP 连接。上面的 TCP 流表明已经建立了一个 TCP 连接，我们使用这个连接。这也解释了我们在数据包完成后没有看到任何`FIN`数据包的事实。

第二个观察结果是，我们有流向不同目的 IP 的流，它们属于我们服务的不同 pod。因此，尽管 DNS 返回服务 IP，我们的 TCP 连接直接对我们服务的不同 pod 开放。这是 Istio 在本地保存每个服务的端点并如前所述直接使用它们的另一个证据。

我们的观察都表明 Istio 维护的连接池的存在。

虽然上面的 tcpdump 捕获是从节点级别进行的，但是让我们仔细检查一下 pod 内部的情况，我们可以看到还没有应用 iptables 规则的本地接口的流量。

让我们看看来自 pod 网络内部的数据包:

我们在这里可以观察到，虽然我们在`lo`接口上看到了 3 次握手数据包，但我们没有看到它们通过`eth0`接口，该接口用于退出 pod 的网络。还值得注意的是，第一个 SYN 包的目的端口是`15001`，这是[默认的 istio-proxy 监听端口](https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/)。

实际情况是，为了让 Istio 保持连接池对任何其他容器透明，使用 iptables 拦截所有流量，并拥有自己的连接，这些连接不同于主容器启动的连接。

在本文中，我们试图使用一个一直以来最受欢迎的工具 tcpdump 来验证和观察 Kubernetes 世界中与网络相关的一些理论和工具。当然，这只是我们在 Kubernetes 世界中看到的网络用例的一小部分，但是希望在阅读完这篇文章之后，您会对将来如何在 Kubernetes 中调试和解释网络行为有一个更好的想法。
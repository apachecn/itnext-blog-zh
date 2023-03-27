# azure—Kubernetes 上的物联网边缘工作负载

> 原文：<https://itnext.io/azure-iot-edge-workloads-on-kubernetes-1065b801cf4f?source=collection_archive---------7----------------------->

![](img/bd5e9ca7ee1cc43a3d93f34b802c1874.png)

Azure 物联网与 Kubernetes 的集成——Kubernetes 上 Edge 的不同解决方案

客户和市场需求通常要求物联网解决方案能够快速部署新功能和更新。Kubernetes 提供了一个统一的部署模型，允许边缘用户使用 Kubernetes 原生原语快速、自动地部署新服务并管理其生命周期，这带来了更高级别的管理功能。Kubernetes 支持滚动更新形式的零停机部署，这允许关键任务物联网解决方案保持最新，而不会对最终用户(客户)产生影响。除了滚动更新，Kubernetes 还提供了丰富的特性集，如高可用性、自动伸缩、入口等。

许多物联网解决方案被认为是需要可靠和可用的业务关键型系统。例如，对工厂运营至关重要的物联网解决方案需要随时可用。Kubernetes 提供了部署高可用性服务所需的工具。它的架构还允许工作负载独立运行。此外，它们可以重新启动或重新创建，对最终用户没有任何影响。

# 这与 Kubernetes 现有的基于虚拟 Kubelet 的 Azure 物联网边缘连接器有何不同？

Azure 已经有了一个基于虚拟 Kubelet 的实现来支持通过 Kubernetes 的物联网边缘部署。 [Azure IoT Edge Connector](https://github.com/Azure/iot-edge-virtual-kubelet-provider) 利用虚拟 Kubelet 项目提供由 Azure IoT hub 支持的虚拟 Kubernetes 节点。它将 Kubernetes pod 规范转换为物联网边缘部署，并将其提交给后台物联网中心。边缘部署包含设备选择器查询，该查询控制部署将应用到边缘设备的哪个子集。

![](img/ad0af6263d13729a1e7715ba637ca589.png)

Azure Kubernetes —基于虚拟 Kubelet 的物联网边缘实施

这里，本地设备必须运行物联网运行时和 Docker as CRI，边缘部署创建为 Docker 容器。

[Kubernetes 上的 IoTedge 网关](https://github.com/Azure-Samples/iotedge-gateway-on-kubernetes)(作为网关部署在 Kubernetes 上的 IoT Edge)使用户能够将 Azure IoT Edge 工作负载部署到本地的 Kubernetes 集群，而无需虚拟节点(虚拟 kubelet)。IoT Edge 注册了一个自定义资源定义(CRD ),代表 Kubernetes API 服务器的边缘部署。此外，它还提供了一个[操作符](https://coreos.com/blog/introducing-operators.html)(物联网边缘代理)，用于协调云管理的期望状态和本地集群状态。有了这个解决方案，EdgeAgent 现在可以与 docker 和 Kubernetes 运行时对话。这种解决方案还以某种方式消除了对主机上特定包的需求(iotedge、iotedgectl 等)。)因为该架构涉及与 Kubernetes API 的直接交互，并且所有控制平面组件都作为 pod 和容器部署在 Kubernetes 上。

该解决方案使用户能够将运行在边缘设备上的 Kubernetes 集群注册为 Azure 上的物联网边缘设备，并管理从中央 Azure 门户到分布式边缘设备的应用部署。所有的 edge 模块都无缝地转换成本地 Kubernetes 对象，方便用户根据 Kubernetes 规范直接操作它们。

# 架构和组件

传统的 Azure 物联网边缘运行时构成了物联网边缘代理:实例化模块，管理模块的生命周期和监控，向物联网中心报告模块状态。edgeAgent 使用它的模块 twin 来存储这些配置数据。第二个是物联网边缘中心(IoT Edge hub):通过公开与物联网中心相同的协议端点，充当物联网中心的本地代理。这种一致性意味着客户端(无论是设备还是模块)可以连接到物联网边缘运行时，就像连接到物联网中枢一样。

通常与以前的方法(如虚拟 Kubelet，docker as CRI 等。)凭证通过主机(OS)上的 iotedge 运行时配置来管理，通过这种方法，使用新的实体“IoTedged”来执行凭证创建和云连接。

![](img/c9fd11500ed0aa9bdb06be409a7b22cd.png)

Azure Kubernetes-Edge 组件

使用该解决方案与 Azure-IoT Hub 进行模块交互的典型工作流包括多个步骤，其中每个组件执行一个明确的操作来设置整个流程:

![](img/6da29f21a14761e1bc8ce15818e52ebd.png)

边缘控制平面和模块工作流

分散每个组件，上述三个组件中的每一个都相互交互，以在本地 Kubernetes 集群上部署来自 Azure 的模块，并促进与 IoT Hub 的交互:

# iotedged:

![](img/f6a7c0d50c0ae9160a71618f5b306f99.png)

库伯内特上的 IotEdged

# 边缘代理:

![](img/487525474604100af9fad522ce1ff407.png)

Kubernetes 上的 edgeAgent

# **edgeHub:**

![](img/261f6075a1e366b63c7cc7c3a86ccbca.png)

Kubernetes 上的 edgeHub

# 试验

1.  创建资源组和 Azure IoT-Hub:

这里的“IoTEdgeResources”是一个资源组，而“kube-iot”是一个物联网中心

![](img/bfda35cdaf421879b11bab4256a6e701.png)

创建 Azure 资源组

![](img/58f6a24b069974ac3924c7552333354d.png)

创建 Azure 物联网中心

2.注册物联网边缘设备

![](img/dceba025e09eead637affae6b1f4dc7c.png)

边缘设备注册

物联网边缘设备在物联网中心注册，设备连接机密用于 Kubernetes 上的 EdgeAgent 部署。

![](img/add6cdda8ebe339bad7401edd36fcb6f.png)

设备连接信息和部署状态

3.在本地 Kubernetes 集群上部署物联网边缘网关:

两个组件“edgeagent”和“iotedged”作为 Kubernetes 部署进行部署:

![](img/2522ddba76227d9c91a1d65c275f926e.png)

Kubernetes 部署配置

![](img/c8106e39f7e1d94cf16d50d2395c17e1.png)

Kubernetes 上的 Edgeagent 和 Iotedged

所有的 pod 都注入了一个代理容器，它负责从 edge 守护进程获取凭证信息。

![](img/9043b32cc4bee60c9555deb445c571f2.png)

EdgeAgent 和代理容器

![](img/11cea9c6293a017b653a79558e27ef1b.png)

容器映射— EdgeAgent 和 Iotedged

Iotedged 使用“设备连接字符串”,这将有助于访问在物联网中心注册的物联网设备。

![](img/43ec6fc90e359b85b741fa05221f4692.png)

Iotedged 模块中的主机连接详细信息

Edgeagent 与 Iotedged 交互以进行身份验证，从而到达注册的边缘设备:

![](img/819d7fa01bfd900431cd1f77b928e720.png)

边缘剂

Iotedged 满足身份验证请求:

![](img/758981237a84cad420d4e35fd9a860e7.png)

iotedged —授权和凭证

4.在本地 Kubernetes 集群上运行的 Edgeagent 向 IoT Hub 上的注册设备报告状态:

![](img/06d1d8d3f5661bd8d90388c8a3d9f932.png)

模块状态仪表板

5.通过物联网中心在本地 Kubernetes 集群上部署模拟温度模块示例:

![](img/898bf5bfe856c8631eac80aa6bf59db9.png)

部署模块

6.模拟温度传感器作为 Kubernetes 部署部署在 Kubernetes 集群内部:

![](img/c1e7ef0129c27c362a88387acb706636.png)

Kubernetes 上的模拟温度传感器

模拟温度传感器盒包含两个容器，一个是应用程序，另一个是代理容器，它与 iotedged 交互以获取物联网中心凭据信息:

![](img/dac6f9629d8d28e9d8b92b3d5da98bc5.png)

具有代理容器的模拟温度传感器

在模拟温度传感器旁边还创建了一个 edgehub pod，它将用作到 IoT Hub 的数据通道，使用 IoT Hub SDK 基于创建部署时指定的路线。

![](img/28d158d53c9648066a81a83f1eff556f.png)

模块路线配置

总体而言，部署以及控制平面组件(edgeagent、edgehub、io tweeded)包括如下所示的拓扑:

![](img/4065ca676b36043f81fb454efefc21d5.png)

Kubernetes 拓扑上的边

模拟温度传感器部署配置在 Kubernetes 集群上，物联网集线器模块被转换为本地 Kubernetes 对象，配置如下所示:

![](img/237a80ad3e0927b31d25510cab15d2a0.png)

模拟温度传感器 Kubernetes 配置

7.Azure IoT Hub 报告本地 Kubernetes 集群上所有代理和部署的状态:

![](img/20aa413ca751043161a5298783d2094a.png)

向物联网中心报告 Kubernetes 上的模块/代理状态

![](img/b683d2e247bcc321f239ab22cd52a087.png)

功能模块和代理

8.EdgeHub 开始轮询来自模拟温度传感器的信息:

![](img/41f8448d33979a71f933a65c3b343aef.png)

Kubernetes 上的功能模拟温度传感器

如上所示，一个 Edge 模块部署在 Azure IoT Hub 的本地 Kubernetes 集群上。上述解决方案还支持用户使用 Kubernetes 集群中的物联网边缘设备作为下游设备的[网关，使用 Kubernetes 的其他方面，如网络负载平衡器(满足服务类型的 IP:负载平衡器)、StorageClass(动态存储供应)和 Voyager、Traefik 等入口控制器。路由外部流量。](https://docs.microsoft.com/en-us/azure/iot-edge/iot-edge-as-gateway)

Azure 社区正在围绕 Kubernetes 平台快速前进和创新。这些进步使得构建可扩展、可靠并部署在分布式联盟中的云原生物联网解决方案成为可能。随着所有这些不断发展的举措，Kubernetes 显然已经成为物联网解决方案的关键支持技术。一个新的 [Kubernetes 物联网工作组](https://www.cncf.io/community/webinars/introducing-kubernetes-iot-edge-working-group/)正在研究如何为物联网云和物联网边缘提供一致的部署模式。
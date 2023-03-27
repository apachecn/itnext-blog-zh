# 虚拟 Kubelet 和 KIP (Kloud 实例提供商)带来的云爆发

> 原文：<https://itnext.io/cloud-bursting-with-virtual-kubelet-and-kip-kloud-instance-provider-4b86a479ce38?source=collection_archive---------4----------------------->

![](img/55cbded2142ed7e4fa9dd393c243135b.png)

Kubernetes —弹性云爆发—无节点 Kubernetes

云爆发是一种应用部署模式，其中应用运行在私有云或数据中心，当计算能力需求激增时，它会爆发到公共云中。这是一种衡量组织使用内部资源托管对其业务至关重要的服务的能力的模型，在需求高峰期间，以按需购买的方式消耗公共云中的资源。

![](img/3a03f2ce2069d3fe8d97c7a0195bf0d8.png)

云爆炸—私有云和公共云

对于在混合云环境中运行应用的现代企业来说，云爆发是一个强大的概念。通过完全自动化这一过程，公司可以节省资金，同时提高他们的 SLA。实现成本节约是因为基础架构可以得到充分利用，允许 IT 部门在内部承载更保守数量的服务器，同时仍然有一个适当的策略，在峰值性能期间，应用程序可以扩展或完全迁移到外部公共云，如 Amazon 的 EC2。

Kubernetes 在突发计算中提供了一个独特的角色，因为它是管理容器的最常见机制，除了减轻工作负载管理的痛苦，Kubernetes 还使用户能够以当代计算所需的规模部署容器。来自 eltol 的 [Kip](https://github.com/elotl/kip) 是一家[虚拟 Kubelet](https://github.com/virtual-kubelet/virtual-kubelet) 提供商，它允许 Kubernetes 集群透明地将 pods 发布到它们自己的云实例 cells 上。Kip 的 virtual-kubelet pod 在集群上运行，并在集群中注册为虚拟 Kubernetes 节点。当一个 pod 被调度到虚拟 Kubelet 上时，Kip 为 pod 的工作负载启动一个大小合适的云实例(单元),并将 pod 分派到该实例上。

![](img/d47d741576bea35a50f635621b7126ae.png)

库伯内特云爆发—基普

上面显示的公共云中的 pod 是单独的 EC2 实例(每个 pod 都在自己的 EC2 实例中运行),称为 cells。单元(云实例)是只有准系统内核+ initrd 的映像，不运行 kubelet 或任何容器运行时(如 docker ), Kip 系统的其他组件负责容器生命周期管理。容器映像被提取到一个平面目录中，并作为流程在云实例(单元)上运行，这里不需要成熟的容器引擎。

*   Itzo 执行 kubelet 的许多任务，比如设置名称空间、运行和重启进程、运行探测器、挂载卷、捕获日志等。Itzo 包含细胞代理和构建细胞图像的代码。
*   [Tosi](https://github.com/elotl/tosi) : Tosi 将图像从图像注册表中提取到单元中，并将它们提取到一个平面目录中，或者创建一个 [overlayfs](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt) 挂载点，将层作为下层。
*   一个最小的云初始化实现，允许用户为单元提供用户数据。

# 试用

下面显示的本地 Kubernetes 集群是一个一体化集群(主+工作)，其中一个虚拟 kubelet pod 注册为虚拟工作节点。任何支持的 CNI 都可以用于本地部署，建议使用与公共云提供商上的云提供商集成的原生 CNI 插件，其中虚拟 kubelet pods 和常规 pod 将从相同的网络(如 VPC)地址空间获取其 IP 地址。

![](img/37a0e9965ded681f76262a82b4c2baba.png)

Kubernetes 云爆发—本地和 AWS

Virtual-Kubelet 配置是通过 configmap 提供的。该配置包含 kip-provider 配置和可选的 cloud-init 配置(该 cloud-init 作为 EC2 用户数据提供给在 AWS 上创建的单元)。

Cloud-init 配置允许用户为 Kip 创建的单元(EC2 或云实例)提供用户数据。用户可以在 provider.yaml 中指定一个 cloud-init 文件，当 Kip 引导单元时，将应用 cloud-init 文件。cloud-init 文件可用于提供用户、ssh 密钥、设置主机名、编写任意文件、使用“runcmd”运行命令等。或者在该单元上设置额外的包。

![](img/d88381aad80f695a136cfa8d2dea2e5d.png)

Kip —虚拟 Kubelet 配置— cloudinit

提供商配置包括云提供商凭证、单元的网络信息(将在其中创建实例的网络)、单元特定信息，如:默认实例类型(单元的默认实例类型，Kip 支持基于 Kubernetes 资源请求动态选择实例)、引导映像信息(用户可以构建自己的 AMI 或使用 eltol 提供/拥有的默认 AMI)、配置路由的 CIDR 信息、名称标签(用于标识属于该控制器的云资源的名称标签)、默认卷大小(单元的根卷的默认大小)、itzo 配置(要使用的 itzo 代理版本)等

![](img/54e5d2832a7c3b95c5fdf810a8b0fd07.png)

Kip —虚拟 Kubelet 配置—提供商配置

Kip provider 在 virtual-kubelet 上的实现从指定的配置中获取 VPC 和其他 AMI(单元图像)信息:

![](img/d3117089f0a2187a2e5e25da0b169bf6.png)

kip——虚拟 Kubelet 提供商

由于在集群上部署了带有 Kip 提供者的虚拟 Kubelet，因此控制平面 daemonset pods，如 kube-proxy、node-local-dns 等。被安排在 virtual-kubelet 节点上(因为 pod 向 kube-api 注册为节点)。

![](img/28c35636bd57236990b54b3625395f87.png)

kip——虚拟 Kubelet 上的 Kubernetes 控制平面吊舱

![](img/7aac7848dfb97b3a23efc7b01c8305b5.png)

kip——虚拟 Kubelet 上的 Kubernetes 控制平面吊舱

Kip 将上述 pod 安排为指定子网内已配置用户帐户上的单个 EC2 实例(单元)。如下所示，kube-proxy pods 被创建为 t3.nano 类型的 EC2 实例

用户可以通过在 pod 上设置以下注释来强制公共子网中的单元仅具有私有地址:` pod . elotl . co/private-IP-only:" true " `。

![](img/f50c1b2e6504e437146faba65ed51afe.png)

Kip —作为 EC2 的自动气象站上的 Kubernetes 控制面板吊舱

![](img/2e5b18829c1147a6d9854a34a12a41ce.png)

Kip —作为 EC2 的自动气象站上的 Kubernetes 控制面板吊舱

virtual-kubelet pod 与云提供商创建的所有单元建立出站连接。安装在 virtual-kubelet 上的 kipctl(类似于 kubectl)可用于交互和获取部署在单元上的节点和单元的信息。

![](img/7df5dda836e7cafaf89486867e6838f7.png)

kip——虚拟库伯勒

![](img/d8d0894c5ee090ee655089ef42db8897.png)

kipctl —虚拟库伯勒

# 将示例 Busybox Pod 分解到 AWS

一个示例 busybox pod 部署在本地集群上，使用 nodeSelector 在 virtual-kubelet 节点上对其进行调度，专门针对 virtual-kubelet 节点。

![](img/584d814856dd24b9272204b2f86f834a.png)

虚拟 Kubelet 上安排的样本 pod

上面的 pod 在 virtual-kubelet 节点上调度，如下所示:

![](img/4eea3fee219c462eb6a7a2610c98e101.png)

虚拟 Kubelet 上安排的样本 pod

virtual-kubelet 中的 Kip 提供者实现开始在 AWS 上创建单元，以获取已配置的(提供者配置)AMI。因为上面的 busybox pod 没有配置任何资源请求，所以实例类型被选择为“t3.nano”(默认实例类型)。动态实例类型选择将在下一节讨论。

![](img/ddb84c1509b0bf0d6af680a3e2509d57.png)

在虚拟 kube let-AWS EC2-T3 . nano 上安排的样本 pod

如下所示，使用提供者配置中提供的 AMI 创建了一个类型为“t3.nano”的单元(EC2 实例)(这里使用的 AMI 是一个轻量级的基于 Alpine 的映像)。Kip 自动配置所需的安全组以启用连接。为该单元提供了来自提供商配置中提供的子网的两个 IP。

![](img/59baa9702a2f612c3e2aa5a111c2e6d3.png)

虚拟 Kubelet 上安排的样本 pod—AWS EC2

![](img/0726d437b97cab02c37f9c563a6c6c7e.png)

在虚拟 kube let-AWS 安全组上安排的样本 pod

如下所示，Kip 无缝地添加了标签:名称空间、名称、节点名称。应用程序标签等。来识别资源。

![](img/3b31a3ffce7461f2df81e062fbefa6cd.png)

虚拟 kube let-AWS 标签上安排的样本 pod

itzo 客户机安装在单元上，itzo 开始向单元部署所需的包，以启用 busybox 应用程序。

![](img/ced6894871b075279de80acb69f19615.png)

虚拟 Kubelet 上安排的样本 pod—AWS EC2

上面的 cloud-init 配置包括设置用户和 SSH-Keys，使用户能够访问创建的单元。

如下所示，busybox 容器作为一个运行在基于瘦 alpine 的实例(单元)中的进程被分派:

![](img/3c8a19a46e35c058141cacecb0ad80bf.png)

在虚拟 Kubelet 上安排的样本 pod—AWS EC2 上的 Pod 流程

Kip 为每个单元分配两个 ip 地址，除非 pod 有 hostNetwork:一个用于管理通信(在提供者和实例上运行的小代理之间)，另一个用于 pod。除非 pod 启用了主机网络，否则将为具有第二个 IP 的 pod 创建一个新的 Linux 网络命名空间。两个 IP 地址都来自 VPC 地址空间。如下所示，实例的 eth0 接口为“10.10.22.13”，网络命名空间“pod”的 veth 接口为“10.10.25.96”。

![](img/d752b30edfe352163c9a4d8eb5a68e1b.png)

kip —蜂窝网络—云实例 ip 寻址

# 基于 Kubernetes 资源请求的单元动态调整大小

除非实例类型注释(pod.elotl.co/instance-type:<c5.large>)出现在 pod 中，否则 kip 将选择满足资源需求的最便宜的实例类型。如果在 pod 的规范中配置了特定的资源(使用 Kubernetes 资源请求),那么 kip 会根据资源需求无缝地选择大小合适的实例。如果没有指定注释或资源，那么它将返回到 provider.yaml 中的 defaultInstanceType。</c5.large>

实例选择器使用如下所示的各种解析器/参数，试图找到满足资源规范中所有约束的最小成本实例:

![](img/0e812cf9a5a533eac17fe5ef7937a1c0.png)

kip —实例选择—属性和模式

Kip 还可以在按需 spot 实例上运行单元，同样可以使用注释进行配置:“pod.elotl.co/launch-type: spot”。

在下面的示例中，名为“busybox-resources”的示例 busybox pod 配置了内存(15Gi)和 cpu (8 个 vcpu)请求，这创建了一个 c5.xlarge 单元。

![](img/e155d89297b43c83bb6d31bdc65365da.png)

kip—动态实例选择/规模调整

如下所示，pod 配置了资源请求，并使用 nodeSelector 在 virtual-kubelet 节点上进行调度:

![](img/f9409dd8a72af0edd002db1efeed1365.png)

kip —动态实例选择/规模调整—包含资源请求的 pod

![](img/e001b56bfa239a3c0d491c8c8d406f5d.png)

虚拟 Kubelet 上安排的样本 pod

如下所示，kip 提供程序识别资源需求并生成 c5.xlarge 单元(云实例),而不选择提供程序配置中指定的默认映像。

![](img/85e5aa795c1aa128a8a6f3967175f30a.png)

虚拟 Kubelet 上安排的样本 pod 实例类型:c5.xlarge

在 AWS 上创建 c5.xlarge 实例以适应资源请求:

![](img/cad7a4260af9cb851a7205247f32303c.png)

实例类型:c5.xlarge，用于带有资源请求的示例 buybox pod

# 本地和云之间的 VPN 连接

Kip 部署在 Kubernetes 云服务上，如 EKS、AKS、GCE 等。由于 pod 和 kip-cell 共享由云提供商提供的相同网络空间，因此不需要 pod 之间的任何额外连接层。为了在本地集群上的 pod 和由 kip 部署为单元的 pod 之间实现联网，需要一个到云提供商网络的传统 VPN 连接(本例中为 AWS-VPC)。客户网络(专用/内部网络)到 VPC 之间的传统站点到站点 VPN 拓扑包括客户网关和虚拟专用网关。

![](img/e9674acad9473375dd107f84d4df57da.png)

AWS — VPN 网关—拓扑

Kip 和一个 [VPN 客户端](https://github.com/elotl/kip/tree/master/deploy/terraform-vpn/aws-vpn-client) (Strongswan)部署在本地 Kubernetes 集群中。VPN 网关部署在 AWS 上，并与配置用于 kip-cell 的 VPC 相关联。一个 VPN 连接将把本地集群连接到 VPC，允许 pod 和服务在双方之间互相访问。基普被配置为在 VPC 创建吊舱。

![](img/9ed785931c271144ca7d9a37062fd7c0.png)

kip — aws_vpn_client —内部数据中心和 AWS VPC 之间的站点到站点 vpn

vpn-client pod 包括一个运行 Strongswan 的 ipsec 容器和一个使用通过 configmap 提供的配置来配置 Strongswan 参数的 init 容器。

![](img/4babb1578a391f1c7d8e98d5f03df735.png)

kip — aws_vpn_client —作为 pod 部署在 Kubernetes 上

运行“ipsec-init”和“ipsec”容器的 aws-vpn-client pod:

![](img/2e76686464da82c37c49db7b4d77c511.png)

kip — aws_vpn_client —容器

Strongswan 配置以配置图的形式提供，其中包括公共隧道端点 PSK 和 CIDR VPC。用户可以使用静态路由或 BGP 来通告路由。

![](img/64efbd5192c5ccc271abdeca3b60ac63.png)

kip — aws_vpn_client —配置

提供商配置中提供的 VPC 与互联网网关相关联:

![](img/259099bdc98cb1354fe7e094192efae2.png)

AWS-VPC

![](img/4083db0de4944017422edfa27b982685.png)

AWS-IGW

VGW 和 CGW 与上面的 VPC 相关联，CGW IP 是在本地 Kubernetes 集群的主机网络上运行的 vpn 客户端 pod 的公共 IP:

![](img/5d9e88f5b88fe66737a2cfa6a87ca457.png)

虚拟专用网关

![](img/d2a6bd71f9a1ca435fd4fa8f3569d462.png)

AWS-客户网关

VGW 和 CGW 之间创建了 VPN 连接:

![](img/91306780fd2d96df6e0a61f63d4e6491.png)

AWS-VPN 连接

在本地 Kubernetes 集群上运行的 vpn-client pod 具有所需的配置，可与 VPC 建立隧道:

![](img/fae152ad2d08eb66310beb9a807a3121.png)

kip — aws_vpn_client —隧道建立

![](img/6298c11100e8b61d3faf133408cee08e.png)

kip — aws_vpn_client —隧道建立

AWS 上的隧道状态:

![](img/6111b236fd37c3db4eb366d1e6fcd7f5.png)

AWS — VPN 连接—隧道状态

vpn 客户端 pod 上的隧道状态:

![](img/3e0c585a61a185187a7a48b56e5fef47.png)

kip — aws_vpn_client —隧道状态

隧道建立后，本地集群上的 pod CIDR 和创建单元的 VPC 子网将通过 VPN 连接建立隧道，从而实现基于 IPSEC 的双向通信。

![](img/a2415b9330f1244e2ce4ed23508e2ad6.png)

基普—通过 VPN 的 VPC Pod 子网 CIDR 路由

![](img/0afdb695fcc8c22460db5b3b40c4b023.png)

基普—通过 VPN 的 VPC Pod 子网 CIDR 路由

在本地集群上创建了一个“busybox-on prem”pod(主集群被污染，以允许创建工作负载)，如下所示:

![](img/a092f53b2777003e410b0ce69e41015b.png)

在实际节点上计划的本地 Kubernetes 集群上的示例 pod

在本地集群上运行的法兰绒提供了 Pod IP (10.244.0.6):

![](img/e37a2e1a45c7cdf4cc61767c6d008220.png)

在实际节点上计划的本地 Kubernetes 集群上的示例 pod

VPC 子网为在 AWS 上作为单元(云实例)运行的 Pod“busybox-cloudburst”提供了 Pod IP (10.10.25.96 ):

![](img/1047eac55c601efd12514ba5fa86af7d.png)

虚拟 Kubelet 上安排的样本 pod

如下所示，从运行在 AWS 上的 kip 创建的 pod 可以访问不同子网中具有 pod_ip 的本地 pod:

![](img/90a8405a711a8ed070c2da569ba1762b.png)

本地和云之间的 Pod-Pod 连接(kip 创建的单元)

*******************************************************************

除了传统的云爆发，kip 在其他场景中也很有用，如*云计算扩展*:在多个云提供商之间部署工作负载子集，这些云提供商从一个中央集群运行，无需在各个提供商上部署单独的控制平面，*高可用性/灾难恢复*:具有严格 SLA 的业务关键型应用程序可以在多个云提供商上运行，而不必担心创建成熟的集群或基础架构， *简化应用程序迁移*:由于同一应用程序的子集可以在不同的提供商上运行，因此不会有供应商锁定的机会，因为工作负载可以在不同的云提供商上轻松安排，*多租户安全*:无节点为每个单元提供单独的计算启动类型。 这导致了 pod 等之间更强的(虚拟机级别)隔离。

带有 kip 的虚拟 Kubelet 以“容器即服务”的方式运行 Kubernetes pods。用户可以根据工作负载和规模要求，利用 kip 动态创建实例，而不是调配未使用的额外容量。Kip 提供了一种智能实例过滤方案，能够选择最具成本效益的实例类型，并且能够准确满足工作负载资源请求—想象一下这样一个场景，我们在一个具有 32gb RAM 的节点上部署 10 个需要 10gb RAM 的 pod，其中该节点上的 22gb RAM 是理想的，会产生每月为理想资源支付的费用。Kip 还可以在具有单个控制平面的高级混合云使用案例中加以利用。
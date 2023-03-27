# 入口运营商——使用公共 IP 公开私有 Kubernetes 集群上的服务

> 原文：<https://itnext.io/inlets-operator-exposing-services-on-private-kubernetes-clusters-with-a-public-ip-e701c64693ae?source=collection_archive---------2----------------------->

![](img/c71fb7cf1ab9f2c4d2e3931d89d4c17a.png)

私有 Kubernetes 服务，具有公共端点—入口/入口—运营商

Kubernetes 云提供商将每个公共和私有云提供商的知识和背景嵌入到大多数 Kubernetes 组件中。有了这些提供者，就可以很容易地使用平台本地负载平衡结构来公开运行在特定平台上的 Kubernetes 服务。如果用户部署到 EC2 实例或 DigitalOcean Droplet，那么他们有一个公共 IPv4 地址，但当在公司防火墙、NAT 之后工作时，或者在 VM 或容器内工作时，这就不起作用了。

在其他多种情况下，这可能是一个问题:

*   没有 DMZ 和其他网络配置，无法将本地主机应用程序直接暴露给 internet
*   进行开发的私有数据中心资源有限。
*   无法出于测试目的共享网站
*   在企业防火墙后面开发任何使用 Webhooks (HTTP 回调)的服务。例如，私有空间中的开发人员代码没有可路由的 IP 地址，GitHub 根本无法发送消息。
*   暂时共享仅在开发人员计算机上运行的网站

像 [Ngrok](https://ngrok.com/) 这样的工具是众所周知的将本地主机暴露给 web 的隧道服务。这些工具实现了多平台隧道、反向代理，使用 WebSocket 建立从公共端点(如互联网)到本地运行的网络服务的安全隧道。WebSocket 是一种自然的全双工、双向、单套接字连接。使用 WebSocket，HTTP 请求变成了打开 WebSocket 连接的单个请求，并重用从客户端到服务器以及从服务器到客户端的相同连接。

客户端运行在内部网络中(与应用程序一起),并通过 HTTP websockets 连接到远程服务器。然后，服务器通过提供的 websockets 之一将请求转发给客户机。

![](img/97c3e6610ca332b017a07b70d04f78b3.png)

Websocket 隧道

[Alex Ellis](https://twitter.com/alexellisuk) — [Inlets](https://github.com/alexellis/inlets) 结合了反向代理和 WebSocket 隧道，通过出口节点将内部和开发端点暴露给公共互联网。出口节点是任何配置了入口服务器进程的公共云平台上的公共可达服务器。

![](img/464a9dbb542c6e35e20cae8b087ef80c.png)

入口—公共云平台上的出口节点

类似的工具，如来自 [Cloudflare](https://www.cloudflare.com/) 的 [ngrok](https://ngrok.com/) 或 [Argo Tunnel](https://developers.cloudflare.com/argo-tunnel/) 都是闭源的，具有有限的内置功能，并且可能非常昂贵。ngrok 也经常被公司防火墙政策禁止，这意味着它可能无法使用。Inlets 旨在通过 websocket 隧道，使用自动 TLS 证书将您的本地服务动态绑定和发现到 DNS 条目的公共 IP 地址。

如果没有入口，用户可能需要为 webhooks 配置所需的防火墙规则，以访问内部网络中的应用程序。这可能是一项艰巨的任务，因为跟踪所有的动态连接是不可能的。

![](img/6d7f181107056829a28d6cb541bd93ef.png)

场景—没有入口的引入网络挂钩

对于入口，所有请求都从公共托管的出口节点(入口服务器)发送到驻留在内部的入口客户端。入口充当交通汇，因此不容易出现任何类型的滥用。它不会向公共互联网转发任何请求。相反，入口遭受相反的问题。

![](img/8f0dec894f2161eb87559e60be123d6f.png)

场景—带入口的引入网钩

如上所述，默认情况下，Kubernetes 服务的负载平衡器服务只在云提供商上可用，而不是私有托管的 Kubernetes 集群。让流量进入集群的最干净的方法似乎是负载均衡器。然而，它需要一个通常由 GCP 或 AWS 提供的外部服务，而这不是 Kubernetes 提供的。借助 Inlets，运营商用户可以无缝地为私有 Kubernetes 服务启用公共负载平衡器，而无需在公共云平台上管理成熟的集群。

# 入口-操作员

在 Kubernetes 设置中，inlets 被实现为一个利用 CRD(自定义资源定义)来管理隧道及其组件的运营商。使用入口-运营商用户可以在任何支持的平台上动态创建出口服务器，如 Digitalocean、Packet、GCP 等。对于每种服务类型“负载平衡器”,将在受支持的公共云平台上使用入口服务器流程创建一台专用服务器。

![](img/aa8c8b257144b6321abf52753326fa30.png)

Kubernetes 上的入口-操作员

下图所示的 CRD 入口使用户能够操作(创建/删除/获取/描述)隧道，作为 Kubernetes API 的扩展。

![](img/001f8a18ec1964b3dab5784f8fd08843.png)

入口-操作员自定义资源定义

在本演练中，操作员和示例应用程序部署在 Kubernetes 集群上，该集群运行在连接到家庭专用网络的 intel-nuc 上。Inlets operator 被部署为 Kubernetes 部署，并不断地从 Kubernetes-API 轮询任何负载平衡器类型的服务的信息。

![](img/cb5a49b5243a840ceed915fda4e67bfd.png)

操作员部署

![](img/93d042954ac161b42fcdbf8810e68f10.png)

操作员控制器和 Kube-apiserver

![](img/867a77ff996a9155769db9316dda8cd7.png)

操作员部署

入口运营商支持多种平台(Digitalocean、Packet、Scaleway、GCP)，在支持的平台上，用户可以利用所需组件的自动供应来无缝启动出口节点。试用入口-数字海洋运营商:

创建用于身份验证的 API 密钥:

![](img/c491e52d6c444a23833023a512eb3935.png)

创建 API 密钥:数字海洋

创建了带有上面创建的访问令牌的 Kubernetes 秘密:

![](img/0a343b285c508b8dbb2d7524559f5d73.png)

Kubernetes 秘密:访问密钥

创建了一个样例部署(Kuard)和一个服务类型:LoadBalancer。

![](img/d9ef9456bb8ff7e816bd8babf0f6a14b.png)

示例部署

![](img/1c9f80b5838291061435e6f2dcbb65f9.png)

示例部署

如下所示，在流程开始的地方创建了一个类型为 LoadBalancer 的 kuard-service。

![](img/4158899c633284bda333f89674596288.png)

示例服务-类型:负载平衡器

Inlets-operator 检测到上述服务，并使用上述 Kubernetes secret 提供的凭证在用户 Digitalocean 帐户上创建一个 droplet。

![](img/3288aaa6abd8890423d672e8c25dda13.png)

操作员在数字海洋上提供液滴

由于此处选择的提供商是 Digitalocean，因此会在用户帐户上自动创建一个 droplet，同时在该帐户上运行 Inlets 服务器进程并分配一个公共 ip。

![](img/28d8595d010f44d1e0d0f9c179a5c047.png)

水滴—数字海洋上的出口节点

![](img/34f99cc48440dad4e744d4c18e7fc90d.png)

带有 public _IP 的 Droplet 数字海洋上的出口节点

如下图所示，inlets-server 进程在上面创建的 droplet 上运行:

![](img/561ad9b47bbb812c687b107019ad3835.png)

Droplet 上的入口服务器

使用操作员在集群上创建入口客户端部署，并使用远程出口节点信息进行自动配置。

![](img/cac773bd44065639b67017f62fac1946.png)

入口-客户

如下所示，客户端配置了上面创建的 droplet 的 public_ip 作为远程服务器目的地，本地服务(kuard-service)作为上游(通常是类似 envoy 的反向代理的工作方式)。

![](img/9818ef8fdad5ae38c47682484c7c77be.png)

入口-配置了服务器信息的客户端

上面创建的具有类型:LoadBalancer 的 Kuard 服务将分配有上面部署的 droplet 的 public_ip 作为 external_ip:

![](img/2321b6c8eae04b9f7c9300b64de961c5.png)

带有 Public_IP 的负载平衡器

使用上面创建的 droplet 的 public_ip 访问在私有环境中运行的服务。

![](img/d5ea2f3af2ddf238c0aec10dfdca60e4.png)

使用 Droplet 的 public_ip 访问应用程序

在 droplet 上运行的服务器进程，将所有请求代理给在内部网络上运行的本地 kuard 应用程序。

![](img/b6eadb59bc95e03123921956f79d3368.png)

入口-数字海洋代理请求的服务器

用户可以使用云提供商 DNS:

![](img/8a55185f0b9f1373e8d2fd03102ce211.png)

DNS 配置

用户可以动态删除触发删除操作的服务，公共云上的所有资源都将被清理。

![](img/f018e8faae7d418ed6108e62f61164f9.png)

删除服务

操作员规范使用户能够选择其他支持的提供商:

![](img/c3d22a2fb9034964388086efcaf7e133.png)

其他支持平台的配置

# 坦克激光瞄准镜（Tank Laser-Sight 的缩写）

默认情况下，for development inlets 配置为使用易受中间人攻击的非加密隧道。用户可以通过在出口节点上运行 Caddy 或 nginx 来实现 TLS，或者通过 nginx 使用 Nginx 运行来代理入口服务器。通过启用 HTTPS，用户可以连接到加密的端点，而入口客户端也可以通过加密的隧道连接到服务器。路线图中还提到了 cert-magic(一个基于 go 的证书生成栈)。

正如所看到的，使用 inlets-operator 用户可以很容易地为运行在私有集群上的应用程序获得一个可公开访问的端点。入口在很多情况下都很方便:

*   在本地主机上开发与 webhooks 的集成。无需将服务部署到云或在服务上配置防火墙/NAT。无论您现在在哪里工作，无论是在家里、办公室还是在咖啡店，都可以使用相同的端点，无需管理 DNS 和 Public_IP。
*   使用 WebSocket 隧道来公开家庭自动化服务，而无需公共 IP 地址或配置 NAT。二进制文件也可用于 arm 架构，可以在 Raspberry Pi 或类似的硬件上运行
*   连接物联网设备，并通过 WebSockets 或 Node-RED 提供的节点直接集成—无需在设备上运行 web 服务器来接收 webhooks。
*   不需要为了测试而不断地在开发环境中将服务重新部署到云中。用户可以在本地私有开发环境中创建一个公共端点。可以使用基本和令牌认证方法来保护隧道。

入口可以部署在任何基于 ARM 的设备上，使其能够在 Raspberry Pi 等设备上运行，并且可以用作内部运行的服务的传入流量的网关。这还可以优化云成本(平均运行成本:每月 5 美元),只需为入口服务器运行一个微小的虚拟机，而不必为了开发目的在云上运行整个 Kubernetes 集群。
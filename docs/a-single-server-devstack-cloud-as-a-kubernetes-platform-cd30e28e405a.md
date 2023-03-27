# 关于 Devstack 的 Kubernetes 第 1 部分:部署 Devstack 云

> 原文：<https://itnext.io/a-single-server-devstack-cloud-as-a-kubernetes-platform-cd30e28e405a?source=collection_archive---------2----------------------->

这篇文章是关于在 Devstack 云上运行 Kubernetes 集群的[系列文章的一部分。它讨论了云的设置。](https://berndbausch.medium.com/running-a-kubernetes-cluster-on-devstack-533d579bb2f9)

目标是在单节点 Devstack 服务器上创建 Kubernetes 集群。本文档涵盖了服务器本身、部署 Devstack 和一些部署后配置步骤。

# 目录

1.  [dev stack 服务器](https://medium.com/p/cd30e28e405a#426c)
2.  [准备 Devstack 服务器并部署云](https://medium.com/p/cd30e28e405a#ad9b)
3.  [配置您的云](https://medium.com/p/cd30e28e405a#305d)
4.  [如果需要重启 Devstack 服务器](https://medium.com/p/cd30e28e405a#8acc)
5.  [可选:测试负载平衡是否工作](https://medium.com/p/cd30e28e405a#833a)

# Devstack 服务器

在具有以下属性的计算机上安装 Ubuntu 18.04 的服务器版本:

*   大约 15GB 的内存，操作舒适。12GB 可能就足够了。除了云开销之外，还需要运行至少两个 Kubernetes 节点(每个 4GB)和一个 1GB 负载平衡器实例。
*   大约 50GB 的存储空间
*   几个 CPU(最少 2 个，越多越好)
*   一个可以连接到互联网并且可以从外部连接的网卡

这台计算机(此后命名为 Devstack server)可以是您实验室或公共提供商的物理计算机，也可以是您实验室或公共云中运行的虚拟机。

如果您选择在实验室的虚拟机中运行 Devstack 服务器，请确保虚拟机管理程序支持**嵌套虚拟化**并允许网络流量流向嵌套虚拟机。KVM 满足这两个条件。以我的经验来看，VirtualBox 和 Xen 会阻塞到嵌套虚拟机的流量，可能是因为它们拒绝与未知的 IP 地址对话(我是 Xen 新手；可能存在改变这种行为的配置设置)。我没有试过 Vmware、Hyper-V 或 WSL。

如果没有嵌套虚拟化，许多操作将非常缓慢，例如，在嵌套虚拟机上安装软件需要一个小时或更长时间。

要在 KVM 上启用嵌套虚拟化，请将`kvm-intel.nested=1`或`kvm-amd.nested=1`添加到 Linux 内核参数中。或者，创建一个`modprobe.conf.d`文件，用`nested=1`参数加载相应的内核模块`kvm_adm`或`kvm_intel`。

**下面是我的设置:**dev stack 服务器运行在一个 KVM 虚拟机上，该虚拟机通过一个 Linuxbridge 连接到外部网络。K8s 集群节点(主节点和工作节点)是运行在 Devstack 服务器内部的 OpenStack 实例。 *br-ex* 是一个 Openvswitch 桥，它将所有 OpenStack 实例连接到外部世界。Devstack 服务器的单个 NIC 插入到 *br-ex* 中。

```
 +---------------- Physical host -----------------+
 |                                                |
 |                                                |
 | +------ Devstack server (a KVM VM) ----------+ |
 | |                                            | |
 | |  +------------+          +------------+    | |
 | |  | K8s master |          | K8s worker |    | |
 | |  +------------+          +------------+    | |
 | |          \                     /           | |
 | |           \-----\   /---------/            | |
 | |                 |   |                      | |
 | |               /-------\                    | |
 | |               | br-ex |                    | |
 | |               \-------/                    | |
 | |                   |                        | |
 | +------------- Virtual NIC ------------------+ |
 |                     |                          |
 |              /-------------\                   |
 |              | Linuxbridge |                   |
 |              \-------------/                   |
 |                     |                          |
 +-----------------Physical NIC-------------------+
                       | 
_______________________|_________External Network_______________
```

由于这些桥，Devstack 服务器和 K8s 节点可以从外部网络获得 IP 地址。这是可取的，因为它允许向外界公开 K8s 集群服务。

# 准备 Devstack 服务器并部署云

Devstack 文档站点有关于[部署 Devstack](https://docs.openstack.org/devstack) 和[启用负载平衡器](https://docs.openstack.org/devstack/latest/guides/devstack-with-lbaas-v2.html)的说明。这些说明并没有涵盖所有内容；例如，他们不知道如何将云连接到外部网络，或者如何启用负载平衡器的 GUI。下面的步骤对我有效。

从带有 SSH 访问的默认 Ubuntu 18.04 安装开始。在设置 Devstack 之前:

*   配置一个静态 IP 地址(DHCP 可能也可以，但静态更安全)
*   创建一个名为 *stack* 的用户，使用 Devstack 站点上记录的无密码 sudo。
*   我有 Ubuntu 默认 DNS 解析的问题，它在云部署期间被神奇地破坏了。虽然我不知道为什么会发生这种情况，但我通过链接/etc/resolv.conf 解决了这个问题，如下所示:
    `ln -s /run/systemd/resolve/resolv.conf /etc`

做好这些准备后，以 *stack* 用户身份登录，克隆 Ussuri 版本的 Devstack:
`git clone https://opendev.org/openstack/devstack -b stable/ussuri`

这将创建$HOME/devstack 目录并将 devstack 复制到其中。Devstack 是 Bash 脚本的集合，我鼓励您分析它们。

Devstack 使用一个名为 *local.conf* 的配置文件。从我的 Github repo 中改编 [local.conf 模板，并将其复制到＄HOME/dev stack。然后推出
云:`cd $HOME/devstack; ./stack.sh`](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/local.conf)

这将生成大量输出，这些输出也被写入`/opt/stack/logs/stack.sh.log`。部署过程从互联网上安装 Ubuntu 和 Python 包，并获取 VM 镜像。因此，安装时间在很大程度上取决于您的互联网带宽，但也取决于您的硬盘速度。粗略估计:一到两个小时。

部署失败的原因有很多，包括:

*   `apt`或者`dpkg`在后台运行，可能是因为 Ubuntu 目前正在寻找更新。等 apt/dpkg 安静了再试试。
*   由于网络问题，对某些包的访问失败。几分钟后重试。
*   Python 包之间的不兼容性。这需要来自软件包维护者或 Devstack 团队的修复。最少 24 小时。
*   你的 Devstack 服务器的问题:
    网络故障，空间不足，…

只要不改变 *local.conf* ，应该可以重新运行一个不成功的部署。部署修改后的 *local.conf* 可能需要从头开始安装 Ubuntu。

当部署以类似以下的消息结束时，您就知道部署成功了:

```
=========================
DevStack Component Timing
 (times are in seconds)
=========================
run_process           39
test_with_retry        4
apt-get-update         5
osc                  437
wait_for_service      18
git_timed            260
dbsync               422
pip_install          455
apt-get              898
-------------------------
Unaccounted time     2074
=========================
Total runtime        4612This is your host IP address: 192.168.1.200
This is your host IPv6 address: ::1
Horizon is now available at http://192.168.1.200/dashboard
Keystone is serving at http://192.168.1.200/identity/
The default users are: admin and demo
The password: pwWARNING:
Using lib/neutron-legacy is deprecated, and it will be removed in the future Services are running under systemd unit files.
For more information see:
https://docs.openstack.org/devstack/latest/systemd.htmlDevStack Version: ussuri
Change: f482957e89d1ee938da679529aa10f13b2d07631 Bionic: Enable Train UCA for updated QEMU and libvirt 2020-09-18 09:10:58 +0100
OS Version: Ubuntu 18.04 bionic
```

# 配置您的云

成功部署后，您可以通过将浏览器定向到 Devstack 服务器的 IP 地址来访问云的 GUI。对于命令行访问，使用 Devstack 服务器上的 shell。

在安装 Kubernetes 集群之前，向云中添加以下内容:项目和用户、网络、安全组、密钥对、Centos 映像以及主实例和工作实例。创建所有这些云资源的准备脚本是可用的。

# 如果需要重新启动 Devstack 服务器

Devstack 被设计成一个崩溃-烧毁系统，它的一些配置是非持久的。如果您计划关闭 Devstack 服务器，您需要使它防重启。

*br-ex* 桥必须配置有 Devstack 服务器的 IP 地址。这可以通过[网络计划配置文件](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/00-installer-config.yaml)来完成。复制给`/etc/netplan`。

为了重新创建剩余的配置设置，我在每次重启后运行脚本 [restore-devstack.sh](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/restore-devstack.sh) 。

最后，重新启动时正在运行的任何实例现在都处于关闭状态。用`openstack server start`命令复活它们。

# 可选:测试负载平衡是否有效

基于 [Devstack 负载平衡器指南](https://docs.openstack.org/devstack/latest/guides/devstack-with-lbaas-v2.html)。

这并不是真正必需的，但是如果您对 OpenStack 云中的负载平衡器的设置感兴趣，它会为您提供一些见解。

在[配置您的云](#305d)之后执行这些步骤。从 *kube* 项目中的两个 Cirros 实例开始。

```
source ~/devstack/openrc kube kube
openstack server create --image cirros --network kubenet \
                  --flavor 1 --key-name kubekey --min 2 --max 2 c
```

这将启动两个名为 *c-1* 和 *c-2* 的实例。要为 ICMP 和网络端口 22 和 80 打开防火墙，请将它们添加到 *kubesg* 安全组。要启用网络访问，请给他们浮动 IP 地址。

```
openstack floating ip create public
openstack floating ip create public   # you need two floating IPs
openstack server add security group c-1 kubesg
openstack server add security group c-2 kubesg
openstack server add floating IP c-1 FLOATING-IP-1
openstack server add floating IP c-2 FLOATING-IP-2
```

在两个实例上安装一个基本的 HTTP 服务器。

```
ssh -i kubekey cirros@FLOATING-IP-1
$ while true
do 
     echo -e "HTTP/1.0 200 OK\r\n\r\nWelcome to server 1" | 
     sudo nc -l -p 80 
done &
$ exit
```

当服务器收到一个 HTTP 请求时，这个简短的脚本会响应“欢迎使用服务器 1”。对另一个实例执行相同的操作。

```
ssh -i kubekey cirros@FLOATING-IP-2
$ while true
do 
     echo -e "HTTP/1.0 200 OK\r\n\r\nThis is server 2" | 
     sudo nc -l -p 80 
done &
$ exit
```

测试这个:
`curl FLOATING-IP-1
curl FLOATING-IP-2`

现在创建一个负载平衡器，将这两个实例作为后端。您需要创建负载平衡器、侦听器、该侦听器的池，并将这两个实例添加为池成员。GUI 允许您直观地做到这一点。以下是 CLI 指令，几乎一字不差地来自 [Devstack 指南](https://docs.openstack.org/devstack/latest/guides/devstack-with-lbaas-v2.html#phase-2-create-your-load-balancer)，并添加了来自[基本负载平衡指南](https://docs.openstack.org/octavia/latest/user/guides/basic-cookbook.html#deploy-a-basic-http-load-balancer-using-a-floating-ip)的内容:

```
# Obtain fixed IP addresses from the Cirros instances
FIXED-IP-1=$(openstack server show c-1 -c addresses -f value | 
sed -e 's/kubenet=//' -e 's/,.*//')
FIXED-IP-2=$(openstack server show c-2 -c addresses -f value | 
sed -e 's/kubenet=//' -e 's/,.*//')openstack loadbalancer create --name testlb \
                              --vip-subnet-id kubesubnet
openstack loadbalancer show testlb  
# Repeat the above `show` command 
# until the provisioning status turns ACTIVE.# Create a listener
openstack loadbalancer listener create --protocol HTTP \
                       --protocol-port 80 --name testlistener testlb# Create a pool
openstack loadbalancer pool create --lb-algorithm ROUND_ROBIN \
             --listener testlistener --protocol HTTP --name testpool# Add the two Cirros instances as members to the pool
openstack loadbalancer member create --subnet-id kubesubnet \
                   --address $FIXED-IP-1 --protocol-port 80 testpool
openstack loadbalancer member create --subnet-id kubesubnet \
                   --address $FIXED-IP-2 --protocol-port 80 testpool
```

负载平衡器已经就位。最后一步，添加一个浮动 IP，这样就可以从云外部访问它。这个有点牵扯。

获取负载平衡器的 ID:
`LB_ID=$(openstack loadbalancer show testlb -c id -f value)`

获取承载负载均衡器虚拟 IP 的中子端口:
`PORT_ID=$(openstack port list --device-id lb-$LB_ID -c ID -f value)`

创建一个新的浮动 IP 并保存其 ID:
`FLOATINGIP_ID=$(openstack floating ip create public -c id -f value)`

将此浮动 IP 与负载平衡器的 VIP 端口关联:
`openstack floating ip set --port $PORT_ID $FLOATINGIP_ID`

现在获取浮动 IP 地址，反复访问。您应该会收到来自运行在 Cirros 实例上的两个“web 服务器”的响应。

```
FIP=$(openstack floating ip show $FLOATINGIP_ID \
                              -c floating_ip_address -f value)
for i in 1 2 3 4 5 6
do curl $FIP
done
```

**接下来:** [使用 *OpenStack 云提供商*和 *Cinder CSI* 插件](https://berndbausch.medium.com/a-kubernetes-cluster-with-openstack-cloud-provider-and-cinder-plugin-b3f058ce898c)创建 Kubernetes 集群
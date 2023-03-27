# 第 2 部分:手动设置 Kubernetes 集群

> 原文：<https://itnext.io/a-kubernetes-cluster-with-openstack-cloud-provider-and-cinder-plugin-b3f058ce898c?source=collection_archive---------4----------------------->

本页是关于在 OpenStack 上运行 Kubernetes 集群的[系列的第二部分。它基于安装 kubernetes 的 kubernetes.io](https://berndbausch.medium.com/running-a-kubernetes-cluster-on-devstack-533d579bb2f9) [博客条目](https://kubernetes.io/blog/2020/02/07/deploying-external-openstack-cloud-provider-with-kubeadm/)和官方 [OpenStack-Kubernetes 云提供商文档](https://github.com/kubernetes/cloud-provider-openstack/tree/master/docs)。

在[部署 Devstack cloud](https://berndbausch.medium.com/a-single-server-devstack-cloud-as-a-kubernetes-platform-cd30e28e405a) 之后，您将使用 *kubeadm* 在主实例和工作实例上启动 Kubernetes 集群。然后，您将添加 OpenStack 云提供商和 Cinder CSI 插件。这将使您能够启动 Kubernetes*load balancer*服务，并使用由 OpenStack 资源实现的持久卷。

1.  [安装 Kubernetes](https://medium.com/p/b3f058ce898c#3aae)
2.  [构建 Kubernetes 集群](https://medium.com/p/b3f058ce898c#99aa)
3.  [安装 OpenStack 云提供商](https://medium.com/p/b3f058ce898c#5b15)
4.  [安装 CSI Cinder 插件](https://medium.com/p/b3f058ce898c#7c34)

# 安装 Kubernetes

首先，登录到主节点和工作节点，并将它们的主机名设置为各自的 OpenStack 实例名。

在 Devstack 服务器上，列出实例:

```
$ source ~/devstack/openrc kube kube 
$ openstack server list +--------+---------+--------+-------------------------------------+---------+---------+
| ID     | Name    | Status | Networks                            | Image   | Flavor  |
+--------+---------+--------+-------------------------------------+---------+---------+
| 32d... | worker1 | ACTIVE | kubenet=172.16.0.99, 192.168.1.228  | centos7 | ds4G    |
| cf8... | master1 | ACTIVE | kubenet=172.16.0.169, 192.168.1.221 | centos7 | ds4G    |
+--------+---------+--------+-------------------------------------+---------+---------+
```

如果这些实例不存在，根据[准备脚本](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/devstack-setup.md#configuring-your-cloud)创建它们。

要登录一个实例，您需要它的 IP 地址和 SSH 密钥。上面输出中每个实例的第二个 IP 地址是它的浮动 IP。如果您使用准备脚本创建了实例，那么键就是$HOME/kubekey。

登录到服务器并修复它们的主机名。还要将主机名添加到/etc/hosts 中。

```
$ ssh -i kubekey centos@192.168.1.221 
$ sudo hostnamectl set-hostname master1 
$ echo 192.168.1.221 master1 | sudo tee -a /etc/hosts 
$ echo 192.168.1.228 worker1 | sudo tee -a /etc/hosts 
$ exit 
$ ssh -i kubekey centos@192.168.1.228 
$ sudo hostnamectl set-hostname worker1 
$ echo 192.168.1.221 master1 | sudo tee -a /etc/hosts 
$ echo 192.168.1.228 worker1 | sudo tee -a /etc/hosts 
$ exit
```

接下来，在主节点和工作节点这两个节点上安装 Docker 和 Kubernetes 软件。你可以平行地做那件事。

最简单的方法是运行[安装节点](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/install-node.sh)脚本。这个脚本逐字摘自 kubernetes.io [博客文章](https://kubernetes.io/blog/2020/02/07/deploying-external-openstack-cloud-provider-with-kubeadm/#install-docker-and-kubernetes)。随意手动执行博客中的步骤，但是在`modprobe br_netfilter`之后停止。

# 构建 Kubernetes 集群

以下说明略微改编自官方 [OpenStack 云提供商](https://github.com/kubernetes/cloud-provider-openstack/blob/release-1.19/docs/using-openstack-cloud-controller-manager.md)文档。

## 引导集群

在主服务器上，下载 [kubeadm-config-os.yaml](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/manifests/kubeadm-config-os.yaml) 并将其输入 *kubeadm* 以构建一个简单的集群。 *kubeadm* 命令**必须以 root** 身份运行。

```
$ sudo kubeadm init --config kubeadm-config-os.yaml
```

这需要几分钟时间。在这段时间里，你可能想看看 *kubeadm-config-os.yaml* 。它配置了一个外部云提供商，并定义了一个 10.244.0.0/16 的 pod 子网。

当 *kubeadm* 完成时，它会告诉你如何创建一个*。kube* 配置目录以及如何向集群添加其他节点:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.0.152:6443 --token arr5on.ecrqdbjpt9vqr44z \
	--discovery-token-ca-cert-hash sha256:8c9cae87374f82f18be7ca1bad17c77c30ff1f58b920d37e6d6e5956badc1114
```

创造*。kube/config* 按照指示。记下 *kubeadm join* 命令，但不要执行它。

## 添加网络插件

例如，以**非特权 centos 用户**的身份安装法兰绒 run *kubectl* :

```
$ kubectl apply -f [https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml](https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml)
```

## 将工作实例加入集群

检查 worker 节点上的 Docker 和 Kubernetes 安装是否完成。一旦完成，在 worker 节点上以 root 身份运行上面的 *kubeadm join* 命令**。**

完成后，在主上**，用检查结果**

```
$ kubectl get nodes
NAME      STATUS   ROLES    AGE     VERSION
master1   Ready    master   7m37s   v1.19.2
worker1   Ready    <none>   58s     v1.19.2
```

如果*工人 1* 未准备好，几秒钟后再次检查。

## 确认集群正在工作

您现在有了一个工作的 Kubernetes 集群。您可以随意使用一些 *kubectl get* 命令来探索它，或者在它上面部署一个应用程序。

```
$ kubectl get pods -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-hwmpf           1/1     Running   0          9m12s
coredns-f9fd979d6-kht96           1/1     Running   0          9m12s
etcd-master1                      1/1     Running   0          9m19s
kube-apiserver-master1            1/1     Running   0          9m19s
kube-controller-manager-master1   1/1     Running   0          9m19s
kube-flannel-ds-c27nb             1/1     Running   0          2m53s
kube-flannel-ds-zpbkr             1/1     Running   0          3m43s
kube-proxy-dxnck                  1/1     Running   0          9m12s
kube-proxy-hl62l                  1/1     Running   0          2m53s
kube-scheduler-master1            1/1     Running   0          9m19s
```

# 安装 OpenStack 云提供商

OpenStack cloud provider 使 Kubernetes 集群能够管理在 OpenStack 实例上实现的节点，并使用 OpenStack 负载平衡器。

要安装和配置它，需要以下组件:

*   云身份验证详细信息
*   其他云详细信息，如负载平衡器所连接的网络
*   云控制器经理的 RBAC 角色
*   云控制器管理器

## 配置身份验证和网络详细信息

下面的模板包含了云的细节(点 1 和点 2)。他们告诉 Cinder CSI 插件如何访问云。

```
[Global]
region=RegionOne
username=kube
password=pw
auth-url=http://<<DEVSTACK-SERVER-IP/identity/v3>>
tenant-id=<<KUBE-PROJECT-ID>>
domain-id=default[LoadBalancer]
network-id=<<ID-OF-KUBENET-NETWORK>> [Networking]
public-network-name=public
```

要获得`KUBE-PROJECT-ID`和`ID-OF-KUBENET-NETWORK`，在 Devstack 服务器上运行这些命令**:**

```
$ source ~/devstack/openrc kube kube
$ openstack project show kube -c id
+-------+----------------------------------+
| Field | Value                            |
+-------+----------------------------------+
| id    | 7a099eff1fb1479b89fa721da3e1a018 |
+-------+----------------------------------+

$ openstack network list -c ID -c Name
+--------------------------------------+---------+
| ID                                   | Name    |
+--------------------------------------+---------+
| 0e80607f-7cd1-4fe5-a782-3459fa447202 | public  |
| dd07c40e-5e6d-4f4c-bb66-cfdcee6dbc70 | kubenet |
| e9bbe2fc-daa3-4cdf-aaf3-4c65c8a1a321 | shared  |
+--------------------------------------+---------+
```

**在主节点**上，将上述模板复制到名为 *cloud.conf* 的文件中，并替换两个 id 以及 Devstack 服务器的 IP 地址。由于此配置包含密码，因此必须将其转换为密码:

```
$ kubectl create secret -n kube-system generic cloud-config \
                        --from-file=cloud.conf
```

## 创建 RBAC 资源

使用 OpenStack 云提供商报告中的清单来创建 RBAC 资源。

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/cluster/addons/rbac/cloud-controller-manager-roles.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/
```

## 启动控制器管理器

使用 Github 上 OpenStack 云提供商 repo 中的清单。

```
kubectl apply -f [https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/manifests/controller-manager/openstack-cloud-controller-manager-ds.yaml](https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/manifests/controller-manager/openstack-cloud-controller-manager-ds.yaml)
```

当控制器管理器启动时，探索它的清单可能是有益的。云控制器被实现为具有单个容器的 *daemonset* 。看看容器中的命令及其选项。分析这个秘密是如何使用的:它被挂载为一个名为 *cloud-config-volume* 的卷。

启动将需要几秒钟才能完成。用…检验它的成功

```
$ kubectl -n kube-system get daemonset,pv,pod
NAME                                                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
daemonset.apps/kube-flannel-ds                      2         2         2       2            2           <none>                            38m
daemonset.apps/kube-proxy                           2         2         2       2            2           kubernetes.io/os=linux            44m
daemonset.apps/openstack-cloud-controller-manager   1         1         0       1            0           node-role.kubernetes.io/master=   34s

NAME                                           READY   STATUS    RESTARTS   AGE
pod/coredns-f9fd979d6-hwmpf                    1/1     Running   0          44m
pod/coredns-f9fd979d6-kht96                    1/1     Running   0          44m
pod/etcd-master1                               1/1     Running   0          44m
pod/kube-apiserver-master1                     1/1     Running   0          44m
pod/kube-controller-manager-master1            1/1     Running   0          44m
pod/kube-flannel-ds-c27nb                      1/1     Running   0          38m
pod/kube-flannel-ds-zpbkr                      1/1     Running   0          38m
pod/kube-proxy-dxnck                           1/1     Running   0          44m
pod/kube-proxy-hl62l                           1/1     Running   0          38m
pod/kube-scheduler-master1                     1/1     Running   0          44m
pod/openstack-cloud-controller-manager-9c2pp   0/1     Error     1          34s
```

(最后一行显示这次发射失败)。

启动成功后，您可以通过创建负载平衡器服务来测试它。或者直接跳到 [Cinder CSI 插件](https://github.com/berndbausch/Devstack-Kubernetes/blob/main/k8s-manual-setup.md#cinder)的安装。

## 故障排除提示

如果过一会儿云控制器管理器 pod 不处于运行状态*，您需要找出问题所在。 *cloud.conf* 可能不正确，机密名称可能与云控制器管理器清单不一致，云控制器管理器可能无法连接到云或向其进行身份验证。有关详细信息，请查看 pod 的日志。*

```
*$ kubectl -n kube-system logs pod/openstack-cloud-controller-manager-9c2pp
...
... openstack.go:300] failed to read config: 10:11: illegal character U+003A ':'
... controllermanager.go:131] Cloud provider could not be initialized: could not init cloud provider "openstack": 10:11: illegal character U+003A ':'*
```

*在这个例子中， *cloud.conf* 在第 10 行包含了一个冒号而不是等号。通过移除秘密、更正 *cloud.conf* 、重新创建秘密并启动新的 daemonset，修复了此问题:*

```
*$ kubectl -n kube-system delete secret cloud-config
$ kubectl create secret -n kube-system generic cloud-config --from-file=cloud.conf
$ kubectl -n kube-system delete daemonset.apps/openstack-cloud-controller-manager
# wait a moment for this to settle...
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/manifests/controller-manager/openstack-cloud-controller-manager-ds.yaml*
```

*如果日志没有足够的信息，请提高日志级别。这是通过 *daemonset* 清单中的选项 `--v`完成的。详细级别 6 包括对 OpenStack 云的请求的详细信息，在身份验证或其他云操作失败时非常有用。将货单复制到主机，将`*--v=1*`替换为`*--v=6*`，并重新应用货单。*

# *安装 CSI Cinder 插件*

*Cinder 的 CSI(云存储接口)允许创建持久卷和由 Cinder 卷支持的持久卷声明。以下说明基于 [OpenStack 云提供商文档](https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/using-cinder-csi-plugin.md#using-the-manifests)。*

## *将 Cinder CSI 清单复制到主机*

*将原始的[煤渣 CSI 清单](https://github.com/kubernetes/cloud-provider-openstack/tree/release-1.19/manifests/cinder-csi-plugin)复制到主机。您可以一个文件一个文件地下载，或者更好的方法是，简单地克隆整个回购协议，或者将回购协议下载为 ZIP 文件(大约 0.5MB)。*

*在**主机**上执行的以下代码下载 ZIP 文件，并将相关清单复制到一个新的`manifest`目录中。不要复制`csi-secret-cinderplugin.yaml`，因为云配置秘密已经存在。*

```
*$ cd
$ mkdir manifests
$ wget https://github.com/kubernetes/cloud-provider-openstack/archive/release-1.19.zip
$ unzip release-1.19.zip
$ cd cloud-provider-openstack-release-1.19/manifests/cinder-csi-plugin
$ cp cinder* csi-cinder-driver.yaml ~/manifests/*
```

## *应用煤渣 CSI 清单*

*转到清单目录。如果您想提高容器的日志记录级别，请将`*--v=6*`选项添加到 *controllerplugin* 和 *nodeplugin* 清单中的所有容器中。该选项生成大量日志消息，包括插件发布给 OpenStack cloud 的 API 的详细信息。*

*应用所有清单。这将创建必要的 RBAC 角色，并启动控制器插件，节点插件和 Cinder 驱动程序。*

```
*$ cd ~/manifests 
$ kubectl apply -f .*
```

*确认所有 pod 都已启动并运行。如果没有，请参见上面的故障排除部分。*

```
*$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-hwmpf                    1/1     Running   0          6h13m
coredns-f9fd979d6-kht96                    1/1     Running   0          6h13m
csi-cinder-controllerplugin-0              5/5     Running   0          7m12s
csi-cinder-nodeplugin-vrczm                2/2     Running   0          7m11s
etcd-master1                               1/1     Running   0          6h14m
kube-apiserver-master1                     1/1     Running   0          6h14m
kube-controller-manager-master1            1/1     Running   0          6h14m
kube-flannel-ds-c27nb                      1/1     Running   0          6h7m
kube-flannel-ds-zpbkr                      1/1     Running   0          6h8m
kube-proxy-dxnck                           1/1     Running   0          6h13m
kube-proxy-hl62l                           1/1     Running   0          6h7m
kube-scheduler-master1                     1/1     Running   0          6h14m
openstack-cloud-controller-manager-t8kfj   1/1     Running   0          5h10m*
```

*现在，您应该可以在该集群上启动利用 OpenStack 负载平衡器和卷服务的应用程序了。*
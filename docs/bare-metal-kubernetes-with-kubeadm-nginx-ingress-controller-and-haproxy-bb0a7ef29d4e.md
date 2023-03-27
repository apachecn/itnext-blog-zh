# 带 Kubeadm、NGINX 入口控制器和 HAProxy 的裸机 Kubernetes

> 原文：<https://itnext.io/bare-metal-kubernetes-with-kubeadm-nginx-ingress-controller-and-haproxy-bb0a7ef29d4e?source=collection_archive---------0----------------------->

![](img/b97b8885fcdadde567a21f38c5c2c6dc.png)

图片由[托尔斯滕·弗伦泽尔](https://pixabay.com/users/thorstenf-7677369/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=4197733)从[皮克斯拜](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=4197733)拍摄

下面是我关于设置“裸机”Kubernetes (k8s)集群的一些笔记。我将裸机放在双引号中，因为我实际上使用了 AWS EC2 实例，但只是作为普通的虚拟机，目标是能够在任何云或内部基础架构中部署此场景，而不依赖于 k8s 云服务，如 EKS、AKS 或 GKE。我选择了 [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) 作为 Kubernetes 集群部署工具，因为它是官方 Kubernetes 项目的一部分。

这种设置的总体架构是:

*   HAProxy 运行在专用 EC2 实例上，充当负载平衡器和从 Kubernetes 公开的端点 DNS 名称的 TLS 终端
*   默认的 HAProxy 后端由 3 个 k8s 节点 IP 地址组成，每个地址的端口对应于 nginx 入口控制器 HTTP 节点端口
*   NGINX 入口控制器作为 NodePort 类型的服务安装在 k8s 集群中
*   基于 NFS 的自定义 k8s 存储类
*   舵图安装时启用入口

**Kubeadm 设置**

我首先提供了 3 个运行 Ubuntu 20.04 的 EC2 实例，一个作为 k8s 主节点，另外两个作为 k8s 工作节点。然后，我在所有 3 个实例上运行以下脚本。它设置一些与网络相关的系统变量，安装 containerd 等先决条件(我想避免 Docker，因为 Kubernetes 放弃了对 Docker 的支持)，然后安装 kubeadm、kubelet 和 kubectl，将这三个都固定到版本 1.19.7(如果没有指定版本，将安装最新的 k8s 1.20，这有点太新，一些工具仍然不太支持):

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.confoverlaybr_netfilterEOFsudo modprobe overlaysudo modprobe br_netfiltercat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.confnet.bridge.bridge-nf-call-iptables  = 1net.ipv4.ip_forward                 = 1net.bridge.bridge-nf-call-ip6tables = 1EOFsudo sysctl --systemsudo apt-get update && sudo apt-get install -y containerdsudo mkdir -p /etc/containerdsudo containerd config default | sudo tee /etc/containerd/config.tomlsudo systemctl restart containerdsudo apt-get update && sudo apt-get install -y apt-transport-https curlcurl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.listdeb https://apt.kubernetes.io/ kubernetes-xenial mainEOFsudo apt-get updatesudo apt-get install -y kubelet=1.19.7-00 kubeadm=1.19.7-00 kubectl=1.19.7-00sudo apt-mark hold kubelet kubeadm kubectl
```

在主节点上，我运行:

```
# kubeadm init
```

并写下了我将需要在工作节点上运行的`kubeadm join`命令。在加入工作节点之前，我还在主节点上安装了 [Calico](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises) 网络覆盖:

```
# export KUBECONFIG=/etc/kubernetes/admin.conf
# curl https://docs.projectcalico.org/manifests/calico.yaml -O
# kubectl apply -f calico.yaml
```

我还取消了主节点，这样我就可以在其上运行常规的 pod:

```
# kubectl taint nodes --all node-role.kubernetes.io/master-
```

此时，我已经准备好将工作节点连接到主节点。我在每个 worker 节点上运行了一个类似于下面的命令(这个命令是从主节点上的 *kubeadm init* 输出粘贴过来的):

```
# kubeadm join MASTER_NODE_IP_ADDRESS:6443 --token TOKEN --discovery-token-ca-cert-hash sha256:CA-CERT-HASH
```

我将文件`/etc/kubernetes/admin.conf`从主节点传输到我的客户端笔记本电脑，并为其设置 KUBECONFIG 变量。然后，我能够对 Kubeadm 集群运行`kubectl`命令。

**NGINX 入口控制器设置**

我在本地机器上下载了最新的 [NGINX 入口控制器](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/) k8s 部署清单:

```
$ wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml
```

然后我创建了一个名为`ingress-nginx`的 k8s 名称空间

```
$ kubectl create ns ingress-nginx
```

我编辑了`deploy.yaml`文件并将`service`类型指定为`NodePort`，然后在`ingress-nginx`名称空间中应用清单:

```
$ kubectl apply -f deploy.yaml
```

如果部署成功，您应该会看到类型为`NodePort`的`ingress-nginx-controller`服务:

```
$ kubectl get services -n ingress-nginxNAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGEingress-nginx-controller             NodePort    10.104.246.104   <none>        80:32012/TCP,443:30572/TCP   52mingress-nginx-controller-admission   ClusterIP   10.101.12.34     <none>        443/TCP                      52m
```

这里的大问题是如何进一步将入口控制器暴露给外界。如果我们要使用云服务，如 EKS、AKS 或 GKE，很容易声明 NGINX 入口控制器服务属于`LoadBalancer`类型，然后依靠云提供商为其创建本地负载平衡器。在裸机场景中，我们需要另一种解决方案。

一个出现在 [NGINX 入口控制器文档](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#a-pure-software-solution-metallb)中的解决方案是使用 [MetalLB](https://metallb.universe.tf/) 。但是，MetalLB 有自己的要求，这些要求依赖于 AWS 中不容易提供的操作系统和网络功能。我试着在 EC2 实例上运行的 Kubeadm 集群中安装它，但遇到了错误。事实上，MetalLB 的[云兼容性](https://metallb.universe.tf/installation/clouds/)矩阵建议在 AWS 环境中使用 EKS。如果我需要在纯裸机本地环境中运行 Kubernetes，并且能够控制我使用的操作系统和网络，我会尝试使用 MetalLB。

我们可以继续在端点中使用 NodePort 值，这些端点由通过 NGINX 入口控制器创建的入口对象公开(在上面我的服务片段中，这些端口对于 http 流量是 32012，对于 https 流量是 30572)。这相当难看，所以我找到的解决方案是在 k8s 集群前面安装 HAProxy(在 NGINX ingress 控制器文档中称为"[自配置边缘](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#using-a-self-provisioned-edge)"场景)，并使用端口 32012 将后端映射到三个 k8s 节点。细节继续往下。

**总部位于 NFS 的 Kubernetes 存储类设置**

在云环境中，我们认为理所当然的另一件事是云提供商在 Kubernetes 请求时自动创建的永久卷。在 AWS 中，默认情况下，这些卷是 EBS 卷。在裸机环境中，我们需要创建特定的存储类别来处理这种自动配置。我发现的一个解决方案是在我的 k8s 集群中使用 [NFS 客户端供应器](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)，并将其配置为自动创建通过我设置的 NFS 服务器提供服务的 NFS 卷。

为了简单起见，我将 k8s 主节点配置为 NFS 服务器。

```
# apt install -y nfs-kernel-server
```

我在`/etc/exports`文件中声明了一个名为`nfs-export`的卷，并通过运行`exportfs -a`将其导出:

```
# cat /etc/exports
/nfs-export               10.0.0.0/8(rw,sync,no_root_squash,no_subtree_check)# exportfs -a# exportfs/nfs-export    10.0.0.0/8
```

我还必须在 k8s 工作节点上安装 NFS 客户端包:

```
# apt install -y nfs-common
```

然后我克隆了`nfs-subdir-external-provisioner` Git repo，其中包含 NFS 客户端供应器的 helm 图表:

```
$ git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git
```

我编辑了文件*nfs-subdir-external-provisioner/deploy/helm/values . YAML*，指定了 NFS 服务器 IP 和导出路径，以及新存储类的名称`nfs-client`:

```
nfs: server: IP_OF_MASTER_NODE path: /nfs-exportstorageClass: create: true name: nfs-client
```

我用 [helm v3 CLI](https://helm.sh/docs/intro/install/) 安装了 helm 图表:

```
$ helm install nfs-subdir-external-provisioner \--namespace default \./nfs-subdir-external-provisioner/deploy/helm
```

此时，我能够看到新的`nfs-client`存储类:

```
$ kubectl get storageclassNAME         PROVISIONER                            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGEnfs-client   cluster.local/nfs-client-provisioner   Delete          Immediate           true                   34s
```

要使用这个存储类，需要在 k8s 集群中安装的舵图的`values.yaml`文件中指定它的名称。大多数图表都有一个`persistence`部分，其中可以指定`storageClass`的名称(在本例中为`nfs-client`)。

**入口启用的舵图设置**

我将给出一个非常简单的安装[无人机](https://www.drone.io/) CI/CD 工具舵图的例子。

我从官方无人机报告中克隆了无人机舵图:

```
$ git clone [https://github.com/drone/charts](https://github.com/drone/charts) drone-charts
```

我编辑了`drone-charts/charts/drone/values.yaml`并设定:

```
service: type: NodePortingress: enabled: true annotations: kubernetes.io/ingress.class: nginx hosts: - host: drone.mydomain.com paths: - "/"persistentVolume: storageClass: nfs-clientenv: DRONE_SERVER_HOST: "drone.mydomain.com" DRONE_SERVER_PROTO: https
```

服务类型需要是`NodePort`，以便 NGINX 入口控制器可以正确地创建`Ingress`对象。我没有为舵图设置 SSL/TLS，但我仍然将外部 URL 指定为[https://drone.mydomain.com](https://drone.mydomain.com)，因为它将由 HAProxy 处理。

我创建了一个`drone`名称空间，并在其中安装了无人机图表:

```
$ kubectl create namespace drone
$ helm install drone -n drone ./drone-charts/charts/drone
```

如果一切顺利，您现在应该在`drone`名称空间中看到一个`Ingress`对象:

```
$ kubectl get ingress -ndroneNAME    CLASS    HOSTS               ADDRESS        PORTS   AGEdrone   <none>   drone.mydomain.com   NODE-IP-ADDRESS  80      61s
```

`ADDRESS`字段将包含 k8s 节点之一的 IP 地址。`drone`服务在端口 80 上运行，但是 NGINX 入口控制器会在`NodePort` 32012 上暴露它。

**HAProxy 负载平衡器设置**

本例中剩下要做的就是在 NGINX 入口控制器前面设置 HAProxy 作为边缘负载平衡器。我还获得了一个用于`*.mydomain.com`的 SSL 证书，并将证书文件、CA 包和私钥(按照这个顺序)组合到一个名为`/etc/haproxy/haproxy-ssl-cert.pem`的 PEM 文件中，然后我在 HAProxy 配置文件中指定了这个文件。

我提供了一个运行 Ubuntu 20.04 的 EC2 实例，并通过以下方式安装了 HAProxy:

```
$ sudo apt install -y haproxy$ haproxy -vHA-Proxy version 2.0.13-2ubuntu0.1 2020/09/08 - https://haproxy.org/
```

我编辑了`/etc/haproxy/haproxy.cfg`并在默认情况下已经包含的`global`和`default`部分下添加了以下行:

```
frontend front_nginx_ingress_controller bind LOCAL-IP-OF-HAPROXY-EC2-INSTANCE:80 bind LOCAL-IP-OF-HAPROXY-EC2-INSTANCE:443 ssl crt /etc/haproxy/haproxy-ssl-cert.pem http-request redirect scheme https unless { ssl_fc } default_backend nginx_ingress_controller_servicebackend nginx_ingress_controller_service balance roundrobin server k8s-node1 IP-OF-K8S-NODE1:32012 check port 32012 server k8s-node2 IP-OF-K8S-NODE2:32012 check port 32012 server k8s-node3 IP-OF-K8S-NODE3:32012 check port 32012
```

上面的行告诉 HAProxy 使用我提到的证书进行 SSL/TLS 终止，将传入的 http 流量重定向到 https，并使用循环算法在 3 个 k8s 节点的 IP 地址之间进行负载平衡，使用`32012`作为端口号，这与 NGINX 入口控制器公开的 http `NodePort`的值完全相同(您还需要修改 k8s 节点的 EC2 安全组，以允许该端口上来自 HAProxy EC2 实例的传入流量)。

然后我重启了 haproxy 服务:

```
$ sudo service haproxy restart
```

此时，我能够在浏览器中点击 https://drone.mydomain.com 的[并看到正确的证书和正确的无人机登录页面。流量流向为:浏览器- > HAProxy 前端- > k8s 节点- > nginx 入口控制器- >入口对象- >运行在端口 80 上的无人机服务- >无人机 pod](https://drone.mydomain.com)

这就是了。一种端到端设置，用于公开运行在具有外部 DNS 名称和 SSL/TLS 的裸机节点上的 kubernetes 服务。像往常一样，HAProxy 拯救了这一天(它从 2008 年开始拯救我的培根！).

**注意**:为了更加安全，您可以安装启用了 TLS 的 helm charts(即使使用自签名证书)，然后配置 HAProxy 使用 nginx 控制器 https `NodePort`值向后端 k8s 节点 IP 地址发送流量，在我的示例中该值为 30572。

**更新**:我在 kubeadm k8s 集群中部署的一些服务实际上需要 https 流量，所以我继续重新配置 HAProxy，将 https 流量发送到后端节点，指定`verify none`，因为我不需要在目标 k8s 节点上验证 SSL 证书，在这种情况下，它是一个自签名的 NGINX 入口控制器证书。以下是`haproxy.cfg`中更新后的`backend`部分:

```
backend nginx_ingress_controller_service balance roundrobin server k8s-node1 IP-OF-K8S-NODE1:30572 ssl check port 30572 verify none server k8s-node2 IP-OF-K8S-NODE2:30572 ssl check port 30572 verify none server k8s-node3 IP-OF-K8S-NODE3:30572 ssl check port 30572 verify none
```
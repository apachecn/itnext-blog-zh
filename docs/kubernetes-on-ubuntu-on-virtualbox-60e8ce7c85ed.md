# kubernetes over Ubuntu on VirtualBox

> 原文：<https://itnext.io/kubernetes-on-ubuntu-on-virtualbox-60e8ce7c85ed?source=collection_archive---------2----------------------->

## Kubernetes 是一个主流。让我们在本地环境中创建用于开发的集群

![](img/69ea6784cc9654be6ecc84b173c0f8f6.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fkubernetes-on-ubuntu-on-virtualbox-60e8ce7c85ed)

## 环境:

*   ArchLinux 上的 VirtualBox 5.2.6
*   每台虚拟机上安装 Ubuntu 16.04。
*   Kubernetes 1.9.3

## 最低[要求](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#before-you-begin):

*   2CPU
*   2GB 内存
*   **禁用交换**
*   20GB 硬盘(仅用于测试)

虚拟机有两个网络接口。一个位于 NAT 网络之后，用于互联网连接，第二个主机仅用于虚拟机之间的内部通信。纯主机网络应该不同于 kubernetes 虚拟网络。我选 172.30.56.0/24。Kubernetes 将有 3 个节点:

*   kube-master(管理、监控)
*   kube-工人-1
*   kube-工人-2

worker-1 和 worker-2 将用于所有用户窗格。

为了构建 kubernetes 集群，我使用了 **kubeadm。**关于如何设置 kubeadm 的所有说明，kubectl 在这里描述[。](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

## 让我们开始:

1.  安装[docker 环境。](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
2.  [安装](https://kubernetes.io/docs/setup/independent/install-kubeadm/) kubelet，kubeadm，kubectl。
3.  为 kubelet 和 docker 更改 **cgroup-driver**

对于码头工人:

```
cat << EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

对于库伯莱:

```
#edit kubeadm.service file
> vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# add to the end **--cgroup-driver=systemd** > systemctl daemon-reload && systemctl restart kubelet docker
```

4.初始化主机

```
> kubeadm init --apiserver-advertise-address=172.30.56.120 --pod-network-cidr=192.168.0.0/16
```

> —API server-advertise-address =<ip adress="">—更改标准 API IP 地址。</ip>
> 
> —pod-network-CIDR = 192 . 168 . 0 . 0/16—引导印花棉布网络时需要。

5.将 kube 配置复制到主文件夹。需要通过 kubectl 管理集群。此外，如果需要在不同于 master 的主机上管理集群，您可以将此配置复制到 workers 或本地机器。

```
> mkdir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

6.安装虚拟网络。我选择[印花布](https://www.projectcalico.org/)。

```
> kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml
```

7.不要忘记复制令牌散列。将工人添加到集群和登录仪表板时需要它。代币将 24 小时开放。

> 不要担心令牌是否会过期 **kubeadm 令牌创建**和 **kubeadm 令牌列表**将帮助创建新令牌用于加入节点和登录仪表板。

每个工人都应该安装 **docker** 软件和 **kubeadm** 。要将节点加入群集，请在每个虚拟机上执行下一个命令。

```
> kubeadm join --token 6a1c63.4d6b405ebfba2021 172.30.56.120:6443 --discovery-token-ca-cert-hash sha256:8fac8a430850de77f17e7eab0bdfbcd59107bf8f60edd3538e098f46e0d2287b
```

显示集群成员(在主服务器上):

```
> kubectl get nodes
```

8.默认情况下，主设备不处理用户单元，但是如果集群很小，主设备可以放置用户单元。

```
> kubectl taint nodes --all node-role.kubernetes.io/master-
```

9.集群已创建并可供使用。只需启动内置 linux 的简单 pod。

```
> kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
```

[busybox](https://www.busybox.net/) —它是简单的 linux，里面有最少的软件。适合从头开始创建容器。

现在 **kubernetes** 集群已经为豆荚做好了准备。 **kubectl** 是管理集群的主命令。

## 仪表板安装

> 仪表板不是管理 kubernetes 集群的安全方式。仪表板以完全管理权限执行。小心共享对 dashboard 的访问。用户可以强奸你的集群。

1.  [安装](https://github.com/kubernetes/dashboard)仪表板。

```
> kubectl apply -f [https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml)
```

仪表板-具有管理功能的用户界面。对于度量收集，需要安装 Heapster。

2.[安装带有 influxDB 的](https://github.com/kubernetes/heapster/) heapster，用于度量存储。

```
> git clone [https://github.com/kubernetes/heapster.git](https://github.com/kubernetes/heapster.git)
> cd heapster
> kubectl create -f deploy/kube-config/influxdb/
> kubectl create -f deploy/kube-config/rbac/heapster-rbac.yaml
```

3.在所有 pod 成功启动后，启动 ku bectl proxy for open dashboard web 应用程序。

```
> kubectl proxy
```

kube 代理开始监听本地主机上的 8001 端口。将下一行放到浏览器中。

```
[**http://localhost:8001/ui**](http://localhost:8001/)
```

kebernetes 给你一个日志页面，有两个选项:用 kubeconfig 或 token 登录。最简单的方法是使用令牌。 **kubeadm 令牌列表**将显示所有创建的令牌。选择其中一个登录应用程序。

## 提示和技巧

*   **如果虚拟机的 CPU 少于 2 个，kube-dns** 不会启动。
*   有时登录仪表板后会返回安全错误:

```
User "system:bootstrap:883ae1" cannot list configmaps in the namespace "default"
```

只需将用户添加到集群管理组:

```
> kubectl create clusterrolebinding --user system:bootstrap:883ae1 kube-system-cluster-admin --clusterrole cluster-admin
```

*   描述系统单元:

```
kubectl describe pod kube-apiserver-kube-master -n kube-system
```

*   显示窗格中的日志:

```
kubectl logs kube-dns-6f4fd4bdf-kr8wh -n kube-system
```

享受吧。
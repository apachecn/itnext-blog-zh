# Kubernetes:第 3 部分— AWS EKS 概述和手动 EKS 集群设置

> 原文：<https://itnext.io/kubernetes-part-3-aws-eks-overview-and-manual-eks-cluster-set-up-f35b6eca2763?source=collection_archive---------6----------------------->

![](img/661b4110ee4351469b87b71655df5c0d.png)

让我们继续我们的库伯内特之旅。

以前的零件:

*   [Kubernetes:第 1 部分—架构和主要组件概述](https://rtfm.co.ua/en/kubernetes-part-1-architecture-and-main-components-overview/)
*   [Kubernetes:第 2 部分——在 AWS 上建立一个集群，使用 AWS 云提供商和 AWS 负载均衡器](https://rtfm.co.ua/en/kubernetes-part-2-a-cluster-set-up-on-aws-with-aws-cloud-provider-and-aws-loadbalancer/)

在这一部分中，我们将开始与 AWS Elastic Kuberneters Service(EKS)合作，这是它的简短概述，然后将创建 Kubernetes 控制平面、带有工作节点的 CloudFormation 堆栈，将启动一个简单的 web 服务，并将添加一个负载平衡器。

*   [弹性 Kubernetes 服务—概述](#f368)
*   [准备 AWS 环境](#1907)
*   [我的角色](#2377)
*   [VPC](#c7be)
*   [安全组](#6d78)
*   [互联网网关](#17c5)
*   [子网](#b1e8)
*   [NAT 网关](#e13f)
*   [路由表](#d93e)
*   [弹性 Kubernetes 服务](#ca27)
*   [创建一个控制平面](#3803)
*   [创建工人节点](#fcfe)
*   [kubectl 安装](#03c0)
*   [AWS 认证器](#7273)
*   [Web-app & &负载平衡器](#2ffe)

## 弹性 Kubernetes 服务—概述

AWS EKS 是一个 Kubernetes 集群，其核心[控制平面](https://rtfm.co.ua/kubernetes-znakomstvo-chast-1-arxitektura-i-osnovnye-komponenty-obzor/#Kubernetes_core_services_aka_Kubernetes_Control_Plane)将由 AWS 自己管理，从而将用户从不必要的麻烦中解放出来。

*   [控制平面](https://rtfm.co.ua/kubernetes-znakomstvo-chast-1-arxitektura-i-osnovnye-komponenty-obzor/#Kubernetes_core_services_aka_Kubernetes_Control_Plane):由 AWS 管理，由不同可用性区域中的三个 EC2 组成
*   [Worker Nodes](https://rtfm.co.ua/kubernetes-znakomstvo-chast-1-arxitektura-i-osnovnye-komponenty-obzor/#Worker_Node) :自动缩放组中的一个普通ес2，在客户的 VPC 中，由用户管理

![](img/e3a862bb10f48d70c9d49e98e95ea9e0.png)

网络概述:

![](img/64be2b2bb61357d45152e93da1b9c871.png)

对于网络——使用了 [amazon-vpc-cni-k8s](https://github.com/aws/amazon-vpc-cni-k8s) 插件，该插件允许在集群内使用 AWS ENI(弹性网络接口)和 vpc 的网络空间。

![](img/1cea452ac13760602f0b4ff91228205c.png)

对于授权，使用了 [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html) ，它允许根据 AWS iam 角色和策略对 Kubernetes 对象进行身份验证(参见[管理集群的用户或 IAM 角色](https://docs.aws.amazon.com/en_us/eks/latest/userguide/add-user-role.html)

此外，AWS 将管理 Kubernetes 的小升级，即 1.11.5 到 1.11.8，但大升级仍然必须由用户完成。

## 准备 AWS 环境

要创建 EKS 集群 firt，我们需要创建一个带有子网的专用 VPC，配置路由，并为集群授权添加一个 IAM 角色。

## IAM 角色

转到 *IAM* ，用 *EKS* 类型创建一个新角色:

![](img/f915860535dd451e89c108459bf41a5e.png)

*权限*将由 AWS 自己填写:

![](img/34240d1b207b89ab738b98893ca5db03.png)

保存它:

![](img/509b926bc2fcaa466aeae144c831cb71.png)

## VPC

接下来，必须创建具有 4 个子网的 VPC，其中两个公共子网用于 LoadBalacner，两个私有子网用于工作节点。

创建 VPC:

![](img/e45aeb3dc7ee878196070b53e97eb465.png)

**安全组**

转到*安全组*，为集群创建一个新组:

![](img/615a5acd953d714e414f9461667dc233.png)

添加所需的规则，这里只是一个 *Allow All to All* 的例子:

![](img/ee991b77213ceaf1a8744087524c39a6.png)![](img/18cb32421e81d9132f5a4363ec711fe5.png)

**互联网网关**

创建将用于路由来自公共子网的流量的 IGW:

![](img/17dd559c0b45b96810acf5083a86c285.png)

将它连接到 VPC:

![](img/f775e4be0dea15502a65220ef41f25e4.png)![](img/7913603cd7eeaf26b498a912b8b63e59.png)

**子网**

Pods 将使用分配的子网中的 IP(参见 [amazon-vpc-cni-k8s](https://github.com/aws/amazon-vpc-cni-k8s) )，因此这些子网必须有足够的地址空间。

使用 *10.0.0.0/18 块* (16384 个地址)创建第一个公共子网:

![](img/372de7ae5171eb03162d376da70aa84a.png)

使用 *10.0.64.0/18* 块的第二个公共子网:

![](img/2599e44ebda2be667bc2c780e1507e14.png)

在公共子网中—启用将公共 IP 自动分配给 ec2:

![](img/573778098acbb1c762f58d83b56fe104.png)![](img/d55ea203ce240aa223d652e627dfcfb4.png)

同样，添加两个专用子网:

![](img/49b9e24100276dc95b4bf45080667821.png)![](img/f3ca3c656237ed2b16cad02b16338ec8.png)

**NAT 网关**

在公共子网中创建一个 NAT 网关，它将用于路由来自私有子网的流量:

![](img/fa273a47293a4ca3eee13ce21894c869.png)

并在此配置路由:

![](img/126118d6a70a2d7c6648b44fd73b04fb.png)

**路由表**

现在，需要为公共子网和私有子网创建两个路由表。

公共路由表

创建公共子网路由表:

![](img/4418a52913d0500383e338d7dcc9da86.png)

编辑路线—通过上面创建的 IGW 将路线设置为 *0.0.0.0/0* :

![](img/84592759ea28f9df949e1f6d887cf2f9.png)

切换到*子网关联* —将两个公共子网连接到此 RTB:

![](img/82d6d560e9dc2b0db9d21db811d17585.png)

私有路由表

同样，为专用子网创建 RTB:

![](img/a1df40b0b662b9d36f85cc88aad742d3.png)

向 *0.0.0.0/0* 添加另一条路线，但通过 NAT GW 而不是 IGW:

![](img/d71c135913ff7320df49e5e8b1e588a9.png)

回到您的子网— *编辑路由表关联*:

![](img/73b5f4e6e873da877f733f307b0b6f2d.png)

将我们的专用 RTB 连接到专用子网，以便它们使用 NAT GW:

![](img/eedc2a2b1f06b90704591d630b1013aa.png)

将我们的公共 RTB 连接到专用子网，以便它们使用互联网网关:

![](img/83373432a602696531bf978da156345f.png)

## 支票

要测试这个 VPC 是否工作，请运行两个 EC2 实例。

公共子网中的第一个:

![](img/109b83f1f5e0487f714d234448d60b4e.png)

设置安全组:

![](img/02e2685e3436c2129afa51adb89f52e8.png)

检查网络:

```
[setevoy@setevoy-arch-work ~] $ ssh -i setevoy-testing-eu-west-2.pem ubuntu@35.178.171.252 ‘ping -c 1 8.8.8.8’
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=51 time=1.33 ms
 — — 8.8.8.8 ping statistics — -
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.331/1.331/1.331/0.000 ms
```

在专用子网中添加另一个 EC2:

![](img/e8e191465ea7b8a009cc794ba4832cd4.png)

不要忘记 SG。

并尝试从第一个实例 ping 它(因为我们不能从互联网 ping 私有网络中的实例):

```
[setevoy@setevoy-arch-work ~] $ ssh -i setevoy-testing-eu-west-2.pem ubuntu@35.178.171.252 ‘ping -c 1 10.0.184.21’
PING 10.0.184.21 (10.0.184.21) 56(84) bytes of data.
64 bytes from 10.0.184.21: icmp_seq=1 ttl=64 time=0.357 ms
 — — 10.0.184.21 ping statistics — -
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.357/0.357/0.357/0.000 ms
```

如果没有对 ping 的回复—首先检查您的安全组和路由表。

我们到此为止——是时候从 EKS 本身开始了。

## 弹性 Kubernetes 服务

## 创建一个控制平面

转到 EKS 并创建主节点—单击*创建集群*:

![](img/c55bd062ea4ec7384abe00742c182367.png)

设置名称，选择在最开始创建的 IAM 角色:

![](img/159f31f4ba6f9e15f76ec533a83da12f.png)

在子网中，选择仅专用子网并设置上面创建的安全性组:

![](img/5b23a8a175b50e43ae24bed85af4366d.png)

如果需要，启用日志:

![](img/ddd02ed91790a0b51140a1587d5972df.png)

并创建集群:

![](img/e5a4436e154ce47f9d893db35e58b55a.png)

## 创建工作节点

当控制平面处于调配状态时，让我们为工作节点创建一个云形成堆栈。

可以从 AWS 中取一个已有的模板—[https://Amazon-eks . S3-us-west-2 . Amazon AWS . com/cloud formation/2019-02-11/Amazon-eks-node group . YAML](https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/amazon-eks-nodegroup.yaml)。

转到*云形成>创建堆栈*:

![](img/5a8fdf9a9e26ab51ccdee1e3c1f062cf.png)

由于我们的工作节点将放置在专用子网中，因此在设计器中打开此模板:

![](img/8a02934a2ed9961e27904ae2ab3f0fbd.png)

找到`AssociatePublicIpAddress`参数，将其值从*真*更改为*假*:

![](img/f2c6533c3c28e6abe08b2907b090d26b.png)

点击*创建堆栈*:

![](img/380c8bb3efceba22eeefe8390b649849.png)

设置堆栈的名称(可以是任意名称)和群集名称—与我们创建主节点时所做的相同，例如 *eks-cluster-manual* 在本例中，选择安全组，填充自动缩放设置:

![](img/8ff3c58af904d0545ca75883976aee44.png)

根据地区查找`NodeImageId`—查看[文档](https://docs.aws.amazon.com/en_us/eks/latest/userguide/getting-started-console.html#eks-launch-workers)获取最新列表。

目前栈是在 London/eu-west-2 创建，不需要在 GPU，于是——*ami-0147919 D2 ff 9 a6 ad 5*(亚马逊 Linux)。

设置这个 AMI ID，选择 VPC 和两个子网:

![](img/2bc0a6ef1fddf75c13c009e176f62f59.png)

点击*下一页*，跳过下一页，点击*创建堆栈*:

![](img/91346d8e3d8c71467da2b7e8aa22dbe1.png)

堆栈创建完成后—检查自动缩放组:

![](img/f498ce0970b499308d8d267433cea629.png)

## `kubectl`安装

当我们使用工作节点时——我们的 EKS 集群被供应，我们可以在一台工作机器上安装`kubectl`。

下载可执行文件:

```
[setevoy@setevoy-arch-work ~] $ curl -o kubectl [https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/kubectl](https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/kubectl)
[setevoy@setevoy-arch-work ~] $ chmod +x kubectl
[setevoy@setevoy-arch-work ~] $ sudo mv kubectl /usr/local/bin/
```

检查:

```
[setevoy@setevoy-arch-work ~] $ kubectl version — short — client
Client Version: v1.13.7-eks-fa4c70
```

要创建其配置文件，请使用 AWS CLI:

```
[setevoy@setevoy-arch-work ~] $ aws eks — region eu-west-2 — profile arseniy update-kubeconfig — name eks-cluster-manual
Added new context arn:aws:eks:eu-west-2:534***385:cluster/eks-cluster-manual to /home/setevoy/.kube/config
```

添加别名只是为了简化工作:

```
[setevoy@setevoy-arch-work ~] $ echo “alias kk=\”kubectl\”” >> ~/.bashrc
[setevoy@setevoy-arch-work ~] $ bash
```

检查一下:

```
[setevoy@setevoy-arch-work ~] $ kk get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 172.20.0.1 <none> 443/TCP 20m
```

## AWS 验证器

尽管工作者节点的云形成已经准备好，并且 EC2 实例已经启动并运行——但是我们仍然不能将它们视为 Kubernetes 集群中的节点:

```
[setevoy@setevoy-arch-work ~] $ kk get node
```

找不到资源。

![](img/4e5730000c109e05210f05cb38d66f92.png)

下载 AWS 验证器:

```
[setevoy@setevoy-arch-work ~] $ cd Temp/
[setevoy@setevoy-arch-work ~] $ curl -so aws-auth-cm.yaml [https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/aws-auth-cm.yaml](https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/aws-auth-cm.yaml)
```

转到в *IAM >角色*，找到角色( *NodeInstanceRole* )的 ARN(亚马逊资源名):

![](img/796a13e9ee0cdec98a5a14a0df8266a3.png)![](img/d1f61591ee07777d3cee4321d286a427.png)

编辑文件`aws-auth-cm.yaml`，设置`rolearn`:

![](img/88133f94f3bf03708dd0e8c4a0a3977e.png)

创造一个`ConfigMap`:

```
[setevoy@setevoy-arch-work ~/Temp] $ kk apply -f aws-auth-cm.yaml
configmap/aws-auth created
```

检查:

```
[setevoy@setevoy-arch-work ~/Temp] $ kubectl get nodes -o wide
NAME STATUS ROLES AGE VERSION INTERNAL-IP EXTERNAL-IP OS-IMAGE KERNEL-VERSION CONTAINER-RUNTIME
ip-10–0–153–7.eu-west-2.compute.internal Ready <none> 47s v1.13.7-eks-c57ff8 10.0.153.7 <none> Amazon Linux 2 4.14.128–112.105.amzn2.x86_64 docker://18.6.1
ip-10–0–196–123.eu-west-2.compute.internal Ready <none> 50s v1.13.7-eks-c57ff8 10.0.196.123 <none> Amazon Linux 2 4.14.128–112.105.amzn2.x86_64 docker://18.6.1
ip-10–0–204–190.eu-west-2.compute.internal Ready <none> 52s v1.13.7-eks-c57ff8 10.0.204.190 <none> Amazon Linux 2 4.14.128–112.105.amzn2.x86_64 docker://18.6.1
```

节点被添加到集群中—太好了。

## 网络应用和负载平衡器

出于测试目的，让我们创建一个简单的 web 服务，例如，一个普通的 NGINX，就像在[上一章](https://rtfm.co.ua/en/kubernetes-part-2-a-cluster-set-up-on-aws-with-aws-cloud-provider-and-aws-loadbalancer/)中一样。

要访问 NGINX——让我们也在 Kubernetes 和 AWS 中创建一个负载平衡器，它将请求代理到工作节点:

```
kind: Service
apiVersion: v1
metadata:
  name: eks-cluster-manual-elb
spec:
  type: LoadBalancer
  selector:
    app: eks-cluster-manual-pod
  ports:
    - name: http
      protocol: TCP
      # ELB's port
      port: 80
      # container's port
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-cluster-manual-deploy
spec:
  # ReplicaSet pods config
  replicas: 1
  # pods selector
  selector:
    matchLabels:
      app: eks-cluster-manual-pod
  # Pod template
  template:
    metadata:
      # a pod's labeles
      labels:
        app: eks-cluster-manual-pod
    spec:
      containers:
        - name: eks-cluster-manual-app
          image: nginx
```

部署它们:

```
[setevoy@setevoy-arch-work ~/Temp] $ kk apply -f eks-cluster-manual-elb-nginx.yml
service/eks-cluster-manual-elb created
deployment.apps/eks-cluster-manual-deploy created
```

检查服务:

```
[setevoy@setevoy-arch-work ~/Temp] $ kk get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
eks-cluster-manual-elb LoadBalancer 172.20.17.42 a05***405.eu-west-2.elb.amazonaws.com 80:32680/TCP 5m23s
```

一个豆荚:

```
[setevoy@setevoy-arch-work ~/Temp] kk get po -o wide -l app=eks-cluster-manual-pod
NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
eks-cluster-manual-deploy-698b8f6df7-jg55x 1/1 Running 0 6m17s 10.0.130.54 ip-10–0–153–7.eu-west-2.compute.internal <none> <none>
```

而`ConfigMap`本身:

```
[setevoy@setevoy-arch-work ~/Temp]kubectl describe configmap -n kube-system aws-auth
Name: aws-auth
Namespace: kube-system
Labels: <none>
Annotations: kubectl.kubernetes.io/last-applied-configuration:
{“apiVersion”:”v1",”data”:{“mapRoles”:”- rolearn: arn:aws:iam::534***385:role/eks-cluster-manual-workers-stack-NodeInstanceRole-12DRN98…
Data
====
mapRoles:
 — — 
- rolearn: arn:aws:iam::534***385:role/eks-cluster-manual-workers-stack-NodeInstanceRole-12DRN987QYB34
username: system:node:{{EC2PrivateDNSName}}
groups:
- system:bootstrappers
- system:nodes
Events: <none>
```

AWS 中的负载平衡器(需要等待大约 5 分钟来启动 pod 并将节点连接到 ELB):

![](img/d87bee5eb39ad65f6a8f3df2b9b4ed99.png)

它的标签:

![](img/59517a9684ac42325ec509419fae5ee8.png)

并测试 AWS 或`kubectl get svc`命令提供的 URL:

```
[setevoy@setevoy-arch-work ~/Temp] $ curl a05***405.eu-west-2.elb.amazonaws.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
…
```

完成了。

## 有用的链接

*   [什么是亚马逊 EKS？](https://docs.aws.amazon.com/en_us/eks/latest/userguide/what-is-eks.html)
*   [亚马逊 EKS 入门](https://docs.aws.amazon.com/en_us/eks/latest/userguide/getting-started-console.html)
*   [管理集群的用户或 IAM 角色](https://docs.aws.amazon.com/en_us/eks/latest/userguide/add-user-role.html)
*   [Kubernetes 配置图](https://unofficial-kubernetes.readthedocs.io/en/latest/tasks/configure-pod-container/configmap/)
*   [亚马逊 EKS 工作室](https://eksworkshop.com/introduction/)
*   [关于亚马逊(EKS)上的 Kubernetes 要知道的十件事](https://cloudgeometry.io/blog/amazon-eks/)
*   [EKS 评论](https://medium.com/@andrewhibbert/eks-review-b4ee523083dc)
*   我的第一个 Kubernetes 集群:亚马逊 EKS 评论
*   【Kubernetes 部署故障排除
*   为初学者详细解释的匹配标签、标记和选择器

*最初发布于* [*RTFM: Linux、DevOps 和系统管理*](https://rtfm.co.ua/en/kubernetes-part-3-aws-eks-overview-and-manual-eks-cluster-set-up/) *。*
# Kubernetes on Devstack 第 3 部分:在集群上运行应用程序

> 原文：<https://itnext.io/kubernetes-on-devstack-part-3-running-applications-on-the-cluster-f15cf4762822?source=collection_archive---------1----------------------->

这是涵盖 OpenStack 云上的 Kubernetes 集群的[系列](https://berndbausch.medium.com/running-a-kubernetes-cluster-on-devstack-533d579bb2f9)的第 3 部分。

到目前为止发生了什么:您已经建立了一个 Devstack 云，并在两个云实例上部署了一个 Kubernetes 集群。您向云中添加了 OpenStack 云提供商和 Cinder CSI 插件。

是时候使用集群了。您将创建使用以下内容的应用程序:

1.  煤渣体积
2.  多附加 Cinder 卷
3.  负载平衡器

# 一个使用煤渣卷的简单应用程序

您首先需要定义一个映射到 Cinder 的存储类。这是通过在存储类定义中将 Cinder CSI 驱动程序引用为*提供者*来实现的。

## 获取煤渣 CSI 驱动程序的名称

煤渣 CSI 司机叫什么名字？它在*主机 1* 上的`csi-cinder-driver.yaml`清单中定义:

```
$ cd manifests
$ cat csi-cinder-driver.yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: cinder.csi.openstack.org
spec:
  attachRequired: true
  podInfoOnMount: true
  volumeLifecycleModes:
  - Persistent
  - Ephemeral
```

或者，列出当前安装的 CSI 驱动程序:

```
$ kubectl get csidrivers.storage.k8s.io
NAME                       ATTACHREQUIRED   PODINFOONMOUNT   MODES                  AGE
cinder.csi.openstack.org   true             true             Persistent,Ephemeral   2d20h
```

司机名叫*cinder.csi.openstack.org*。

## 定义存储

三个清单定义了测试应用程序:一个 storageclass 定义、一个 PVC 和一个带有 NGINX pods 的复制控制器。将三份清单从 Github **复制到 master1** 。

存储类通过其*置备程序*键链接到 CSI 驱动程序:

```
$ cat cinder-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-sc-cinderplugin
provisioner: cinder.csi.openstack.org
```

首先，应用两个存储清单。

```
$ kubectl apply -f cinder-storageclass.yaml
$ kubectl apply -f cinder-pvc-claim1.yaml
```

## 检查成功

如果一切配置正确，PVC 应该有一个煤渣卷作为后盾。**在 Devstack 服务器**上，检查卷。

```
$ source ~/devstack/openrc kube kube
$ openstack volume list -f yaml
- Attached to: []
  ID: 1edb1b91-6fdc-4f20-8ead-283aaf878390
  Name: pvc-a0c8e55f-f766-44bd-80a5-ff2832e4365f
  Size: 1
  Status: available
```

PVC 对应于当前*可用*的 Cinder 卷，即未连接到任何服务器。

## 启动应用程序

**在主节点**上，启动应用程序。

```
$ kubectl apply -f example-pod.yaml
$ kubectl get rc,pod,pvc
NAME                           DESIRED   CURRENT   READY   AGE
replicationcontroller/server   4         4         4       106sNAME               READY   STATUS    RESTARTS   AGE
pod/server-bnwd7   1/1     Running   0          105s
pod/server-jklpp   1/1     Running   0          105s
pod/server-kj26p   1/1     Running   0          106s
pod/server-r4mqd   1/1     Running   0          105sNAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
persistentvolumeclaim/claim1   Bound    pvc-a0c8e55f-f766-44bd-80a5-ff2832e4365f   1Gi        RWO            csi-sc-cinderplugin   7m48s
```

## 检查启动成功

**在 Devstack 服务器**上，再次列出卷。您会发现卷状态已经从*可用*变为*使用中*，并且它被附加到 *worker1* 实例。

**在 worker1 节点**上，验证该卷是否已连接和挂载。

```
[centos@worker1 ~]$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    252:0    0  20G  0 disk
`-vda1 252:1    0  20G  0 part /
vdb    252:16   0   1G  0 disk /var/lib/kubelet/pods/de103e52-227b-4117-b1b6-647c68507127/volumes/kubernetes.io~cs
[centos@worker1 ~]$ sudo blkid
/dev/vda1: UUID="6cd50e51-cfc6-40b9-9ec5-f32fa2e4ff02" TYPE="xfs"
/dev/vdb: UUID="028085bf-2dd1-47c0-ab2a-e925a4150c36" TYPE="ext4"
```

这个卷被称为 *vdb* ，并被挂载到一个 kubelet 目录中。

# 多附加卷

默认情况下，Cinder 卷只能连接到一个 OpenStack 实例。现在，实现 PVC 的卷只连接到 *worker1* 。

本节将探讨共享一个卷的多个节点上的**pod，以及为此需要做哪些更改。我们将:**

1.  允许在控制器上安排 pod。
2.  尝试在两个节点上部署 pod，但是失败。
3.  添加煤渣体积类型。
4.  更改存储类清单。
5.  使 PVC *读写多个*。
6.  尝试在两个节点上部署 pod，并获得成功。

## 1.从 master1 中删除 NoSchedule 污点

我们想在 *master1* 节点上运行 pods，但是现在它的 *NoSchedule* 污点使得这不可能:

```
$ kubectl describe no master1 
...
Taints:             node-role.kubernetes.io/master:NoSchedule
...
```

去掉这个污点，这样就可以在那里安排吊舱了。

```
$ kubectl taint nodes master1 \
                          node-role.kubernetes.io/master:NoSchedule-
```

**命令末尾的破折号是重要的**。再次使用`kubectl describe`再次检查污点是否已被清除。

## 2.将 pod 添加到控制器并检查它们的运行状况

在`example-pod.yaml`中，将副本号从 4 设置为 8，然后应用更新的清单。列出豆荚。

```
$ kubectl get pods -o wide
NAME           READY   STATUS              RESTARTS   AGE     IP            NODE      NOMINATED NODE   READINESS GATES
server-8swjk   0/1     ContainerCreating   0          7s      <none>        master1   <none>           <none>
server-bnwd7   1/1     Running             0          3h21m   10.244.1.30   worker1   <none>           <none>
server-j8mnp   0/1     ContainerCreating   0          7s      <none>        master1   <none>           <none>
```

过了一分钟左右，再检查一遍。您将会发现新 pod 的状态仍然是 *ContainerCreating* 。当您描述其中一个 pod 时，您会发现它们的创建可以完成，因为该卷无法连接，因此无法挂载:

```
$ kubectl describe pod server-vl8cl
...
Events:
  Type     Reason              Age                  From                     Message
  ----     ------              ----                 ----                     -------
  Normal   Scheduled           15m                  default-scheduler        Successfully assigned default/server-vl8cl to master1
  Warning  FailedAttachVolume  15m                  attachdetach-controller  Multi-Attach error for volume "pvc-a0c8e55f-f766-44bd-80a5-ff2832e4365f" Volume is already used by pod(s) server-bnwd7, server-jklpp, server-kj26p, server-r4mqd
  Warning  FailedMount         2m16s (x6 over 13m)  kubelet                  Unable to attach or mount volumes: unmounted volumes=[cinderpvc], unattached volumes=[cinderpvc default-token-9tnjm]: timed out waiting for the condition
```

这是因为 *master1* 无法附加已经附加到 *worker1* 的卷。要将一个 Cinder volume 附加到多个实例，必须设置其 *multiattach* 标志。

您现在将设置一个允许多连接的存储类，但首先要删除副本控制器、PVC 和存储类。

```
$ kubectl delete rc server
$ kubectl delete pvc claim1
$ kubectl delete sc csi-sc-cinderplugin
```

## 3.创建允许多附件的 Cinder 卷类型

要将一个 Cinder 卷附加到多个实例，它必须有一个*多附加卷类型*。目前不存在这样的卷类型，因此您必须先创建一个。更多信息请参见 [Cinder 管理指南](https://docs.openstack.org/cinder/latest/admin/blockstorage-volume-multiattach.html)。

**在 Devstack 服务器上**:

```
$ source ~/devstack/openrc admin admin
$ openstack volume type create multiattach-type \
                                 --property multiattach="<is> True"
```

请确保 multiattach 属性具有正确的值。 *True* 中的 T 必须是大写字母。

## 4.将新的卷类型添加到 Kubernetes 存储类

**在 master1 节点**上，通过添加新的卷类型作为参数来修改**存储类**清单:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-sc-cinderplugin
provisioner: cinder.csi.openstack.org
parameters:
  type: multiattach-type
```

## 5.更改 PVC 的访问模式并重新创建 PVC

还要修改 **PVC 清单**并将其 accessMode 设置为 *ReadWriteMany* 。

创建存储类和 PVC。

```
$ kubectl apply -f cinder-storageclass.yaml
$ kubectl apply -f cinder-pvc-claim1.yaml
```

**在 Devstack 服务器**上，验证是否已创建 Cinder 卷，以及它是否具有 multiattach 标志。

```
$ openstack volume list --all-projects
+--------------------------------------+------------------------------------------+-----------+------+-------------+
| ID                                   | Name                                     | Status    | Size | Attached to |
+--------------------------------------+------------------------------------------+-----------+------+-------------+
| 73c3e003-c535-42e1-8c4a-a1f9379ffba6 | pvc-6d770b8d-fc35-4f8d-800d-0ec8925f58d6 | available |    1 |             |
+--------------------------------------+------------------------------------------+-----------+------+-------------+
$ openstack volume show 73c3e003-c535-42e1-8c4a-a1f9379ffba6 | grep multi
| multiattach                  | True                                          |
| type                         | multiattach-type                              |
```

## 6.运行吊舱并检查它们的健康状况

```
$ kubectl apply -f example-pod.yaml
replicationcontroller/server created
$ kubectl get pod -o wide
NAME           READY   STATUS              RESTARTS   AGE   IP       NODE      NOMINATED NODE   READINESS GATES
server-5m8bg   0/1     ContainerCreating   0          13s   <none>   master1   <none>           <none>
server-9djx2   0/1     ContainerCreating   0          13s   <none>   master1   <none>           <none>
...
```

过一会儿重复 *get* 命令时，所有的 pod 都应该处于*运行*的状态。

**在 Devstack 服务器**上，列出卷。

```
$ openstack volume list --fit-width
+----------------------+----------------------+--------+------+-----------------------+
| ID                   | Name                 | Status | Size | Attached to           |
+----------------------+----------------------+--------+------+-----------------------+
| 66ea38fd-4d08-4afa-  | pvc-37ab30fb-c27c-4e | in-use |    1 | Attached to worker1   |
| ae3b-fa477cf4b26a    | 86-a067-81b0e0353fe1 |        |      | on /dev/vdb Attached  |
|                      |                      |        |      | to master1 on         |
|                      |                      |        |      | /dev/vdb              |
+----------------------+----------------------+--------+------+-----------------------+
```

*附加到*列显示煤渣卷附加到两个节点。

# 一个使用负载平衡的简单应用程序

为了演示负载平衡，pod 需要在两个或更多节点上运行。有关在控制器上启用 pod 调度，请参见上一节中的说明。

## 新清单

创建一个**新的复制控制器清单**，它使用容器本地存储而不是卷:

```
$ cat example-pod-lb.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: lbserver
spec:
  replicas: 8
  selector:
	role: server
  template:
	metadata:
	  labels:
		role: server
		app: lbserver
	spec:
	  containers:
	  - name: server
		image: nginx
```

该清单基于官方云提供商 OpenStack repo 的[示例](https://github.com/kubernetes/cloud-provider-openstack/blob/master/examples/persistent-volume-provisioning/cinder/example-pod.yaml)。和原版有两点不同:pod 多了一个标签`app: lbserver`，没有体积。

创建名为 *service.yaml* 的**服务清单**。确保其选择器通过上面的标签指向复制控制器:

```
kind: Service
apiVersion: v1
metadata:
  name: lb-webserver
spec:
  selector:
	app: lbserver
  type: LoadBalancer
  ports:
  - name: http
	port: 80
	targetPort: 80
```

应用这两个清单。

## OpenStack 资源

OpenStack 云提供商将创建一个负载平衡器，这可能需要几分钟时间。大部分时间将用于启动一个新的、所谓的“amphora”实例，该实例运行负载平衡代码。

**在 Devstack 服务器**上，探索 OpenStack 负载平衡器。以下命令使用 YAML 作为输出格式，以获得更漂亮的输出。

```
$ openstack loadbalancer list -f yaml
- id: 474f2c35-1d45-4b49-99ba-d745082e9d33
  name: kube_service_kubernetes_default_lb-webserver
  operating_status: OFFLINE
  project_id: 7a099eff1fb1479b89fa721da3e1a018
  provider: amphora
  provisioning_status: PENDING_CREATE
  vip_address: 172.16.0.134
```

预配状态当前为 PENDING_CREATE，但将在负载平衡器实例启动并运行时变为活动状态。将您的身份更改为 admin 后，您可以查看负载平衡器实例:

```
$ source ~/devstack/openrc admin admin
$ openstack server list
```

查看负载平衡器详细信息的一个有用命令是`openstack loadbalancer status show <LOADBALANCER-ID>`。

云提供商将浮动 IP 与负载平衡器的 VIP 地址相关联。可以用`openstack floating ip list | grep 172.16.0.134`查看，其中 172.16.0.134 是 VIP 地址。

## 负载平衡器测试

**在 master1** 上，浮动 IP 被用作服务的外部 IP:

```
$ kubectl get service
NAME           TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
kubernetes     ClusterIP      10.96.0.1     <none>          443/TCP        4d2h
lb-webserver   LoadBalancer   10.97.192.7   192.168.1.225   80:30007/TCP   34m
```

您刚刚启动的 pod 包含一个 NGINX web 服务器，带有一个默认的*index.htm*l 页面。您可以使用`curl <EXTERNAL-IP>`访问 web 服务器，但这并不能向您证明使用了多个 pod。

为每个吊舱创建不同的*index.html*。例如:

```
server=1
$ for pod in $LIST_OF_PODS
do 
	kubectl exec $i -- /bin/bash -c "echo Server $server > /usr/share/nginx/html/index.html"
	((server++))
done
```

LIST_OF_PODS 是您刚刚启动的 lbserver pods 的列表。这个脚本用一个定制的字符串替换了每个 pod 中的*index.html*。

通过重复运行上面的 curl 命令来测试这一点。您应该看到不同的 pod 如何响应，证明发生了负载平衡。

```
$ curl 192.168.1.225
Server 7
$ curl 192.168.1.225
Server 5
$ curl 192.168.1.225
Server 4
$ curl 192.168.1.225
Server 4
$ curl 192.168.1.225
Server 2
```

接下来:[使用 OpenStack Magnum 建立 Kubernetes 集群](https://berndbausch.medium.com/using-openstack-magnum-to-create-a-kubernetes-cluster-af8ec3cfbda5)。
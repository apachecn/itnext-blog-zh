# Kubernetes: PersistentVolume 和 PersistentVolumeClaim —示例概述

> 原文：<https://itnext.io/kubernetes-persistentvolume-and-persistentvolumeclaim-an-overview-with-examples-3c5688222f99?source=collection_archive---------4----------------------->

![](img/194f7c00055c0d75e23c5372361844e3.png)

对于持久数据，Kubernetes 提供了两种主要类型的对象——persistent volume 和 PersistentVolumeClaim。

*PersistentVolume* —是一个存储设备和其上的文件系统卷，例如，它可以是连接到 AWS EC2 的 AWS EBS，从群集的角度来看，PersistentVolume 是一种类似的资源，比如说 Kubernetes 工作节点。

*PersistentVolumeClaim* 依次请求使用这样的 PersistentVolume 资源，类似于 Kubernetes pod——当 Pod 请求工作节点的资源时，PersistentVolumeClaim 将从 PersistentVolume 请求资源:当 Pod 请求 CPU、工作节点的内存时——PersistentVolumeClaim 将请求必要的存储大小和访问类型— `ReadWriteOnce`、`ReadOnlyMany`或`ReadWriteMany`，请参见[访问模式](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)。

可以通过两种方式创建持久卷—静态和动态(推荐)。

当静态创建 PV 时，您必须首先创建一个存储设备，例如 AWS EBS，它将由 PersistentVolume 使用。

如果群集无法为 PersistentVolumeClaim н找到合适的 PV，它可以完全为此 PVC 创建新的存储设备，这将是动态 PV 创建方式。

为了实现这一点，PVC 必须有一个[存储类](https://kubernetes.io/docs/concepts/storage/storage-classes/)设置相同，并且这个类必须得到集群的支持。

例如，对于 AWS EKS，我们有*GP2*T3:

```
$ kubectl get storageclass
NAME PROVISIONER AGE
gp2 (default) kubernetes.io/aws-ebs 64d
```

# 存储类型

为了更好地理解持久卷的概念，我们来看一下所有可用的存储:

*   节点本地存储(`emptyDir`和`hostPath`)
*   云卷(例如，`awsElasticBlockStore`、`gcePersistentDisk`和`azureDiskVolume`)
*   文件共享卷，如网络文件系统
*   分布式文件系统(例如，CephFS、RBD 和 GlusterFS)
*   特殊类型，如`PersistentVolumeClaim`、`secret`和`gitRepo`

`emptyDir`和`hostPath`直接连接到 pod，并且仅当这样的 pod 处于活动状态时才能存储数据，而云卷、NFS 和 PersistentVolume 独立于 pod，并且将存储数据，直到这样的卷被删除。

# 创建 PersistentVolumeClaim

## 静态持续卷配置

## 创建一个 EBS

首先，对于静态资源调配，我们需要创建一个存储设备，在本例中，它将是 AWS EBS，然后我们将创建一个使用此 EBS 的持久卷。

创建一个 EBS:

```
$ aws ec2 --profile arseniy --region us-east-2 create-volume --availability-zone us-east-2a --size 50
{
“AvailabilityZone”: “us-east-2a”,
“CreateTime”: “2020–07–29T13:10:12.000Z”,
“Encrypted”: false,
“Size”: 50,
“SnapshotId”: “”,
“State”: “creating”,
“VolumeId”: “vol-0928650905a2491e2”,
“Iops”: 150,
“Tags”: [],
“VolumeType”: “gp2”
}
```

存储其 id—*“vol-0928650905 a 2491 e 2”*。

## 创建持久卷

写一个清单文件，姑且称之为`pv-static.yaml`:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-static
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2
  awsElasticBlockStore:
    fsType: ext4
    volumeID: vol-0928650905a2491e2
```

这里:

*   `capacity`:存储大小
*   `accessModes`:接入类型，这里是`ReadWriteOnce`，表示该 PV 同一时间只能挂一个 WorkerNode
*   `storageClassName`:存储访问，见下文
*   `awsElasticBlockStore`:使用的设备类型
*   `fsType`:要在该卷上创建的文件系统类型
*   `volumeID`:AWS EBS 光盘 ID

创建持久卷:

```
$ kubectl apply -f pv-static.yaml
persistentvolume/pv-static created
```

检查一下:

```
$ kubectl get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
pv-static 5Gi RWO Retain Available 69s
```

## `StorageClass`

`storageClassName`参数将设置存储类型。

PVC 和 PV 必须具有相同的类别，否则，PVC 将找不到 PV，并且这种 PVC 的状态将为*待定*。

如果 PVC 没有设置`StorageClass`，那么将使用默认值:

```
$ kubectl get storageclass -o wide
NAME PROVISIONER AGE
gp2 (default) kubernetes.io/aws-ebs 65d
```

在此期间，如果没有为 PV 设置`StorageClass`——此 PV 将被创建为没有类，并且我们具有默认类的 PVC 将无法使用此 PV，并且“ ***无法绑定到请求的卷“PV name”:storage class name 与*** 不匹配”错误:

```
…
Events:
Type Reason Age From Message
 — — — — — — — — — — — — -
Warning VolumeMismatch 12s (x17 over 4m2s) persistentvolume-controller Cannot bind to requested volume “pvname”: storageClassName does not match
…
```

参见文档[此处> > >](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1) 和[此处> > >](https://kubernetes.io/docs/concepts/storage/storage-classes/) 。

## 创建 PersistentVolumeClaim

现在，我们可以创建一个 PersistentVolumeClaim，它将使用我们在上面创建的 PersistentVolume 到`pvc-static.yaml`文件:

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-static
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeName: pv-static
```

创建此 PVC:

```
$ kubectl apply -f pvc-static.yaml
persistentvolumeclaim/pvc-static created
```

检查一下:

```
$ kubectl get pvc pvc-static
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-static Bound pv-static 5Gi RWO gp2 31s
```

## 动态持续卷配置

创建 PersistentVolume 的动态方法类似于静态方法，唯一的区别是您不需要手动创建 AWS EBS 和 PersistentVolume 资源，相反，您只需创建一个 PersistentVolumeClaim 对象，Kubernetes 将通过 AWS API 创建一个 EBS，并挂载到在 Kubernetes 集群中扮演 WorkerNode 角色的 AWS EC2:

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

创建此 PVC:

```
$ kubectl apply -f pvc-dynamic.yaml
persistentvolumeclaim/pvc-dynamic created
```

检查一下:

```
$ kubectl get pvc pvc-dynamic
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-dynamic Pending gp2 45s
```

好吧，但为什么它处于待定状态？检查其事件:

```
$ kubectl describe pvc pvc-dynamic
…
Events:
Type Reason Age From Message
 — — — — — — — — — — — — -
Normal WaitForFirstConsumer 1s (x4 over 33s) persistentvolume-controller waiting for first consumer to be created before binding
Mounted By: <none>
```

## `WaitForFirstConsumer`

让我们看看默认的`StorageClass`设置:

```
$ kubectl describe sc gp2
Name: gp2
IsDefaultClass: Yes
…
Provisioner: kubernetes.io/aws-ebs
Parameters: fsType=ext4,type=gp2
…
VolumeBindingMode: WaitForFirstConsumer
Events: <none>
```

在这里，`[VolumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode)`定义了如何创建一个持久卷。使用 *Immediate* 值，当请求者 VPC 出现时，将立即创建这样一个 PV，但是使用`*WaitForFirstConsumer*`值，如在本例中，Kubernetes 将等待第一个消费者，如一个 pod，它将请求此 PV，然后根据该 pod 正在运行的 WorkerNode 的 AvailbiltyZone，Kubernetes 将创建一个新的 PV 和一个 AWS EBS 磁盘。

现在，让我们创建 pod 来消耗这些卷。

# 在 pod 中使用 PersistentVolumeClaim

## 动态持续量声明

让我们描述一个将使用我们的动态 PVC 的 pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: pv-dynamic-pod
spec:
  volumes:
    - name: pv-dynamic-storage
      persistentVolumeClaim:
        claimName: pvc-dynamic
  containers:
    - name: pv-dynamic-container
      image: nginx
      ports:
        - containerPort: 80
          name: "nginx"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-dynamic-storage
```

这里:

*   `volumes`:
*   `persistentVolumeClaim`:
*   `claimName`:创建 pod 时需要的 PVC 名称
*   `containers`:
*   `volumeMounts`:将 *pv-dynamic-storage* 卷挂载到 pod 中的`/usr/share/nginx/html`目录

创建它:

```
$ kubectl apply -f pv-pods.yaml
pod/pv-dynamic-pod created
```

再次检查我们的 PVC:

```
$ kubectl get pvc pvc-dynamic
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-dynamic Bound pvc-6d024b40-a239–4c35–8694-f060bd117053 5Gi RWO gp2 21h
```

现在我们可以看到一个 ID 为*PVC-6d 024 b 40-a239–4c 35–8694-f 060 BD 117053*的新卷—检查一下:

```
$ kubectl describe pvc pvc-dynamic
Name: pvc-dynamic
Namespace: default
StorageClass: gp2
Status: Bound
Volume: pvc-6d024b40-a239–4c35–8694-f060bd117053
…
Finalizers: [kubernetes.io/pvc-protection]
Capacity: 5Gi
Access Modes: RWO
VolumeMode: Filesystem
Events: <none>
Mounted By: pv-dynamic-pod
```

检查音量:

```
$ kubectl describe pv pvc-6d024b40-a239–4c35–8694-f060bd117053
Name: pvc-6d024b40-a239–4c35–8694-f060bd117053
…
StorageClass: gp2
Status: Bound
Claim: default/pvc-dynamic
Reclaim Policy: Delete
Access Modes: RWO
VolumeMode: Filesystem
Capacity: 5Gi
Node Affinity:
Required Terms:
Term 0: failure-domain.beta.kubernetes.io/zone in [us-east-2b]
failure-domain.beta.kubernetes.io/region in [us-east-2]
Message:
Source:
Type: AWSElasticBlockStore (a Persistent Disk resource in AWS)
VolumeID: aws://us-east-2b/vol-040a5e004876f1a40
FSType: ext4
Partition: 0
ReadOnly: false
Events: <none>
```

以及 AWS EBS*vol-040 a5e 004876 f1 a40*:

```
$ aws ec2 — profile arseniy — region us-east-2 describe-volumes — volume-ids vol-040a5e004876f1a40 — output json
{
“Volumes”: [
{
“Attachments”: [
{
“AttachTime”: “2020–07–30T11:08:29.000Z”,
“Device”: “/dev/xvdcy”,
“InstanceId”: “i-0a3225e9fe7cb7629”,
“State”: “attached”,
“VolumeId”: “vol-040a5e004876f1a40”,
“DeleteOnTermination”: false
}
],
…
```

检查吊舱内部:

```
$ kk exec -ti pv-dynamic-pod bash
root@pv-dynamic-pod:/# lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme0n1 259:0 0 50G 0 disk
|-nvme0n1p1 259:1 0 50G 0 part /etc/hosts
`-nvme0n1p128 259:2 0 1M 0 part
nvme1n1 259:3 0 5G 0 disk /usr/share/nginx/html
```

*nvme 1n 1*——这是我们的分区。

让我们写一些数据:

```
root@pv-dynamic-pod:/# echo Test > /usr/share/nginx/html/index.html
```

放下吊舱:

```
$ kk delete pod pv-dynamic-pod
pod “pv-dynamic-pod” deleted
```

重新创建它:

```
$ kubectl apply -f pv-pods.yaml
pod/pv-dynamic-pod created
```

检查数据:

```
$ kk exec -ti pv-dynamic-pod cat /usr/share/nginx/html/index.html
Test
```

一切都还在原处。

## 静态 PersistentVolumeClaim

现在，让我们尝试使用我们静态创建的 PV。

我们可以使用同一个清单——`pv-static.yaml`:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-static
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2
  awsElasticBlockStore:
    fsType: ext4
    volumeID: vol-0928650905a2491e2
```

让我们使用 PVC 的`pvc-static.yaml`清单:

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-static
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeName: pv-static
```

创建 PV:

```
$ kk apply -f pv-static.yaml
persistentvolume/pv-static created
```

检查一下:

```
$ kk get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
pv-static 5Gi RWO Retain Available gp2 58s
…
```

创建 PVC:

```
$ kk apply -f pvc-static.yaml
persistentvolumeclaim/pvc-static created
```

检查一下:

```
$ kk get pvc pvc-static
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-static Bound pv-static 5Gi RWO gp2 9s
```

*状态绑定*表示 PVC 能够找到其 PV 并成功连接。

## Pod `nodeAffinity`

接下来，我们需要确定一个 AWS 可用性区域，在该区域中我们为静态 PV 创建了 AWS EBS:

```
$ aws ec2 — profile arseniy — region us-east-2 describe-volumes — volume-ids vol-0928650905a2491e2 — query '[Volumes[*].AvailabilityZone]' — output text
us-east-2a
```

*us-east-2a* —好的，那么我们需要在同一个 AvailabilityZone 中的 Kubernetes Worker 节点上创建一个 pod。

创建清单:

```
apiVersion: v1
kind: Pod
metadata:
  name: pv-static-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: failure-domain.beta.kubernetes.io/zone
            operator: In
            values:
            - us-east-2a
  volumes:
    - name: pv-static-storage
      persistentVolumeClaim:
        claimName: pvc-static
  containers:
    - name: pv-static-container
      image: nginx
      ports:
        - containerPort: 80
          name: "nginx"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-static-storage
```

与动态 PVC 相反，这里我们使用了`nodeAffinity`来指定我们想要使用来自 *s-east-2a* AZ 的节点。

创建 pod:

```
$ kk apply -f pv-pod-stat.yaml
pod/pv-static-pod created
```

检查事件:

```
0s Normal Scheduled Pod Successfully assigned default/pv-static-pod to ip-10–3–47–58.us-east-2.compute.internal
0s Normal SuccessfulAttachVolume Pod AttachVolume.Attach succeeded for volume “pv-static”
0s Normal Pulling Pod Pulling image “nginx”
0s Normal Pulled Pod Successfully pulled image “nginx”
0s Normal Created Pod Created container pv-static-container
0s Normal Started Pod Started container pv-static-container
```

pod 中的分区:

```
$ kk exec -ti pv-static-pod bash
root@pv-static-pod:/# lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme0n1 259:0 0 50G 0 disk
|-nvme0n1p1 259:1 0 50G 0 part /etc/hosts
`-nvme0n1p128 259:2 0 1M 0 part
nvme1n1 259:3 0 50G 0 disk /usr/share/nginx/html
```

*nvme1n1* 已安装，所有工作正常。

## 持久卷`nodeAffinity`

另一个选择是与 t 相对的 T2。

在创建将使用该 PV 的单元时，Kubernetes 首先会检查哪些工作节点可用于连接该卷，然后在这样的节点上创建单元。

在 pod 的清单中删除`nodeAffinity`:

```
apiVersion: v1
kind: Pod
metadata:
  name: pv-static-pod
spec:
  volumes:
    - name: pv-static-storage
      persistentVolumeClaim:
        claimName: pvc-static
  containers:
    - name: pv-static-container
      image: nginx
      ports:
        - containerPort: 80
          name: "nginx"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-static-storage
```

并添加到 PV 的清单中:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-static
spec:
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: failure-domain.beta.kubernetes.io/zone
          operator: In
          values:
          - us-east-2a    
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2
  awsElasticBlockStore:
    fsType: ext4
    volumeID: vol-0928650905a2491e2
```

创建此 PV:

```
$ kk apply -f pv-static.yaml
persistentvolume/pv-static created
```

创建它的 PVC —这里没有任何改变:

```
$ kk apply -f pvc-static.yaml
persistentvolumeclaim/pvc-static created
```

创建 pod:

```
$ kk apply -f pv-pod-stat.yaml
pod/pv-static-pod created
```

检查日志:

```
0s Normal Scheduled Pod Successfully assigned default/pv-static-pod to ip-10–3–47–58.us-east-2.compute.internal
0s Normal SuccessfulAttachVolume Pod AttachVolume.Attach succeeded for volume “pv-static”
0s Normal Pulling Pod Pulling image “nginx”
0s Normal Pulled Pod Successfully pulled image “nginx”
0s Normal Created Pod Created container pv-static-container
0s Normal Started Pod Started container pv-static-container
```

# 删除 PersistentVolume 和 PersistentVolumeClaim

当用户想要删除当前由活动 pod 使用的 PVC 时，这样的 PVC 不会被立即删除，它将一直存在，直到相应的 pod 正在运行。

类似地，当从持久卷声明中删除具有绑定的持久卷时，这样的 PV 将不会被删除，直到这样的绑定存在，例如直到其 PVC 存在。

## 改造

这里的文档是[>>>](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming)。

当我们想要完成 PersistentVolume 的工作时，我们可以从集群中删除它以释放相应的 AWS EBS ( *回收*)。

用于持久卷的*回收策略*向集群指定它与这种释放的卷有什么关系，并且可以具有`Retained`、`Recycled`或`Deleted`值。

## `Retain`

`Retain`策略允许我们手动清理磁盘。

删除相关的 PersistentVolumeClaim 后，PersistentVolume 将不会被删除，并将被标记为“ *released* ”，但它将可用于新的 PersistentVolumeClaim，因为它仍保留以前的 PersistentVolumeClaim 中的一些数据。

要使其可供下次使用，您需要从群集中删除 PersistentVolume 对象。

## `Delete`

使用`Delete`值，当您删除一个 PVC 时，它将删除其对应的 PersistentVolume 和卷的设备，如 AWS EBS、GCE PD 或 Azure Disk。

请记住，以动态方式创建的卷将从所使用的`StorageClass`继承策略，该策略默认设置为`Delete`。

## `Recycle`

已弃用，用于通过常用`rm -rf`删除数据。

## 删除 PV 和 PVC —一个示例

所以，我们有一个 pod 在运行:

```
$ kk get pod pv-static-pod
NAME READY STATUS RESTARTS AGE
pv-static-pod 1/1 Running 0 19s
```

使用聚氯乙烯:

```
$ kk get pvc pvc-static
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-static Bound pv-static 50Gi RWO gp2 19h
```

这个 PVC 绑定到 PV:

```
$ kk get pv pv-static
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
pv-static 50Gi RWO Retain Bound default/pvc-static gp2 19h
```

我们的 PV 将其`RECLAIM POLICY`设置为*保留*——因此，在我们丢弃其 PVC 和 PV 后，所有数据都必须保留。

让我们检查一下—添加一些数据:

```
$ kk exec -ti pv-static-pod bash
root@pv-static-pod:/# echo Test > /usr/share/nginx/html/test.txt
root@pv-static-pod:/# cat /usr/share/nginx/html/test.txt
Test
```

退出 pod 并删除它，然后删除它的 PVC:

```
$ kubectl delete pod pv-static-pod
pod “pv-static-pod” deleted
kubectl delete pvc pvc-static
persistentvolumeclaim “pvc-static” deleted
```

检查 PV 的状态:

```
$ kubectl get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
pv-static 50Gi RWO Retain Released default/pvc-static gp2 25s
```

`STATUS` == *发布*，目前我们无法通过新的 PVC 重新连接该卷。

让我们检查一下—再次创建一个 PVC:

```
$ kubectl apply -f pvc-static.yaml
persistentvolumeclaim/pvc-static created
```

创建一个 pod:

```
$ kubectl apply -f pv-pod-stat.yaml
pod/pv-static-pod created
```

并检查其 PVC 状态:

```
$ kubectl get pvc pvc-static
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-static Pending pv-static 0 gp2 59s
```

`STATUS`是*待定*。

删除 pod、PVC，此时还要删除永久卷:

```
$ kubectl delete -f pv-pod-stat.yaml
pod “pv-static-pod” deleted$ kubectl delete -f pvc-static.yaml
persistentvolumeclaim “pvc-static” deleted$ kubectl delete -f pv-static.yaml
persistentvolume “pv-static” deleted
```

从头再来:

```
$ kubectl apply -f pv-static.yaml
persistentvolume/pv-static created$ kubectl apply -f pvc-static.yaml
persistentvolumeclaim/pvc-static created$ kubectl apply -f pv-pod-stat.yaml
pod/pv-static-pod created
```

检查 PVC:

```
$ kubectl get pvc pvc-static
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-static Bound pv-static 50Gi RWO gp2 27s
```

并检查我们之前添加的数据:

```
$ kubectl exec -ti pv-static-pod cat /usr/share/nginx/html/test.txt
Test
```

一切正常—数据仍在原处。

## 更改永久卷的回收策略

这里的文档是[>>>](https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/)。

目前，我们的 PV 具有`Retain`值:

```
$ kubectl get pv pv-static -o jsonpath=’{.spec.persistentVolumeReclaimPolicy}’
Retain
```

应用补丁—将其`persistentVolumeReclaimPolicy`参数更新为`Delete`值:

```
$ kubectl patch pv pv-static -p ‘{“spec”:{“persistentVolumeReclaimPolicy”:”Delete”}}’
persistentvolume/pv-static patched
```

检查一下:

```
$ kubectl get pv pv-static -o jsonpath=’{.spec.persistentVolumeReclaimPolicy}’
Delete
```

删除 pod 及其 PVC:

```
$ kubectl delete -f pv-pod-stat.yaml
pod “pv-static-pod” deleted$ kubectl delete -f pvc-static.yaml
persistentvolumeclaim “pvc-static” deleted
```

检查持久卷:

```
$ kubectl get pv pv-static
Error from server (NotFound): persistentvolumes “pv-static” not found
```

以及用于本次 PV 的 AWS EBS:

```
$ aws ec2 --profile arseniy --region us-east-2 describe-volumes --volume-ids vol-0928650905a2491e2
```

出现错误(无效卷。NotFound)调用 DescribeVolumes 操作时:卷“vol-0928650905a2491e2”不存在。

其实就这些了。

# 有用的链接

*   [Kubernetes 中支持拓扑感知的卷供应](https://kubernetes.io/blog/2018/10/11/topology-aware-volume-provisioning-in-kubernetes/)
*   [使用预先存在的永久磁盘作为永久卷](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/preexisting-pd)
*   [具有永久磁盘的永久卷](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes)
*   [Kubernetes 持久存储:原因、地点和方式](https://cloud.netapp.com/blog/kubernetes-persistent-storage-why-where-and-how)
*   [Kubernetes 上使用持久卷和 Amazon EBS 的有状态容器](https://blog.couchbase.com/stateful-containers-kubernetes-amazon-ebs/)

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/kubernetes-persistentvolume-and-persistentvolumeclaim-an-overview-with-examples/) *。*
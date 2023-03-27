# OpenShift 的本地存储

> 原文：<https://itnext.io/local-storage-for-openshift-121224daf2d9?source=collection_archive---------4----------------------->

![](img/ad44c71d7aea419b2084137018f88cde.png)

有时出于性能考虑，您可能希望使用本地存储而不是网络存储。*当然，你会失去围绕节点自由移动 pod 的灵活性。pod 将绑定到提供本地存储的节点。*

本文探讨了如何在 OpenShift 4.x 环境中的不可变操作系统(RHCOS)上添加磁盘和创建文件系统。进一步创建用于容器的持久卷(PV)和存储类。

## **在 RHCOS 中创建文件系统**

在工作节点上，让我们添加一个额外的磁盘。我使用 KVM，所以第二个磁盘将显示为/dev/vdb。如下所示创建 LVM 文件系统，

```
ssh core@wokernodesudo pvcreate /dev/vdb
sudo vgcreate vg-disk2 /dev/vdb
sudo lvcreate -n lv-disk2 -l 100%FREE vg-disk2
sudo mkfs.xfs /dev/vg-disk2/lv-disk2
```

RHCOS 是容器的不可变操作系统，没有/etc/fstab 来保存文件系统定义。我们需要创建一个 systemd 挂载文件，

```
[Unit]
Before=local-fs.target
[Mount]
What=/dev/vg-disk2/lv-disk2
Where=/var/mnt/disk2
Type=xfs
[Install]
WantedBy=local-fs.target
```

将该文件保存在`/etc/systemd/system/var-mnt-disk2.mount`中，注意文件名遵循核心操作系统挂载文件命名惯例。

启用并安装它，

```
sudo systemctl enable var-mnt-disk2.mount
sudo systemctl start var-mnt-disk2.mount
sudo mkdir /var/mnt/disk2/pv1
```

现在我们已经挂载了本地文件系统/var/mnt/disk2。我创建了一个子目录 pv1 来保存持久卷。

## 创建本地卷和存储类

创建 PV 和各自的存储类，如下图所示:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-woker1-pv1
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  storageClassName: local-pv-woker1-pv1
  local:
    path: /var/mnt/disk2/pv1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - dev2-worker1.ocp.ibmcloud.io.cpak
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-pv-woker1-pv1
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

nodeAffinity 会将卷绑定到工作节点(dev2-worker1)。存储类是这样定义的，我们可以像使用动态存储类一样使用它。与动态存储类的区别在于，PV 和它的存储类是一对一的映射。如果您需要第二个 PVC，则必须创建一对单独的 PV 和存储类。

只有当 pod 请求与其一起运行时，卷才会绑定。

使用 PV 中定义的 nodeAffinity，Kubernetes 的调度程序将调度 pod 在创建本地 PV 的特定节点上运行。*如果节点关闭，这将影响可用性。*
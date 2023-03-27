# Rook CephFS 安装故障排除

> 原文：<https://itnext.io/troubleshooting-rook-cephfs-mounting-6009e1130ec8?source=collection_archive---------1----------------------->

![](img/22d6d4302d918b3c5ce3f9903aaeb0ed.png)

图片来自[Thanapat pimphol](https://pixabay.com/users/oadtz-3657813/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1862077)来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1862077)

我在设置最新的轮班。像往常一样，我使用 Rook 操作符来提供私有注册中心所需的存储。RWX (ReadWriteMany)访问持久性卷是通过使用 CephFS 存储类实现的。

克隆最新的 Rook 存储库，设置操作符，创建 Rook 集群和存储类，创建 PVC，将其分配给私有注册表。一切进展顺利，所有的车舱都在运行，存储类已创建，PVC 已绑定。但是注册表窗格在长时间等待后无法运行。

通过描述 pod 来检查 pod 事件，

```
oc -n openshift-image-registry describe pods image-registry-c45fcf748-szrnq...
 Warning  FailedMount             58s                  kubelet                  **MountVolume.MountDevice failed** for volume "pvc-0787c032-46a8-4e2a-9c74-38d592cbb550" : rpc error: code = Internal desc = an error (exit status 32) occurred while running mount args: [-t ceph 172.30.129.111:3300,172.30.139.94:3300,172.30.254.66:3300:/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00 /var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-0787c032-46a8-4e2a-9c74-38d592cbb550/globalmount -o name=csi-cephfs-node,secretfile=/tmp/csi/keys/keyfile-3975689489,mds_namespace=rook-cephfs,_netdev] stderr: mount error 110 = Connection timed out
```

挂载失败。但是让我们确认其余的工作正常。

## 验证车

运行`oc -n rook-ceph get pods`以确认所有的 pod 正在运行并准备就绪。

Exec 进入工具舱检查 ceph 状态，

```
oc exec -it rook-ceph-tools-d6d7c985c-mn8hr -- ceph status
  cluster:
    id:     f0f2a152-ece9-491d-a45b-2f60a439c16a
    health: HEALTH_OK services:
    mon: 3 daemons, quorum a,b,c (age 3d)
    mgr: a(active, since 3d)
    mds: 1/1 daemons up, 1 hot standby
    osd: 3 osds: 3 up (since 3d), 3 in (since 3d) data:
    volumes: 1/1 healthy
    pools:   4 pools, 97 pgs
    objects: 27 objects, 119 KiB
    usage:   22 MiB used, 450 GiB / 450 GiB avail
    pgs:     97 active+clean io:
    client:   1.2 KiB/s rd, 2 op/s rd, 0 op/s wr
```

看起来 Rook/ Ceph 运行良好。

## 验证安装

pod 的错误消息表明错误出在“mount”命令上。让我们测试挂载本身，如 pod 事件所示

```
[-t ceph 172.30.129.111:3300,172.30.139.94:3300,172.30.254.66:3300:/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00 /var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-0787c032-46a8-4e2a-9c74-38d592cbb550/globalmount -o name=csi-cephfs-node,secretfile=/tmp/csi/keys/keyfile-3975689489,mds_namespace=rook-cephfs,_netdev]
```

描述 PVC 的相应 PV，

```
$ oc describe pv pvc-0787c032-46a8-4e2a-9c74-38d592cbb550
Name:            pvc-0787c032-46a8-4e2a-9c74-38d592cbb550
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: rook-ceph.cephfs.csi.ceph.com
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    rook-cephfs
Status:          Bound
Claim:           openshift-image-registry/pvc-image-registry
Reclaim Policy:  Delete
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        200Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            rook-ceph.cephfs.csi.ceph.com
    FSType:
    VolumeHandle:      0001-0009-rook-ceph-0000000000000001-0c0965ad-a826-11ec-8b93-0a580a80020f
    ReadOnly:          false
    VolumeAttributes:      clusterID=rook-ceph
                           fsName=rook-cephfs
                           pool=rook-cephfs-data0
                                    storage.kubernetes.io/csiProvisionerIdentity=1647491897305-8081-rook-ceph.cephfs.csi.ceph.com
                           subvolumeName=csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f
                           subvolumePath=/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00
Events:                <none>
```

子卷路径与 pod 事件中的 mount 命令相匹配。

让我们通过测试 mount 命令，在运行 pod 的节点上对它进行测试。

在此之前，先拿到 Ceph 的证书。

```
$ oc exec -it rook-ceph-tools-d6d7c985c-mn8hr -- ceph auth ls
mds.rook-cephfs-a
 key: AQCVvDJi1uQGNhAAuOt9w4pSNUmbVZWDD+DFAA==
 caps: [mds] allow
 caps: [mon] allow profile mds
 caps: [osd] allow *
...
client.admin
 key: **AQD9ujJiKPRqMRAA2a65UJMMV844FgSFNrrKAQ==**
 caps: [mds] allow *
 caps: [mgr] allow *
 caps: [mon] allow *
 caps: [osd] allow *
...
```

我们也可以使用`ceph auth print-key client.admin`来打印出密钥。

登录到运行 pods 的节点，sudo 到 root，运行 mount 命令，

```
[core@dev-410-worker2 ~]$ sudo -i
# mkdir -p /mnt/ceph
# mount -t ceph 172.30.129.111:3300,172.30.139.94:3300,172.30.254.66:3300:/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00 /mnt/ceph -o name=admin,secret=AQD9ujJiKPRqMRAA2a65UJMMV844FgSFNrrKAQ==
```

该命令的输出是，

```
mount: /var/mnt/ceph: mount(2) system call failed: Connection timed out.
```

同时，

```
# dmesg | tail -f
[284657.294544] libceph: mon0 (1)172.30.129.111:3300 socket closed (con state V1_BANNER)
[284658.318751] libceph: mon1 (1)172.30.139.94:3300 socket closed (con state V1_BANNER)
[284658.582452] libceph: mon1 (1)172.30.139.94:3300 socket closed (con state V1_BANNER)
[284659.095492] libceph: mon1 (1)172.30.139.94:3300 socket closed (con state V1_BANNER)
[284660.302497] libceph: mon1 (1)172.30.139.94:3300 socket closed (con state V1_BANNER)
[284661.326709] libceph: mon2 (1)172.30.254.66:3300 socket closed (con state V1_BANNER)
[284661.590463] libceph: mon2 (1)172.30.254.66:3300 socket closed (con state V1_BANNER)
[284662.102229] libceph: mon2 (1)172.30.254.66:3300 socket closed (con state V1_BANNER)
[284663.310139] libceph: mon2 (1)172.30.254.66:3300 socket closed (con state V1_BANNER)
[284664.205628] ceph: No mds server is up or the cluster is laggy
```

端口阻塞？验证连接，

```
# curl 172.30.129.111:3300
ceph v2
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
```

连接很好。由于 rook 集群处于健康状态，连接也没有被阻塞，所以问题更可能出在客户端。

错误`mon2 (1)172.30.254.66:3300 socket closed (con state V1_BANNER)`似乎表明其使用 V1 旗帜。而 3300 端口在 V2 是相对较新的东西。让我们转到 V1 港 6789，

```
mount -t ceph 172.30.129.111:6789,172.30.139.94:6789,172.30.254.66:6789:/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00 /mnt/ceph -o name=admin,secret=AQD9ujJiKPRqMRAA2a65UJMMV844FgSFNrrKAQ==
```

命令立即返回，没有任何错误，这可以用

```
# df -h /mnt/ceph
Filesystem                                                                                                                                                Size  Used Avail Use% Mounted on
172.30.129.111:6789,172.30.139.94:6789,172.30.254.66:6789:/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00  200G     0  200G   0% /var/mnt/ceph
```

## 修复

要修复它，我们需要将端口从 V2(3300)改回 V1 (6789)。但是通过查看 ceph [mount 命令](https://docs.ceph.com/en/latest/man/8/mount.ceph/#basic)的手册，我们找到了一个选项`ms_mode`来指定消息的 V1 或 V2 格式。用 V2 选项进行测试，

```
mount -t ceph 172.30.129.111:3300,172.30.139.94:3300,172.30.254.66:3300:/volumes/csi/csi-vol-0c0965ad-a826-11ec-8b93-0a580a80020f/269b71dd-1dd4-4ee0-961d-10831f05fd00 /mnt/ceph -o name=admin,secret=AQD9ujJiKPRqMRAA2a65UJMMV844FgSFNrrKAQ==,ms_mode=crc
```

CephFS 已成功装载。

在 Kubernetes 中，可以在 StorageClass 定义中添加 CephFS 挂载选项，

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  clusterID: rook-ceph
  fsName: rook-cephfs

  pool: rook-cephfs-data0
  **kernelMountOptions: "ms_mode=crc"**
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
reclaimPolicy: Delete
```

重新创建此存储类，重新创建 PVC ( `*need to edit the PVC to delete the finalizers*`)，删除 pod。现在，卷已连接，并且 pod 已成功运行。

小贴士:

验证“rook-ceph-csi-config”的配置图，找出监视器使用的默认端口。
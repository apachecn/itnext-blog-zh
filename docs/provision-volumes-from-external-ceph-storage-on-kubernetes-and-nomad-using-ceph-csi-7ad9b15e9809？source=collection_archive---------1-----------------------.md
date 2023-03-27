# 使用 Ceph CSI 提供关于 Kubernetes 和 Nomad 的资料

> 原文：<https://itnext.io/provision-volumes-from-external-ceph-storage-on-kubernetes-and-nomad-using-ceph-csi-7ad9b15e9809?source=collection_archive---------1----------------------->

![](img/e1c7ca575325226b3879e0bc057905c0.png)

如果您想在 Kubernetes 上运行有状态的应用程序，CSI(容器存储接口)在动态供应卷方面有着重要的作用。一般来说，CSI 不仅用于为 Kubernetes 提供存储卷，还用于所有其他容器编排器，如 Mesos、Nomad。
在这里，我将讨论使用外部 Ceph 存储中的 Ceph CSI 为 Hashicorp 的 Kubernetes 和 Nomad 进行卷配置。

我在这篇文章中使用的重要组件如下。

*   Kubernetes v1.17 版
*   [Nomad v1.0.4](https://github.com/hashicorp/nomad/tree/v1.0.4)
*   [Ceph 存储 v14(鹦鹉螺)](https://docs.ceph.com/en/latest/releases/nautilus/)
*   [Ceph CSI v3.3.1](https://github.com/ceph/ceph-csi/tree/v3.3.1)

# Kubernetes 上的供应卷

在本例中，您将看到如何使用 Ceph CSI 从 Kubernetes 上的外部 Ceph 存储供应卷。

首先，让我们安装 helm 在 Kubernetes 上部署 Ceph CSI 图表。

```
curl -fsSL -o get_helm.sh [https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3](https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3)
chmod 700 get_helm.sh
./get_helm.sh
```

为 Ceph CSI 创建一个命名空间。

```
kubectl create namespace ceph-csi-rbd; 
```

克隆 Ceph CSI 并切换到 v3.3.1。

```
git clone [https://github.com/ceph/ceph-csi.git](https://github.com/ceph/ceph-csi.git);
cd ceph-csi;
git checkout v3.3.1;# move to rbd chart.
cd charts/ceph-csi-rbd;
```

或者，您可以将 Kubernetes v1.17.x 的 csi 版本更改为`v1beta1`

```
sed -i 's/storage.k8s.io\/betav1/storage.k8s.io\/v1beta1/g' templates/csidriver-crd.yaml;
```

现在，我们需要图表值文件来部署 ceph csi。看起来是这样的。

```
cat <<EOF > ceph-csi-rbd-values.yaml
csiConfig:
  - clusterID: "c628ebf1-d03f-4806-9941-8b5840338b14"
    monitors:
      - "10.0.0.3:6789"
      - "10.0.0.4:6789"
      - "10.0.0.5:6789"
provisioner:
  name: provisioner
  replicaCount: 2
EOF
```

`clusterID`可以通过命令获得。

```
sudo ceph fsid;
```

ceph 监视器的`monitors`可以用这个获得。

```
sudo ceph mon dump;
```

让我们在 Kuberntes 上安装 Ceph CSI 图表。

```
helm install \
--namespace ceph-csi-rbd \
ceph-csi-rbd \
--values ceph-csi-rbd-values.yaml \
./;
kubectl rollout status deployment ceph-csi-rbd-provisioner -n ceph-csi-rbd;
helm status ceph-csi-rbd -n ceph-csi-rbd;
```

现在，您已经准备好 ceph csi 驱动程序来配置卷。

在创建 Ceph 存储类之前，需要在 Ceph 存储中创建几个资源。

要从外部 Ceph 存储在 Kubernetes 上挂载卷，首先需要创建一个池。在 ceph 中创建一个池。

```
sudo ceph osd pool create kubePool 64 64
```

并将池初始化为块设备。

```
sudo rbd pool init kubePool
```

要使用策略访问池，您需要一个用户。在此示例中，将创建池的管理员用户。让我们为池`kubePool`创建一个管理员用户，并将生成的密钥编码为 base64。

```
sudo ceph auth get-or-create-key client.kubeAdmin mds 'allow *' mgr 'allow *' mon 'allow *' osd 'allow * pool=kubePool' | tr -d '\n' | base64;
QVFEMWxvTmdzMTZyRVJBQTZJalBHcDBWUi8wcUd6TW9sSmlaTXc9PQ==
```

将管理员用户`kubeAdmin`编码为 base64。

```
echo "kubeAdmin" | tr -d '\n' | base64;
a3ViZUFkbWlu
```

现在，已经为池`kubePool`创建了管理员用户和密钥。

让我们为 admin 用户创建一个秘密资源。

```
cat > ceph-admin-secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: ceph-admin
  namespace: default
type: kubernetes.io/rbd
data:
  userID: a3ViZUFkbWlu
  userKey: QVFEMWxvTmdzMTZyRVJBQTZJalBHcDBWUi8wcUd6TW9sSmlaTXc9PQ==
EOF# create a secret for admin user.
kubectl apply -f ceph-admin-secret.yaml;
```

最后，我们准备创建 Ceph 存储类。让我们创造它。

```
# ceph storage class.
cat > ceph-rbd-sc.yaml <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: c628ebf1-d03f-4806-9941-8b5840338b14
   pool: kubePool
   imageFeatures: layering
   csi.storage.k8s.io/provisioner-secret-name: ceph-admin
   csi.storage.k8s.io/provisioner-secret-namespace: default
   csi.storage.k8s.io/controller-expand-secret-name: ceph-admin
   csi.storage.k8s.io/controller-expand-secret-namespace: default
   csi.storage.k8s.io/node-stage-secret-name: ceph-admin
   csi.storage.k8s.io/node-stage-secret-namespace: default
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
   - discard
EOFkubectl apply -f ceph-rbd-sc.yaml;
```

`clusterID`是`sudo ceph fsid`的值，`pool`是上面已经创建的`kubePool`的值。

现在，您已经准备好使用 Ceph 存储类轻松地为有状态应用程序提供卷了。

让我们用 pvc 创建一个 pod，以使用 ceph 存储类从外部 ceph 存储挂载卷。

```
## create pvc and pod.
cat <<EOF > pv-pod.yaml   
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ceph-rbd-sc-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: ceph-rbd-sc
---    
apiVersion: v1
kind: Pod
metadata:
  name: ceph-rbd-pod-pvc-sc
spec:
  containers:
  - name:  ceph-rbd-pod-pvc-sc
    image: busybox
    command: ["sleep", "infinity"]
    volumeMounts:
    - mountPath: /mnt/ceph_rbd
      name: volume
  volumes:
  - name: volume
    persistentVolumeClaim:
      claimName: ceph-rbd-sc-pvc
EOF

kubectl apply -f pv-pod.yaml;
```

看一下`storageClassName`也就是`ceph-rbd-sc`的值。将使用 ceph 存储类动态配置该卷，此示例 pod 将使用路径`/mnt/ceph_rbd`挂载配置的卷。

让我们检查一下容器中的容量。

```
kubectl exec pod/ceph-rbd-pod-pvc-sc -- df -k | grep rbd;
/dev/rbd0              1998672      6144   1976144   0% /mnt/ceph_rbd
```

如图所示，ceph rbd 卷已经安装到容器中的路径`/mnt/ceph_rbd`上。

并且，检查是否在 ceph 池中创建了映像。

```
sudo rbd ls -p kubePool;
csi-vol-c545c641-a4b3-11eb-b242-26d41aad22d3
```

现在，您已经看到了如何在 Kubernetes 上使用 CSI 从外部 Ceph 存储供应卷。

# 提供关于游牧的书籍

正如您在上面看到的，在 kubernetes 上动态供应卷很简单。游牧部落怎么样？

首先，您需要将以下内容添加到 nomad 客户端配置中。

```
plugin "docker" {
  config {
    allow_privileged = true
  }
}
```

这样，docker 容器可以在 Nomad 客户机节点上以特权身份运行。

如前所述，CSI 不仅可以在 Kubernetes 上运行，还可以在所有其他容器编排器上运行。CSI 由`controller`和`node`组成。让我们首先创建一个 Ceph CSI 控制器作业。

```
cat <<EOC > ceph-csi-plugin-controller.nomad
job "ceph-csi-plugin-controller" {
  datacenters = ["dc1"]group "controller" {
    network {    
      port "metrics" {}
    }
    task "ceph-controller" {template {
        data        = <<EOF
[{
    "clusterID": "62c42aed-9839-4da6-8c09-9d220f56e924",
    "monitors": [
        "10.0.0.3:6789",
  "10.0.0.4:6789",
  "10.0.0.5:6789" 
    ]
}]
EOF
        destination = "local/config.json"
        change_mode = "restart"
      }
      driver = "docker"
      config {
        image = "quay.io/cephcsi/cephcsi:v3.3.1"
        volumes = [
          "./local/config.json:/etc/ceph-csi-config/config.json"
        ]    
        args = [
          "--type=rbd",
          "--controllerserver=true",
          "--drivername=rbd.csi.ceph.com",         
          "--endpoint=unix://csi/csi.sock",
          "--nodeid=\${node.unique.name}",
    "--instanceid=\${node.unique.name}-controller", 
          "--pidlimit=-1",
    "--logtostderr=true",
          "--v=5",    
          "--metricsport=\$\${NOMAD_PORT_metrics}"
        ]  
      }   
   resources {
        cpu    = 500
        memory = 256
      }
      service {
        name = "ceph-csi-controller"
        port = "metrics"
        tags = [ "prometheus" ]
      }csi_plugin {
        id        = "ceph-csi"
        type      = "controller"
        mount_dir = "/csi"
      }
    }
  }
}
EOC
```

`clusterID`和`monitors`应该改成你的值。该 Ceph CSI 控制器将被创建为`service`类型。

并创建一个 Ceph CSI 节点作业。

```
cat <<EOC > ceph-csi-plugin-nodes.nomad
job "ceph-csi-plugin-nodes" {
  datacenters = ["dc1"]
  type        = "system"
  group "nodes" {
    network {    
      port "metrics" {}
    }

    task "ceph-node" {
      driver = "docker"
      template {
        data        = <<EOF
[{
    "clusterID": "62c42aed-9839-4da6-8c09-9d220f56e924",
    "monitors": [
        "10.0.0.3:6789",
  "10.0.0.4:6789",
  "10.0.0.5:6789" 
    ]
}]
EOF
        destination = "local/config.json"
        change_mode = "restart"
      }
      config {
        image = "quay.io/cephcsi/cephcsi:v3.3.1"
        volumes = [
          "./local/config.json:/etc/ceph-csi-config/config.json"
        ]
        mounts = [
          {
            type     = "tmpfs"
            target   = "/tmp/csi/keys"
            readonly = false
            tmpfs_options = {
              size = 1000000 # size in bytes
            }
          }
        ]
        args = [
          "--type=rbd",
          "--drivername=rbd.csi.ceph.com",        
          "--nodeserver=true",
          "--endpoint=unix://csi/csi.sock",
          "--nodeid=\${node.unique.name}",
          "--instanceid=\${node.unique.name}-nodes", 
          "--pidlimit=-1",
    "--logtostderr=true",
          "--v=5",       
          "--metricsport=\$\${NOMAD_PORT_metrics}"
        ]  
        privileged = true
      }   
   resources {
        cpu    = 500
        memory = 256
      }
      service {
        name = "ceph-csi-nodes"
        port = "metrics"
        tags = [ "prometheus" ]
      }csi_plugin {
        id        = "ceph-csi"
        type      = "node"
        mount_dir = "/csi"
      }
    }
  }
}
EOC
```

这个 Ceph 节点作业是`system`的类型，也就是说，ceph csi 节点容器将在所有的 nomad 客户机节点上创建。

让我们在 Nomad 上运行 Ceph CSI 控制器和节点作业。

```
nomad job run ceph-csi-plugin-controller.nomad;
nomad job run ceph-csi-plugin-nodes.nomad;
```

查看 ceph csi 插件的状态。

```
nomad plugin status ceph-csi;
ID                   = ceph-csi
Provider             = rbd.csi.ceph.com
Version              = v3.3.1
Controllers Healthy  = 1
Controllers Expected = 1
Nodes Healthy        = 2
Nodes Expected       = 2Allocations
ID        Node ID   Task Group  Version  Desired  Status   Created    Modified
b6268d6d  457a8291  controller  0        run      running  1d21h ago  1d21h ago
ec265d25  709ee9cc  nodes       0        run      running  1d21h ago  1d21h ago
4cd7dffa  457a8291  nodes       0        run      running  1d21h ago  1d21h ago
```

现在，它可以使用 ceph csi 驱动程序从外部 ceph 存储挂载卷了。

在继续之前，在所有的 nomad 客户机节点上加载 RBD 模块。

```
sudo modprobe rbd;
sudo lsmod |grep rbd;
rbd                    83733  0
libceph               306750  1 rbd
```

让我们创建一个 ceph 池`myPool`和管理员用户`myPoolAdmin`。

```
# Create a ceph pool:
sudo ceph osd pool create myPool 64 64
sudo rbd pool init myPool;# create admin user for pool.
sudo ceph auth get-or-create-key client.myPoolAdmin mds 'allow *' mgr 'allow *' mon 'allow *' osd 'allow * pool=myPool'
AQCKf4JgHPVxAxAALZ8ny4/R7s6/3rZWC2o5vQ==
```

现在我们需要在 Nomad 上注册一个卷，创建一个卷。

```
cat <<EOF > ceph-volume.hcl
type = "csi"
id   = "ceph-mysql"
name = "ceph-mysql"
external_id     = "0001-0024-62c42aed-9839-4da6-8c09-9d220f56e924-0000000000000009-00000000-1111-2222-bbbb-cacacacacaca"                          
access_mode     = "single-node-writer"
attachment_mode = "file-system"
mount_options {
  fs_type = "ext4"
}
plugin_id       = "ceph-csi"
secrets {
  userID  = "myPoolAdmin"
  userKey = "AQCKf4JgHPVxAxAALZ8ny4/R7s6/3rZWC2o5vQ=="  
}
context {  
  clusterID = "62c42aed-9839-4da6-8c09-9d220f56e924"
  pool      = "myPool" 
  imageFeatures = "layering"
}
EOF
```

`userID`和`userKey`是上面创建的值，`clusterID`是 Ceph Cluster ID，`pool`是之前创建的`myPool`的值。

看一下`external_id`，它是单个卷的唯一 ID。该 ID 基于 [CSI ID 格式](https://github.com/ceph/ceph-csi/blob/71ddf51544be498eee03734573b765eb04480bb9/internal/util/volid.go#L27)。让我们看看`external_id`的约定。

```
<csi-id-version>-<cluster-id-length>-<cluster-id>-<pool-id>-<uuid>
```

按照这个惯例，下面的`external_id`可以分成独立的部分。

```
0001-0024-62c42aed-9839-4da6-8c09-9d220f56e924-0000000000000009-00000000-1111-2222-bbbb-cacacacacaca
```

*   CSI ID 版本:`0001`
*   集群 ID 长度:`0024`
*   集群 ID: `62c42aed-9839–4da6–8c09–9d220f56e924`
*   人才库 ID: `0000000000000009`
*   UUID: `00000000–1111–2222-bbbb-cacacacacaca`

对于池 ID，您可以在 ceph 中获得池的 ID`myPool`。

```
sudo ceph osd lspools
1 cephfs_data
2 cephfs_metadata
3 foo
4 bar
5 .rgw.root
6 default.rgw.control
7 default.rgw.meta
8 default.rgw.log
9 myPool
10 default.rgw.buckets.index
11 default.rgw.buckets.data
```

`myPool`的 ID 是 9。

UUID 是卷的唯一 ID。如果要从同一个池创建新卷，需要设置新 UUID。

在 Nomad 上注册卷之前，需要先在池`myPool`中创建一个映像。

```
sudo rbd create csi-vol-00000000-1111-2222-bbbb-cacacacacaca --size 1024 --pool myPool --image-feature layering;
```

看一下图片的名字`csi-vol-00000000–1111–2222-bbbb-cacacacacaca`。Ceph CSI 参数`volumeNamePrefix`的默认值为`csi-vol-`。剩下的就是上面提到的基于 CSI ID 格式的`external_id`的 UUID。要创建的图像的名称有以下约定。

```
<volume-name-prefix><uuid>
```

现在，在 Nomad 上注册音量。

```
nomad volume register ceph-volume.hcl;
```

查看注册卷的状态。

```
nomad volume status;
Container Storage Interface
ID        Name        Plugin ID  Schedulable  Access Mode
ceph-mys  ceph-mysql  ceph-csi   true         single-node-writer
```

让我们通过在 Nomad 上运行示例 MySQL 服务器作业来挂载 Ceph CSI 提供的卷。

```
cat <<EOF > mysql-server.nomad
job "mysql-server" {
  datacenters = ["dc1"]
  type        = "service"group "mysql-server" {
    count = 1volume "ceph-mysql" {
      type      = "csi"
      read_only = false
      source    = "ceph-mysql"
    }network {
      port "db" {
        static = 3306
      }
    }restart {
      attempts = 10
      interval = "5m"
      delay    = "25s"
      mode     = "delay"
    }task "mysql-server" {
      driver = "docker"volume_mount {
        volume      = "ceph-mysql"
        destination = "/srv"
        read_only   = false
      }env {
        MYSQL_ROOT_PASSWORD = "password"
      }config {
        image = "hashicorp/mysql-portworx-demo:latest"
        args  = ["--datadir", "/srv/mysql"]
        ports = ["db"]
      }resources {
        cpu    = 500
        memory = 1024
      }service {
        name = "mysql-server"
        port = "db"check {
          type     = "tcp"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
EOF# run mysql job.
nomad job run mysql-server.nomad;
```

让我们检查来自 Ceph RBD 的卷是否在进入分配的 mysql 服务器容器时被挂载。

```
nomad alloc exec bfe37c92 sh
# df -h
Filesystem      Size  Used Avail Use% Mounted on
...
/dev/rbd0       976M  180M  781M  19% /srv
...
```

如图所示，卷`ceph-mysql`已经安装到路径`/srv`中。

现在，我们将检查在 Nomad 上重新提交 mysql 服务器作业后，MySQL 数据是否丢失。

让我们连接到容器中的 MySQL 服务器。

```
mysql -u root -p -D itemcollection;
... type the password of MYSQL_ROOT_PASSWORD mysql> select * from items;
+----+----------+
| id | name     |
+----+----------+
|  1 | bike     |
|  2 | baseball |
|  3 | chair    |
+----+----------+
```

让我们添加一些行。

```
INSERT INTO items (name) VALUES ('glove');
INSERT INTO items (name) VALUES ('hat');
INSERT INTO items (name) VALUES ('keyboard');
```

确保成功插入行。

```
mysql> select * from items;
+----+----------+
| id | name     |
+----+----------+
|  1 | bike     |
|  2 | baseball |
|  3 | chair    |
|  4 | glove    |
|  5 | hat      |
|  6 | keyboard |
+----+----------+
6 rows in set (0.00 sec)
```

现在，停止 mysql 服务器的工作。

```
nomad stop -purge mysql-server;
```

并且，再次提交 mysql 服务器作业。

```
nomad job run mysql-server.nomad
```

检查 mysql 数据是否无损存在。进入分配的 mysql 服务器容器后，像前面一样键入以下内容。

```
mysql -u root -p -D itemcollection;
... type the password of MYSQL_ROOT_PASSWORDmysql> select * from items;
+----+----------+
| id | name     |
+----+----------+
|  1 | bike     |
|  2 | baseball |
|  3 | chair    |
|  4 | glove    |
|  5 | hat      |
|  6 | keyboard |
+----+----------+
6 rows in set (0.00 sec)
```

即使 mysql 服务器作业在 Nomad 上重新启动，mysql 服务器也不会丢失数据。

您已经看到了目前如何使用相同的 Ceph CSI 在 Kubernetes 和 Nomad 上提供卷。

要使用 CSI 从外部 Ceph 存储供应卷，在 Kubernetes 上比在 Nomad 上更简单。您可以在 Kubernetes 上动态配置卷，但是目前，Nomad 还不支持动态卷配置。最近， [Nomad 1.1 Beta 版](https://www.hashicorp.com/blog/announcing-hashicorp-nomad-1-1-beta)宣布了对 CSI 支持的改进，例如，`nomad volume create <volume-hcl>`将自动在 ceph 中创建一个池的图像。

## 参考

*   [https://rancher . com/docs/rancher/v2 . x/en/cluster-admin/volumes-and-storage/ceph/](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/volumes-and-storage/ceph/)
*   [https://learn . hashi corp . com/tutorials/nomad/stateful-workloads-CSI-volumes？in =移动/有状态工作负载](https://learn.hashicorp.com/tutorials/nomad/stateful-workloads-csi-volumes?in=nomad/stateful-workloads)
*   [https://github . com/hashicorp/nomad/tree/main/demo/CSI/ceph-CSI-plugin](https://github.com/hashicorp/nomad/tree/main/demo/csi/ceph-csi-plugin)
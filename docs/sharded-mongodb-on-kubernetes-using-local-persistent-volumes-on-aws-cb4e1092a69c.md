# 使用 AWS 上的本地持久卷在 Kubernetes 上分片 MongoDB

> 原文：<https://itnext.io/sharded-mongodb-on-kubernetes-using-local-persistent-volumes-on-aws-cb4e1092a69c?source=collection_archive---------1----------------------->

本文将展示如何在 AWS 上使用本地持久卷创建分片的 mongodb 集群。

i3 系列实例配有高随机 IO 性能 SSD 存储、出色的网络带宽，强烈建议用于 MongoDB 等 NoSQL 工作负载。我们对它的 MongoDB 集群使用相同的方法。现在，这要求我们使用本地持久卷，Kubernetes 在 1.9 中通过延迟卷绑定改进了对本地持久卷的支持。本地持久卷也记录在这里[https://github . com/kubernetes-incubator/external-storage/tree/master/local-volume](https://github.com/kubernetes-incubator/external-storage/tree/master/local-volume)。通过所有这些，不明显的是 Kubernetes 使得使用本地持久卷变得非常容易。

# 启用本地持久卷

我们将使用 Kubespray v2.5.0 来启动和管理我们的 Kubernetes 集群。要启用本地永久卷，请在`inventory/<folder>/group_vars/k8s-cluster.yml`中将以下属性设置为真

```
persistent_volumes_enabled: true
local_volume_provisioner_enabled: true
```

这将启用 Kubernetes `PersistentLocalVolumes`功能门以及`VolumeScheduling`和`MountPropagation`。

还要注意以下两个变量

```
local_volume_provisioner_base_dir: /mnt/disks
local_volume_provisioner_mount_dir: /mnt/disks
```

现在开始启动集群。在这篇文章中，我启动了一个集群，其中有 3 个配置节点 i3/other 和 6 个 i3 . 2x 大型实例用于分片。

一旦您使用 Kubespray 启动了集群，您应该能够看到您的节点

```
kubectl get nodes
```

接下来，您应该标记您的节点，以区分什么是配置，什么是碎片。做这件事有不同的方法。但基本上它相当于发布了类似

```
kubectl label nodes <node name> component=mongo-config
kubectl label nodes <node name> component=mongo-shard
```

我们稍后将使用这些标签为我们的 pod 指定`nodeAffinity`，以便我们将它们绑定到正确的实例集。

# 设置本地永久卷

现在，我们将了解如何调配本地 pvs。我们需要做的就是将本地磁盘挂载到/mnt/disks。它们将被自动检测，然后作为本地 pv 自动提供。

有几种方法可以做到这一点。您可以使用 daemonset 或者使用一个简单的脚本来为每个标记为 mongo-shard 的节点执行此操作。

上面的脚本会发现 i3 实例中的每个磁盘，并使用 xfs 格式化它们(这是 MongoDB 推荐的[)并挂载到/mnt/disks。它还将条目添加到 fstab 中，以便在节点重新启动后装载是持久的。](https://scalegrid.io/blog/xfs-vs-ext4-comparing-mongodb-performance-on-aws-ec2/)

现在 Kubernetes 魔术发生了，这些卷是自动可用的！下面的代码片段显示了同样的情况(但是在绑定到 pod 之后)

```
kubectl get pvNAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                        STORAGECLASS    REASON    AGElocal-pv-1756e029                          442Gi      RWO            Delete           Bound     default/data-mongo-shard0-1                  local-storage             9dlocal-pv-277d7925                          442Gi      RWO            Delete           Bound     default/data-mongo-shard1-0                  local-storage             9dlocal-pv-5e05e368                          442Gi      RWO            Delete           Bound     default/data-mongo-shard1-1                  local-storage             9dlocal-pv-6836da3e                          442Gi      RWO            Delete           Bound     default/data-mongo-shard0-2                  local-storage             9dlocal-pv-a66723f4                          442Gi      RWO            Delete           Bound     default/data-mongo-shard0-0                  local-storage             9dlocal-pv-d3c10f22                          442Gi      RWO            Delete           Bound     default/data-mongo-shard1-2                  local-storage             9d
```

# 创建密钥文件

我们现在将创建密钥文件。我们将以可重复的方式做这件事。

这个脚本基本上创建了一个 mongodb-keyfile，如果它在 Kubernetes 中不作为秘密存在的话

我们稍后将在 mongo-config、mongo-shard 等的所有 yaml 中引用这个密钥文件秘密。

# 启动 mongo 配置

接下来，我们将调出 mongo-config 吊舱

你可以在这里找到 YAML[https://github . com/sabar ishs/sharded-mongo-kube-local-PV/blob/master/src/mongo-config . YAML](https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-config.yaml)

请注意我们是如何使用 statefulset 并将我们的 pods 绑定到 mongo-config 标记的节点上的

```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
      - key: node.ignite.harman.com/component
      operator: In
      values: ["mongo-config"]
```

现在我们需要初始化配置复制集

```
kubectl exec mongo-config-0 -- mongo --eval "rs.initiate({_id: \"crs\", configsvr: true, members: [ {_id: 0, host: \"mongo-config-0.mongo-config-svc.default.svc.cluster.local:27017\"}, {_id: 1, host: \"mongo-config-1.mongo-config-svc.default.svc.cluster.local:27017\"}, {_id: 2, host: \"mongo-config-2.mongo-config-svc.default.svc.cluster.local:27017\"} ]});"
```

# 旋转 mongo 查询路由器盒

接下来，我们将介绍几个查询路由器。Mongo 最佳实践建议您应该在每个需要 mongo 访问的节点上运行这个。您可以将它作为`daemonset`运行，并将其绑定到所有需要访问 mongo 的节点。

你可以在这里找到 YAML[https://github . com/sabar ishs/sharded-mongo-kube-local-PV/blob/master/src/mongo-QR . YAML](https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-qr.yaml)

注意我们如何使用 nodeAffinity 将它绑定到所有类型为“api”的节点

```
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: node.ignite.harman.com/component
      operator: In
      values:
      - api
```

因此，所有 API pods 现在都可以使用 <node-host-name>:27017 访问本地查询路由器</node-host-name>

您可以在 API pod 中将它作为一个额外的容器来运行，但是这将意味着多个查询路由器将在一个 API 节点中运行，并将导致开放 mongo 连接的数量膨胀。通过作为 daemonset 运行，我们将其限制为每个候选节点一个。

# 旋转 mongo-shard

接下来我们将带来一个 mongo-shard。因为您想要处理多个分片，所以标准的做法是拥有一个模板，然后通过替换分片索引来为每个分片生成实际的 yaml 工件。

你可以在这里找到 YAML[https://github . com/sabar ishs/sharded-mongo-kube-local-PV/blob/master/src/mongo-shard . YAML](https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-shard.yaml)

我们将为每个分片生成一个副本，然后初始化副本集，最后在循环中将分片添加到查询路由器。

```
while [[ $shards -lt $shardcount ]]
do sed "s/shard0/shard$shards/g" mongo-shard.yaml > mongo-shard$shards.yaml kubectl apply -f mongo-shard$shards.yaml kubectl exec mongo-shard$shards-0 -- mongo --eval "rs.initiate({_id: \"shard$shards\", members: [ {_id: 0, host: \"mongo-shard$shards-0.mongo-shard$shards-svc.default.svc.cluster.local:27017\"}, {_id: 1, host: \"mongo-shard$shards-1.mongo-shard$shards-svc.default.svc.cluster.local:27017\"}, {_id: 2, host: \"mongo-shard$shards-2.mongo-shard$shards-svc.default.svc.cluster.local:27017\"} ]});" kubectl exec $qrPod -- mongo --eval "sh.addShard(\"shard$shards/mongo-shard$shards-0.mongo-shard$shards-svc.default.svc.cluster.local:27017\");" shards=$(($shards+1))done
```

精彩！我们完了。如果我们现在检查 pv，我们会看到它们被绑定。我们的碎片现在与 Kubernetes 上的本地持久卷同步。干得好！

在我的 github[https://github.com/sabarishs/sharded-mongo-kube-local-pv](https://github.com/sabarishs/sharded-mongo-kube-local-pv)中可以找到所有这些工件和将它们联系在一起的引导脚本

我建议您查看引导脚本。它可以重新运行任何次数，也有助于扩展碎片的数量。

```
#the following sets up 2 sharded cluster
mongo-bootstrap.sh 2#the following adds another shard to make it a 3 shard cluster
mongo-bootstrap.sh 3
```

# 确认

如果没有 Paul Done 和 Sunny 的这些精彩博文，我不可能在 3 天内完成所有这些工作。

[Paul Done 的优秀博客](http://pauldone.blogspot.in/2017/06/deploying-mongodb-on-kubernetes-gke25.html)
Sunny 的中等博客
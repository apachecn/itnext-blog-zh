# Apache Druid: PostgreSQL 作为元数据存储，并用 Kubernetes 扩展替换 ZooKeeper

> 原文：<https://itnext.io/apache-druid-postgresql-as-metadata-storage-and-replace-zookeeper-with-kubernetes-extensions-668c93f0a389?source=collection_archive---------9----------------------->

![](img/b1cc264af56203fcad582e91385e792e.png)

我们继续一系列关于阿帕奇德鲁伊的帖子。在[的第一部分](https://rtfm.co.ua/en/apache-druid-overview-running-in-kubernetes-and-monitoring-with-prometheus/)中，我们看了一下 [Apache Druid](https://rtfm-co-ua.translate.goog/uk/apache-druid-oglyad-zapusk-v-kubernetes-ta-monitoring-z-prometheus/?_x_tr_sl=ru&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp) 本身——它的架构和监控，在[的第二部分](https://rtfm.co.ua/en/postgresql-postgresql-operator-for-kubernetes-and-its-prometheus-monitoring/) —我们运行了一个 PostgreSQL 集群并设置了它的监控。

后续任务:

*   将 Druid 切换到 PostgreSQL 作为[元数据存储](https://druid.apache.org/docs/latest/design/architecture.html#metadata-storage)而不是 [Apache Derby](https://db.apache.org/derby/)
*   去掉[阿帕奇动物园管理员](https://zookeeper.apache.org/)，换上`[druid-kubernetes-extensions](https://druid.apache.org/docs/latest/development/extensions-core/kubernetes.html)`

让我们从 PostgreSQL 开始。

# 用 PostgreSQL 配置 Apache Druid

参见 [PostgreSQL 元数据存储](https://druid.apache.org/docs/latest/development/extensions-core/postgresql.html)和[元数据存储](https://apache.googlesource.com/druid/+/refs/heads/0.14.0-incubating/docs/content/configuration/index.md#metadata-storage)。

## PostgreSQL 用户

让我们回到用于创建 PostgreSQL 集群的`[manifests/minimal-master-replica-svcmonitor.yaml](https://github.com/zalando/postgres-operator/blob/master/manifests/minimal-master-replica-svcmonitor.yaml)`文件——添加一个`druid`用户和一个数据库`druid`:

```
...
  users:
    zalando:  # database owner
    - superuser
    - createdb
    foo_user: []  # role for application foo
    druid:       
    - createdb
  databases:
    foo: zalando  # dbname: owner
    druid: druid
...
```

升级群集:

```
$ kubectl apply -f maniapplyminimal-master-replica-svcmonitor.yaml
```

取回用户*德鲁伊*的密码:

```
$ kubectl -n test-pg get secret druid.acid-minimal-cluster.credentials.postgresql.acid.zalan.do -o ‘jsonpath={.data.password}’ | base64 -d
Zfqeb0oJnW3fcBCZvEz1zyAn3TMijIvdv5D8WYOz0Y168ym6fXahta05zJjnd3tY
```

打开 PostgreSQL 主服务器的端口:

```
$ kubectl -n test-pg port-forward acid-minimal-cluster-0 6432:5432
Forwarding from 127.0.0.1:6432 -> 5432
Forwarding from [::1]:6432 -> 5432
```

连接:

```
$ psql -U druid -h localhost -p 6432
Password for user druid:
psql (14.5, server 13.7 (Ubuntu 13.7–1.pgdg18.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type “help” for help.
druid=>
```

检查底座—现在必须是空的:

```
druid-> \dt
Did not find any relations.
```

## 阿帕奇德鲁伊`metadata.storage`配置

使用与创建 Apache 德鲁伊集群相同的`[druid-operator/examples/tiny-cluster.yaml](https://github.com/druid-io/druid-operator/blob/master/examples/tiny-cluster.yaml)`配置(参见[启动德鲁伊集群](https://rtfm.co.ua/en/apache-druid-overview-running-in-kubernetes-and-monitoring-with-prometheus/#Spin_Up_Druid_Cluster))。

目前，它是为 DerbyDB 配置的，因此数据存储在本地磁盘上:

```
...
    druid.metadata.storage.type=derby
    druid.metadata.storage.connector.connectURI=jdbc:derby://localhost:1527/druid/data/derbydb/metadata.db;create=true
    druid.metadata.storage.connector.host=localhost
    druid.metadata.storage.connector.port=1527
    druid.metadata.storage.connector.createTables=true
...
```

对于 PostgreSQL，我们需要指定`connectURI`，所以找到对应的 Kubernetes 服务:

```
$ kubectl -n test-pg get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
acid-minimal-cluster ClusterIP 10.97.188.225 <none> 5432/TCP 14h
```

编辑清单—注释 Derbi 行，并添加一个带有`type=postgresql`的新配置:

```
...
    # Extensions
    #
    druid.extensions.loadList=["druid-kafka-indexing-service", "postgresql-metadata-storage", "druid-kubernetes-extensions"]
...
    # Metadata Store
    #druid.metadata.storage.type=derby
    #druid.metadata.storage.connector.connectURI=jdbc:derby://localhost:1527/druid/data/derbydb/metadata.db;create=true
    #druid.metadata.storage.connector.host=localhost
    #druid.metadata.storage.connector.port=1527
    #druid.metadata.storage.connector.createTables=true

    druid.metadata.storage.type=postgresql
    druid.metadata.storage.connector.connectURI=jdbc:postgresql://acid-minimal-cluster.test-pg.svc.cluster.local/druid
    druid.metadata.storage.connector.user=druid
    druid.metadata.storage.connector.password=Zfqeb0oJnW3fcBCZvEz1zyAn3TMijIvdv5D8WYOz0Y168ym6fXahta05zJjnd3tY
    druid.metadata.storage.connector.createTables=true
...
```

更新 Druid 集群:

```
$ kubectl -n druid apply -f examples/tiny-cluster.yaml
```

检查 Postgre 数据库中的数据:

```
druid-> \dt
List of relations
Schema | Name | Type | Owner
 — — — — + — — — — — — — — — — — -+ — — — -+ — — — -
public | druid_audit | table | druid
public | druid_config | table | druid
public | druid_datasource | table | druid
public | druid_pendingsegments | table | druid
public | druid_rules | table | druid
public | druid_segments | table | druid
public | druid_supervisors | table | druid
```

不错！

如果需要将数据从 Derby 迁移到 Postgre——参见[元数据迁移](https://druid.apache.org/docs/latest/operations/metadata-migration.html)。

接下来，让我们去掉 ZooKeeper 实例中的必要项，用 Kubernetes 本身替换它。

# 配置 Druid Kubernetes 服务发现

此处> > > 为模块的文档。

回到`[druid-operator/examples/tiny-cluster.yaml](https://github.com/druid-io/druid-operator/blob/master/examples/tiny-cluster.yaml)`，更新配置-打开 ZooKeeper，添加一个新的扩展`druid-kubernetes-extensions`和附加参数:

```
...
    druid.extensions.loadList=["druid-kafka-indexing-service", "postgresql-metadata-storage", "druid-kubernetes-extensions"]
    ...
    druid.zk.service.enabled=false
    druid.serverview.type=http
    druid.coordinator.loadqueuepeon.type=http
    druid.indexer.runner.type=httpRemote
    druid.discovery.type=k8s

    # Zookeeper
    #druid.zk.service.host=tiny-cluster-zk-0.tiny-cluster-zk
    #druid.zk.paths.base=/druid
    #druid.zk.service.compress=false
...
```

升级群集:

```
$ kubectl -n druid apply -f examples/tiny-cluster.yam
```

## 德鲁伊 RBAC 角色

添加一个 RBAC 角色和一个角色绑定，否则，我们将得到如下授权错误:

> ERROR[org . Apache . druid . k8s . discovery . k 8 sdruidnodedicovery provider＄NodeRoleWatcherbroker]org . Apache . druid . k8s . discovery . k 8 sdruidnodedicovery provider＄NodeRoleWatcher—观察节点类型时出错[BROKER]
> org . Apache . druid . Java . util . common . re:观察窗格时出现错误，代码[403]和错误[{"kind":"Status "，" apiVersion":"v1 "，" metadata":{}，" Status "

根据[文档](https://druid.apache.org/docs/latest/development/extensions-core/kubernetes.html#gotchas)创建一个清单:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: druid-cluster
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - configmaps
  verbs:
  - '*'
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: druid-cluster
subjects:
- kind: ServiceAccount
  name: default
roleRef:
  kind: Role
  name: druid-cluster
  apiGroup: rbac.authorization.k8s.io
```

在德鲁伊的名称空间中创建新资源:

```
$ kubectl -n druid apply -f druid-serviceaccout.yaml
role.rbac.authorization.k8s.io/druid-cluster created
rolebinding.rbac.authorization.k8s.io/druid-cluster created
```

一两分钟后再次检查日志:

```
…
2022–09–21T17:01:15,916 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Starting NodeRoleWatcher for [HISTORICAL]…
2022–09–21T17:01:15,916 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Started NodeRoleWatcher for [HISTORICAL].
2022–09–21T17:01:15,916 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Created NodeRoleWatcher for nodeRole [HISTORICAL].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Creating NodeRoleWatcher for nodeRole [PEON].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Starting NodeRoleWatcher for [PEON]…
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Started NodeRoleWatcher for [PEON].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Created NodeRoleWatcher for nodeRole [PEON].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Creating NodeRoleWatcher for nodeRole [INDEXER].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Starting NodeRoleWatcher for [INDEXER]…
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Started NodeRoleWatcher for [INDEXER].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Created NodeRoleWatcher for nodeRole [INDEXER].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Creating NodeRoleWatcher for nodeRole [BROKER].
2022–09–21T17:01:15,917 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Starting NodeRoleWatcher for [BROKER]…
2022–09–21T17:01:15,918 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider$NodeRoleWatcher — Started NodeRoleWatcher for [BROKER].
2022–09–21T17:01:15,918 INFO [main] org.apache.druid.k8s.discovery.K8sDruidNodeDiscoveryProvider — Created NodeRoleWatcher for nodeRole [BROKER].
…
```

完成了。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/apache-druid-postgresql-as-metadata-storage-and-replace-zookeeper-with-kubernetes-extensions/) *。*
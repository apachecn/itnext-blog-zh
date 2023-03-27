# 和卡斯科普一起在库贝内特斯部署卡珊德拉

> 原文：<https://itnext.io/deploying-cassandra-in-kubernetes-with-casskop-98956edfe32b?source=collection_archive---------2----------------------->

![](img/320db31ef7e6bd443fec4ee83b50818a.png)

CassKop 是 Cassandra 的 Kubernetes 操作员。它可以创建、配置和管理运行在 Kubernetes 上的 Cassandra 集群。卡斯科普有很多有趣的特点。其中一个特性是数据中心和机架感知部署，这是本文的重点。

CassKop 引入了一个新的自定义资源定义`CassandraCluster`，它允许您指定一些东西，比如:

*   CPU 和内存资源
*   [节点反亲和](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)
*   资源限制
*   存储类
*   拓扑(就数据中心和机架配置而言)

这篇文章探讨了如何配置一些不同的 Cassandra 集群拓扑。

# 安装 CassKop

卡斯科普可作为[舵](https://helm.sh/)图。Helm 是部署操作员的最简单方法。设置 Helm 超出了这篇文章的范围；因此，我将简单地向您介绍[用户指南](https://helm.sh/docs/using_helm/#installing-helm)中有关其安装的详细信息。请务必同时查看[为舵柄设置 RBAC](https://helm.sh/docs/using_helm/#role-based-access-control)。

最新的操舵图可以在 GitHub 的 CassKop 库中找到。首先要做的是添加存储库:

```
$ helm repo add casskop [https://Orange-OpenSource.github.io/cassandra-k8s-operator/helm](https://Orange-OpenSource.github.io/cassandra-k8s-operator/helm)
```

然后部署 CassKop:

```
$ helm install --name casskop casskop/cassandra-operator
NAME:   casskop
LAST DEPLOYED: Tue Jul 30 21:13:51 2019
NAMESPACE: default
STATUS: DEPLOYEDRESOURCES:
==> v1/Deployment
NAME                        READY  UP-TO-DATE  AVAILABLE  AGE
casskop-cassandra-operator  0/1    1           0          1s==> v1/Pod(related)
NAME                                         READY  STATUS             RESTARTS  AGE
casskop-cassandra-operator-79f77bb4c8-b4mf7  0/1    ContainerCreating  0         1s==> v1/RoleBinding
NAME                AGE
cassandra-operator  1s==> v1/ServiceAccount
NAME                SECRETS  AGE
cassandra-operator  1        1s==> v1beta1/Role
NAME                AGE
cassandra-operator  1sNOTES:
Congratulations. You have just deployed CassKop the Cassandra Operator.
Check its status by running:
kubectl --namespace default get pods -l "release=casskop"Visit [https://github.com/Orange-OpenSource/cassandra-k8s-operator](https://github.com/Orange-OpenSource/cassandra-k8s-operator) for instructions on hot to create & configure Cassandra clusters using the operator.
```

继续之前，请确保操作员已成功部署:

```
$ kubectl get pods -l name=cassandra-operator
NAME                                          READY   STATUS    RESTARTS   AGE
casskop-cassandra-operator-79f77bb4c8-b4mf7   1/1     Running   0          108s
```

# 部署简单集群

我们从最小的`CassandraCluster`开始，它没有定义任何数据中心或机架。

```
# simple-cluster.yamlapiVersion: "db.orange.com/v1alpha1"
kind: "CassandraCluster"
metadata:
  name: simple-cluster
  labels:
    cluster: simple-cluster
spec:
  **nodesPerRacks: 3**
  resources:
    requests:
      cpu: 400m
      memory: 1Gi
    limits:
      cpu: 400m
      memory: 1Gi
```

因为既没有定义数据中心也没有定义机架，`nodesPerRacks`实际上指定了集群的大小。

使用以下内容创建集群:

```
$ kubectl apply -f simple-cluster.yaml
```

最终我们会看到三个豆荚组成了卡珊德拉星团。让我们查询一下 pod:

```
$ kubectl get pods  -l cassandracluster=simple-cluster
NAME                         READY   STATUS    RESTARTS   AGE
simple-cluster-dc1-rack1-0   1/1     Running   0          12h
simple-cluster-dc1-rack1-1   1/1     Running   0          12h
simple-cluster-dc1-rack1-2   1/1     Running   0          12h
```

**注:** CassKop 增加了`cassandracluster`标签。

卡斯科普并没有直接创造豆荚。这些豆荚是由卡斯科普创建的一个声明集管理的。

```
$ kubectl get statefulset -l cassandracluster=simple-cluster
NAME                       DESIRED   CURRENT   AGE
simple-cluster-dc1-rack1    3         3        12h
```

状态集要求使用无头服务来负责 pod 的网络识别。让我们查询无头服务:

```
$ kubectl get svc -l cassandracluster=simple-cluster | awk {'print $1" " $2" "$3" "$4'} | column -t
NAME                          TYPE       CLUSTER-IP  EXTERNAL-IP
simple-cluster               ClusterIP  None        <none>
simple-cluster-exporter-jmx  ClusterIP  None        <none>
```

我们有我们的无头服务`simple-cluster`。另一个服务是`simple-cluster-exporter-jmx`，用于向普罗米修斯公开指标。

现在让我们用`nodetool`检查 Cassandra 集群的状态:

```
$ kubectl exec simple-cluster-dc1-rack1-0 -- nodetool status
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Rack
UN  10.0.2.18  69.94 KiB  256          69.0%             rack1
UN  10.0.5.6   87.38 KiB  256          65.0%             rack1
UN  10.0.3.7   104.86 KiB  256         65.9%             rack1
```

**注意:**为了提高可读性，已经从输出中删除了主机 ID 列。

所有三个 Cassandra 节点都处于`Up/Normal`状态。卡珊德拉集群此时已全面投入运行。通过最低限度的规范，我们能够启动并运行一个集群。

# 告密者

我想简单说说在`cassandra.yaml`中配置了`endpoint_snitch`属性的飞贼。Cassandra 开箱即用默认为`SimpleSnitch`。卡斯科普配置卡桑德拉节点使用`GossipingPropertyFileSnitch`。

# 多个机架

接下来，我们将看一个涉及多个机架的稍微复杂一点的示例:

```
# racks-cluster.yamlapiVersion: "db.orange.com/v1alpha1"
kind: "CassandraCluster"
metadata:
  name: racks-cluster
spec:
  nodesPerRacks: 2
  resources:
    requests:
      cpu: 400m
      memory: 1Gi
    limits:
      cpu: 400m
      memory: 1Gi
  **topology:
    dc:
      - name: dc1**
        **rack:
          - name: rack1
          - name: rack2
          - name: rack3**
```

这里我们第一次看到了`topology`属性。它声明了一个数据中心和三个机架。`topology`接受数据中心列表。每个数据中心接受一个机架列表。这个规格应该产生一个 Cassandra 集群，其中 6 个节点分布在三个机架上。

让我们查询一下 pod:

```
$ kubectl get pods -l cassandracluster=racks-cluster
NAME                        READY   STATUS    RESTARTS   AGE
racks-cluster-dc1-rack1-0   1/1     Running   0          18m
racks-cluster-dc1-rack1-1   1/1     Running   0          17m
racks-cluster-dc1-rack2-0   1/1     Running   0          15m
racks-cluster-dc1-rack2-1   1/1     Running   0          14m
racks-cluster-dc1-rack3-0   1/1     Running   0          13m
racks-cluster-dc1-rack3-1   1/1     Running   0          11m
```

我们如预期的有六个豆荚。我之前提到过，CassKop 创建了一个 StatefulSet 来管理 pod。让我们查询 StatefulSet:

```
$ kubectl get statefulset -l cassandracluster=racks-cluster
NAME                      DESIRED   CURRENT   AGE
racks-cluster-dc1-rack1   2         2         21m
racks-cluster-dc1-rack2   2         2         18m
racks-cluster-dc1-rack3   2         2         15m
```

这次我们有三个状态集。CassKop 为每个机架创建一个状态集。

让我们查询无头服务:

```
$ kubectl get svc -l cassandracluster=racks-cluster | awk {'print $1" " $2" "$3" "$4'} | column -t
NAME                          TYPE       CLUSTER-IP  EXTERNAL-IP
racks-cluster               ClusterIP  None        <none>
racks-cluster-exporter-jmx  ClusterIP  None        <none>
```

可以对多个状态集使用同一个无头服务。这正是卡斯科普所做的。

现在让我们用`nodetool`检查 Cassandra 集群的状态:

```
$ kubectl exec racks-cluster-dc1-rack1-0 -- nodetool status
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Rack
UN  10.0.2.19  87.34 KiB  256          30.5%             rack2
UN  10.0.4.5   92.34 KiB  256          32.8%             rack1
UN  10.0.0.6   81.63 KiB  256          31.9%             rack3
UN  10.0.5.7   69.94 KiB  256          36.0%             rack1
UN  10.0.3.8   87.38 KiB  256          33.9%             rack2
UN  10.0.1.8   69.92 KiB  256          35.0%             rack3
```

**注意:**为了提高可读性，已经从输出中删除了主机 ID 列。

所有六个 Cassandra 节点都处于`Up/Normal`状态，并如预期的那样分布在三个机架上。

# 多个数据中心

最后，我们将看看多 DC 部署。

```
# multidc-cluster.yamlapiVersion: "db.orange.com/v1alpha1"
kind: "CassandraCluster"
metadata:
  name: multidc-cluster
spec:
  nodesPerRacks: 2
  resources:
    requests:
      cpu: 400m
      memory: 1Gi
    limits:
      cpu: 400m
      memory: 1Gi
  topology:
    **dc:
      - name: dc1
        rack:
          - name: rack1
          - name: rack2
          - name: rack3
      - name: dc2
        nodesPerRacks: 3
        rack:
          - name: rack1
          - name: rack2**
```

注意，除了声明数据中心`dc1`和`dc2`之外，我们还为`dc2`设置了`nodesPerRacks`。CassKop 强制数据中心内每个机架的节点数量保持不变。每个机架的节点数量可能因数据中心而异。每个数据中心的机架数量也可能不同。这为配置集群拓扑提供了很大的灵活性。

这个规范应该产生一个包含 12 个 Cassandra 节点的 Cassandra 集群。让我们查询一下 pod:

```
$ kubectl get pods -l cassandracluster=multidc-cluster | awk {'print $1" " $2" "$3'} | column -t
NAME                         READY  STATUS
multidc-cluster-dc1-rack1-0  1/1    Running
multidc-cluster-dc1-rack1-1  1/1    Running
multidc-cluster-dc1-rack2-0  1/1    Running
multidc-cluster-dc1-rack2-1  1/1    Running
multidc-cluster-dc1-rack3-0  1/1    Running
multidc-cluster-dc1-rack3-1  1/1    Running
multidc-cluster-dc2-rack1-0  1/1    Running
multidc-cluster-dc2-rack1-1  1/1    Running
multidc-cluster-dc2-rack1-2  1/1    Running
multidc-cluster-dc2-rack2-0  1/1    Running
multidc-cluster-dc2-rack2-1  1/1    Running
multidc-cluster-dc2-rack2-2  1/1    Running
```

我们有 12 个豆荚。让我们查询状态集:

```
$ kubectl get statefulset -l cassandracluster=multidc-cluster
NAME                        DESIRED   CURRENT   AGE
multidc-cluster-dc1-rack1   2         2         8h
multidc-cluster-dc1-rack2   2         2         8h
multidc-cluster-dc1-rack3   2         2         7h58m
multidc-cluster-dc2-rack1   3         3         7h55m
multidc-cluster-dc2-rack2   3         3         7h51m
```

正如所料，我们有五个状态设置，每个机架一个。

现在让我们查询无头服务:

```
$ kubectl get svc -l cassandracluster=multidc-cluster | awk {'print $1" " $2" "$3" "$4'} | column -t
NAME                          TYPE       CLUSTER-IP  EXTERNAL-IP
multidc-cluster               ClusterIP  None        <none>
multidc-cluster-exporter-jmx  ClusterIP  None        <none>
```

希望现在很清楚，无论数据中心和机架的数量有多少，CassKop 只创建一组无头服务。

最后，让我们用`nodetool`检查 Cassandra 集群的状态:

```
$ kubectl exec multidc-cluster-dc1-rack1-0 -- nodetool status
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Rack
UN  10.0.2.2   200.27 KiB  256          14.3%            rack2
UN  10.0.4.6   299.47 KiB  256          14.4%            rack2
UN  10.0.0.7   314.06 KiB  256          15.5%            rack3
UN  10.0.5.9   294.07 KiB  256          15.0%            rack1
UN  10.0.1.9   270.8 KiB   256          16.3%            rack3
UN  10.0.3.10  289.84 KiB  256          15.6%            rack1
Datacenter: dc2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Rack
UN  10.0.2.22  327.84 KiB  256          15.9%            rack1
UN  10.0.4.7   233.92 KiB  256          14.9%            rack2
UN  10.0.0.8   254.63 KiB  256          14.8%            rack2
UN  10.0.5.10  243.91 KiB  256          15.7%            rack1
UN  10.0.1.10  254.97 KiB  256          15.5%            rack2
UN  10.0.3.11  309.45 KiB  256          15.9%            rack1
```

**注意:**为了提高可读性，已经从输出中删除了主机 ID 列。

我们有 12 个 Cassandra 节点分布在两个数据中心。`dc1`有六个节点分布在三个机架上。`dc2`有六个节点分布在两个机架上。一切都检查过了！

# 结论

CassKop 在配置 Kubernetes 中运行的 Cassandra 集群的拓扑时提供了很大的灵活性。数据中心是 Cassandra 的核心功能，CassKop 通过使其创建的每个集群都能够感知数据中心来充分利用该功能。
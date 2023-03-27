# Redis:在 Kubernetes 中运行主从复制

> 原文：<https://itnext.io/redis-running-master-slave-replication-in-kubernetes-327776a37918?source=collection_archive---------0----------------------->

![](img/23c6c90ae3811873cdac3b872c12308a.png)

任务是在 Kubernetes 集群中运行 Redis 实例。

将使用主从复制设置和 Sentinels 进行监控和故障转移操作。

查看 [Redis: replication，part 2-Master-Slave replication 和 Redis Sentinel](https://rtfm.co.ua/en/redis-replication-part-2-master-slave-replication-and-redis-sentinel/) 帖子了解更多详细信息。

# Redis 集群与 Redis 复制

参见 [Redis:复制，第 1 部分——概述。复制与分片。哨兵 vs 集群。Redis 拓扑。](https://rtfm.co.ua/en/redis-replication-part-1-overview-replication-vs-sharding-sentinel-vs-cluster-redis-topology/)和[在 Redis 掌舵图和 Redis 集群掌舵图](https://github.com/bitnami/charts/tree/master/bitnami/redis#choose-between-redis-helm-chart-and-redis-cluster-helm-chart)之间选择。

简而言之:

*   *副本* —包括一个 Redis 主实例，该实例执行读写操作，并将数据复制到它的 Redis 从实例，该实例提供只读操作。在此期间，如果主人失败，这样的奴隶可以提升到主人的角色。
*   *集群* —当你的 Redis 拥有的数据超过你服务器的 RAM 时，你会感觉到。集群可以使用[分片](https://rtfm.co.ua/redis-replikaciya-chast-1-obzor-replication-vs-sharding-sentinel-vs-cluster-topologiya-redis/#Sharding)，请求数据的客户端将被重定向到保存该数据的节点。

# 在 Kubernetes 运行 Redis 的方法

让我们看看如何执行这个任务——在 Kubernetes 集群中运行带有复制功能的 Redis。

*   手动设置—参见[如何在 Kubernetes 中创建主副本 Redis 集群](https://tech.ringieraxelspringer.com/blog/cloud/how-to-create-a-primary-replica-redis-cluster-in-kubernetes/r8lt028)
*   Redis 运算符— [redis 运算符](https://github.com/spotahome/redis-operator)
*   雷迪斯星团的掌舵图—[https://bitnami.com/stack/redis-cluster](https://bitnami.com/stack/redis-cluster)
*   带 Redis 主从复制的掌舵人—[https://bitnami.com/stack/redis](https://bitnami.com/stack/redis)(本帖我们的选择)

在我们目前的情况下，我们不需要担心数据的持久性，因为我们的 Redis 将仅用作缓存服务，所以我们不需要[Kubernetes persistent volume](https://rtfm.co.ua/kubernetes-persistentvolume-i-persistentvolumeclaim-obzor-i-primery/)。

# 舵图展开

首先，我们将从图表中运行 Redis 服务，简单查看一下，然后继续查看可用的[参数](https://github.com/bitnami/charts/tree/master/bitnami/redis#parameters)。

将 Bitnami 存储库添加到您的头盔中:

```
$ helm repo add bitnami [https://charts.bitnami.com/bitnami](https://charts.bitnami.com/bitnami)
“bitnami” has been added to your repositories
```

部署 Redis 图表:

```
$ helm install backend-redis bitnami/redis
NAME: backend-redis
LAST DEPLOYED: Tue Sep 22 14:48:02 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:** Please be patient while the chart is being deployed **Redis can be accessed via port 6379 on the following DNS names from within your cluster:backend-redis-master.default.svc.cluster.local for read/write operations
backend-redis-slave.default.svc.cluster.local for read-only operationsTo get your password run:export REDIS_PASSWORD=$(kubectl get secret — namespace default backend-redis -o jsonpath=”{.data.redis-password}” | base64 — decode)To connect to your Redis server:1\. Run a Redis pod that you can use as a client:kubectl run — namespace default backend-redis-client — rm — tty -i — restart=’Never’ \
 — env REDIS_PASSWORD=$REDIS_PASSWORD \
 — image docker.io/bitnami/redis:6.0.8-debian-10-r0 — bash2\. Connect using the Redis CLI:redis-cli -h backend-redis-master -a $REDIS_PASSWORD
redis-cli -h backend-redis-slave -a $REDIS_PASSWORDTo connect to your database from outside the cluster execute the following commands:kubectl port-forward — namespace default svc/backend-redis-master 6379:6379 &
redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD
```

获取图表部署期间生成的密码:

```
$ kubectl get secret — namespace default backend-redis -o jsonpath=”{.data.redis-password}” | base64 — decode
TySS43UhAW
```

运行`[port-forward](https://rtfm.co.ua/en/kubernetes-clusterip-vs-nodeport-vs-loadbalancer-services-and-ingress-an-overview-with-examples/#kubectl_portforward)`连接到 Redis 主实例:

```
$ kubectl port-forward — namespace default svc/backend-redis-master 6379:6379
Forwarding from [::1]:6379 -> 6379
Forwarding from 127.0.0.1:6379 -> 6379
```

连接:

```
$ redis-cli -h 127.0.0.1 -p 6379 -a TySS43UhAW
Warning: Using a password with ‘-a’ or ‘-u’ option on the command line interface may not be safe.
127.0.0.1:6379>
```

*“管用！”(с)*

查看我们这里有哪些服务:

```
$ kk get svc redis-service
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGEredis-service NodePort 172.20.119.242 <none> 6379:32445/TCP 117d
```

它的型号是`NodePort`，而我们需要的是`LoadBalancer`。

此外，我们需要 Redis Sentinel，它在默认情况下是关闭的:

> `sentinel.enabled`启用哨兵集装箱`false`

好了，让我们转到选项来启用 Sentinel 并配置一个负载平衡器。

## Redis 选项

创建一个对你有用的选项列表。

在我的情况下，这将是:

*   global.redis.password
*   指标:
*   指标.已启用
*   metrics.serviceMonitor.enabled
*   metrics . service monitor . namespace
*   坚持:
*   master .持久性.已启用
*   slave.persistence.enabled
*   服务
*   master.service.type
*   master.service .批注
*   slave.service.type
*   slave.service .批注
*   哨兵
*   sentinel .启用
*   sentinel.service.type —需要 LBso 客户端可以请求主/从
*   哨兵.服务.注释

创建一个新文件`~/Temp/redis-opts.yaml`来保存我们想要的参数:

```
global:
  redis:
    password: "blablacar"

metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    namespace: "monitoring"

master:
  persistence:
    enabled: false
  service:
    type: LoadBalancer
    annotations: 
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/scheme: internal 

slave:
  persistence:
    enabled: false
  service:
    type: LoadBalancer
    annotations:
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/scheme: internal

sentinel:
  enabled: true
  service:
    type: LoadBalancer
    annotations: 
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/scheme: internal
```

用`-f`更新部署以指定我们的参数文件:

```
$ helm upgrade — install backend-redis bitnami/redis -f ~/Temp/redis-opts.yaml
```

检查密码:

```
$ kubectl get secret — namespace default backend-redis -o jsonpath=”{.data.redis-password}” | base64 — decode
blablacar
```

和负载平衡器:

```
$ kk get svc -l app=redis
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGEbackend-redis LoadBalancer 172.20.204.235 af5e1294a4a73426692c7e25f7bb947d-915967.us-east-2.elb.amazonaws.com 
6379:30647/TCP,26379:32523/TCP 80sbackend-redis-headless ClusterIP None <none> 6379/TCP,26379/TCP 80sbackend-redis-metrics ClusterIP 172.20.66.2 <none> 9121/TCP 80s
```

连接:

```
$ redis-cli -h af5e1294a4a73426692c7e25f7bb947d-915967.us-east-2.elb.amazonaws.com -a blablacar
Warning: Using a password with ‘-a’ or ‘-u’ option on the command line interface may not be safe.\
af5e1294a4a73426692c7e25f7bb947d-915967.us-east-2.elb.amazonaws.com:6379>
```

有效，好吧。

但是为什么负载均衡器在被设置为内部*的时候是公共的呢？*

这是因为`alb.ingress.kubernetes.io/scheme: internal`用于 [ALB 入口控制器](https://rtfm.co.ua/en/aws-elastic-kubernetes-service-running-alb-ingress-controller/)，而图表创建了一个简单的带有`LoadBalancer`类型的 Kubernetes 服务，它将创建一个 AWS 经典负载平衡器。

阅读文档—[https://kubernetes . io/docs/concepts/services-networking/service/# load balancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)并更新我们的注释:指定“`alb.ingress.kubernetes.io/scheme: internal`”而不是“`service.beta.kubernetes.io/aws-load-balancer-internal: "true"`”:

```
...
master:
  service:
    type: LoadBalancer
    annotations: 
      service.beta.kubernetes.io/aws-load-balancer-internal: "true"

slave:
  service:
    type: LoadBalancer
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-internal: "true"

sentinel:
  enabled: true
  service:
    type: LoadBalancer
    annotations: 
      service.beta.kubernetes.io/aws-load-balancer-internal: "true"
```

更新部署并再次检查:

```
$ kk get svc -l app=redis
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGEbackend-redis LoadBalancer 172.20.178.72 internal-***-1786016015.us-east-2.elb.amazonaws.com 6379:31192/TCP,26379:32239/TCP 17s\backend-redis-headless ClusterIP None <none> 6379/TCP,26379/TCP 17sbackend-redis-metrics ClusterIP 172.20.35.163 <none> 9121/TCP 17s
```

现在，进行写操作—必须由 Redis 主实例提供服务:

```
$ admin@bttrm-dev-app-1:~$ redis-cli -h internal-***-1786016015.us-east-2.elb.amazonaws.com -p 6379 -a blablacar SET testkey testvalue
OK
```

和读操作:

```
admin@bttrm-dev-app-1:~$ redis-cli -h internal-***-1786016015.us-east-2.elb.amazonaws.com -p 6379 -a blablacar GET testkey
“testvalue”
```

复制状态:

```
admin@bttrm-dev-app-1:~$ redis-cli -h internal-***-1786016015.us-east-2.elb.amazonaws.com -p 6379 -a blablacar info replication
Replication
role:master
connected_slaves:1
slave0:ip=10.3.50.119,port=6379,state=online,offset=144165,lag=1
…
```

和哨兵状态:

```
admin@bttrm-dev-app-1:~$ redis-cli -h internal-***-1786016015.us-east-2.elb.amazonaws.com -p 26379 -a blablacar info sentinel
Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=10.3.33.107:6379,slaves=1,sentinels=2
```

虽然写操作需要在通过 Sentinels 获得主设备的地址之后执行:

```
admin@bttrm-dev-app-1:~$ redis-cli -h internal-***-1786016015.us-east-2.elb.amazonaws.com -p 26379 -a blablacar sentinel get-master-addr-by-name mymaster
1) “10.3.33.107”
r2) “6379”
```

检查[文档](https://github.com/bitnami/charts/tree/master/bitnami/redis/#master-slave-with-sentinel)。

完成了。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/redis-running-master-slave-replication-in-kubernetes/) *。*
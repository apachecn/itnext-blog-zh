# Cheat 林克德备忘单

> 原文：<https://itnext.io/cncf-linkerd-cheat-sheet-32cb206ec899?source=collection_archive---------4----------------------->

![](img/b9cb9176cc08d0e3d9dac78de37bc122.png)

> Kubernetes 超轻、安全第一的服务网。Linkerd 2.x 的主回购

*   [Github](https://github.com/linkerd/linkerd2)
*   [文件](https://linkerd.io/2.11/overview/)

# 装置

```
curl -fsL [https://run.linkerd.io/install](https://run.linkerd.io/install) | sh
```

## 设置集群

使用[类](https://kind.sigs.k8s.io/docs/user/quick-start/)集群

```
kind create cluster --name cncf-cheat-sheet
linkerd check --pre
linkerd install | kubectl apply -f -
linkerd check
```

## 演示应用程序

```
curl -fsL [https://run.linkerd.io/emojivoto.yml](https://run.linkerd.io/emojivoto.yml) | kubectl apply -f -
```

# 自动完成

## 尝试

```
echo "source <(linkerd completion bash) >> ~/.bashrc"
```

## Zsh

```
source <(linkerd completion zsh)
```

# 基础

## 注射

更新部署批注“linkerd.io/inject: enabled”。因此 linkerd 控制平面知道在 pod 创建期间插入一个侧柜。

```
kubectl get deploy -o yaml | linkerd inject - | kubectl apply -f -
```

# 交通操纵

使用[服务配置文件](https://linkerd.io/2.11/reference/service-profiles/) CRD 来管理和操作进入服务的流量。

## 分流流量

将去往“svc-a”的流量的 33%发送到“svc-b”。

```
cat <<EOF | kubectl apply -f -
apiVersion: split.smi-spec.io/v1alpha1
kind: TrafficSplit
metadata:
  name: traffic-split
spec:
  service: svc-a
  backends:
  - service: svc-a
    weight: 2
  - service: svc-b
    weight: 1
```

## 超时设定

将进入服务的所有请求的 HTTP 超时设置为 10 秒。

```
cat << EOF | kubectl apply -f -
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: svc-name.namespace.svc.cluster.local
  namepspace: default
spec:
  routes:
  - name: 'all'
    conditions:
    - pathRegex: /*
    timeout: 30s
```

## 重试次数

对于具有空主体的幂等路由

```
cat << EOF | kubectl apply -f -
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: svc-name.namespace.svc.cluster.local
  namepspace: default
spec:
  routes:
  - name: 'route-name'
    conditions:
    - pathRegex: /*
    isRetryable: true
```

重试预算

```
cat << EOF | kubectl apply -f -
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: svc-name.namespace.svc.cluster.local
  namepspace: default
spec:
  routes:
  - name: 'route-name'
    conditions:
    - pathRegex: /*
  retryBudget:
    retryRatio: 0.2
    minRetriesPerSecond: 10
    ttl: 10s
```

# 扩展ˌ扩张

## 即

Viz extension 将度量堆栈添加到集群中。

安装

```
linkerd viz install | kubectl apply -f -
```

运行仪表板 UI

```
linkerd viz dashboard
```
# Kubernetes 种类备忘单

> 原文：<https://itnext.io/kubernetes-kind-cheat-shee-2605da77984?source=collection_archive---------0----------------------->

![](img/ee10687986e14aa23cba1d059cbb08a4.png)

> [kind](https://sigs.k8s.io/kind) 是一个使用 Docker 容器“节点”运行本地 Kubernetes 集群的工具。
> kind 主要是为测试 Kubernetes 本身而设计的，但也可能用于本地开发或 CI。

# 自动完成

## 尝试

```
# Setup the current shell
source <(kind completion bash)########
## OR ##
######### Update permanently
echo “source <(kind completion bash) >> ~/.bashrc”
```

## ZSH

```
source <(kind completion zsh)
```

# 基础

创建一个名为 cncf-cheat-sheet 的集群

```
kind create cluster — name cncf-cheat-sheet
```

创建集群，并等待所有组件准备就绪

```
kind create cluster — wait 2m
```

获取正在运行的集群

```
kind get clusters
```

删除名为 cncf-备忘单的分类

```
kind delete cluster — name cncf-cheat-sheet
```

# 高级配置

对于更高级的用例，使用 kind.yaml 配置文件

## 港口

[Info](https://kind.sigs.k8s.io/docs/user/configuration/#extra-port-mappings)将 80 端口从集群控制平面映射到主机。

```
cat <<EOF | kind create cluster — name cncf-cheat-sheet — config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
 extraPortMappings:
 — containerPort: 80
 hostPort: 80
 protocol: TCP
EOF
```

## 挂载目录

[Info](https://kind.sigs.k8s.io/docs/user/configuration/#extra-mounts)
将当前目录挂载到位于`/app `的集群控制平面

*注意:MacOS 用户:确保在 docker-for-mac 偏好设置中共享资源*

```
cat <<EOF | kind create cluster — name cncf-cheat-sheet — config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
 extraMounts:
 — hostPath: .
 containerPath: /app
EOF
```

## 添加本地注册表

[信息](https://kind.sigs.k8s.io/docs/user/local-registry/)

步骤 1:创建本地注册表

```
docker run -d — restart=always -p 127.0.0.1:5000:5000 — name cncf-cheat-sheet-registry registry:2
```

步骤 2:创建集群

```
cat <<EOF | kind create cluster — name cncf-cheat-sheet — config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
 [plugins.”io.containerd.grpc.v1.cri”.registry.mirrors.”localhost:5000"]
 endpoint = [“[http://cncf-cheat-sheet-registry:5000](http://cncf-cheat-sheet-registry:5000)"]
nodes:
- role: control-plane
EOF
```

步骤 3:用创建的网络连接注册表

```
docker network connect kind cncf-cheat-sheet-registry
```

步骤 4:更新关于新注册表的集群

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
 name: local-registry-hosting
 namespace: kube-public
data:
 localRegistryHosting.v1: |
 host: “localhost:5000”
EOF
```

## 多个工人

[信息](https://kind.sigs.k8s.io/docs/user/configuration/#nodes)
默认配置将创建具有一个节点(控制平面)的集群。

```
cat <<EOF | kind create cluster — name cncf-cheat-sheet — config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
EOF
```
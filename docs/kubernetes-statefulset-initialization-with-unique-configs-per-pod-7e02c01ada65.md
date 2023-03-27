# Kubernetes 使用每个 Pod 的唯一配置进行状态集初始化

> 原文：<https://itnext.io/kubernetes-statefulset-initialization-with-unique-configs-per-pod-7e02c01ada65?source=collection_archive---------0----------------------->

如何为有状态应用程序的每个 pod 安装唯一的配置(例如，如何为主数据库 pod 和从数据库 pod 安装单独的配置)

![](img/da3358f7c659870e66fc85170076e362.png)

由[费伦茨·阿尔马西](https://unsplash.com/@flowforfrank?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

Kubernetes 中的 StatefulSets 用于管理需要以下一项或多项功能的有状态应用程序:

*   稳定、唯一的网络标识符
*   稳定、持久的存储
*   有序、优雅的部署和扩展
*   有序的自动滚动更新

实际上，StatefulSets 最常用于在 Kubernetes 上部署数据库(例如 MySQL、PostgreSQL、Redis、Elasticsearch)。Kubernetes 的声明性使得运行复制的有状态应用程序变得很容易。例如，如果您需要为一个数据库运行多个只读副本，可以使用 StatefulSets 为每个副本装载相同的配置。

但是，当您需要在 StatefulSet 集群中的每个 pod 上运行不同的配置时，会发生什么呢？一个常见的场景是当您需要区分主节点(读写)和从节点(只读)时。这可以通过为主节点和从节点创建单独的 StatefulSet 来解决。如果你需要一个更复杂的设置呢？可能状态集运行一些分布式分类帐网络，并且每个状态集具有不同的角色(例如以太坊的完整节点、轻型节点和归档节点)。这可以扩展到这样的场景，其中具有相同角色的不同状态集 pod 可能具有不同的许可方案或数据同步需求，这取决于网络拓扑(例如，pod-0 可能需要将其所有数据与外部数据库同步，而 pod-1 只能读取数据的子集并写入消息队列)。

目前，Kubernetes 没有提供一个简单的解决方案来支持上述用例。有一种观点认为，具有如此不同配置需求的每个 StatefulSet 应该被视为单独的部署，但是我希望有一种方法将具有不同角色的 stateful set 集群分组，并轻松地对它们进行扩展。

# 利用普通性

用唯一的配置初始化每个 StatefulSet pod 的一种变通方法是利用 initContainers 和 pod 的普通性。Kubernetes 为每个 StatefulSet pod 分配一个唯一的网络标识，该标识具有序号索引(从 0 到 N-1 个副本)、`$statefulsetName-$ordinal`形式的网络 ID 和稳定存储(即 PersistentVolume)。由此，我们可以从网络 ID 中提取每个 pod 的序号索引，并使用 initContainer 有选择地加载每个 pod 的相应配置。

这个想法源于 Kubernetes 运行一个复制的 MySQL 数据库的例子:

[](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/) [## 运行复制的有状态应用程序

### 本页显示如何使用 StatefulSet 控制器运行复制的有状态应用程序。这个应用程序是一个…

kubernetes.io](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/) 

在本例中，创建了一个配置映射来指定主 pod 和副本 pod 的配置:

```
**apiVersion**: v1
**kind**: ConfigMap
**metadata**:
  **name**: mysql
  **labels**:
    **app**: mysql
**data**:
  **primary.cnf**: | *# Apply this config only on the primary.
    [mysqld]
    log-bin*    
  **replica.cnf**: | *# Apply this config only on replicas.
    [mysqld]
    super-read-only*
```

然后在 StatefulSet 定义中，使用 initContainer 提取序号索引，如果是 pod-0，则复制到`primary.cnf`,如果不是，则使用`replica.cnf`:

```
**apiVersion**: apps/v1
**kind**: StatefulSet
**metadata**:
  **name**: mysql
**spec**:
  **selector**:
    **matchLabels**:
      **app**: mysql
  **serviceName**: mysql
  **replicas**: 3
  **template**:
    **metadata**:
      **labels**:
        **app**: mysql
    **spec**:
      **initContainers**:
      - **name**: init-mysql
        **image**: mysql:5.7
        **command**:
        - bash
        - "-c"
        - | *set -ex
          # Generate mysql server-id from pod ordinal index.
          [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          echo [mysqld] > /mnt/conf.d/server-id.cnf
          # Add an offset to avoid reserved server-id=0 value.
          echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
          # Copy appropriate conf.d files from config-map to emptyDir.
          if [[ $ordinal -eq 0 ]]; then
            cp /mnt/config-map/primary.cnf /mnt/conf.d/
          else
            cp /mnt/config-map/replica.cnf /mnt/conf.d/
          fi*          
        **volumeMounts**:
        - **name**: conf
          **mountPath**: /mnt/conf.d
        - **name**: config-map
          **mountPath**: /mnt/config-map
[...]
      **volumes**:
      - **name**: conf
        **emptyDir**: {}
      - **name**: config-map
        **configMap**:
          **name**: mysql
```

这里的核心思想是安装包含各种 pod 的所有配置的单个 ConfigMap，并基于序号索引，将每个 pod 的特定配置复制到主容器使用的 emptyDir 卷。

# GitOps 友好的方法

虽然上面的解决方案非常有效，但是有时将所有的配置组合到一个配置图中是不可行的。不同的团队可能负责维护不同的配置，或者您只是喜欢将配置组织到不同的目录中。

让我们来看一个具有以下结构的样本舵图表库:

```
my-awesome-chart
├── Chart.yaml
├── config
│   ├── node-0
│   │   ├── postgres.conf
│   │   └── httpSettings
│   └── node-1
│       ├── nodeid.dat
│       └── application.conf
├── templates
│   └── configmap.yaml
└── values.yaml
```

我们希望为每个目录生成一个配置映射:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: node-0
data:
  postgres.conf: ...
  httpSettings: ...

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: node-1
data:
  nodeid.dat: node-1
  application.conf: ...
```

为了遍历每个配置目录，我们需要使用 glob 模式`config/**`，并且每个目录只处理一次。为此，我们可以使用 dict 来跟踪已经创建的配置图:

```
{{- $processedDict := dict -}}
{{- range $path, $bytes := .Files.Glob "config/**" }}
{{- $name := base (dir $path) }}
{{- if not (hasKey $processedDict $name) -}}
{{ $_ := set $processedDict $name "true" }}
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ $name }}
data:
{{ ($.Files.Glob (printf "config/%s/*" $name)).AsConfig | indent 2 }}
---
{{- end }}
{{- end }}
```

然后，配置映射将产生两个配置映射，称为`node-0`和`node-1`。

现在，在我们的 initContainer 中，我们可以使用 range 函数挂载不同的配置图:

```
volumeMounts:
{{ range $i, $e := until (int .Values.replicaCount) }}
- name: node-{{ $i }}
  mountPath: /tmp/node-{{ $i }}
{{ end }}
```

然后，在 initContainer 中，我们可以使用序号索引来复制每个 pod 的所有配置:

```
SET_INDEX=${HOSTNAME##*-};
echo "Starting initializing for node $SET_INDEX";cp /tmp/node-${SET_INDEX}/* /network/conf;
```

如果您在掌舵图之外创建配置图，那么您也可以利用此模式来使用现有的配置图，并将它们加载到特定的 pod 上。

# 最后的想法

目前在 Kubernetes 上管理 StatefulSets 不像运行可以水平伸缩的无状态应用程序那么容易。然而，StatefulSet API 提供了强大的方法来利用普通性来配置您的 pod，以接收独特的配置并在运行时承担不同的角色。因为 Kubernetes 为状态集提供了有序、优雅的部署和扩展，所以这也可以用于留出具有较小索引的单元来初始化网络并促进顺序部署。对于高级用例，RollingUpdates 可以配置分区，以便只更新序号大于或等于分区的 pod，而不更新其他 pod。
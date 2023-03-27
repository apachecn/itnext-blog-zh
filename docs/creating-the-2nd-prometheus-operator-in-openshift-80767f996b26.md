# 在 OpenShift 中创建第二个普罗米修斯运算符

> 原文：<https://itnext.io/creating-the-2nd-prometheus-operator-in-openshift-80767f996b26?source=collection_archive---------2----------------------->

![](img/7beac2d88293256213e718e1c5d3629e.png)

维克多·哈纳切克摄影

OpenShift 4.x 提供开箱即用的 Prometheus 操作员监控。但是，这个操作符只限于某些特定的名称空间，专门用于集群监控。对于应用程序监控，需要单独的 Prometheus 操作员。必须考虑仔细的规划，以避免两个管理者(操作者)管理完全相同的资源的情况(CRD)。虽然我使用 OpenShift 作为集群，但是一般步骤适用于其他 Kubernetes 实现。

## 安装第二个操作器

OpenShift 监控操作员使用与 Prometheus 操作员完全相同的 CRDs，

```
oc get crd -o name| grep monitoring.coreos
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com
```

因此，对于第二个运算符，我们不能使用默认的 OpenShift 方式来安装带有 OLM 的运算符。我们必须跳过 CRD 安装，以避免任何覆盖。基于取自 prometheus-operator repo 的原始源代码， [bundle.yaml](https://github.com/coreos/prometheus-operator/blob/master/bundle.yaml) ，我们相应地修改了 operator 安装。

忽略集群角色。假设我们将在不同于 openshift-monitoring 的名称空间中安装操作符。创建服务帐户，并重命名集群角色绑定的名称，如下所示(*使用 golang 模板格式*)

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/name: prometheus-operator
    app.kubernetes.io/version: v0.39.0
  name: {{ .ns }}-prometheus-operator
  namespace: {{ .ns }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-operator
subjects:
- kind: ServiceAccount
  name: prometheus-operator
  namespace: {{ .ns }}
```

部署也如下所示进行了修改，

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/name: prometheus-operator
    app.kubernetes.io/version: v0.39.0
  name: prometheus-operator
  namespace: {{ .ns }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/name: prometheus-operator
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/name: prometheus-operator
        app.kubernetes.io/version: v0.39.0
    spec:
      containers:
      - args:
        - --kubelet-service=kube-system/kubelet
        - --logtostderr=true
        - --config-reloader-image=jimmidyson/configmap-reload:v0.3.0
        - --prometheus-config-reloader=quay.io/coreos/prometheus-config-reloader:v0.39.0
        # manage the following listed namespaces only , separated
        **- --namespaces={{ .listOfNamespaces }}
**        image: quay.io/coreos/prometheus-operator:v0.39.0
        name: prometheus-operator
        ports:
        - containerPort: 8080
          name: http
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        securityContext:
          allowPrivilegeEscalation: false
      nodeSelector:
        beta.kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
      serviceAccountName: prometheus-operator
```

对于容器的命令参数，我们添加了一个名称空间选项，这是一个逗号分隔的名称空间字符串。然后，操作符将只关注在列出的名称空间中定义的 CRD。这一点至关重要，否则所有 CRD，包括由 OpenShift 监控操作符创建的 CRD，都将由该操作符监视和管理，与默认的 OpenShift 操作符竞争。

## **创建 AlertManager 资源**

一旦部署了操作符，我们就可以创建 prometheus CRD 实例。但是由于我们的设置需要 AlertManager 的依赖性，我们首先创建 AlertManager 实例。

```
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  name: {{ .ns }}-alert-manager
  namespace: {{ .ns }}
spec:
  replicas: 1
```

警报管理器需要一个名为`alertmanager-{{ .nameOfAlertManager }}`的 K8s 秘密配置。定义以下内容另存为`alertmanager.yaml`

```
global:
  resolve_timeout: 5m
route:
  group_by: ['job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'webhook'
receivers:
- name: 'webhook'
  webhook_configs:
  - url: '[http://amwebhook.{{](http://amwebhook.{{) .ns }}.svc:8080/'
```

用固定的名称和文件名创建秘密。

```
kubectl -n {{ .ns }} create secret generic alertmanager-{{ .ns }}-alert-manager --from-file=alertmanager.yaml
```

现在，将创建 Alertmanager statefulSet。使用`alertmanager: name of the alert manager`选择器创建以下服务

```
apiVersion: v1
kind: Service
metadata:
  name: svc-alert-manager
  namespace: {{ .ns }}
spec:
  type: ClusterIP
  ports:
  - name: web
    port: 9093
    protocol: TCP
    targetPort: web
  selector:
    alertmanager: {{ .ns }}-alert-manager
```

## 创建普罗米修斯资源

为普罗米修斯创建以下 CRD 对象，

```
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  namespace: {{ .ns }}
  labels:
    prometheus: prometheus
spec:
  replicas: 1
  serviceAccountName: prometheus
 ** serviceMonitorSelector:**
    matchLabels:
      managedBy: my-prometheus
 **serviceMonitorNamespaceSelector:**     matchLabels:
       monitored-by: my-prometheus
  **ruleSelector:**
    matchLabels:
      managedBy: my-prometheus
  alerting:
    alertmanagers:
    - namespace: {{ .ns }}
      name: svc-alert-manager
      port: web
```

这将创建一个普罗米修斯状态集。请注意 Prometheus 将抓取的服务监视器资源，警报规则是用标签选择器定义的。现在，当且仅当满足以下所有条件时，这些类型的资源将由这个 Prometheus 实例管理。

1.  名称空间在被监视的操作员名称空间列表中。
2.  资源具有与定义为`serviceMonitorSelector`和`ruleSelector`相匹配的标签。
3.  如果未定义 serviceMonitorNamespaceSelector 和 ruleNamespaceSelector，则操作符将只在自己的名称空间中查找 serviceMonitors 和规则。如果要监视其他命名空间，必须在 serviceMonitorNamespaceSelector 或 ruleNamespaceSelector 中定义它

应用 yaml，观察普罗米修斯舱启动并运行。一旦准备就绪，我们就可以创建 ServiceMonitor 资源来收集指标。下面列出了一个示例服务监视器资源，

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: qmon-svc-mon
  namespace: {{ .ns }}
 **labels:
    managedBy: my-prometheus**
spec:
  selector:
    matchLabels:
      app: mq
  endpoints:
  - port: metrics
```

和一个警报规则示例，

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: monitoring-rules
  namespace: {{ .ns }}
 **labels:
    managedBy: my-prometheus**
spec:
  groups:
  - name: queueDepthHigh
    rules:
    - alert: QueueDepthTooHigh
      annotations:
        summary: Queue Depth is more than 100
      expr:
        mq_mon_queue_depth > 100
      for: 5m
      labels:
        severity: warning
```

第二艘普罗米修斯号已经准备好投入使用。这些步骤可以应用于多个 Prometheus 操作符和多租户集群。
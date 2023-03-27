# 使用来自不同命名空间的自定义指标进行水平窗格自动缩放

> 原文：<https://itnext.io/horizontal-pod-autoscaling-with-custom-metric-from-different-namespace-f19f8446143b?source=collection_archive---------4----------------------->

![](img/a73db0c1e9c81a4805b8754c5c9b9c46.png)

基于自定义指标的水平窗格自动缩放通常要求自定义指标在窗格运行的名称空间中可用。现在，假设客户指标出现在不同的名称空间中，我们如何利用该指标实现 HPA？

## 普罗米修斯适配器

首先，让我们安装 Prometheus 适配器来提供定制指标。给定以下值. yaml，

```
metricsRelistInterval: 1m
listenPort: 6443
prometheus:
  url: http://prometheus-operated.monitoring.svc
  port: 9090
rbac:
  create: trueserviceAccount:
  create: true
  name: prom-adaptor
rules:
  default: false
```

将图表安装在命名空间中，比如 monitoring，

```
helm install prom-adaptor stable/prometheus-adapter -f values.yaml -n monitoring
```

## 创建普罗米修斯服务监视器

使用 Prometheus 操作符，创建定制的 ServiceMonitor CRD 资源，以便可以监视 MQ 队列深度(参见我的[上一篇论文](https://medium.com/@zhimin.wen/programing-exec-into-a-pod-5f2a70bd93bb)了解更多细节)

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: qmon-svc-mon
  namespace: monitoring
  labels:
    app: abt
spec:
  selector:
    matchLabels:
      app: abt
  endpoints:
  - port: metrics
    metricRelabelings:
    - sourceLabels: [namespace]
      regex: '(.*)'
 **replacement: myapp
      targetLabel: target_namespace**
```

请注意，我们通过使用 metricRelabelings 添加了一个自定义标签“target_namespace”，其固定值为“myapp”。收集的指标将有一个额外的标签，如下所示，

`{…, target_namespace=“myapp”}`

## 自定义指标规则

现在，我们更新 configMap，命名为图表的发布名称，以提供我们的自定义指标规则。

```
kind: ConfigMap
apiVersion: v1
metadata:
  #release name of the chart
  name: prom-adaptor-prometheus-adapter
  namespace: monitoring
data:
  config.yaml: |
    rules:
    - seriesQuery: 'mq_mon_queue_depth' 
      resources:
        overrides:
 **target_namespace:
            resource: "namespace"**
          pod: 
            resource: "pod"
          service: 
            resource: "service"
      name:
        matches: "^mq_mon_queue_depth"
        as: "mq_queue_depth" 
      metricsQuery: 'mq_mon_queue_depth{queue_name="myqueue"}'
```

我们将度量重命名为 mq_quque_depth，并查询感兴趣的特定队列的度量。我们还覆盖了资源，名称空间将遵循我们定义为 target_namsepace 的自定义标签，该标签设置为“myapp”。我们将服务名映射到公开监控的服务，即“qmon-svc”。

然后，将在“myapp”的名称空间中为虚拟服务“qmon-svc”定义一个名为“mq_queue_depth”的度量。使用 kubectl 命令进行测试，

```
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/myapp/services/qmon-svc/mq_queue_depth
```

格式化的 JSON 是，

```
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {
    "selfLink": "/apis/custom.metrics.k8s.io/v1beta1/namespaces/myapp/services/qmon-svc/mq_queue_depth"
  },
  "items": [
    {
      "describedObject": {
        "kind": "Service",
        "namespace": "myapp",
        "name": "qmon-svc",
        "apiVersion": "/v1"
      },
      "metricName": "mq_queue_depth",
      "timestamp": "2020-06-21T08:35:58Z",
      "value": "8",
      "selector": null
    }
  ]
}
```

## HPA 资源

有了这个指标，我们可以用对象的类型来定义 HPA 资源。

```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-by-q-depth
  namespace: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deploy
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Object
    object:
      metric:
        name: mq_queue_depth
      describedObject:
        apiVersion: v1
        kind: Service
        name: qmon-svc
      target:
        type: Value
        averageValue: 200
```

我们预计，当队列深度较高时，pod 将自动缩放到目标值，即每个 pod 将平均分配和处理 200 个队列深度值。
# 带有自定义 Prometheus 指标的水平 Pod 自动缩放

> 原文：<https://itnext.io/horizontal-pod-autoscale-with-custom-metrics-8cb13e9d475?source=collection_archive---------1----------------------->

![](img/d1c3a2f894f673fee070438d852dfb3d.png)

*图片引自 Amazon.com*

在[的上一篇论文](https://medium.com/@zhimin.wen/custom-prometheus-metrics-for-apps-running-in-kubernetes-498d69ada7aa)中，我探讨了如何创建应用程序的定制指标，并在 Grafana 仪表板中呈现结果。在本文中，我将根据选定的自定义指标实现水平 Pod 自动缩放(HPA)目标。

这些步骤总结如下

1.  更新并安装 Prometheus-Adapter helm 图表，以便 HPA 使用自定义指标。
2.  定义和创建自定义指标
3.  创建 HPA 对象
4.  横向扩展测试。

我们开始吧。

## **1。**普罗米修斯适配器

[Prometheus 适配器](https://github.com/DirectXMan12/k8s-prometheus-adapter)通过实现[自定义指标](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/custom-metrics-api.md) API 来扩展 Kubernetes，这使得 HorizontalPodAutoscaler 控制器能够使用“custom . metrics . k8s . io”API 来检索指标。通过适配器的配置定义我们自己的指标，我们能够让 HPA 基于我们的自定义指标进行扩展。

我的 Kubernetes 环境是 IBM 云私有(ICP) 3.1.1。尽管在这个版本中安装了 Prometheus 适配器，但是创建定制指标的配置并没有公开。我们将根据[最新舵图](https://github.com/helm/charts/tree/master/stable/prometheus-adapter)安装适配器。

> 通过重新安装，一些重复的 Kubernets 对象将被覆盖。如果集群用于探索之外的其他目的，您需要确保这些默认 pod 指标没有被使用。

如本白皮书中[所述，ICP 中的 Prometheus 需要客户端认证身份验证，我们需要更新 helm 图表以允许客户端认证身份验证的配置。](https://medium.com/@zhimin.wen/https-client-certificate-authentication-with-sidecar-9b07d82a6389)

克隆存储库并在 values.yaml 文件中引入新变量。

```
prometheus:
  url: [http://prometheus.default.svc](http://prometheus.default.svc)
  port: 9090
 **clientAuthentication:
    use: true
    ca: |-
      # CA
    cert: |-
      #cert of client
    key: |-
      # key for the cert**
```

使用额外的命令参数更新部署 yaml 文件，以允许 Prometheus 身份验证。

```
#...
        - /adapter
        - --secure-port=6443
{{- if .Values.tls.enable }}
        - --tls-cert-file=/var/run/serving-cert/tls.crt
        - --tls-private-key-file=/var/run/serving-cert/tls.key
{{- end }}
        - --cert-dir=/tmp/cert
        - --logtostderr=true
        - --prometheus-url={{ .Values.prometheus.url }}:{{ .Values.prometheus.port }}
        - --metrics-relist-interval={{ .Values.metricsRelistInterval }}
        - --v={{ .Values.logLevel }}
        - --config=/etc/adapter/config.yaml
**{{- if .Values.prometheus.clientAuthentication.use }}
        - --prometheus-auth-config=/etc/prometheus/prometheus.kubecfg.yaml
{{- end }}**#...
```

对于用户来说,“prometheus-auth-config”利用 kubecfg 文件格式与 prometheus 服务器对话并不明显。(*有个* [*讨论*](https://github.com/DirectXMan12/k8s-prometheus-adapter/issues/144) *接着来解决这个*)。这里，我在图表中创建新的 configMap 文件，

```
{{- if .Values.prometheus.clientAuthentication.use -}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ template "k8s-prometheus-adapter.prometheus.kubecfg" . }}
  labels:
    app: {{ template "k8s-prometheus-adapter.name" . }}
    chart: {{ template "k8s-prometheus-adapter.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
data:
  prometheus.kubecfg.yaml: |-
    apiVersion: v1
    kind: Config
    preferences: {}
    clusters:
    - cluster:
        # insecure-skip-tls-verify: true
        certificate-authority-data: {{ b64enc .Values.prometheus.clientAuthentication.ca }}
        server: {{ .Values.prometheus.url }}
      name: prometheus-server
    users:
    - name: with-cert
      user:
        client-certificate-data: {{ b64enc .Values.prometheus.clientAuthentication.cert  }}
        client-key-data: {{ b64enc .Values.prometheus.clientAuthentication.key }}
    contexts:
    - context:
        cluster: prometheus-server
        user: with-cert
      name: promethues-server-connection
    current-context: promethues-server-connection
{{- end -}}
```

在配置图中，群集服务器指向由 prometheus.url 变量定义的 URL。它还定义了 CA 证书、客户端证书和客户端密钥。

CA 证书是 Prometheus 使用的 CA。查看 Kubernetes 对象定义，是“cluster-ca-cert”的秘密。检索 CA 证书及其私钥，

```
*# Save the following output as ca.pem
kubectl -n kube-system get secret cluster-ca-cert -o jsonpath="{ .data.tls\.crt }" | base64 -di*# Save the following output as ca.key
*kubectl -n kube-system get secret cluster-ca-cert -o jsonpath="{ .data.tls\.key }" | base64 -di*
```

使用此 CA 和 cfssl 工具创建客户端证书和密钥。(详见[论文](https://medium.com/@zhimin.wen/https-client-certificate-authentication-with-sidecar-9b07d82a6389))

```
*cfssl gencert -ca=ca.pem -ca-key=ca.key -config=ca-config.json -profile=client -hostname="" clientRequest.json | cfssljson -bare prometheus-client*
```

更新图表中的部署文件，以将此配置图作为供 Prometheus 使用的文件进行装载。

```
 volumeMounts:
        - mountPath: /etc/adapter/
          name: config
          readOnly: true
        - mountPath: /tmp
          name: tmp
{{- if .Values.tls.enable }}
        - mountPath: /var/run/serving-cert
          name: volume-serving-cert
          readOnly: true
{{- end }}
**{{- if .Values.prometheus.clientAuthentication.use }}
        - mountPath: /etc/prometheus
          name: prometheus-client-authentication
          readOnly: true
{{- end }}**# ...**{{- if .Values.prometheus.clientAuthentication.use }}
      - name: prometheus-client-authentication
        configMap:
          name: {{ template "k8s-prometheus-adapter.prometheus.kubecfg" . }}
{{- end }}**
```

创建一个 vars.yaml 文件，如下所示，

```
prometheus:
  url: [https://monitoring-prometheus.kube-system](https://monitoring-prometheus.kube-system)
  port: 9090
  clientAuthentication:
    use: true
    ca: |-
      -----BEGIN CERTIFICATE-----
      MIIFmzCCA4OgAwIBAgIJAMVsH80dT0MRMA0GCSqGSIb3DQEBCwUAMGMxCzAJBgNV
      ...
      -----END CERTIFICATE----- cert: |-
      -----BEGIN CERTIFICATE-----
      MIIEnjCCAoagAwIBAgIULumvDA5hcKj4JmGMsH8SN2CtN90wDQYJKoZIhvcNAQEN
      ...
      -----END CERTIFICATE----- key: |-
      -----BEGIN RSA PRIVATE KEY-----
      MIIEpAIBAAKCAQEAqJJw+u2NHy3CEQwZgE76ZzkleBmls+nMGY5qHdCI0XEF2oLH
      ...
      -----END RSA PRIVATE KEY-----replicas: 1
rbac:
  create: trueserviceAccount:
  create: true
  name:rules:
  default: true
  custom: []tls:
  enable: falsenodeSelector:
  management: "true"
tolerations:
  - key: dedicated
    value: infra
    effect: "NoSchedule"logLevel: 10
```

Prometheus URL 是“https ”,服务器名称是群集中 Prometheus 的服务名称，因为适配器 pod 将在同一个群集中运行。

复制 yaml 文件中的证书。

我将在专用管理节点上部署 pod，这就是为什么要应用 nodeSelectot 和 tolerations。

运行 helm 命令来安装图表。“Prometheus-adapter . chart . updated”是更新后的图表所在的目录。

```
helm install prometheus-adapter.chart.updated --tls --values var.yaml
```

验证 pod 正在运行。现在定制指标 API 可用了。通过运行的命令来验证它

```
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1 | jq .
```

您应该会看到适配器的默认指标列表。

## 2.创建自定义指标

为了简单起见，我将删除所有默认的自定义指标，只从我上一篇文章中的示例应用程序创建一个指标。

准备以下配置映射文件，替换{{。chart }}和{{。release }}包含 helm 创建的默认配置映射文件的内容。

```
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app: prometheus-adapter
    chart: {{ .chart }}
    heritage: Tiller
    release: {{ .release}}
  name: {{ .release }}-prometheus-adapter
  namespace: default
data:
  config.yaml: |
    rules:
 **- seriesQuery: '{__name__= "myapp_client_connected"}'
      seriesFilters: []
      resources:
        overrides:
          k8s_namespace:
            resource: namespace
          k8s_pod_name:
            resource: pod
      name:
        matches: "myapp_client_connected"
        as: ""
      metricsQuery: <<.Series>>{<<.LabelMatchers>>,container_name!="POD"}**
```

从 Prometheus 中选择名为“myapp_client_connected”的指标。给命名空间分配标签“k8s_namespace”，给 pod 分配标签“k8s _ pod _ name”*(参考普罗米修斯 relabel_config 的最后一篇论文，了解如何定义标签)*

使用相同的指标名称“myapp _ client _ connected”**，不使用任何正则表达式替换。最后是 metricQuery，它使用 PromQL 类型的查询来获取指标的值。**

**用此新配置图替换旧配置图。删除旧的 pod 以生成新的 pod，从而使配置更改生效。现在我们可以测试自定义指标 API 了。**

```
$ kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1  | jq
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "custom.metrics.k8s.io/v1beta1",
  "resources": [
    {
      "name": "namespaces/myapp_client_connected",
      "singularName": "",
      "namespaced": false,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "pods/myapp_client_connected",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    }
  ]
}
```

**并通过调用进一步列出该值**

```
$ kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/myapp_client_connected  | jq
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {
    "selfLink": "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/%2A/myapp_client_connected"
  },
  "items": [
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "hpa-sim-5cd9d7c47b-nd6c7",
        "apiVersion": "/v1"
      },
      "metricName": "myapp_client_connected",
      "timestamp": "2019-01-04T17:16:11Z",
      "value": "0"
    },
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "hpa-sim-5cd9d7c47b-nfc9h",
        "apiVersion": "/v1"
      },
      "metricName": "myapp_client_connected",
      "timestamp": "2019-01-04T17:16:11Z",
      "value": "0"
    },
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "hpa-sim-5cd9d7c47b-wlqqp",
        "apiVersion": "/v1"
      },
      "metricName": "myapp_client_connected",
      "timestamp": "2019-01-04T17:16:11Z",
      "value": "0"
    }
  ]
}
```

**请注意，所有三个 pod 的指标都已返回。**

## ****3。为部署创建 HPA****

**回想在上一篇论文中，我已经部署了“hpa-sim”。准备好自定义指标后，我们可以根据定义的指标自动扩展它。在这里，我只需选择活动会话的数量。HPA 定义如下:**

```
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-sim
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-sim
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: myapp_client_connected
      targetAverageValue: 20
```

**我们将名为“hpa-sim”的部署从 1 个副本扩展到 10 个副本。这些指标将基于 pod 类型的自定义指标。指标名称为“myapp_client_connected ”,测量基于所有相关 pod 的平均值。*(实际上只支持平均值目标)*。将 targetAverageValue 设置为 20。因此，当 HTTP 处理程序的平均并发会话数超过 20 时，就会触发 HPA。**

**应用配置，监控 HPA 开始收集度量值，**

```
k get hpa hpa-sim
NAME      REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-sim   Deployment/hpa-sim   0/20      1         10        1          3h
```

## **4.泵送负载并监控结垢**

**让我们用一些 Golang 函数使用 [magefile](https://github.com/magefile/mage) 任务来测试一下。**

```
func (Load) T01_PumpUniformLoad() {
 var wg sync.WaitGroup
 for index := 0; index < config.LoadCount; index++ {
  wg.Add(1)
  go func() {
   url := fmt.Sprintf("[http://192.168.64.244:30543/service?cost=%f](http://192.168.64.244:30543/service?cost=%f)", config.FixedLatency)
   _, err := http.Get(url)
   if err != nil {
    log.Printf("Request Failed")
   }
   wg.Done()
  }()
 } wg.Wait()
 log.Printf("ALL request are returned.")
}
```

**默认的 LoadConfig 设置为 100，FixedLatency 设置为 120。环境文件。运行法师任务，触发持续 120 秒的 100 并发连接。打开另一个 shell，在前 2 分钟加载完成之前运行相同的任务以保持加载。**

**使用上一篇文章中创建的 Grafana 仪表板进行监视，**

**![](img/47c2abf5c9985b1bc6e83c6debe13f9f.png)**

**我们可以清楚地看到，这些单元从一个单元扩大到多个单元，然后又缩小到一个单元。运行命令`kubectl get hpa hpa-sim -w`，下面显示了 pod 如何在峰值期间增长并在峰值后减少。**

**![](img/91916b89c1756890dd436e1081f341c7.png)**

## **结论**

**在 4 篇文章中，我研究了 Prometheus、使用客户端 API 的自定义指标、Grafana 的自定义指标和 HPA 的自定义指标，并对这些主题有了一些透彻的理解。希望你喜欢阅读，并和我一起学习一些东西。**
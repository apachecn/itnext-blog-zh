# Prometheus 配置，带有针对平台和应用级警报的定制警报标签

> 原文：<https://itnext.io/prometheus-configuration-with-custom-alert-labels-for-platform-and-application-level-alerts-4a356ed2488d?source=collection_archive---------1----------------------->

最近，我们(我和我的队友 [Markus](https://de.linkedin.com/in/markus-bukowski-a0728851) )与一位客户合作开发 Kubernetes 监控解决方案，我们必须满足以下要求:

1.  部署[普罗米修斯](https://prometheus.io)带着简单的生命循环的想法。我所说的简单生命周期是指用最少的努力升级 Prometheus 实例，并有可重复的部署
2.  准备好一套初始警报规则，以满足基本的平台和应用监控需求
3.  将警报转发给 [IBM 的 Tivoli Netcool/OMNIbus](https://www.ibm.com/support/knowledgecenter/SSSHTQ/landingpage/NetcoolOMNIbus.html) ,并附上唯一标识警报来源的标签

在这篇文章中，我们将谈论我们如何想出一个设计来满足上述要求。我们不会在这篇文章中涉及普罗米修斯的基础知识，网上有足够的资源让你自己学习基础知识。

本文提到的所有文件都可以在[这里](https://github.com/mcelep/blog/tree/master/prometheus-advanced)找到。

## 舵图:库贝-普罗米修斯-斯塔克

为了满足前两个要求，可以使用 [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/a3e6d68b12284f26d216250b57b2240cc9519224/charts/kube-prometheus-stack) 。这张舵图以前被命名为[普罗米修斯-操作员](https://github.com/helm/charts/tree/b9278fa98cef543f5473eb55160eaf45833bc74e/stable/prometheus-operator)。你可以在这里找到 kube-prometheus 的细节(包括 prometheus 操作符，以及其他有用的位)[。](https://github.com/prometheus-operator/kube-prometheus)

[Helm charts](https://helm.sh/docs/topics/charts/) 是目前在 Kubernetes 上部署软件最常见的方式之一，拥有一个软件的[操作员](https://kubernetes.io/docs/concepts/extend-kubernetes/operator)对于升级软件版本、备份/恢复和许多其他通常由软件管理员在第 2 天手动触发的操作也很方便。

## 普罗米修斯规则:)

kube-Prometheus-stackhelm chart 附带的另一个有趣的东西是一组规则，涵盖了 Kubernetes 平台的大多数常见需求。这里列出了规则和普罗米修斯规则，它们被简单地包装在一个[自定义资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)中，其类型如下:

```
apiVersion: monitoring.coreos.com/v1 
kind: PrometheusRule
```

人们应该记住，尽管这些规则从一开始就很好，但通常负责管理 K8S 平台的团队很可能会决定调整这些规则，添加新的规则，随着时间的推移，在他们认为合适的时候删除一些规则。

此处[列出的规则](https://github.com/prometheus-community/helm-charts/tree/a3e6d68b12284f26d216250b57b2240cc9519224/charts/kube-prometheus-stack/templates/prometheus/rules-1.14)涵盖了警报的两个主题。大多数规则都是为了监控 Kubernetes 平台本身，应该由平台的操作者来负责。但是，也有一些关于 K8s 上运行的工作负载的规则，例如 [kubernetes-apps.yaml](https://github.com/prometheus-community/helm-charts/blob/a3e6d68b12284f26d216250b57b2240cc9519224/charts/kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-apps.yaml) 。由平台事件触发的警报应该以不同于工作负载警报的方式发送，这就是为什么我们需要区分平台/系统和工作负载。换句话说，根据普罗米修斯规则，应该提醒不同的个人/团队。在本文的其余部分，我们将详细介绍 Prometheus 是如何配置的，以便始终包含一个带有正确值的自定义标签，即平台 ID 或工作负载 ID。

## 解决方案设计

下图展示了整体系统设计:

总体目标是区分平台级警报和源自集群上运行的工作负载的警报，即应用程序警报。负责操作 K8S 集群的团队通常不同于在这些集群上运行工作负载的团队。因此，为了确保警报可以被路由到正确的团队，我们需要一些标识符，可以在 Tivoli Netcool/OMNIbus 端使用，以进行正确的路由调用。我们不会详细讨论如何配置 Tivoli Netcool/OMNIbus 来将警报发送给不同的团队。

由于 Tivoli Netcool/OMNIbus 可以接收 http 调用，我们将使用 webhooks 从 [Prometheus/Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) 向 Tivoli Netcool/OMNIbus 发送警报。在 Tivoli 端进行正确的路由所需要的唯一东西是一个标签，我们将把它作为警报数据的一部分发送出去。该标签的键被确定为 example.com/ci_monitoring 的**。**

## **规则配置**

**[这个文件](./prometheus-operator/values.yml.j2)包括我们为 Prometheus 操作员部署设计的定制配置。这是一个 jinja 模板文件，原因是我们根据目标 K8S 集群替换了模板中的某些值，如集群名称、容器映像注册表主机名、警报发送目的地(电子邮件、webhook)。所有的舵图表都有一个默认的 *values.yml* 文件，我们的 jinja 模板是基于[这个文件](https://github.com/prometheus-community/helm-charts/blob/a3e6d68b12284f26d216250b57b2240cc9519224/charts/kube-prometheus-stack/values.yaml)。**

**Prometheus operator 附带了一组[默认启用]的警报规则(https://github . com/Prometheus-community/helm-charts/blob/564086 af 6157 D2 da 6a 9 a5 f 086010 f1 CB 3a 93 babd/charts/kube-Prometheus-stack/values . YAML # L30)并且由于我们需要覆盖[kuberneteapps 规则](https://github.com/prometheus-community/helm-charts/blob/a3e6d68b12284f26d216250b57b2240cc9519224/charts/kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-apps.yaml)，所以在 values 文件中包含了以下部分以禁止创建**kuberneteapps****

```
defaultRules:
  create: true
  rules:
    kubernetesApps: false
```

**之后，我们开始添加我们自己版本的 *kubernetesApps* 规则，下面是一个示例摘录:**

```
## Provide custom recording or alerting rules to be deployed into the cluster.
additionalPrometheusRules:
  - name: kubernetes-apps
    groups:
      - name: kubernetes-apps
        rules:
        - alert: KubePodCrashLooping-System
          annotations:
            message: Pod {{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
              }}) is restarting {{ printf "%.2f" $value }} times / 5 minutes.
            runbook_url: https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubepodcrashlooping
          expr: rate(kube_pod_container_status_restarts_total{job="kube-state-metrics",
            namespace=~"[% system_namespaces_regex %]"}[15m]) * 60 * 5 > 0
          for: 15m
          labels:
            severity: critical
            label_example_com_ci_monitoring: [% ci_cluster_id %]
        - alert: KubePodCrashLooping
          annotations:
            message: Pod {{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
              }}) is restarting {{ printf "%.2f" $value }} times / 5 minutes.
            runbook_url: https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubepodcrashlooping
          expr: rate(kube_pod_container_status_restarts_total{job="kube-state-metrics",
            namespace!~"[% system_namespaces_regex %]"}[15m]) * 60 * 5 > 0
          for: 15m
          labels:
            severity: critical
```

**规则基于最初的 [kubernetes-apps.yaml](https://github.com/helm/charts/blob/b9278fa98cef543f5473eb55160eaf45833bc74e/stable/prometheus-operator/templates/prometheus/rules-1.14/kubernetes-apps.yaml) ，我们为每个规则添加了两个版本。一个用于应用程序命名空间，一个用于系统命名空间。为了控制什么是系统名称空间，我们使用了一个 jinja 占位符:`[% system_namespaces_regex %]`，我们为这个占位符使用的值是:`default|kube-system|pks-system|monitoring`。(有关所有被覆盖的值，请参见“vars/test.yaml”下的文件)**

**我们为从 AlertManager 发送到 Tivoli 端的 Prometheus 警报添加标签，并确保与应用程序相关的警报查询总是包含该标签。在我们的配置中，这个标签叫做*label _ example _ com _ ci _ monitoring*。**

**对于与系统相关的警报，我们包括一个标签来表示集群 ID，它同样由 jinja 占位符`[% ci_cluster_id %]`控制。以下是相关部分:**

```
- alert: KubePodCrashLooping-System
          annotations:
            message: Pod {{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
              }}) is restarting {{ printf "%.2f" $value }} times / 5 minutes.
            runbook_url: https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubepodcrashlooping
          expr: rate(kube_pod_container_status_restarts_total{job="kube-state-metrics",
            namespace=~"[% system_namespaces_regex %]"}[15m]) * 60 * 5 > 0
          for: 15m
          labels:
            severity: critical
            label_example_com_ci_monitoring: [% ci_cluster_id %]
```

**对于工作负载警报，我们有一种基于集群和名称空间动态自动填充*label _ example _ com _ ci _ monitoring*的机制。此外，如果在带有关键字*label _ example _ com _ ci _ monitoring*的指标上有一个现有的标签，它将保持原样。在下一节中，我们将解释这种机制是如何工作的。**

## **附加警报重新标记**

**在 helm values 文件中，在路径为*prometheus . Prometheus spec . external labels*的部分中，我们引入了一些额外的警报重新标记，以便在由该 Prometheus 实例触发的每个警报上添加一些额外的标签。我们认为 K8S 集群 ID，由 jinja 占位符(`[% ci_cluster_id %]`)控制，在将来可以带来很多好处来表示警报的来源。该标签的关键字设置为*label _ example _ com _ ci _ cluster*，不与*label _ example _ com _ ci _ monitoring*混淆。此标签的目的是始终显示源 K8S 平台，无论它是系统还是命名空间/工作负载警报。相关配置如下所示:**

```
prometheusSpec:
    logLevel: info
    externalLabels:
      label_example_com_ci_cluster: [% ci_cluster_id %]
```

**path 下的以下部分:*Prometheus . Prometheus spec . additionalalertrelabelconfigs*控制我们如何控制从 AlertManager 发出的警报的[重新标记](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#alert_relabel_configs):**

```
additionalAlertRelabelConfigs:
    - source_labels:
        - label_example_com_ci_cluster
      action: replace
      regex: (.*)
      replacement: "[% ci_cluster_id %]"
      target_label: __tmp_monitoring
    - source_labels:
       - namespace
      regex: (.+)
      replacement: "NS-${1}-[% ci_cluster_id %]"
      target_label: __tmp_monitoring
      action: replace
    - source_labels:
        - label_example_com_ci_monitoring
      action: replace
      regex: (.+)
      target_label: __tmp_monitoring
    - source_labels:
        - __tmp_monitoring
      action: replace
      regex: (.*)
      replacement: "$1"
      target_label: label_example_com_ci_monitoring
```

**Prometheus 重新标记规则的处理顺序与它们在 yaml 文档中的顺序相同，这使我们能够:**

1.  **用来自`label_example_com_ci_cluster`的值填充名为 *__tmp_monitoring* 的临时变量。**
2.  **如果有一个带关键字`namespace`的标签，我们取它的值并用它来创建我们自己的表达式`"NS-${1}-[% ci_cluster_id %]"`。这里的`${1}`部分捕获名称空间值，第二部分`[% ci_cluster_id %]`是我们的 jinja 占位符。**
3.  **如果存在名为*label _ example _ com _ ci _ monitoring*的现有标签，则将该标签的值放入现有变量: *__tmp_monitoring***
4.  **最后，用临时标签值替换标签*标签 _ 示例 _ com _ ci _ 监控*。**

**换句话说，在上面的配置中，我们总是应用一个带有关键字的标签:*label _ example _ com _ ci _ monitoring*，这个关键字的值要么是关键字*label _ example _ com _ ci _ monitoring*的原始值，如果它存在，如果该标签不存在，我们使用一个模式基于名称空间和集群生成一个值:`NS-${1}-[% ci_cluster_id %]`如果有一个标签将*名称空间*作为其关键字。如果没有带有关键字:*名称空间*的标签，那么我们回退到 *ci_cluster_id* 。**

**在这种情况下，最好提醒您，将数据从 Kubernetes 抓取到 Prometheus 时会发生一个转换过程。在这个转换过程中，像**example.com/ci_monitoring**这样的关键字会在 Prometheus 中被翻译成一个名为*label _ example _ com _ ci _ monitoring*的标签。**

## **持续**

**默认情况下，Prometheus 操作员舵图带有禁用的永久卷。为了启用和配置持久卷，使用了以下部分:**

```
volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi
```

**此部分可在路径:*Prometheus . prometheusspec . storage spec*下找到。我们尚未指定存储类，并假设为群集配置了默认的存储类。或者，在*规范*部分，您可以指定一个存储类，如下所示:**

```
volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          storageClassName: XYZ
          resources:
            requests:
              storage: 50Gi
```

## **警报管理器配置**

**以下节选显示了如何配置 Prometheus Alertmanager 转发警报:**

```
alertmanager:
  enabled: true

  config:
    global: {}
    receivers:
    - name: default-receiver
      webhook_configs:
      - url: "[% alertmanager_webhook_url %]"
        send_resolved: true
      email_configs:
      - send_resolved: true
        to: [% alertmanager_smtp_to %]
        from: [% alertmanager_smtp_from %]
        smarthost: [% alertmanager_smtp_host %]
        require_tls: false
    - name: 'null'
    route:
      group_by: ['job']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 24h
      receiver: default-receiver
```

**上一节中总共配置了 2 个接收器。第一个叫做*默认接收者*，这个有 **web_hook** 和 **email** 动作，当一个警报被触发时。Webhook 集成允许您向任何支持 http 的端点发送警报，比如 IBM Tivoli Netcool/OMNIbus。有关配置 webhook 监听器的详细信息，请参见此处的[和](https://prometheus.io/docs/alerting/latest/configuration/#webhook_config)。在*默认接收者*部分，电子邮件是任何点的第二个接收者，如果您可以访问使用 smtp 协议的电子邮件服务器，这可以是一种快速的测试方式。在测试过程中，如果您想避免向第三方发送警报，您可以使用`null`接收器。**

## **死亡开关**

**如果 prometheus 部署在您想要观察的 K8S 平台上，您可能收不到任何警报，因为如果平台运行不正常或根本不运行，Prometheus 可能无法运行。为了解决这个问题，一种常见的模式是让 Prometheus 定期向在不同故障域中运行的另一个监控平台发送警报(*心跳*，因为它在其他一些域中被引用)。这种模式通常被称为‘安全开关’,它由[一般规则 yaml](https://github.com/prometheus-community/helm-charts/blob/b643f58714a063e5a0ddf6832cd00ed29c0f4fab/charts/kube-prometheus-stack/templates/prometheus/rules-1.14/general.rules.yaml) 中的以下部分控制:**

```
- alert: Watchdog
      annotations:
        message: 'This is an alert meant to ensure that the entire alerting pipeline is functional.
          This alert is always firing, therefore it should always be firing in Alertmanager
          and always fire against a receiver. There are integrations with various notification
          mechanisms that send a notification when this alert is not firing. For example the
          "DeadMansSnitch" integration in PagerDuty.
          '
      expr: vector(1)
      labels:
        severity: none
```

## **行动解决方案**

## **所需工具**

*   **Kubernetes CLI**
*   **舵> = 3.2**
*   **可译剧本**
*   **K8S 星团**

## **安装 Nginx 入口控制器**

**运行以下命令安装 nginx-ingress:**

```
ansible-playbook -v playbooks/nginx-ingress.yml -i inventory-local.yaml --extra-vars @vars/test.yml
```

**这个剧本将与您执行`ansible-playbook`二进制文件的本地主机对话，它将:**

*   **添加必要的舵 repos**
*   **更新舵 repos**
*   **安装 Nginx 入口控制器，我们稍后将在 Prometheus 安装中使用。**

## **安装普罗米修斯**

**运行以下命令安装并配置 Prometheus 及其依赖项:**

```
ansible-playbook -v playbooks/prometheus-operator.yml -i inventory-local.yaml --extra-vars @vars/test.yml
```

## **创建 webhook 列表器**

**在本节中，我们将运行一个简单的 webhook 侦听器，它将接收到的 http 调用的详细信息(包括有效负载)打印到 stdout 上。以下命令将运行包含 webhook 侦听器程序的映像。**

```
kubectl -n monitoring create deployment webhook --image=mcelep/python-webhook-listener
kubectl expose deployment/webhook --port 8080
```

## **创建测试应用程序**

**运行以下命令创建一个名为 *test* 的名称空间，并创建一个名为 *test* 的部署**

```
kubectl create ns test
kubectl -n test create deployment test --image=no-such-image
```

**很快您应该会有一些处于挂起状态的警报，并且在配置的时间量之后(对于 *KubePodNotReady* ，在 *values.yml.j2* 文件中设置为 15 分钟)应该会有带有正确标签的警报触发。**

**当我们检查 webhook 侦听器容器的日志时，我们看到收到了以下数据:**

```
{
  "status": "firing",
  "labels": {
    "__tmp_monitoring": "NS-test-CL-XYZ",
    "alertname": "KubePodNotReady",
    "label_example_com_ci_cluster": "CL-XYZ",
    "label_example_com_ci_monitoring": "NS-test-CL-XYZ",
    "namespace": "test",
    "pod": "test-7c8844b6fb-kbj8l",
    "prometheus": "monitoring/platform-kube-prometheus-s-prometheus",
    "severity": "critical"
  },
  "annotations": {
    "message": "Pod test/test-7c8844b6fb-kbj8l has been in a non-ready state for longer than 15 minutes.",
    "runbook_url": "https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubepodnotready"
  },
  "startsAt": "2020-10-29T10:49:18.473Z",
  "endsAt": "0001-01-01T00:00:00Z",
  "generatorURL": "http://prometheus.monitoring.k8s.example.com/graph?g0.expr=sum+by%28namespace%2C+pod%2C+label_example_com_ci_monitoring%29+%28max+by%28namespace%2C+pod%2C+label_example_com_ci_monitoring%29+%28kube_pod_status_phase%7Bjob%3D%22kube-state-metrics%22%2Cnamespace%21~%22default%7Ckube-system%7Cpks-system%7Cmonitoring%22%2Cphase%3D~%22Pending%7CUnknown%22%7D%29+%2A+on%28namespace%2C+pod%29+group_left%28owner_kind%29+max+by%28namespace%2C+pod%2C+owner_kind%2C+label_example_com_ci_monitoring%29+%28kube_pod_owner%7Bowner_kind%21%3D%22Job%22%7D%29%29+%3E+0\\u0026g0.tab=1",
  "fingerprint": "0b23d99b9740b21e"
}
```

**名称空间*测试*，没有带关键字**example.com/ci_monitoring**的标签，因此标签*label _ example _ com _ ci _ monitoring*是基于我们的重新标记配置自动生成的，因此得到了集群 id 和名称空间的组合值。就是这样！我们的端到端解决方案正在发挥作用。**
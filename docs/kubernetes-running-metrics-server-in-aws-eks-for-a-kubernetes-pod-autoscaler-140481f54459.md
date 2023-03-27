# Kubernetes:在 EKS AWS 中为 Kubernetes Pod 自动缩放器运行 metrics-server

> 原文：<https://itnext.io/kubernetes-running-metrics-server-in-aws-eks-for-a-kubernetes-pod-autoscaler-140481f54459?source=collection_archive---------6----------------------->

![](img/b507bdfa1910ba0d8b988a12cd2d2352.png)

假设我们已经有了一个带有工作节点的 AWS EKS 集群。

在本文中，我们将连接到一个新创建的集群，使用 HPA 创建一个测试部署—[Kubernetes Horizontal Pod auto scaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)，并尝试使用`[kubectl top](https://www.mankier.com/1/kubectl-top)`获取有关资源使用情况的信息。

## 库伯内特星团

使用`[eksctl](https://eksctl.io/)`创建一个测试集群:

```
$ eksctl create cluster — profile arseniy — region us-east-2 — name eks-dev-1
…
[ℹ] node “ip-192–168–54–141.us-east-2.compute.internal” is ready
[ℹ] node “ip-192–168–85–24.us-east-2.compute.internal” is ready
[ℹ] kubectl command should work with “/home/setevoy/.kube/config”, try ‘kubectl get nodes’
[✔] EKS cluster “eks-dev-1” in “us-east-2” region is ready
```

然后切换到它。

## Kubernetes 集群上下文

配置您的`kubectl`:

```
$ aws eks — profile arseniy — region us-east-2 update-kubeconfig — name eks-dev-1
Added new context arn:aws:eks:us-east-2:534***385:cluster/eks-dev-1 to /home/setevoy/.kube/config
```

在*美国东部-2* 地区查看您的 AWS 帐户中可用的 EKS 集群:

```
$ aws eks — profile arseniy — region us-east-2 list-clusters — output text
CLUSTERS eksctl-bttrm-eks-production-1
CLUSTERS mobilebackend-dev-eks-0-cluster
CLUSTERS eks-dev-1
```

并检查当前使用的配置文件:

```
$ kubectl config current-context
arn:aws:eks:us-east-2:534***385:cluster/eks-dev-1
```

`aws eks`已经为我们配置了`kubectl`的最后一个集群。

如果需要，您可以使用`get-contexts`随时获取所有可用的配置文件:

```
$ kubectl config get-contexts
```

并切换到一个必要的:

```
$ kubectl config use-context arn:aws:eks:us-east-2:534***385:cluster/eks-dev-1
Switched to context “arn:aws:eks:us-east-2:534***385:cluster/eks-dev-1”.
```

## 部署

出于测试目的，让我们创建一个带有自动缩放器的部署。

HPA 将使用从`mertics-server`收集的指标来获取关于节点和单元上的资源使用情况的数据，以了解何时必须扩大或缩小特定部署的单元。

创建 HPA 和部署:

```
---
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hello-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hello
  minReplicas: 1
  maxReplicas: 2
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello
          image: gcr.io/google-samples/node-hello:1.0
          resources:
            limits:
              cpu: "0.1"
            requests:
              cpu: "0.1"
```

应用它们:

```
$ kubectl apply -f example-deployment.yml
horizontalpodautoscaler.autoscaling/hello-hpa created
deployment.apps/hello created
```

检查:

```
$ kubectl get hpa
NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
hello-hpa Deployment/hello <unknown>/80% 1 2 1 26s
```

## HPA —目标<targets>无法获取指标</targets>

如果您现在检查 HPA，您会发现它无法收集有关其目标(节点和单元)的数据:

```
$ kubectl get hpa
NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
hello-hpa Deployment/hello <unknown>/80% 1 2 1 26s
```

在*条件*和*事件*中，您可以找到关于该问题的更多详细信息:

```
$ kubectl describe hpa hello-hpa
…
Conditions:
Type Status Reason Message
 — — — — — — — — — — — -AbleToScale True SucceededGetScale the HPA controller was able to get the target’s current scaleScalingActive False FailedGetResourceMetric the HPA was unable to compute the replica count: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
…
Events:Type Reason Age From Message
 — — — — — — — — — — — — -
Warning FailedGetResourceMetric 12s (x3 over 43s) horizontal-pod-autoscaler unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)Warning FailedComputeMetricsReplicas 12s (x3 over 43s) horizontal-pod-autoscaler failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
```

事实上，我们的目标是:

> 无法获取 cpu 利用率:无法获取资源 cpu 的度量

## 服务器出错(未找到):服务器找不到请求的资源(获取服务 http:heapster:)

另外，如果现在尝试对节点和 pod 使用`top`——Kubernetes 将抛出另一个错误:

```
$ kubectl top node
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)$ kubectl top pod
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)
```

为了使`top`能够显示它尝试连接到[堆](https://github.com/kubernetes-retired/heapster)服务时的资源使用情况，请参见[源代码](https://github.com/kubernetes/kubectl/blob/master/pkg/cmd/top/top_pod.go#L143):

```
...
o.Client = metricsutil.NewHeapsterMetricsClient(clientset.CoreV1(), o.HeapsterOptions.Namespace, o.HeapsterOptions.Scheme, o.HeapsterOptions.Service, o.HeapsterOptions.Port)
...
```

但是 Heaspter 是不赞成使用的服务，以前用于收集指标。

如今，对于 CPU 和内存指标，使用的是`[metrics-server](https://github.com/kubernetes-incubator/metrics-server)`服务。

## 运行指标-服务器

克隆存储库:

```
$ git clone [https://github.com/kubernetes-sigs/metrics-server.git](https://github.com/kubernetes-sigs/metrics-server.git)
$ cd metrics-server/
```

## 度量-AWS EKS 的服务器配置

要使 metrics-server 能够找到 AWS Elastic Kubernetes 服务集群中的所有资源，请编辑其部署文件`deploy/kubernetes/metrics-server-deployment.yaml`，并添加带有四个参数的`command`:

```
...
        command:
          - /metrics-server
          - --logtostderr
          - --kubelet-insecure-tls=true
          - --kubelet-preferred-address-types=InternalIP
          - --v=2
...
```

*   `kubelet-insecure-tls` -不检查节点上的 kubelet-clients CA 证书
*   `kubelet-preferred-address-types` -如何在 Kubernetes 空间中找到资源-通过使用*主机名*、*内部域名*、*内部域名*、*外部域名*或*外部域名*，对于 EKS 将其设置为*内部域名*值
*   `v=2` -日志详细程度

保存并部署它:

```
$ kubectl apply -f deploy/kubernetes/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```

检查:

```
$ kubectl -n kube-system get pod
NAME READY STATUS RESTARTS AGE
aws-node-mt9pq 1/1 Running 0 2m5s
aws-node-rl7t2 1/1 Running 0 2m2s
coredns-74dd858ddc-xmrhj 1/1 Running 0 7m33s
coredns-74dd858ddc-xpcwx 1/1 Running 0 7m33s
kube-proxy-b85rv 1/1 Running 0 2m5s
kube-proxy-n647l 1/1 Running 0 2m2s
metrics-server-546565fdc9–56xwl 1/1 Running 0 6s
```

或者通过使用:

```
$ kubectl get apiservices | grep metr
v1beta1.metrics.k8s.io kube-system/metrics-server True 91s
```

服务日志:

```
kubectl -n kube-system logs -f metrics-server-546565fdc9-qswck
I0215 11:59:05.896014 1 serving.go:312] Generated self-signed cert (/tmp/apiserver.crt, /tmp/apiserver.key)
I0215 11:59:06.439725 1 manager.go:95] Scraping metrics from 0 sources
I0215 11:59:06.439749 1 manager.go:148] ScrapeMetrics: time: 2.728µs, nodes: 0, pods: 0
I0215 11:59:06.450735 1 secure_serving.go:116] Serving securely on [::]:4443
E0215 11:59:10.096632 1 reststorage.go:160] unable to fetch pod metrics for pod default/hello-7d6c85c755-r88xn: no metrics known for pod
E0215 11:59:25.109059 1 reststorage.go:160] unable to fetch pod metrics for pod default/hello-7d6c85c755-r88xn: no metrics known for pod
```

尝试`top`获取节点:

```
$ kubectl top node
error: metrics not available yet
```

对于吊舱:

```
$ kubectl top pod
W0215 13:59:58.319317 4014051 top_pod.go:259] Metrics not available for pod default/hello-7d6c85c755-r88xn, age: 4m51.319306547s
error: Metrics not available for pod default/hello-7d6c85c755-r88xn, age: 4m51.319306547s
```

1-2 分钟后，再次检查`metrics-server`日志:

```
I0215 12:00:06.439839 1 manager.go:95] Scraping metrics from 2 sources
I0215 12:00:06.447003 1 manager.go:120] Querying source: kubelet_summary:ip-192–168–54–141.us-east-2.compute.internal
I0215 12:00:06.450994 1 manager.go:120] Querying source: kubelet_summary:ip-192–168–85–24.us-east-2.compute.internal
I0215 12:00:06.480781 1 manager.go:148] ScrapeMetrics: time: 40.886465ms, nodes: 2, pods: 8
I0215 12:01:06.439817 1 manager.go:95] Scraping metrics from 2 sources
```

并再次尝试`top`:

```
$ kubectl top node
NAME CPU(cores) CPU% MEMORY(bytes) MEMORY%
ip-192–168–54–141.us-east-2.compute.internal 25m 1% 406Mi 5%
ip-192–168–85–24.us-east-2.compute.internal 26m 1% 358Mi 4%
```

豆荚:

```
$ kubectl top pod
NAME CPU(cores) MEMORY(bytes)
hello-7d6c85c755-r88xn 0m 8Mi
```

以及 HPA 服务:

```
$ kubectl describe hpa hello-hpa
…
Conditions:
Type Status Reason Message
 — — — — — — — — — — — -AbleToScale True ReadyForNewScale recommended size matches current sizeScalingActive True ValidMetricFound the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
ScalingLimited True TooFewReplicas the desired replica count is more than the maximum replica countEvents:
Type Reason Age From Message
 — — — — — — — — — — — — -
Warning FailedComputeMetricsReplicas 4m23s (x12 over 7m10s) horizontal-pod-autoscaler failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)Warning FailedGetResourceMetric 4m8s (x13 over 7m10s) horizontal-pod-autoscaler unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
```

请注意以下信息:

*   *HPA 能够根据 cpu 资源利用率成功计算副本数量* — HPA 现在能够收集指标
*   *无法获取资源 cpu 的指标* —检查*年龄*，—其计数器必须停止增长(保持[13]的数字)

实际上，运行一个`metrics-server`来使用 HPA 就需要这些了。

下图—使用 HPA 和`metrics-server`时的一个常见错误。

## HPA 无法计算副本数:缺少对 cpu 的请求

有时，HPA 会报告以下错误:

```
$ kubectl describe hpa hello-hpa
…
Conditions:
Type Status Reason Message
 — — — — — — — — — — — -
AbleToScale True SucceededGetScale the HPA controller was able to get the target’s current scaleScalingActive False FailedGetResourceMetric the HPA was unable to compute the replica count: missing request for cpu
…Warning FailedGetResourceMetric 2s (x2 over 17s) horizontal-pod-autoscaler missing request for cpu
```

如果部署中的 pod 模板没有定义`resources`和`[requests](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container)`，可能会发生这种情况。

在当前情况下，我注释掉了这些行:

```
...
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello
          image: gcr.io/google-samples/node-hello:1.0
#          resources:
#            limits:
#              cpu: "0.1"
#            requests:
#              cpu: "0.1"
```

并且在没有`[requests](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container)`的情况下部署了 pod，这导致了“ ***缺少对 cpu*** 的请求”错误消息。

将`[requests](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container)`拨回——它现在肯定可以工作了。

完成了。

*最初发布于* [*RTFM: Linux、DevOps 和系统管理*](https://rtfm.co.ua/en/kubernetes-running-metrics-server-in-aws-eks-for-a-kubernetes-pod-autoscaler/) *。*
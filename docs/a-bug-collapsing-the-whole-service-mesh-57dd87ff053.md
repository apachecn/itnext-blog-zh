# 一个破坏整个服务网的错误

> 原文：<https://itnext.io/a-bug-collapsing-the-whole-service-mesh-57dd87ff053?source=collection_archive---------1----------------------->

![](img/5f44462a049f8d54ec24baa87e4eb58b.png)

我正准备用 IBM CloudPak 演示 Knative 和 Istio 的特性，以便在 open shift Container Platform(OCP)4.2 上应用。一切都很顺利，Knative 服务能够缩小到零个 pod，并在流量到来时自动激活，直到我打开跟踪功能并使用 Kiali 可视化流量。

# 1.打开 Kiali

要启用 Kiali，请修改名为 basic-install 的 CRD ServiceMeshControlPlane，以启用 Kiali。要启用跟踪，必须使用特使边车来捕获跟踪和监控数据。让我们启用 sidecarInjectorWebhook，这样 sidecar 就可以在创建时注入到 Pod 中。

```
spec:
  istio:
    kiali:
      enabled: true
    tracing:
      enabled: true
    global:
      defaultPodDisruptionBudget:
        enabled: false
      disablePolicyChecks: false
      multitenant: true
      omitSidecarInjectorConfigMap: false
      proxy:
        autoInject: disabled
    grafana:
      enabled: true
    sidecarInjectorWebhook:
      enabled: true
    mixer:
      enabled: true
      policy:
        enabled: false
      telemetry:
        enabled: true
    prometheus:
      enabled: true
```

请注意，遥测和普罗米修斯启用了监测指标。

在改变之后，操作员魔术开始工作，Kiali UI 正在工作。webhook 开始将 sidecar 注入到加入服务网格的名称空间的 Pod 中。一切似乎都很好。

然而…

# **2。Knative activator 进入崩溃循环**

由于 knative-serving 名称空间是服务网格的成员，并且操作者 OpenShift 无服务器操作者协调 Knative activator 的部署以具有以下注释，

```
sidecar.istio.io/inject: 'true'
```

activator pod 将注入 istio 边车，现在 activator pod 无法正常工作，并在初始启动阶段失败，出现以下错误。

```
Error loading/parsing logging configuration:Get https://172.30.0.1:443/api/v1/namespaces/knative-serving/configmaps/config-logging: http: server gave HTTP response to HTTPS client
```

当激活器失败时，它作为下面的功能失败，Knative 服务将不再工作。

1.  *接收&不活动修订的缓冲请求。*
2.  *向自动缩放器报告指标。*
3.  *在自动缩放器根据报告的指标缩放修订后，重试对修订的请求。*

# **3。模拟问题**

让我们试着重复这个问题。在同一命名空间中，使用与 activator pod 相同的服务帐户创建一个 debug pod。

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: debug
  annotations:
    sidecar.istio.io/inject: 'true'
  name: debug
spec:
  serviceAccountName: controller
  containers:
  - image: curlimages/curl
    name: debug
    command:
      - sh
      - -c
      - 'sleep 3600; exit'
  restartPolicy: Never
```

exec 进入 pod，运行以下命令，

```
export CACERT=/run/secrets/kubernetes.io/serviceaccount/ca.crtexport TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)curl -H "Authorization: Bearer $TOKEN" --cacert $CACERT https://172.30.0.1/api/v1/namespaces/knative-serving/configmaps/config-logging
```

那我就有错误了

```
curl: (35) error:1408F10B:SSL routines:ssl3_get_record:wrong version number
```

它与 activator pod 相同。正常的 http 响应是对 https 请求的回应。

## 测试 1。正常的 http 运行正常吗？

答案如下图是肯定的。

```
kubectl -n knative-serving exec debug -i -c debug -- sh -c 'curl [http://httpbin.org/headers'](http://httpbin.org/headers')...{
  "headers": {
    "Accept": "*/*",
    "Content-Length": "0",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.69.1-DEV",
    "X-Amzn-Trace-Id": "Root=1-5e747c1d-41fbfb8274a15620acf0d30b",
    "X-B3-Sampled": "0",
    "X-B3-Spanid": "c645c09f1ab9c9c0",
    "X-B3-Traceid": "685234a1177a2ebcc645c09f1ab9c9c0",
    "X-Envoy-Expected-Rq-Timeout-Ms": "15000",
    "X-Istio-Attributes":nVnLmtuYXRpdmUtc2VydmluZw=="
  }
}
```

## 测试 2。https 在工作吗？

```
kubectl -n knative-serving exec debug -i -c debug -- sh -c 'curl [https://httpbin.org/headers'](http://httpbin.org/headers')curl: (35) error:1408F10B:SSL routines:ssl3_get_record:wrong version number
command terminated with exit code 35
```

https 不工作。

## 测试 3。服务网格中的 http 在工作吗？

上述测试是针对外部服务的。在 activator 的错误消息中，172.30.0.1 指的是 kubernetes.default 的服务。默认名称空间实际上不在服务网格中，这是用以下命令显示的，

```
oc -n istio-system get smmr
NAME      MEMBERS
default   [knative-serving tekton-pipelines kabanero plant-by-websphere plant-by-websphere-dev redis-service]
```

因此，让我们在服务网格中找出一些服务，比如说 knative-serving，

```
oc -n knative-serving get svcNAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
activator-service   ClusterIP   172.30.99.179    <none>        80/TCP,81/TCP,9090/TCP      10d
**autoscaler          ClusterIP   172.30.108.136   <none>        8080/TCP,9090/TCP,443/TCP   10d**
controller          ClusterIP   172.30.218.61    <none>        9090/TCP                    10d
webhook             ClusterIP   172.30.125.111   <none>        443/TCP                     10d
```

选择自动缩放服务。

```
kubectl -n knative-serving exec debug -i -c debug -- sh -c 'curl -s [http://autoscaler.knative-serving:8080'](http://autoscaler.knative-serving:8080')
Bad Request
```

http 服务器已响应，网格内 HTTP 服务正在工作。

## 测试 4。服务网格中的 https 是否正常工作？

```
kubectl -n knative-serving exec debug -i -c debug -- sh -c 'curl -ks [https://autoscaler.knative-serving'](http://autoscaler.knative-serving:8080'){
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```

所以 https in-mesh 也在工作。

# 4.回避问题

通过上面的一组测试，问题的行为是清楚的，服务网格之外的 HTTPS 服务不能被正确访问。走查非常简单，让默认名称空间加入服务网格。

编辑 *ServiceMeshMemberRoll* 以添加*默认*名称空间。我们应该有如下的东西，

```
oc -n istio-system get smmr default
NAME      MEMBERS
default   [knative-serving tekton-pipelines kabanero plant-by-websphere plant-by-websphere-dev redis-service default]
```

过了一会儿，激活器吊舱恢复正常。唷！

# 5.真正的罪犯

绕场就是绕场。真正的问题是什么？经过一番搜索，本期日志(【https://github.com/istio/istio/issues/14264】T4)描述了真正的问题。

首先，确定代理。

```
istioctl  proxy-status
NAME                                                              CDS        LDS        EDS        RDS        PILOT                            VERSION
activator-67f8b69697-sqwgb.knative-serving                        SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
cluster-local-gateway-6647b96c4d-8t9bp.istio-system               SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
debug.knative-serving                                             SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
istio-ingressgateway-8465bbf788-x52pk.istio-system                SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
pbw-dev-v1-deployment-5bc69b6f49-gfqqd.plant-by-websphere-dev     SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
pbw-v1-deployment-5bc474485b-vgszq.plant-by-websphere             SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
pbw-v2-deployment-6c7b986d8-bgbc4.plant-by-websphere              SYNCED     SYNCED     SYNCED     SYNCED     istio-pilot-59fc69bd66-t7zbr     1.1.11*
```

然后，特别寻找 443 这个名字，

```
istioctl -n knative-serving proxy-config routes activator-67f8b69697-sqwgb.knative-serving --name 443 -o json
[
    {
        "name": "443",
        "virtualHosts": [
            {
                "name": "tekton-dashboard.tekton-pipelines.svc.cluster.local:443",
                "domains": [
                    "tekton-dashboard.tekton-pipelines.svc.cluster.local",
                    "tekton-dashboard.tekton-pipelines.svc.cluster.local:443",
                    "tekton-dashboard.tekton-pipelines",
                    "tekton-dashboard.tekton-pipelines:443",
                    "tekton-dashboard.tekton-pipelines.svc.cluster",
                    "tekton-dashboard.tekton-pipelines.svc.cluster:443",
                    "tekton-dashboard.tekton-pipelines.svc",
                    "tekton-dashboard.tekton-pipelines.svc:443",
                    "172.30.118.86",
                    "172.30.118.86:443"
                ],
```

请注意，虚拟主机同时具有普通 http 和 443。检查 K8s 服务，

```
kubectl -n tekton-pipelines get svc tekton-dashboard -o yaml...
spec:
  clusterIP: 172.30.118.86
  ports:
  - name: http
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    app: tekton-dashboard
  sessionAffinity: None
  type: ClusterIP
...
```

443 被命名为 http。这是罪魁祸首。编辑它，并将“http”重命名为“https”以解决问题。(*我们不需要让默认名称空间成为服务网格中的成员*)

Istio 中问题的根本原因记录在问题日志[https://github.com/istio/istio/issues/16458](https://github.com/istio/istio/issues/16458)中。它现在还开着。
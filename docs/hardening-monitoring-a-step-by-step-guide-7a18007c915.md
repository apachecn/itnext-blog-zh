# 强化监控:逐步指南

> 原文：<https://itnext.io/hardening-monitoring-a-step-by-step-guide-7a18007c915?source=collection_archive---------3----------------------->

内部的 Kubernetes 服务如果不向外界公开的话，已经在一个私有网络上了。但是有必要强化内部服务吗？甚至在 Kubernetes 成为主流之前，只有当数据进出数据中心时，数据中心的敏感数据才会应用于外围安全层。然而，当内部网络受到威胁时，事情就会出错。在 Kubernetes，想象一个脆弱的豆荚成为一个目标，并最终妥协。这可能会对群集中所有未加密的流量造成威胁。证书主要用于加密，可以缓解这一问题。但是，如果没有正确的身份验证，这些 pod 可能会被用来从网络中的其他 pod 获取敏感信息。为此，证书还为内部网络中的身份识别和身份验证提供了基础设施。

![](img/042172a38b142564e58949d88ac3b921.png)

普罗米修斯标志:[来源](https://cncf-branding.netlify.app/projects/prometheus/)

在本文中，我将介绍如何安全地为 *metrics-server* 、 *prometheus-server* 和 *prometheus-adapter* 提供服务。

# 无聊的部分

资料来源:giphy.com

我们需要为每个组件生成三个证书。

Kubernetes 为提供 TLS 证书提供了 API，这些证书由内置的 Kubernetes 签名者或自定义证书颁发机构签名。

为了请求证书，我们使用类似于官方指南的 Cloudflare 命令行工具`cfssl` [。](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/#requesting-a-certificate)

安装是通过获得它

```
$ go get github.com/cloudflare/cfssl/cmd/cfssl
```

或者通过自制

```
$ brew install cfssl
```

# 正在生成私钥

下一步是通过在 JSON 文件中定义`certificate signing request`为我们的每个服务生成一个私钥:

```
{
  "hosts": [
    "<service name>.<namespace>",
    "<service name>.<namespace>.svc.cluster.local",
    "<service name>.<namespace>.pod.cluster.local",
    "<service IP>",
    "<Pod IP>"
  ],
  "CN": "<service name>.<namespace>.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
}
```

例如，对于`kube-system`名称空间内的`metrics-server`，文件看起来像这样:

```
// csr-metrics-server.json
{
  "hosts": [
    "metrics-server.kube-system",
    "metrics-server.kube-system.svc.cluster.local",
    "metrics-server.kube-system.pod.cluster.local",
    "<service IP>",
    "<Pod IP>"
  ],
  "CN": "<service name>.<namespace>.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
}
```

然后使用以下命令应用该文件:

```
$ cfssl genkey <csr filename>.json | cfssljson -bare server
```

# 创建签名请求

现在，它已经生成了`server.csr`、`server-key.pem`。前者包含认证请求，后者包含编码的私钥。

接下来，让我们利用 Kubernetes API 来创建一个`CertificateSigningRequest`对象。

```
# csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: <Your CSR Name>
spec:
  request: <base64-encoded server.csr>
  signerName: <Your signer name>
  usages:
  - digital signature
  - key encipherment
  - server auth
```

在这里，对于`signer`，您可以选择使用自定义签名器或者 [Kubernetes 签名器](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#kubernetes-signers)。

然后使用以下命令创建并批准请求:

```
$ kubectl apply -f csr.yaml
$ kubectl approve certificate approve <Your CSR Name>
```

如果您已经将`Condition`字段设为`Approved`，那么您已经准备好签署证书了:

```
$ kubectl get csr
NAME                  AGE   SIGNERNAME            REQUESTOR              REQUESTEDDURATION   CONDITION
metrics-server-csr    10m   example.com/serving   yourname@example.com   <none>              Approved
```

# 通过 CA 签署请求

如果您在上一步中使用了 Kubernetes signer，您可以跳过这一步，下载对象状态的签名证书，如下所示:

```
$ kubectl get csr <csr request name> -o json | \\
  jq '.status.certificate = "'$(base64 ca-signed-server.pem | tr -d '\\n')'"' | \\
  kubectl replace --raw /apis/certificates.k8s.io/v1/certificatesigningrequests/<csr request name>/status -f -
$ kubectl get csr <csr request name> -o jsonpath='{.status.certificate}' \\
    | base64 --decode > server.crt
```

然后直接创建证书机密:

```
$ kubectl create secret tls <secret name> --cert server.crt --key server-key.pem
```

如果您需要创建自己的 CA，首先创建 CA JSON 配置:

```
// ca.json
{
  "CN": "My Example Signer",
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}
```

```
$ cfssl gencert -initca ca.json |  cfssljson -bare ca
$ kubectl create configmap <CA configmap name> --from-file ca.crt=ca.pem
```

现在，让`ca-key.pem`和`ca.pem`使用签名配置对其进行签名:

```
// server-signing-config.json
{
    "signing": {
        "default": {
            "usages": [
                "digital signature",
                "key encipherment",
                "server auth"
            ],
            "expiry": "876000h",
            "ca_constraint": {
                "is_ca": false
            }
        }
    }
}
```

```
$ kubectl get csr <csr request name> -o jsonpath='{.spec.request}' | \\
  base64 --decode | \\
  cfssl sign -ca ca.pem -ca-key ca-key.pem -config server-signing-config.json - | \\
  cfssljson -bare ca-signed-server
```

终于，你完成了！如前所述，下载证书并在此基础上创建一个秘密。

# 将证书附加到服务

既然我们已经创建了 TLS 秘密，我们只需要将它附加到目标资源文件中。我将用普罗米修斯和普罗米修斯适配器的头盔文件来连接它们。

# 附加证书

# 普罗米修斯适配器

让我们从普罗米修斯适配器开始

```
# prometheus-adapter-values.yaml
namespaceOverride: <namespace>
logLevel: 10
runAsUser: 12100
podSecurityContext:
  fsGroup: 11000
prometheus:
  url: https://<prometheus service name>.<prometheus namespace>
extraArguments:
  - --tls-cert-file=/var/run/serving-cert/tls.crt
  - --tls-private-key-file=/var/run/serving-cert/tls.key
  - --client-ca-file=/etc/ssl/certs/ca.crt
extraVolumes:
  - name: volume-serving-cert
    secret:
      secretName: <prometheus-adapter certificate secret name>
  - name: ssl-certs
    configMap:
      name: <CA configmap Name>
extraVolumeMounts:
  - mountPath: /var/run/serving-cert
    name: volume-serving-cert
    readOnly: true
  - name: ssl-certs
    mountPath: /etc/ssl/certs
    readOnly: true
resources:
  limits:
    cpu: 100m
    memory: 128Mi
    ephemeral-storage: 1Gi
rules:
  default: false
  custom:
    - seriesQuery: 'container_network_receive_packets_total{namespace!="",pod!=""}'
      resources:
        template: "<<.Resource>>"
      name:
        matches: "^(.*)"
        as: "packets-per-minutes"
      metricsQuery: sum(rate(<<.Series>>{<<.LabelMatchers>>}[1m])) by (<<.GroupBy>>)
```

# 普罗米修斯

```
# prometheus-values.yaml
server:
  extraSecretMounts:
  - mountPath: /var/run/serving-cert
    name: volume-serving-cert
    secretName: <prometheus certificate secret name>
		readOnly: true
	extraConfigmapMounts:
  - name: ssl-certs
    mountPath: /etc/ssl/certs
    configMap: <CA configmap Name>
		readOnly: true
extraScrapeConfigs: |
  - job_name: myjob
    scrape_interval: 15s
    metrics_path: /metrics
    scheme: https
    static_configs:
      - targets:
          - ###.##.###.###:#####
    tls_config:
      ca_file: /etc/ssl/certs/ca.pem
      key_file: /var/run/serving-cert/key.pem
      cert_file: /var/run/serving-cert/cert.pem
      insecure_skip_verify: true
```

# 度量-服务器

度量服务器可能默认来自您的 Kubernetes 发行版。编辑资源清单，并将以下参数添加到容器中:

```
spec:                                                                                                                                                                                     
    containers:                                                                                                                                                                             
    - args:                                                                                                                                                                                 
      - --client-ca-file=/etc/ssl/certs/ca.pem
      - --tls-cert-file=/var/run/serving-cert/cert.pem
      - --tls-private-key-file=/var/run/serving-cert/key.pem
```

除了参数之外，将卷附加到资源，这样它就可以访问`secret`和`configmap`:

```
volumeMounts:
  - mountPath: /var/run/serving-cert
    name: volume-serving-cert
    readOnly: true
  - mountPath: /etc/ssl/certs
    name: ssl-certs
    readOnly: true
volumes:
  - name: volume-serving-cert
    secret:
      secretName: <metrics-server TLS secret name>
  - configMap:
      name: <CA configmap name>
    name: ssl-certs
```

资料来源:giphy.com

精彩！现在您已经安全地成功部署了您的监控服务，您可以看一下本指南[中关于为 Prometheus-Adapter 配置一些自定义指标的内容。然后，您可以使用](https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/config-walkthrough.md) [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics) 使用指标来扩展您的工作负载。

我会多写 CS 的文章；因此，如果你觉得有趣和有帮助，请跟随我的媒介。另外，请随时通过 [LinkedIn](https://www.linkedin.com/in/ali--moezzi/) 直接联系我。
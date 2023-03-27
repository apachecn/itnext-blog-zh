# K8s 技巧:从 Pod 访问 API 服务器

> 原文：<https://itnext.io/k8s-tips-accessing-the-api-server-from-a-pod-f6f72bc847de?source=collection_archive---------2----------------------->

## 除非真的有必要，否则不要让这种情况发生

![](img/bab0c3e104d4b758343dc5e2a75280fd.png)

照片由 [Dim 侯](https://unsplash.com/@dimhou?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/not-authorized?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

默认情况下，标识默认 ServiceAccount 的令牌安装在 Pod 的容器中。这个令牌允许容器与 API 服务器通信。应用程序能做的不多，但是能够针对 API 服务器进行身份验证已经太多了，尤其是在不需要这样做的情况下。只有需要对 Kubernetes 的资源执行操作的应用程序(例如:列出节点、创建服务、部署、配置映射等)才需要能够针对 API 服务器进行身份验证。

在这篇短文中，我们将探索一个 Pod 在默认情况下与 API 服务器通信的方式。

首先，我们使用下面的命令创建一个基于**幽灵**图像的 Pod:

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ghost
spec:
  containers:
  - image: ghost:4
    name: ghost
EOF
```

接下来，我们检查群集中现在已知的 Pod 的完整规格:

```
$ kubectl get po ghost -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: c7b73bd3bbb97438285987733aece40a84b73c96d39babfce624df10303dea29
    cni.projectcalico.org/podIP: 192.168.6.153/32
    cni.projectcalico.org/podIPs: 192.168.6.153/32
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"ghost","namespace":"default"},"spec":{"containers":[{"image":"ghost:4","name":"ghost"}]}}
    kubernetes.io/psp: privileged
  creationTimestamp: "2022-12-08T15:12:19Z"
  name: ghost
  namespace: default
  resourceVersion: "3858826580"
  uid: 61b901a1-cd5b-41a1-9cc6-3f703cc555c1
spec:
  containers:
  - image: ghost:4
    imagePullPolicy: IfNotPresent
    name: ghost
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-fsjkt
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: pool-9d740-qcgsu
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-fsjkt
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T15:12:19Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T15:12:21Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T15:12:21Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T15:12:19Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://0d8abbb5d2ec63b5a7367b073a87557dc3d19daab6dc5fc45e7d361b9743c9bb
    image: docker.io/library/ghost:4
    imageID: docker.io/library/ghost@sha256:46e434734d979066bcea050dc0fb156b763283168ae080e00616543d62cc3706
    lastState: {}
    name: ghost
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-12-08T15:12:20Z"
  hostIP: 194.182.168.142
  phase: Running
  podIP: 192.168.6.153
  podIPs:
  - ip: 192.168.6.153
  qosClass: BestEffort
  startTime: "2022-12-08T15:12:19Z"
```

与我们在创建 Pod 时使用的属性相比，上面的规范中有更多的属性，例如:

*   网络插件已经添加了注释
*   已创建内部标识符
*   已经为我们在初始规范中没有指定的属性添加了默认值
*   添加了*状态*属性，它给出了关于 Pod 和容器生命周期的信息
*   添加了 Pod 及其运行节点的 IP 地址

在本帖中，我们将重点关注以下部分:

*   *卷*属性下*。规格*:

```
volumes:
  - name: kube-api-access-gh2kv
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

*   *volume mounts*下的*属性. spec.containers*

```
volumeMounts:
  - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
    name: kube-api-access-gh2kv
    readOnly: true
```

那些*卷*和*卷装*是干什么用的？

首先，*卷*属性定义了一个*投影*卷。这种卷允许将多个现有卷源放入同一目录。在当前示例中，预计体积使用:

*   标识默认命名空间(在其中创建 Pod 的命名空间)的 ServiceAccount 的令牌
*   包含群集证书颁发机构的配置映射
*   Pod 所在的命名空间。这一个是使用 DownwardAPI 检索的，它允许从属于 Pod 规范的字段中获取值

这 3 个元素中的每一个都将在投影卷内的专用路径中可用:分别是*令牌*、 *ca.crt* 和*命名空间*

接下来， *volumesMounts* 属性用于将投影卷的内容挂载到 Pod 的文件系统中的*/var/run/secrets/kubernetes . io/service account*文件夹中。

为了验证这一点，我们可以在 ghost 容器中运行一个 shell，并确保文件夹*/run/secrets/kubernetes . io/service account/*包含 3 个文件:

```
$ ls /run/secrets/kubernetes.io/serviceaccount/
ca.crt namespace  token
```

从容器中，我们尝试使用 *kubernetes* 内部服务调用 API 服务器:

```
$ curl https://kubernetes/api/v1
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

我们可以使用*/run/secrets/kubernetes . io/service account/*中的证书来消除这个错误:

```
$ curl --cacert /run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes/api/v1
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/api/v1\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```

我们得到了一个禁止的/ 403 错误，因为上面的命令没有使用任何认证。为了更进一步，我们可以使用在*/run/secrets/kubernetes . io/service account/token*中定义的令牌，如下所示:

```
$ TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token) 

$ curl -H "Authorization: Bearer $TOKEN" --cacert /run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes/api/v1/ 
{
  "kind": "APIResourceList",
  "groupVersion": "v1",
  "resources": [
    {
      "name": "bindings",
      "singularName": "",
      "namespaced": true,
      "kind": "Binding",
      "verbs": [
        "create"
      ]
    },
    {
      "name": "componentstatuses",
      "singularName": "",
      "namespaced": false,
      "kind": "ComponentStatus",
      "verbs": [
        "get",
        "list"
      ],
      "shortNames": [
        "cs"
      ]
    },
    {
      "name": "configmaps",
      "singularName": "",
      "namespaced": true,
      "kind": "ConfigMap",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "cm"
      ],
      "storageVersionHash": "qFsyl6wFWjQ="
    },
    ...
  ]
}
```

我们现在能够获得集群中所有可用资源类型的列表。让我们尝试获取节点列表:

```
$ curl -H "Authorization: Bearer $TOKEN" --cacert /run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes/api/v1/nodes
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "nodes is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"nodes\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "nodes"
  },
  "code": 403
}
```

由于我们使用链接到默认 ServiceAccount 的授权令牌，因此我们被标识为**system:service account:default:default**。这个 ServiceAccount 没有允许它列出节点的关联角色，这是我们得到一个新的禁止/ 403 错误的原因。

让我们删除 ghost Pod，并使用以下命令创建一个新的。Pod 的规范将*. spec . automountserviceaccountoken*设置为 *false* 以防止 ServiceAccount 令牌在容器的文件系统中可用。

```
$ kubectl delete po ghost

$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ghost
spec:
  automountServiceAccountToken: false
  containers:
  - image: ghost:4
    name: ghost
EOF
```

正如我们之前所做的那样，我们可以查看一下集群中现在已知的完整规范:

```
$ kubectl get po ghost -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: ee795d6656add15dbbd6dce8c045a86b69c97d72333b1a930d73213b36d80e2d
    cni.projectcalico.org/podIP: 192.168.6.157/32
    cni.projectcalico.org/podIPs: 192.168.6.157/32
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"ghost","namespace":"default"},"spec":{"automountServiceAccountToken":false,"containers":[{"image":"ghost:4","name":"ghost"}]}}
    kubernetes.io/psp: privileged
  creationTimestamp: "2022-12-08T18:21:19Z"
  name: ghost
  namespace: default
  resourceVersion: "3860621158"
  uid: b51f5883-d8b9-4835-9436-db6cc845aac0
spec:
  automountServiceAccountToken: false
  containers:
  - image: ghost:4
    imagePullPolicy: IfNotPresent
    name: ghost
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: pool-9d740-qcgsu
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T18:21:19Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T18:21:20Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T18:21:20Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-12-08T18:21:19Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://f7a5127800534ad87cdecf14275d679fedc956b97bd45aeec2b91d5e55529774
    image: docker.io/library/ghost:4
    imageID: docker.io/library/ghost@sha256:46e434734d979066bcea050dc0fb156b763283168ae080e00616543d62cc3706
    lastState: {}
    name: ghost
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-12-08T18:21:20Z"
  hostIP: 194.182.168.142
  phase: Running
  podIP: 192.168.6.157
  podIPs:
  - ip: 192.168.6.157
  qosClass: BestEffort
  startTime: "2022-12-08T18:21:19Z"
```

这一次，在容器的文件系统中没有安装预计的卷，因此阻止了它向 API 服务器发送经过验证的查询(尽管受到限制)

## 关键要点

默认情况下，Pod 可以与 API 服务器通信，因为默认的 ServiceAccount 令牌安装在它的容器中。如果应用程序不需要与 API 服务器通信，我们需要确保这个令牌在容器中不可用。这可以通过 Pod 规范中的 automountServiceAccountToken 属性来完成。
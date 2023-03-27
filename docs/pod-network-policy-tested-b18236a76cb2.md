# Pod 网络策略已测试

> 原文：<https://itnext.io/pod-network-policy-tested-b18236a76cb2?source=collection_archive---------5----------------------->

![](img/2f4f11cfbd3fd9146a4686944beed02a.png)

图片由 [41330](https://pixabay.com/users/41330-41330/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1549437) 来自 [Pixabay](https://pixabay.com//?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1549437)

根据 CIS benchmark 的建议，一个名称空间至少需要一个网络策略。本文研究了网络政策中微妙的 YAML 语法，这种语法最终可能会导致完全不同的结果。

## 测试名称空间和窗格

让我们创建以下名称空间

*   myapp
*   ns1
*   ns2
*   ns3

在 myapp 名称空间中部署 Hello World Golang 应用程序。

## 默认无网络策略

网络策略是基于命名空间的资源。默认情况下，如果在名称空间中没有定义网络策略资源，则同一名称空间内或名称空间外的任何 pod 都将能够自由地与 pod 对话。

使用以下命令获取 Pod ip

```
$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP            NODE                                          NOMINATED NODE   READINESS GATES
app-6bd494586d-tdfgc   1/1     Running   0          23h   10.129.2.48   dev-410-worker3.dev-ocp410.ibmcloud.io.cpak   <none>           <none>
```

现在发射一个吊舱来测试连接。从同一个名称空间，我们可以连接并获得预期的响应。

```
kubectl -n myapp run netpol-1665322564 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080)Hello World! 2022-10-09 13:36:11.439497626 +0000 UTC m=+84136.506355255
```

从外部名称空间，相同的结果，

```
kubectl -n ns1 run netpol-1665322724 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080)Hello World! 2022-10-09 13:38:50.211197392 +0000 UTC m=+84295.278054970
```

## 最低网络策略—全部拒绝

使用最低配置在命名空间中定义网络策略，如下所示，

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: minimum-default
spec:
```

如果我们得到了网络策略，Kubernetes 会用默认设置扩展它，

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  annotations:
    ...<Skipped>...
  name: minimum-default
  namespace: myapp
  ...
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

在 myapp 名称空间中运行测试 pod，

```
oc -n myapp run netpol-1665323143 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080')If you don't see a command prompt, try pressing enter.
curl: (28) Connection timeout after 10000 ms
```

类似地，另一个名称空间中的测试将无法连接到 app pod。

检查网络策略，由于没有定义进入规则，所有传入流量都被阻止。

## 网络策略—全部允许

现在，让我们定义以下网络策略来允许所有传入流量。

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  ingress: [{}]
```

注意这里我们定义了入口对象，它是一个数组，我们用默认的空对象{}设置了第一个元素。这使得来自任何地方的输入流量成为可能。

在相同的命名空间和不同的命名空间中测试 agin，所有的流量再次被允许。

请注意，我们没有删除默认的最小规则，这两个规则仍然存在。

```
kubectl get netpol
NAME              POD-SELECTOR   AGE
allow-all         <none>         4s
minimum-default   <none>         27m
```

原因是网络策略是相加的，它们按照下面的顺序被评估，如这里提到的。

> 如果没有策略选择 pod，则该 pod 不受限制
> 
> 如果策略选择了一个 pod，则该 pod 被限制为那些策略的入口/出口规则的**联合**所允许的内容

在我们的例子中，minumim-default 阻止所有传入流量，但是 allow-all 规则再次允许传入流量，将它们加在一起(顺序无关紧要)，净效果是允许传入流量。

为了收紧流量规则，让我们取消允许所有策略。

## 网络策略—示例

现在让我们为名称空间 myapp 中的所有 pod 定义传入网络策略，要求如下

*   允许同一命名空间内的所有窗格
*   允许命名空间“ns1”中的所有窗格
*   允许命名空间“ns2”中的 pod 标签为 myapp=allow 的 pod

网络策略如下所列，

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-sample
  namespace: myapp
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ns1
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ns2
      podSelector:
        matchLabels:
          myapp: allow
```

该网络策略适用于 myapp 名称空间中的所有 pod，因为 podSelector 为空。

对于 ingress 关键字，我们在 parrallel 中定义了三个 from 规则。

在第一个“from”中，因为没有 namespaceSelector，所以它默认为当前名称空间。因为 podSelector 是空的，所以它适用于所有的 pod。允许来自同一命名空间中任何 pod 的传入流量。

第二个“from”允许名称空间“ns1”连接到 myapp 的名称空间中的 pod。我使用的是 Kubernetes 添加的 metadata.name 标签。

最后一个“from”允许来自 ns2 命名空间中的那些 pod 的传入流量，并且必须标有“myapp=allow”。注意，podSelector 和 namespaceSelector 在同一个对象中。如果它被写成如下带有意外“-”，则“与”逻辑被改为“或”。

```
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: ns2
  - podSelector:
      matchLabels:
        myapp: allow
```

## 测试

在同一命名空间中，允许的传入流量:

```
kubectl -n myapp run netpol-1665327691 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080)Hello World! 2022-10-09 15:01:37.087078709 +0000 UTC m=+89262.153936307
```

在 ns1 的命名空间中，允许:

```
kubectl -n ns1 run netpol-1665327708 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080)Hello World! 2022-10-09 15:01:55.520024859 +0000 UTC m=+89280.586882448
```

允许在 ns2 的命名空间中使用标签:

```
kubectl -n ns2 run netpol-1665327725 --image=alpine/curl --restart=Never --rm -i --labels myapp=allow -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080')Hello World! 2022-10-09 15:02:11.885730744 +0000 UTC m=+89296.952588342
```

在没有标签的 ns2 命名空间中不允许:

```
kubectl -n ns2 run netpol-1665327744 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080'](http://10.129.2.48:8080')If you don't see a command prompt, try pressing enter.
curl: (28) Connection timeout after 10000 ms
```

在其他命名空间中不允许:

```
kubectl -n ns3 run netpol-1665327768 --image=alpine/curl --restart=Never --rm -i -- -fsSL --connect-timeout 10 [http://10.129.2.48:8080](http://10.129.2.48:8080')If you don't see a command prompt, try pressing enter.
curl: (28) Connection timeout after 10001 ms
```

## 讨论

出口规则可以类似地用“出口”和“目的地”关键字来定义，而不影响入口规则。
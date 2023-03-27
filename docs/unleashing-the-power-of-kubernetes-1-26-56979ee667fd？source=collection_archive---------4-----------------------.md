# 释放 Kubernetes 1.26 的威力:探索 CEL 的新 ValidatingAdmissionPolicy 特性

> 原文：<https://itnext.io/unleashing-the-power-of-kubernetes-1-26-56979ee667fd?source=collection_archive---------4----------------------->

Kubernetes 工作人员几天前刚刚推出了最新版本 k8s 1.26，它包含了一些非常酷的新功能。最吸引我的是用于准入控制的 CEL，它允许我们创建 ValidatingAdmissionPolicy，将我们的集群安全性提升到一个新的水平。

> *验证准入策略为验证准入 webhooks 提供了一个声明性的进程内替代方案。*
> 
> *验证准入策略使用通用表达式语言(CEL)来声明策略的验证规则。*

# 什么是 CEL？

> *通用表达式语言(CEL)实现了表达式求值的通用语义，使不同的应用程序更容易进行互操作。* [*https://github.com/google/cel-spec*](https://github.com/google/cel-spec)

它是一种编程语言，在 Kubernetes 中用来创建定制的准入控制策略。它允许创建复杂的细粒度策略，以在集群中实施安全性和合规性。

在之前的[文章](https://dev.to/douglasmakey/implementing-a-simple-k8s-admission-controller-in-go-2dcg)中，我们探讨了如何使用 Kubernetes 准入控制器来执行验证，比如防止容器图像使用最新的标签。在这篇文章中，我们将深入研究新的 ValidatingAdmissionPolicy 特性，它允许我们以一种更简单的方式进一步利用我们的验证功能，而无需处理 webhooks！

# 本地测试 K8s 1.26

对于本地测试，您可以使用类似于`kind`或`microk8s`的工具。

在使用`kind`或`microk8s`之前，确保启用`ValidatingAdmissionPolicy`特征门和`admissionregistration.k8s.io/v1alpha1` API。

# micro k8s—[https://micro k8s . io](https://microk8s.io/)

```
microk8s install --channel 1.26
```

您需要编辑位于下面代码片段中指定的地址的 kube-apiserver 文件来激活必要的特性。

```
$ vim /var/snap/microk8s/current/args/kube-apiserver
```

```
...
--feature-gates=ValidatingAdmissionPolicy=true
--runtime-config=admissionregistration.k8s.io/v1alpha1=true
...
```

如果您使用的是 Mac 并且安装了 microk8s，它会将 multipass 用作虚拟机。要访问虚拟机，请运行以下命令:

```
$ multipass list
Name                    State             IPv4             Image
microk8s-vm             Running           192.168.64.5     Ubuntu 18.04 LTS
```

```
$ multipass shell microk8s-vm
```

# kind—[https://kind . sigs . k8s . io](https://kind.sigs.k8s.io/)

通过 kind，您可以使用在 YAML 编写的包含所有必要设置的配置文件来创建集群。

```
kind: Cluster
name: demo-cel
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  "ValidatingAdmissionPolicy": true
runtimeConfig:
  "admissionregistration.k8s.io/v1alpha1": true
nodes:
- role: control-plane
  image: kindest/node:v1.26.0
```

奔跑

```
$ kind create cluster --config=cluster.yaml
```

对于`kind`和`microk8s`，您可以使用 kubectl 来检查是否启用了所有必需的特性。

```
$ kubectl api-versions | grep admissionregistration
```

```
admissionregistration.k8s.io/v1alpha1$ kubectl api-resources | grep ValidatingAdmissionPolicyvalidatingadmissionpolicies                      admissionregistration.k8s.io/v1alpha1   false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                admissionregistration.k8s.io/v1alpha1   false        ValidatingAdmissionPolicyBinding
```

现在您已经设置了集群，您已经准备好开始试验`ValidatingAdmissionPolicy`。

# (演奏等的)表现，风格；(乐曲)演奏

要创建策略，您需要创建以下内容:

1.  概述策略抽象逻辑的 ValidatingAdmissionPolicy(例如，“此策略确保将特定标签设置为特定值”)。
2.  以及将 ValidatingAdmissionPolicy 连接到特定范围或上下文的 ValidatingAdmissionPolicy binding。"

假设您想要创建一个策略，通过要求最少 3 个副本来确保生产部署中的高可用性。该政策可使用以下`ValidatingAdmissionPolicy`和`ValidatingAdmissionPolicyBinding`资源实施:

```
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicy
metadata:
  name: "force-ha-in-prod"
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups:   ["apps"]
      apiVersions: ["v1"]
      operations:  ["CREATE", "UPDATE"]
      resources:   ["deployments"]
  validations:
    - expression: "object.spec.replicas >= 3"
      message: "All production deployments should be HA with at least three replicas"
```

上面的 YAML 显示了如何指定哪些资源和操作将受到策略的影响。在`spect.validations`部分中，您可以使用 CEL 来定义表达式，资源必须满足这些表达式才能被策略接受。如果表达式的计算结果为 false，则根据`spec.failurePolicy`字段执行验证检查。

根据官方的 Kubernetes [文档](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/#validation-expression)，CEL 表达式可以通过许多变量访问接纳请求和响应的内容:

*   **对象**:来自传入请求的对象(对于删除请求为空)。
*   **oldObject** :现有对象(对于创建请求为空)。
*   **请求**:准入请求的属性。
*   **params** :正在评估的策略绑定所引用的参数资源(如果 ParamKind 未设置，则为 null)。

现在您有了第一个策略，您需要使用`ValidatingAdmissionPolicyBinding`来绑定它:

```
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: "force-ha-in-prod-binding"
spec:
  policyName: "force-ha-in-prod"
  matchResources:
    namespaceSelector:
      matchLabels:
        env: production
```

`MatchResources`根据对象是否满足匹配标准来决定是否对其运行准入控制策略。在这种情况下，您使用`namespaceSelector`将策略应用于所有标有`env: production`的名称空间。

也许您想使用`objectSelector`，它根据对象是否有匹配的标签来决定是否运行验证。它根据对象的旧版本和新版本进行评估，允许您设置验证以确保资源不会以特定方式发生更改。

```
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: "force-ha-in-prod-object-binding"
spec:
  policyName: "force-ha-in-prod"
  matchResources:
    objectSelector:
      matchLabels:
        from: kungfudev
```

当然也可以用空的`LabelSelector`，会匹配所有对象。如果您希望您的策略在所有资源上运行，而不考虑它们的标签，这将非常有用。

```
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: "force-ha-in-prod-for-all-binding"
spec:
  policyName: "force-ha-in-prod"
  matchResources:
```

要测试该策略，您可以尝试应用以下 YAML，其中包括一个命名空间和一个只有两个副本的部署。

```
apiVersion: v1
kind: Namespace
metadata:
  name: payments-system
  labels:
    env: production
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: payments-system
  name: payment-gateway
  labels:
    app: nginx
    from: kungfudev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.3
        ports:
        - containerPort: 80
```

测试它:

```
$ kubectl apply -f force-ha-in-prod.yaml
validatingadmissionpolicy.admissionregistration.k8s.io/force-ha-in-prod created
```

```
$ kubectl apply -f force-ha-in-prod-binding.yaml
validatingadmissionpolicybinding.admissionregistration.k8s.io/force-ha-in-prod-binding created$ kubectl apply -f payment-deployment.yaml
The deployments "payment-gateway" is invalid: : ValidatingAdmissionPolicy 'force-ha-in-prod' with binding 'force-ha-in-prod-binding' denied request: All production deployments should be HA with at least three replicas
```

该策略拒绝部署，因为它不符合策略验证中定义的要求。但是，如果您修改部署以使用 4 个复制副本而不是 2 个，策略应该允许。

```
...
spec:
  replicas: 4
  selector:
    matchLabels:
...
```

```
$ kubectl apply -f payment-deployment.yaml
deployment.apps/payment-gateway created
```

哇，我们只需几个简单的动作，就能确保我们所有的生产部署高度可用！目前，我们的策略非常基本，但是没有什么能阻止我们创建更复杂的验证。想象一下，不允许使用“latest”标签，只允许来自我们自己的注册表的图像，或者确保所有容器都设置了 CPU 限制。有了我们指尖的`ValidatingAdmissionPolicy`的力量，可能性是无穷的！

通过利用 CEL 的全部功能，我们可以链接多个表达式来创建更复杂的验证。

要了解 CEL 提供的其他特性和功能，请访问此[链接](https://github.com/google/cel-spec/blob/master/doc/langdef.md)。

```
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicy
metadata:
  name: "prod-ready-policy"
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups:   ["apps"]
      apiVersions: ["v1"]
      operations:  ["CREATE", "UPDATE"]
      resources:   ["deployments"]
  validations:
    - expression: "object.spec.template.spec.containers.all(c, !c.image.endsWith(':latest'))"
      message: "cannot use the latest tag"
    - expression: "object.spec.template.spec.containers.all(c, c.image.startsWith('myregistry.com/'))"
      message: "image from an untrusted registry"
    - expression: "has(object.metadata.labels.env) && object.metadata.labels.env in ['prod', 'production']"
      message: "deployment should have an env and have to be prod or production"
    - expression: |
        object.spec.template.spec.containers.all(
          c, has(c.resources) && has(c.resources.limits) && has(c.resources.limits.cpu)
        )
      message: "container does not have a cpu limit set"
---
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: "prod-reay-policy-binding"
spec:
  policyName: "prod-reay-policy"
  matchResources:
    namespaceSelector:
      matchExpressions:
      - key: "env"
        operator: "In"
        values: [prod, production,]
```

在上面的 YAML 中，你会注意到我们可以在绑定期间使用`matchExpressions`,给我们的策略一个更动态的范围。

ValidatingAdmissionPolicy 功能还包括参数资源，允许您将策略配置与其定义分开。这意味着，例如，您可以使用一个参数并为不同的环境配置不同的值，而不是在 HA 策略中为所有命名空间硬编码最小数量的副本。查看官方[文档](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/#parameter-resources)了解更多关于如何使用该功能的信息。

# 最后

有了 CEL 和 k8s 1.26 中新的 ValidatingAdmissionPolicy 特性，我们现在有能力创建定制的复杂策略，以在集群中实施安全性和合规性。

我们知道开放策略代理(OPA)被广泛使用，但是 ValidatingAdmissionPolicy + CEL 被设计成比 OPA 更轻量级，更易于使用。ValidatingAdmissionPolicy 与 Kubernetes 进行了本机集成，这意味着它可以直接在 Kubernetes API 服务器中使用，而不需要单独的策略引擎。

在 Kubernetes 中使用 OPA 或 ValidatingAdmissionPolicy + CEL 来执行策略各有利弊。OPA 可能是一个功能更加丰富和强大的策略引擎，但它可能需要更多的设置和维护。ValidatingAdmissionPolicy + CEL 更容易使用，与 Kubernetes 的集成更加无缝，但它可能没有 OPA 的所有高级功能。最终，选择使用哪个策略引擎将取决于用户的特定需求和偏好。

在这个[库](https://github.com/douglasmakey/k8s-validating-admission-policy)中找到所有的 YAML 文件甚至更多的例子。

感谢您的关注，我希望这篇文章对您有所帮助，并且易于理解。快乐集群建设！
# Gloo 的两阶段金丝雀展示，第 3 部分

> 原文：<https://itnext.io/two-phased-canary-rollout-with-gloo-part-3-a523114a7d9a?source=collection_archive---------6----------------------->

在本系列的[第一部分中，我们试图提出一个健壮的工作流，您可以用它来执行金丝雀测试和渐进交付，这样团队就可以在生产环境中安全地向用户交付他们服务的新版本。](/two-phased-canary-rollout-with-gloo-part-1-ec5b267cdc9e)

[在第二部分](/two-phased-canary-rollout-with-gloo-part-2-ffa556a503b0)，我们看了如何将它扩展到潜在的许多团队，同时保持域和路由级配置之间的清晰分离。这有助于最大限度地减少开发团队为促进这些工作流而需要维护的配置数量，并使平台能够自助服务，同时防止错误配置。

在这一部分中，我们将创建一个掌舵图，我们的开发团队可以使用它将应用程序部署到他们的 Kubernetes 集群上的 Gloo。这意味着他们可以通过提供一些 helm 值来安装应用程序，并且他们可以通过执行 helm 升级来调用 canary 升级工作流。这样做有很多好处:

*   它降低了进入的门槛；这个工作流程很容易执行。
*   它极大地减少了不同团队需要维护的(通常)复制粘贴配置的数量。
*   它提供了很好的防护栏，最大限度地减少了错误配置的可能性。
*   集成到 GitOps /连续交付工作流中变得微不足道。

作为免责声明，许多服务最终都需要相当广泛的配置，其中一些是特定用例所特有的。然而，在一定程度上，你的团队可以遵循标准的惯例，这些惯例可以像我们在这里做的那样被编码在掌舵图中。对于这一部分，我们将为我们的场景创建一个最小的定制集；将来，我们会让它更通用。

# 设置我们的图表

Helm 包括一个创建图表的工具。运行`helm create gloo-app`之后，您将得到一个包含一个`Chart.yaml`、一个默认的`values.yaml`、一个空的 crds 目录和一个包含许多文件的 templates 目录的目录。

我们将更新`Chart.yaml`，删除空的 crds 目录，并且我们将删除模板目录的内容。在接下来的几个步骤中，我们将开始添加模板来模拟我们的服务。

# 分解我们的需求

在最后一部分中，我们通过应用四个资源来部署每个服务——echo 和 foxtrot:部署、服务、gloo 上游和 gloo 路由表。这些将成为我们图表中的四个模板。我们将假设每个服务的名称空间都是预先或自动创建的，并且不包含在图表本身中。

请注意，该图表不包含 Gloo 控制平面本身，也不包含将所有路由表绑定到我们的域的单一虚拟服务。这些将由一个中央平台团队建立，而掌舵图旨在供开发团队使用，以便加入平台。

# 设计我们的初始价值观

我们将提取最少的最小值，以便图表可以在特定版本部署我们的`echo`或`foxtrot`服务。我们需要为我们的应用程序提供四个值:

*   服务名(`echo`):我们将从 Helm release 的名称中提取，并在模板中用`{{ .Release.Name }}`访问它
*   名称空间(`echo`):这也将由 Helm 命令提供，所以我们将使用`{{ .Release.Namespace }}`来访问它。
*   App Version ( `v1`):这将决定在 pod 上用于子集路由的版本标签。现在，我们将使用`{{ .Values.appVersion }}`访问它。
*   Api Group ( `example`):这将是我们在路由表中使用的标签，因此它与我们的虚拟服务选择器相匹配，现在我们将使用`{{ .Values.apiGroup }}`

# 创建我们的模板

现在我们可以创建模板了。

## 部署

首先，让我们在`templates/deployment.yaml`中创建一个部署。我们最初的 echo 部署如下所示:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-v1
  namespace: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
      version: v1
  template:
    metadata:
      labels:
        app: echo
        version: v1
    spec:
      containers:
        - image: hashicorp/http-echo
          args:
            - "-text=version:echo-v1"
            - -listen=:8080
          imagePullPolicy: Always
          name: echo-v1
          ports:
            - containerPort: 8080
```

为了将此归纳为一个可以部署`echo`或`foxtrot`的模板，在一个特定的版本中，让我们提取值。这就产生了这样一个模板:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Values.appVersion }}
  namespace: {{ .Release.Namespace }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Release.Name }}
      version: {{ .Values.appVersion }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
        version: {{ .Values.appVersion }}
    spec:
      containers:
        - image: hashicorp/http-echo
          args:
            - "-text=version:{{ .Release.Name }}-{{ .Values.appVersion }}"
            - -listen=:8080
          imagePullPolicy: Always
          name: {{ .Release.Name }}-{{ .Values.appVersion }}
          ports:
            - containerPort: 8080
```

## 服务

我们的服务也是如此。我们最初的服务是这样的:

```
apiVersion: v1
kind: Service
metadata:
  name: echo
  namespace: echo
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: echo
```

我们将基于此创建一个模板，并提取我们的值:

```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: {{ .Release.Name }}
```

## 向上游

现在我们已经有了部署和服务，我们可以添加我们的上游。我们最初的上游是这样的:

```
apiVersion: gloo.solo.io/v1
kind: Upstream
metadata:
  name: echo
  namespace: gloo-system
spec:
  kube:
    selector:
      app: echo
    serviceName: echo
    serviceNamespace: echo
    servicePort: 8080
    subsetSpec:
      selectors:
        - keys:
            - version
```

我们将提取到以下模板中:

```
apiVersion: gloo.solo.io/v1
kind: Upstream
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
spec:
  kube:
    selector:
      app: {{ .Release.Name }}
    serviceName: {{ .Release.Name }}
    serviceNamespace: {{ .Release.Namespace }}
    servicePort: 8080
    subsetSpec:
      selectors:
        - keys:
            - version
```

## 路由表

最后，我们需要创建包含初始路由的路由表。首先，这看起来像是:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
spec:
  routes:
    - matchers:
        - prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v1
```

我们将基于该路由表创建以下模板:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: {{ .Release.Name }}-routes
  namespace: {{ .Release.Namespace }}
  labels:
    apiGroup: {{ .Values.apiGroup }}
spec:
  routes:
    - matchers:
        - prefix: "/{{ .Release.Name }}"
      routeAction:
        single:
          upstream:
            name: {{ .Release.Name }}
            namespace: {{ .Release.Namespace }}
          subset:
            values:
              version: {{ .Values.appVersion }}
```

# 默认值

我们可以将图表中的默认`values.yaml`更新为以下内容:

```
appVersion: v1
apiGroup: example
```

# 测试我们的图表

现在，我们应该已经有了使用以下命令部署 echo 和 foxtrot 所需的组件:

```
kubectl create ns echo
helm install echo  ./gloo-app --namespace echo
kubectl create ns foxtrot
helm install foxtrot  ./gloo-app --namespace foxtrot
```

## 测试舵图的侧边栏

关于如何测试舵图，有不同的学派。在 Gloo 中，我们创建了一些库，这样我们就可以确保当提供不同的值集时，图表可以使用预期的资源进行呈现。头盔 3 也有一些内置的测试起点。实际上，用于生产的图表应该经过良好的测试，但是在这篇文章中，我们将跳过单元测试。

在开发这些模板时，我们可能使用的另一种技术是定期运行带有`--dry-run`标志的`helm install`命令，以确保模板没有语法错误，并且资源按照我们的预期呈现。

# 更新我们的图表以支持两阶段升级工作流程

现在我们有了一个基础图表，我们需要公开一些值来帮助执行我们的分阶段升级工作流。请记住，我们的推广战略有以下几个阶段:

*   第 1 阶段:金丝雀测试，使用一个特殊的报头，将少量有目标的流量路由到新版本进行功能测试
*   阶段 2:将负载转移到新版本，同时监控性能

# 设计我们的掌舵价值观

我们可以从设计一些能够表达工作流中不同点的值开始。特别是，如果我们考虑在工作流程中需要修改哪些资源，我们的价值观中有两个主要部分:

*   部署:该部分将用于控制要安装的部署版本。我们可能想要部署单个版本，或者多个版本。可能需要部署一个版本而根本不路由到它，例如做阴影。不同的版本需要不同的配置，所以我们在设计时会有这样的假设。
*   Routes:这个部分将用于配置我们处于流的哪个阶段。我们将使用`routes.primary`来配置我们的主要路线，使用`routes.canary`来对 canary 工作流程的不同阶段进行所有必要的修改。

考虑到这些需求，我们将把我们的`deployment` helm 值做成一个映射，其中键是版本名，值是特定于版本的配置。然后，为了配置我们的主版本和金丝雀版本，我们将使用这些键来指定一个版本。

我们还将使用一种简单的方法来完成第 2 阶段。我们将添加一个`routes.canary.weight`参数，这样金丝雀目的地可以被赋予 0 到 100 之间的权重。主重量将按`100 - routes.canary.weight`计算。

## 金丝雀工作流程，表示为舵值

既然我们已经决定了一个设计，让我们实际上用一系列的舵值来表达这个工作流程。

首先，我们有我们的开始状态，在这里我们部署应用程序的 v1:

```
deployment:
  v1: {} 
routes:
  apiGroup: example
  primary:
    version: v1
```

接下来，我们可以部署 v2，但还没有设置任何金丝雀路由:

```
deployment:
  v1: {} 
  v2: {} 
routes:
  apiGroup: example
  primary:
    version: v1
```

现在，我们可以开始阶段 1，并提供金丝雀配置:

```
deployment:
  v1: {} 
  v2: {} 
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    headers: 
      stage: canary
```

当我们准备开始第二阶段时，我们可以开始调整金丝雀路线的权重。请注意，为简单起见，前面的值等效于此值(0 重量):

```
deployment:
  v1: {} # we'll most likely want version-specific deployment configuration in the future;
  v2: {} # for now, we just need the list of versions that should be deployed
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    weight: 0 # 0 to 100 to facilitate traffic shift
    headers: 
      stage: canary
```

当我们完成工作流程的一半时，我们的值可能如下所示:

```
deployment:
  v1: {} # we'll most likely want version-specific deployment configuration in the future;
  v2: {} # for now, we just need the list of versions that should be deployed
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    weight: 50 # 0 to 100 to facilitate traffic shift
    headers: 
      stage: canary
```

最后，我们将所有流量转移到金丝雀版本:

```
deployment:
  v1: {} # we'll most likely want version-specific deployment configuration in the future;
  v2: {} # for now, we just need the list of versions that should be deployed
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    weight: 100 # 0 to 100 to facilitate traffic shift
    headers: 
      stage: canary
```

最后一步是淘汰旧版本:

```
deployment:
  v2: {} # for now, we just need the list of versions that should be deployed
routes:
  apiGroup: example
  primary:
    version: v2
```

太棒了——我们现在可以用声明性的舵值来表达我们工作流程的每一步。这意味着一旦我们更新了图表，我们将能够简单地通过运行`helm upgrade`来执行我们的工作流。

# 更新我们的模板

现在我们知道了我们想要如何表达我们的价值观，我们可以更新模板了。由于上游和服务模板没有使用来自 Helm 值的任何内容，我们不需要更改它们，我们只需要更新部署和路由表。

## 部署

我们的部署模板需要更新，因为`deployment`是版本到配置的映射。我们希望为地图中的每个版本创建一个部署资源，因此我们将包装模板以覆盖部署值。看起来是这样的:

```
{{- $relname := .Release.Name -}}
{{- $relns := .Release.Namespace -}}
{{- range $version, $config := .Values.deployment }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ $relname }}-{{ $version }}
  namespace: {{ $relns }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ $relname }}
      version: {{ $version }}
  template:
    metadata:
      labels:
        app: {{ $relname }}
        version: {{ $version }}
    spec:
      containers:
        - image: hashicorp/http-echo
          args:
            - "-text=version:{{ $relname }}-{{ $version }}"
            - -listen=:8080
          imagePullPolicy: Always
          name: {{ $relname }}-{{ $version }}
          ports:
            - containerPort: 8080
---
{{- end }}
```

关于这一点，请注意以下几点:

*   使用范围影响范围，所以`.Release.Name`和`.Release.Namespace`在范围内不可用。相反，我们将把它们保存到我们可以访问的变量中。
*   我们添加了一个 YAML 分隔符，因为我们有时会最终生成多个对象。

否则，这看起来非常类似于我们以前的模板。

## 路由表

现在，让我们更新路由表，在指定这些值后，开始表达我们的金丝雀路由配置。

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: {{ .Release.Name }}-routes
  namespace: {{ .Release.Namespace }}
  labels:
    apiGroup: {{ .Values.routes.apiGroup }}
spec:
  routes:
    {{- if .Values.routes.canary }}
    - matchers:
        - headers:
            {{- range $headerName, $headerValue := .Values.routes.canary.headers }}
            - name: {{ $headerName }}
              value: {{ $headerValue }}
            {{- end }}
          prefix: "/{{ .Release.Name }}"
      routeAction:
        single:
          upstream:
            name: {{ .Release.Name }}
            namespace: {{ .Release.Namespace }}
          subset:
            values:
              version: {{ .Values.routes.canary.version }}
    {{- end }}
    - matchers:
        - prefix: "/{{ .Release.Name }}"
      routeAction:
        {{- if .Values.routes.canary }}
        multi:
          destinations:
            - destination:
                upstream:
                  name: {{ .Release.Name }}
                  namespace: {{ .Release.Namespace }}
                subset:
                  values:
                    version: {{ .Values.routes.primary.version }}
              weight: {{ sub 100 .Values.routes.canary.weight }}
            - destination:
                upstream:
                  name: {{ .Release.Name }}
                  namespace: {{ .Release.Namespace }}
                subset:
                  values:
                    version: {{ .Values.routes.canary.version }}
              weight: {{ add 0 .Values.routes.canary.weight }}
        {{- else }}
        single:
          upstream:
            name: {{ .Release.Name }}
            namespace: {{ .Release.Namespace }}
          subset:
            values:
              version: {{ .Values.routes.primary.version }}
        {{- end }}
```

我们一段一段来。我们对顶部的元数据做的唯一改变是我们把`apiGroup`移到了一个新的头盔值。

在`routes`部分，事情开始变得有趣起来。首先，我们添加了一个条件块来呈现金丝雀路线:

```
{{- if .Values.routes.canary }}
    - matchers:
        - headers:
            {{- range $headerName, $headerValue := .Values.routes.canary.headers }}
            - name: {{ $headerName }}
              value: {{ $headerValue }}
            {{- end }}
          prefix: "/{{ .Release.Name }}"
      routeAction:
        single:
          upstream:
            name: {{ .Release.Name }}
            namespace: {{ .Release.Namespace }}
          subset:
            values:
              version: {{ .Values.routes.canary.version }}
    {{- end }}
```

只要`canary`不为零，这条路线就会被包括在内。它将使用我们引入的`version`和`headers`值来指定匹配和子集路由应该如何工作。

我们还需要定制另一条路线上的路线操作，因为在第 2 阶段，我们将其从单目的地更改为多目的地，并建立权重:

```
 routeAction:
        {{- if .Values.routes.canary }}
        multi:
          destinations:
            - destination:
                upstream:
                  name: {{ .Release.Name }}
                  namespace: {{ .Release.Namespace }}
                subset:
                  values:
                    version: {{ .Values.routes.primary.version }}
              weight: {{ sub 100 .Values.routes.canary.weight }}
            - destination:
                upstream:
                  name: {{ .Release.Name }}
                  namespace: {{ .Release.Namespace }}
                subset:
                  values:
                    version: {{ .Values.routes.canary.version }}
              weight: {{ add 0 .Values.routes.canary.weight }}
        {{- else }}
        single:
          upstream:
            name: {{ .Release.Name }}
            namespace: {{ .Release.Namespace }}
          subset:
            values:
              version: {{ .Values.routes.primary.version }}
        {{- end }}
```

为了简单起见，一旦提供了金丝雀值，我们就切换到多目的地。除非用户明确地给金丝雀增加一个权重，否则这不会有任何影响，所有的流量将继续被路由到主版本。这只是保持我们的模板和值最小。

另一个有趣的注意事项是，我们在权重上使用了`Helm`算术函数，因此我们总是以总权重 100 结束，并且用户只需要在值中指定一个权重。

# 使用我们的新图表运行工作流程

我们现在准备好执行我们的整个 canary 工作流，通过执行 helm 安装，然后升级和更改值。

我假设我们在 Kubernetes 集群上部署了 Gloo，并且没有任何现有的虚拟服务。如果您之前已经安装了`echo`和`foxtrot`，只需运行`kubectl delete ns echo foxtrot`即可。我们将从为我们的`example` API 组部署通用虚拟服务开始。然后，当我们用我们的舵图安装新的服务时，这些航线将被自动选取。

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: app
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - '*'
    routes:
      - matchers:
          - prefix: /
        delegateAction:
          selector:
            labels:
              apiGroup: example
            namespaces:
              - "*"
```

我们可以使用以下命令将其应用于集群:

```
kubectl apply -f vs.yaml
```

现在，我们已经准备好开始使用图表部署服务了。

# 安装 echo

让我们为 echo 创建一个名称空间。

```
kubectl create ns echo
```

现在，我们可以使用初始值进行安装:

```
deployment:
  v1: {}
routes:
  apiGroup: example
  primary:
    version: v1
```

我们将发出以下命令来安装 helm:

```
➜ k create ns echo
namespace/echo created➜ helm install echo  ./gloo-app --namespace echo -f values-1.yaml
NAME: echo
LAST DEPLOYED: Thu Apr 16 16:04:29 2020
NAMESPACE: echo
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

我们可以验证事情是否正常，并测试路线:

```
➜ glooctl check
Checking deployments... OK
Checking pods... OK
Checking upstreams... OK
Checking upstream groups... OK
Checking auth configs... OK
Checking secrets... OK
Checking virtual services... OK
Checking gateways... OK
Checking proxies... OK
No problems detected.➜ curl $(glooctl proxy url)/echo
version:echo-v1
```

太好了！我们的安装已经完成。通过安装我们的图表，我们能够将 Gloo 中的新服务的新路线上线。

为了更好地衡量，让我们也安装狐步舞:

```
➜ kubectl create ns foxtrot
namespace/foxtrot created➜ helm install foxtrot  ./gloo-app --namespace foxtrot -f values-1.yaml
NAME: foxtrot
LAST DEPLOYED: Thu Apr 16 16:08:23 2020
NAMESPACE: foxtrot
STATUS: deployed
REVISION: 1
TEST SUITE: None➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v1
```

与以前相比，这是一个巨大的改进，在以前，我们需要复制粘贴大量的 yaml，并且需要做更多的手工工作来使新的服务上线。让我们看看这如何扩展到推动我们的升级工作流程。

# 开始升级到 echo-v2

我们将使用以下值进行头盔升级，最初部署 echo-v2。请注意，这还没有创建任何金丝雀路线，它只是添加了新的部署:

```
deployment:
  v1: {}
  v2: {}
routes:
  apiGroup: example
  primary:
    version: v1
```

使用`helm upgrade`，我们可以执行这个步骤:

```
➜ helm upgrade echo --values values-2.yaml  ./gloo-app --namespace echo
Release "echo" has been upgraded. Happy Helming!
NAME: echo
LAST DEPLOYED: Thu Apr 16 16:12:03 2020
NAMESPACE: echo
STATUS: deployed
REVISION: 4
TEST SUITE: None
```

我们可以看到，新部署已经创建，v2 pod 现在正在运行:

```
➜ kubectl get pods -n echo
NAME                       READY   STATUS    RESTARTS   AGE
echo-v1-8569c78bcf-dtl6m   1/1     Running   0          9m5s
echo-v2-9c4b669cd-ldkdv    1/1     Running   0          90s
```

我们的路线仍然有效:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1
```

# 进入第一阶段

现在我们要在标题`stage: canary`上设置一个匹配的路由，并路由到新版本。所有其他请求应该继续路由到旧版本。我们可以用另一个`helm upgrade`来部署它。我们将使用这些值:

```
deployment:
  v1: {}
  v2: {}
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    headers:
      stage: canary
```

我们将使用以下命令进行升级:

```
➜ helm upgrade echo --values values-3.yaml  ./gloo-app --namespace echo
Release "echo" has been upgraded. Happy Helming!
NAME: echo
LAST DEPLOYED: Thu Apr 16 16:16:29 2020
NAMESPACE: echo
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

现在，在我们的测试中，我们将能够开始使用金丝雀路线:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/echo -H "stage: canary"
version:echo-v2
```

# 进入第二阶段

正如我们上面讨论的，最后一组值将为阶段 2 设置我们的加权目的地，但是将在金丝雀路线上将权重设置为 0。所以现在，我们可以做另一个头盔升级来改变重量。如果我们想改变权重，使 50%的流量流向上游的金丝雀，我们可以使用这些值:

```
deployment:
  v1: {}
  v2: {}
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    weight: 50
    headers:
      stage: canary
```

这是我们升级的命令:

```
➜ helm upgrade echo --values values-4.yaml  ./gloo-app --namespace echo
Release "echo" has been upgraded. Happy Helming!
NAME: echo
LAST DEPLOYED: Thu Apr 16 16:19:26 2020
NAMESPACE: echo
STATUS: deployed
REVISION: 6
TEST SUITE: None
```

现在，这些路线的行为符合我们的预期。金丝雀路线仍然存在，我们已经将主要路线上一半的流量转移到新版本:

```
➜ curl $(glooctl proxy url)/echo -H "stage: canary"
version:echo-v2➜ curl $(glooctl proxy url)/echo
version:echo-v2➜ curl $(glooctl proxy url)/echo
version:echo-v1
```

# 完成阶段 2

让我们更新路由，以便 100%的流量都流向新版本。我们将使用这些值:

```
deployment:
  v1: {}
  v2: {}
routes:
  apiGroup: example
  primary:
    version: v1
  canary:
    version: v2
    weight: 100
    headers:
      stage: canary
```

我们可以使用以下命令进行部署:

```
➜ helm upgrade echo --values values-5.yaml  ./gloo-app --namespace echo
Release "echo" has been upgraded. Happy Helming!
NAME: echo
LAST DEPLOYED: Thu Apr 16 16:21:37 2020
NAMESPACE: echo
STATUS: deployed
REVISION: 7
TEST SUITE: None
```

当我们测试路由时，我们现在看到所有流量都流向新版本:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v2➜ curl $(glooctl proxy url)/echo
version:echo-v2➜ curl $(glooctl proxy url)/echo
version:echo-v2➜ curl $(glooctl proxy url)/echo  -H "stage: canary"
version:echo-v2
```

# 退役 v1

我们工作流程的最后一步是淘汰旧版本。我们的价值观可以归结为:

```
deployment:
  v2: {}
routes:
  apiGroup: example
  primary:
    version: v2
```

我们可以使用以下命令进行升级:

```
➜ helm upgrade echo --values values-6.yaml  ./gloo-app --namespace echo
Release "echo" has been upgraded. Happy Helming!
NAME: echo
LAST DEPLOYED: Thu Apr 16 16:24:05 2020
NAMESPACE: echo
STATUS: deployed
REVISION: 8
TEST SUITE: None
```

我们可以确保我们的路线是健康的:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v2
```

我们可以确保我们的旧舱被清理干净:

```
➜ kubectl get pod -n echo
NAME                      READY   STATUS    RESTARTS   AGE
echo-v2-9c4b669cd-ldkdv   1/1     Running   0          12m
```

# 结束语

就是这样！我们已经使用 Helm 安装了我们的服务，然后通过执行`helm upgrade`和定制值来执行我们的整个两阶段升级工作流。

从这里，我们现在可以看到两种不同的改进。

首先，我们需要在图表中引入更多的值，以便开始支持更多的工作流、路由功能和开发人员用例。我们可能需要定制很多东西，比如部署映像、资源限制、启用的选项等等。我们可能还需要支持多条路线和替代路径。希望在这一点上，您确信我们可以将这些作为我们迄今为止工作的扩展来处理，并且我们可以在未来的部分中对此进行深入研究。

其次，我们现在已经准备好完全集成到 CI/CD 流程中。许多用户通过定制 helm 值和使用自动化(如 Helm operator 或 Flux project)来与他们的生产集群进行交互，以便在事实的来源(通常是 git repo)发生变化时帮助将这些更新交付给集群。有了 Helm，在我们的平台上加入和升级新服务已经变得微不足道，并且可以通过 GitOps 实现自动化。

# 加入 Gloo 社区

除了企业客户群之外，Gloo 还有一个庞大且不断增长的开源用户社区。要了解更多关于 Gloo 的信息:

*   检查一下[回购](https://github.com/solo-io/gloo)，在那里你可以看到代码和文件问题
*   查看[文档](https://docs.solo.io/gloo/latest)，其中有大量的指南和例子
*   加入 [slack 频道](http://slack.solo.io/)并开始与 Solo 工程团队和用户社区聊天

如果你想与我联系(反馈总是被感激！)，你可以在 Slack 上找到我或者发邮件给我 [rick.ducott@solo.io](mailto:rick.ducott@solo.io) 。
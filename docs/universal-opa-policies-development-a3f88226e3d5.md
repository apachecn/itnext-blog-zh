# 通用 OPA 政策开发

> 原文：<https://itnext.io/universal-opa-policies-development-a3f88226e3d5?source=collection_archive---------1----------------------->

# 介绍

编写 [**【减压阀】**](https://www.openpolicyagent.org/docs/latest/#rego) 文件以及如何解开与 [**看门人**](https://github.com/open-policy-agent/gatekeeper)vs[**Conftest**](https://github.com/open-policy-agent/conftest)验证 Kubernetes 资源的繁琐过程背后的故事。

如今，使用符合策略的 Kubernetes 集群已经成为许多公司关注的焦点，尤其是如果你正在走向流行的认证，如信息安全管理的 **ISO/IEC 27001** 。

# 在 Otomi 容器平台中实际使用 OPA

正如你们中的一些人可能已经知道的，kubernetes 本地的 **PodSecurityPolicy** 资源将被弃用，参见 [*此处*](https://github.com/kubernetes/kubernetes/pull/97171) *和* [*此处*](https://docs.google.com/document/d/1VKqjUlpU888OYtIrBwidL43FOLhbmOD5tesYwmjzO4E/edit#)*——这为类似于 [**开放策略代理**](https://www.openpolicyagent.org/) 这样的外部项目留下了余地，它们将被用作开发和执行策略规则的新标准。*

*[Otomi](https://otomi.io/) 正在使用 **OPA** 作为提供政策执行的标准，因为它在 kubernetes 社区中很受欢迎，易于使用，也是为了未来的项目开发计划。*

*[Otomi 容器平台](https://otomi.io/)的基础是易于使用并提供清晰的集成，允许开发人员轻松扩展任何平台功能，并从头开始为一切提供集成的安全性。—访问 redkubes.com了解更多信息。*

# *OPA 生态系统常识*

*![](img/522f1d6bb01030f084f658ca9ce93f9f.png)*

*照片由[伊莉莎·卡尔韦 B.](https://unsplash.com/@elisa_cb?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄*

*决策是通过**准入控制器**来处理的，比如 **OPA** **kube-mgmt** 项目或者 [**Gatekeeper**](https://github.com/open-policy-agent/gatekeeper) **，**，我马上会谈到这些，但是也要记住，我们可以使用 [**减压阀查询**](https://www.openpolicyagent.org/docs/latest/#rego) 语言，使用静态分析工具，比如[**【Conftest】**](https://github.com/open-policy-agent/conftest)对任何普通文件进行验证。conftest 支持的格式列表包括但不限于:json、yaml、Dockerfile、INI 文件、XML 等。*

*问题来了，[](https://github.com/open-policy-agent/conftest)**和**看门人**正在分道扬镳，虽然他们说着相同的语言，但两人在某些方面存在分歧。***

# ***集群内与静态资源策略之战***

***![](img/877743bc0a43dd6d17ed3e5c0b3826bd.png)***

***由 [jean wimmerlin](https://unsplash.com/@jwimmerli?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片***

***在策略受限的环境中工作就像为非特权用户打开“家长控制”,允许管理员决定哪种资源和设置对他们的 Kubernetes 集群最安全。***

***从应用程序开发人员的角度来看，拒绝访问部署一些资源意味着他/她没有遵守为该环境强加的规则，并且应该决定找到并修复该设置中缺少的链接。***

***另一方面，策略管理员/开发人员努力寻找正确的实施策略，并根据期望的状态调整策略参数，或者在策略实施没有意义的情况下允许某些排除(考虑系统名称空间、云供应商特定的名称空间、默认情况下应该避免干预的任何内容)。政策采纳没有金科玉律，如果有什么不对劲，你要负责克服自己的错误。***

***我将从针对任何种类的 YAML 资源运行策略检查的简单用例开始，然后继续讨论关于集群内 Kubernetes 准入审查对象的更多细节。***

## ***行动中的冲突***

***通过一个简单的命令，我们可以测试舵图表是否违反了所提供的策略文件夹中的任何规则。***

```
*****$** helm template stable/keycloak | conftest test --policy ./policies/   --all-namespaces -**FAIL** - Policy: container-limits - container <keycloak> has no resource limits
**FAIL** - Policy: container-limits - container <keycloak-test> has no resource limits**162 tests, 160 passed, 0 warnings, 2 failures, 0 exceptions*****
```

***生成的 yaml 文件被传输到 conftest 中，并逐个测试策略。***

***通过检查日志消息，我们发现**容器限制**策略将两个资源标记为失败——现在我们所要做的就是修改模板，为指定的容器提供“敏感”数量的资源限制，我们的策略检查将成功通过！好哇..***

***如果我们想采用新的 helm 应用程序，但不想在集群中部署任何东西，除非*很好地检查了*是否有任何违规，这是非常有用的。***

***Conftest 支持使用`--data`选项向策略传递值，这允许策略设计者通过参数配置不同的设置。***

*****参数**可以帮助我们控制为资源创建可配置规则的任何方面。***

***我一会儿会回到这个话题上…***

## ***运行看门人***

***Gatekeeper 正在成为 Kubernetes 中实施安全策略的新标准，支持一个广泛的生态系统来传播思想。***

***实施策略决策的工作方式是使用一个验证 web 挂钩，拦截任何由 api-server 验证的请求，并检查请求有效负载是否满足已定义策略的要求，否则拒绝它。***

***尝试在启用了网关守护设备的群集中创建不一致的资源对象，会导致出现一条错误消息，解释被拒绝的请求。***

```
*****$** helm template charts/redis | kubectl apply -f -

secret/redis created
configmap/redis created
service/redis-master createdError from server (**[denied by banned-image-tags]** Policy: banned-image-tags - container <redis> has banned image tag <latest>, banned tags are {"latest", "master"}
**[denied by psp-allowed-users]** Policy: psp-allowed-users - Container redis is attempting to run as disallowed user 0\. Allowed runAsUser: {"rule": "MustRunAsNonRoot"})error when creating "redis/templates/redis-master-statefulset.yaml": admission webhook "validation.gatekeeper.sh" denied the request:[...]***
```

***正如我们所看到的，创建了一些资源，但是命令失败了，因为 admission webhook 拒绝了。***

***为了使这个资源有效，对**图像标签**和 **securityContext** 字段做一些小的调整就可以了。***

***[有关非根/ securityContext.runAsUser 的更多信息… ]***

# ***把关者背景下的政策制定***

***与 Conftest 使用的普通策略文件类似， [**网守策略库**](https://github.com/open-policy-agent/gatekeeper-library) 的工作原理是将 kubernetes **CRDs** 中一组包装好的 rego 文件加载到内存中，称为 **ConstraintTemplates** 。***

***为了对某些资源类型实施策略，我们需要实例化一个**约束** ( **CR** resource)，它的行为类似于带有所需值或参数的蓝图。***

***配置和约束资源的网关守护设备设置示例:***

```
***--- # Gatekeeper **Config**apiVersion: config.gatekeeper.sh/v1alpha1
kind: Config 
metadata: 
  name: config 
  namespace: gatekeeper-system 
spec: 
  match: 
  - excludedNamespaces: 
    - gatekeeper-system 
    - kube-system 
    processes: 
    - '*' 
sync: 
  syncOnly: 
  - group: "" 
    kind: Pod 
    version: v1--- # ContainerLimits **Constraint** apiVersion: constraints.gatekeeper.sh/v1beta1
kind: ContainerLimits
metadata:
  name: containerlimits
spec:
  match:
    kinds:
    - apiGroups:
        - apps
        - ""
      kinds:
        - DaemonSet
        - Deployment
        - StatefulSet
        - Pod
 parameters:
   container-limits:
     enabled: false
     cpu: "2"
     memory: 2000Mi***
```

***到目前为止，这看起来很容易——我们决定对除“gatekeeper-system”和“kube-system”之外的所有名称空间启用该策略，gatekeeper 将测试我们所有容器的资源限制。***

***继续..这个政策的定义在哪里？***

*****减压阀规则**在名为**约束模板**的 CRD 文件中定义，它们需要在**约束**实例之前部署。***

```
***--- # **ConstraintTemplate** for container-limits policyapiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:  
  name: containerlimits  
spec:  
  crd:    
    spec:      
      names:        
        kind: ContainerLimits      
      validation:        # Schema for the `parameters` field
        openAPIV3Schema:          
...
targets:    
- target**:** admission.k8s.gatekeeper.sh      
  **libs:**
    - |-
      package lib.utils
      ...
  **rego:** |-        
    package containerlimits
    ...***
```

***为了简化这个例子，我删除了我们文件的一部分，但是你也可以在 [Gatekeeper Library repo](https://github.com/open-policy-agent/gatekeeper-library/blob/master/library/general/containerlimits/template.yaml) 上查看它。***

***`"rego”`属性是实际策略定义所在的地方，任何依赖库都可以在`"libs"`的列表中声明。***

***我们现在不会深入研究减压阀违规规则是如何工作的，但是你可以享受学习一种受几十年前的 Datalog 启发的强大查询语言的乐趣。***

# ***统一规划***

***虽然创建新策略的过程似乎很重要，但幸运的是有一个简单的 CLI 工具可以加快我们的工作，它叫做 [Konstraint](https://github.com/plexsystems/konstraint/) 。***

***您可以在**减压阀文件**中定义规则，而 **Konstraint** 会将我们的策略和所有依赖库包装在必要的 **CRD** **文件**中，以供看门人使用。Konstraint 拥有处理各种 kubernetes 本地对象的核心库函数，这使得它成为减压阀开发不可或缺的工具。***

***好的，这意味着我们只关心写**减压阀文件**，我们可以在 Gatekeeper 和 Conftest 中测试它们。***

## ***OPA 社区中的开放问题***

***虽然这是事实，但在深入研究 Gatekeeper 和 Conftest 之后，我发现了一些相互冲突的设计概念，它们完全破坏了这种统一的策略开发思维模式。***

***在两个库中定义参数的方式是不同的，Gatekeeper 有一个限制性的解析器，不允许在 rego 定义中任意导入。***

***在这里的[讨论中，有人试图改变这一硬性限制](https://github.com/open-policy-agent/gatekeeper/issues/1046)！***

## ***减压阀统一后的现状***

> ***为了再次呼应这个主题，我们对减少样板文件和简化制定政策的过程感兴趣。***
> 
> ***[相同的减压阀==不同的上下文]***

***使用一个**通用减压阀策略文件集**并使用相同的源代码来测试“静态文件”，这些文件是从 CI 构建或跨任何“把关上下文”生成的，其中对象是通过 kubernetes api 创建的。***

***让我们切入正题，说我们已经做到了，并在 [Otomi](https://otomi.io/) 中交付了一个工作解决方案。通过使用可用的社区工具，大量的集成工作和大量的定制添加。***

***我们现在有了一个为我们的 Kubernetes 集群定义的丰富的策略和效用函数集合——您可以在 [otomi-core 库](https://github.com/redkubes/otomi-core/tree/master/policies/)中浏览它们。***

## ***Istio 如何在背景中改变物体***

***到目前为止，我们已经了解了减压阀政策和把关者的所有本质细节，但总会有外部因素改变世界的状态，你会发现自己处于一个封闭的盒子里，无处可去。***

***当使用 [**Istio mesh**](https://istio.io/latest/) 进行联网时，这种情况就变成了一场噩梦。实际上[**Istio**](https://istio.io/)**正在通过将一个 Sidecar 容器注入到启用了服务网格通信的名称空间中的所有 pod 来创建一个资源“子空间”。*****

*****这种容器有时会干扰安全**约束**设计策略。*****

# *****Otomi 策略功能*****

******—参数、注释、排除、基线参数。******

*****为了创造一定的灵活性，我们进一步扩展了策略异常功能，检查每个被分析资源的粒度注释信息。*****

*****回到前面描述的**参数**想法，如果策略文件可以读取 kubernetes 对象(或普通静态文件)作为原始输入，为什么不通过注释我们想要跳过检查的资源来允许某些异常，类似于看门人的 **excludeNamespace** 选项。*****

*****减压阀有一个丰富的内置库系统，是一种强大的语言，允许我们轻松地为这个设计创建健壮的实用函数。*****

*****举例来说，使用以下注释将确保从一个或多个策略检查中排除整个 pod 或 pod 中的某些容器:*****

```
*****# Ignore Annotation for entire pod
**policy.otomi.io/ignore:** psp-allowed-repos# Ignore Annotation for Istio sidecar based containers 
**policy.otomi.io/ignore-sidecar:** psp-allowed-users,psp-capabilities# Ignore Annotation for specific container (name=app-container)
**policy.otomi.io/ignore/app-container:** banned-image-tags# Parameters Annotation to extend policy configuration
**policy.otomi.io/parameters.psp-allowed-repos: '**{"repos":["*"]}'*****
```

*****排除的另一个示例是通过从基线设置中禁用策略来完全关闭策略:*****

```
*****# baseline configurationpolicies:
  **container-limits:**
    enabled: false
    cpu: '2'
    memory: 2Gi
  **banned-image-tags:**
    enabled: false
    tags:
      - latest
      - master
 **psp-allowed-users:**
    enabled: true
    runAsUser:
      rule: MustRunAs
      ranges:
        - min: 1
          max: 65535
 ** psp-allowed-repos:**
    enabled: true
    repos:
      - docker.io
      - gke.otomi.cloud
      - aks.otomi.cloud
      - eks.otomi.cloud*****
```

*****查看我们在 [otomi-core 仓库](https://github.com/redkubes/otomi-core/tree/master/policies/)中收集的其他策略。*****

# *****结论*****

*****为了确定现状 kubernetes 策略的设计仍在向新的高度发展，我可以想象在不久的将来会有越来越多的项目从类似于 [policy-hub](https://github.com/policy-hub/policy-hub-cli) 的注册中心共享相同的策略。*****

*****在花了很多时间反复研究政策定义后，我认为就统一的减压阀达成共识是光明未来的关键。*****

*****—乐意分享我的想法，敬请期待**第二部。*******

*****谢谢大家！*****
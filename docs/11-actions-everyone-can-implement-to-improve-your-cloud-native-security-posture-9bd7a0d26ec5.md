# 每个人都可以采取的 11 项行动来改善他们的云原生安全状况

> 原文：<https://itnext.io/11-actions-everyone-can-implement-to-improve-your-cloud-native-security-posture-9bd7a0d26ec5?source=collection_archive---------4----------------------->

如何保护您的中小型云基础架构

![](img/522a4883ae3b2cb8ee152f2f66427e54.png)

由 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的 [CHUTTERSNAP](https://unsplash.com/@chuttersnap?utm_source=medium&utm_medium=referral) 拍摄

**设置:**您在一个中小型组织中维护着一个中小型 Kubernetes 集群，有几个应用程序，几乎没有专用的开发运维/SRE 资源。那么，从哪里开始保护这些 API、pod 和容器呢？这里有 11 个捕捉低挂水果的动作，提高你的 Kubernetes 安全姿态。他们没有排名，但有些可能会参考其他人。

# 1.创建资产清单

“如果你不知道你有什么，你怎么捍卫它？”

更新资产清单对于跟踪组织中的资产至关重要。这至少应该是组织使用和维护的不同软件组件的列表、当前版本、它们是现成的、定制的还是定制的、在哪里可以找到它们(例如，到文档、存储库等的链接)。).

这对于分布式系统尤其重要，在分布式系统中，软件组件作为容器化的工作负载分布在多个微服务上。理想情况下，资产清单还应该包含给定组件通信的服务、它使用的 API 等的概述。

Anchore Syft 是一个 CLI 工具和 Golang 库，用于从容器映像和文件系统生成软件材料清单(SBOM)。Syft 支持来自多个生态系统的包和库，如 APK、DEB、RPM、Ruby Bundles、Python Wheel/Egg/requirements . txt、JavaScript NPM/Yarn、Java JAR/EAR/WAR、Jenkins plugins JPI/HPI、Go 模块等。，以及像 Alpine、BusyBox、RedHat 和 Debian/Ubuntu 这样的 Linux 发行版:[https://github.com/anchore/syft](https://github.com/anchore/syft)

# 2.创建一个威胁模型

一般来说，威胁建模允许您识别威胁，了解每个威胁的影响，并为它们制定缓解策略。目标是突出您的 IT 生态系统中的风险。

在 Kubernetes 中，你会想要描绘出不同的组件交互(api-server，etcd，kubelet，kube-proxy，等等)。)因为这些在默认情况下不一定是安全的。

不同的威胁参与者将具有不同的能力，从广义上讲，我们将最终用户、内部攻击者和特权攻击者区分开来。终端用户通常从群集外部连接到某个入口，而内部攻击者在群集内部拥有一些有限的(不一定是 root 权限)访问权限:例如，pod 中的外壳。另一方面，有特权的攻击者可以访问 api-server(想想 kubectl 和 sysadmin 特权)。

对您的基础架构和应用程序进行威胁建模。在 Kubernetes 中，一个应用程序可能在多个名称空间的多个 pods 中有多个组件，因此在应用程序级别上规划风险也很重要。如果攻击者设法逃离一个 pod 并访问底层节点，该节点上的其他 pod 也将可见并很容易受到危害。

您不会发现威胁模型中的所有内容，但它会让您对生态系统中的风险有一个总体了解，并为它们提供一个缓解策略。

威胁建模有多种详细的框架。例如，参见微软的 STRIDE 框架或 Tony UcedaVélez 和 Marco M. Morana 的关于 PASTA(攻击模拟和威胁分析过程)威胁建模的书:[“以风险为中心的威胁建模:攻击模拟和威胁分析过程。”](https://www.amazon.com/Risk-Centric-Threat-Modeling-Simulation/dp/0470500964)

# 3.将基础设施用作代码(IaC)

使用人类可读和机器可使用的模板文件来调配和管理云资源。在较小的环境中，您可能会满足于使用 Gitlab 的一键式 Kubernetes 集成在 CI/CD 管道内部或外部提供一个集群并安装带有`kubectl apply -f resource.yaml`的实用程序。

在这种方法中，您会失去很多可见性。IaC 设置为不同的基础设施相关资源*及其设置*提供了清晰的参考。您不必查看 web 控制台或依赖 kubectl 来处理“所有事情”它也比“点击操作”方法更加稳定和安全。如果您在 web 控制台中更改了错误的设置，将会损坏东西。IaC 为您提供了一个更加健壮的工作流，尤其是在与 VCS(例如 Git)结合使用时。当谈到集群强化时，我们希望对于不同的配置有一个真实的来源。如果您设置了名称空间、策略、访问控制、安全机制等。，有了 IaC，你总是知道它的当前配置和版本存在于什么地方。

IaC 会在您的工作流中产生开销，但是 ROI 是巨大的，尤其是在保护您的云生态系统时。

# 4 使用名称空间对工作负载进行逻辑隔离

使用名称空间来加强应用程序的逻辑隔离。通过将应用程序隔离在各自的名称空间中，我们可以通过预定义的规则(网络策略)来管理通信

# 4.1 资源配额

资源配额可以应用于资源和命名空间级别。资源配额提供了限制在给定命名空间(或给定应用程序)中可以请求的资源总数(CPU/内存)的约束。这对于阻止加密挖掘攻击至关重要，在加密挖掘攻击中，攻击者消耗集群资源来挖掘加密货币，或者限制内存泄漏的后果以及应用程序可能出现的其他性能问题。

# 5 使用基于角色的访问控制(RBAC)

在 Kubernetes 中实现“最小特权原则”的第一步是使用 RBAC。RBAC 是一种基于授予用户或组的角色来管理对 API 和资源的访问的模型。RBAC 建立在角色和角色绑定的基础上，其中角色是权限的集合，而角色绑定将权限与主题相关联。

以下代码片段描述了一个 Kubernetes 角色，用 Terraform 的配置语言 HCL 编写。开发人员角色有两种不同的规则，一组是他们可以做任何事情的资源，另一组是他们只能读取的资源。

```
resource "kubernetes_cluster_role" "dev" { metadata { name = "developer" } rule { api_groups = ["*"] resources = var.rbac_developer_allowed_resources verbs = ["*"] } rule { api_groups = ["*"] resources = var.rbac_developer_readonly_resources verbs = ["get", "list", "watch"] }}
```

集群角色只是一个全局角色(为所有名称空间定义一次)，而我们更喜欢在每个名称空间级别上保留角色绑定本身:

```
resource "kubernetes_role_binding" "developer_binding" { metadata { name = "developer-binding" namespace = "namespace0" } role_ref { api_group = "rbac.authorization.k8s.io" kind = "ClusterRole" name = "developer" } subject { kind = "User" name = "Martin" api_group = "rbac.authorization.k8s.io" }}
```

这为用户“Martin”提供了名称空间“namespace0”中开发人员角色的权限。这意味着该用户不能访问其他名称空间中的任何资源。除非在这些命名空间中也创建了 RoleBinding。

Kubernetes 支持各种主题，LDAP，Keycloak 等。

链接:

*   [kubernetes 文档](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
*   [地形上的 RBAC](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/role)

# 6.实施网络策略

网络策略允许您指定允许哪些命名空间中的哪些 pod 或 IP 相互通信以及通过哪些端口通信。您的网络插件需要支持网络策略资源。

网络策略可用于在 IP 和端口级别(OSI 第 3 层和第 4 层)管理入口和出口。我建议至少使用出口策略。如果一个 pod 遭到破坏，攻击者只能通过网络策略中明确定义的端口向这些组件发送出站流量。虽然您应该同时使用入口和出口，但我们通常没有对入口流量的不同来源和端口的完整概述。明确定义允许这个 pod 与哪些应用程序对话要容易得多。

网络政策可能会让人不知所措，但这里有一些很棒的资源可供参考:

*   等值网在线 netpol 编辑:【https://editor.cilium.io/ 
*   Ahmet Alp Balkan 的 github repo of reference netpols，载有实际例子和解释清楚的理论:[https://github.com/ahmetb/kubernetes-network-policy-recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)

**注意:** Kubernetes 将总是选择可能的最宽松的权限集(包括入口和出口)。如果同一个 pod 存在两个规则，“允许所有内容”和“允许访问特定 IP”，那么 Kubernetes 将对它们进行 and 运算，这样 pod 就由“允许所有内容和允许访问特定 IP”来管理。因此，您必须确保默认策略必须拒绝所有内容，然后其他规则必须开放特定的端口、名称空间和 IP 范围。

# 7.保护好你的集装箱

保护您的容器运行时应该是您的首要任务之一，因为不安全的容器是极好的攻击媒介。

容器应该总是以它们需要的最少特权运行。如果容器中的进程以 root 用户权限运行，则该容器在主机本身上也将具有 root 用户权限。因此，容器应该作为非根用户运行。这里的另一个陷阱是使用`--privileged`标志以 root 身份运行整个容器。

```
apiVersion: v1kind: Podmetadata: name: pod-security-context-demospec: securityContext: runAsUser: 1000 containers: - name: pod image: busybox command: [ "sh", "-c", "sleep 1h" ] securityContext: allowPrivilegeEscalation: false
```

当然，有些容器有特定的需求，要求它们以提升的权限运行。尽管如此，如果不尽可能多地放弃特权，就不应该以特权身份运行容器。PodSecurityContext 对象(在 SecurityContext 字段中设置)是一种更好的方法，它可以查看是否可以使用 addCapabilities 参数为容器提供所需的最低权限。记住还要将 allowPrivilegeEscalation 设置为“false”。大部分强化工作可以在 pod 规范/部署中完成。

容器应该是不可变的。通常，没有什么可以阻止一个正在运行的容器将额外的软件下载到它的文件系统中。攻击者可以利用这一点来安装恶意软件，并使用容器作为攻击的矛头。通过利用只读文件系统，可执行代码只在构建时添加到容器中，而不是在运行时安装，使得容器*不可变*。如果容器需要对本地存储进行写访问，可以根据需要为这些操作添加一个临时文件系统。

安全配置文件应该用来限制容器在功能和权限方面可以做什么。虽然应该为每个应用程序定制安全配置文件，但这会带来大量开销和维护。推荐使用默认的 Docker seccomp 和 AppArmor 配置文件，因为它们引入了一些通用功能和防护栏，而不会对应用程序造成过多干扰。默认的 docker seccomp 配置文件阻塞了大约 300 个对 Linux 内核的系统调用中的 40 多个，而不会对绝大多数容器化的应用程序产生负面影响。如果您的 Kubernetes 节点支持 SELinux，这也应该被认为是启用的。

最大限度地减少容器映像的内容，从而在容器受损时减少攻击面。理想情况下，容器应该只包含应用程序运行所需的包和依赖项。考虑通过使用临时构建的“发行版”基础映像来精简容器映像，因为这些映像只包含应用程序及其运行时依赖项，而不包含软件包管理器、shells 或其他在完整的 Linux 发行版中常见的实用程序。

参见 Liz Rice 的书:集装箱安全

[https://kubernetes . io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

# 8.设置安全扫描

在现代环境中，自动安全扫描是“强制性的”。恶意软件大量存在，并且大多数工具易于添加，引入的开销很小。至少，您应该在容器上运行漏洞扫描和依赖扫描。除此之外，您应该有一个工具来检测源代码中的凭证，或者 SAST/DAST 工具，如 Sonarqube 或 OWASP Zap 基线扫描。

# 图像扫描

图像扫描可以作为识别漏洞和违反最佳实践/内部策略的一种方式。通常，我们可以区分内嵌扫描、在 CI/CD 管道中运行安全扫描工具和注册表扫描。注册表扫描是扫描存储在容器注册表中的图像的过程，而不是在构建时。这个特性内置于容器注册中心和大多数公共云中。有许多容器图像扫描工具，但像 Clair 或 Anchore 这样的开源工具有丰富的插件解决方案，如 Jenkins 插件和 Github 操作。例如参见[https://github . com/market place/actions/anchore-container-scan](https://github.com/marketplace/actions/anchore-container-scan)。

其他资源:

*   [sysdig 图像扫描最佳实践](https://sysdig.com/blog/image-scanning-best-practices/)

# 依赖性扫描

现代软件项目有许多依赖，它们的依赖有依赖(看看你，NPM)。随着有毒的 NPM 包裹等供应链攻击变得越来越普遍，执行依赖性扫描的需求变得突出。检查 [Snyk](https://snyk.io/) (商业)或 [OWASP 依赖检查](https://owasp.org/www-project-dependency-check/)(免费)。传统上，OWASP 项目在 NPM 软件包上表现不佳，并使用 NVD 数据作为威胁情报的唯一来源，而 Snyk，一个商业产品，也从其他来源获取威胁情报。

# 9.使用准入控制器

准入控制器是 Kubernetes——本地组件，允许我们定义允许在集群中运行什么。你可以用 Webhooks 编写自己的准入控制器，或者使用例如 [OpenPolicyAgent](https://www.openpolicyagent.org/) 。

OPA 是一个“策略即代码”框架，允许您实施定制策略，例如要求白名单中的主机名，或者确保同一名称空间中没有两个入口资源具有相同的主机名。OPA gatekeeper 是一个准入控制器，您可以使用它来验证从自定义策略到 PodSecurityContexts 的所有内容。网守库有一组广泛的策略示例。

另请参见:

*   [通过准入控制器上的图像扫描屏蔽您的 Kubernetes 运行时](https://sysdig.com/blog/image-scanning-admission-controller/)

# 10.启用审核日志记录

审计日志跟踪谁在何时做了什么。根据 [Kubernetes 文档](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)，审计日志允许集群管理员回答以下问题:

*   发生了什么事？
*   什么时候发生的？
*   是谁发起的？
*   它发生在什么地方？
*   在哪里观察到的？
*   它是从哪里发起的？
*   它要去哪里？

要在集群中启用审计日志记录，您必须定义一个审计策略。记录事件后，会将其与设置事件审核级别的规则列表进行比较:

*   无—不记录符合此规则的事件。
*   元数据—日志请求元数据(请求用户、时间戳、资源、动词等)。)而不是请求或响应主体。
*   请求—记录事件元数据和请求正文，但不记录响应正文。这不适用于非资源请求。
*   RequestResponse —记录事件元数据、请求和响应正文。这不适用于非资源请求。

这是摘自 Sysdig 的默认审计策略脚本:

```
apiVersion: audit.k8s.io/v1beta1 # This is required.kind: Policy# Don't generate audit events for all requests in RequestReceived stage.omitStages: - "RequestReceived"rules: *# Log "pods/log", "pods/status" at Metadata level* - level: Metadata resources: - group: "" resources: ["pods/log", "pods/status"] *# Log the request body of configmap changes in kube-system.* - level: Request resources: - group: "" *# core API group* resources: ["configmaps"] *# This rule only applies to resources in the "kube-system" namespace.* *# The empty string "" can be used to select non-namespaced resources.* namespaces: ["kube-system"] *# A catch-all rule to log all other requests at the Metadata level.* - level: Metadata *# Long-running requests like watches that fall under this rule will not* *# generate an audit event in RequestReceived.* omitStages: - "RequestReceived"
```

如本例所示，您可以将审核级别指定到不同资源上的特定动词。该策略将首先从顶部评估事件，并从第一个匹配的规则设置事件的审核级别。在底部，所有其他事件都记录在元数据级别(除了`omitStages`中的请求)。请注意，审计事件需要记录在某个地方或由审计接收器处理。大多数公共云提供商提供的托管 Kubernetes 产品都启用了审计日志。

# 10.1 Falco 异常检测

[Falco](https://falco.org/) 是一款开源的 Kubernetes 威胁检测引擎。Falco 检测运行时的意外应用程序行为，并就此发出警报。默认规则集已经相当广泛了，但是还可以创建自定义规则和例外，以更好地反映您的应用程序。Falco 引擎将检测并警告你任何事情，从一个意外的系统调用到一个 pod 突然产生一个外壳。

Falco 可以设置为审计事件接收器，通过 Falco 引擎处理审计事件。这对于捕获审计事件中的异常非常有用，而不仅仅是将日志保存在文件中，保存时间为建议的六个月。因为据说要花六个月的时间才能发现漏洞。通过 Falco exporter 或 Falco sidekick 等支持应用程序，Falco 可以将其事件导出到 Prometheus、Splunk、Grafana、Logstash ++或通过 slack、电子邮件、短信等发送通知。多个可以组合使用。

# 11.设置有效的监控

在理想情况下，当观察到的行为与预期不符时，您可以使用日志进行调试和审计，使用指标进行应用程序监控和警告。确保充分覆盖业务关键型应用程序，以便能够及早检测到异常行为。如果你使用的是[普罗米修斯堆栈](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)，记住一个没有相应警告的 Grafana 面板是没有意义的。我们大多数人不会坐在那里看着仪表盘，等着事故发生。

# 结论

向 Kubernetes 的转变、其他云原生技术和容器的主流采用，让基础架构安全面临挑战，因为我们正在脱离传统基础架构安全的概念，被迫适应新的攻击面。通过在基于 Kubernetes 的生态系统中实现这些操作，攻击面被极大地最小化，而不必购买昂贵的安全解决方案或引入不必要的复杂性。但是请记住，基础设施安全只能带你走这么远。您仍然需要保护应用程序本身！

如果您想了解更多关于这些概念的知识，我强烈推荐 Kaizhe Huang 和 Pranjal Jumde 的书 [Learn Kubernetes Security](https://www.packtpub.com/product/learn-kubernetes-security/9781839216503) 。
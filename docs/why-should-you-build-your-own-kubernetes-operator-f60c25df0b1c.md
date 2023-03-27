# 为什么您应该构建自己的 Kubernetes 运营商

> 原文：<https://itnext.io/why-should-you-build-your-own-kubernetes-operator-f60c25df0b1c?source=collection_archive---------3----------------------->

## 这比你想象的更容易，也更有益

![](img/fd2921ba4dfd2357a35603d257ace41a.png)

贷项:CorrieMiracle/P [ixabay](https://pixabay.com/photos/beach-sand-ocean-sand-castle-2445836/)

Kubernetes Operator 是 Kubernetes 社区中最流行的技术之一。截至 2020 年 5 月， [Awesome Operators](https://github.com/operator-framework/awesome-operators/blob/master/README.md) repo 列出了 145 家运营商。Kube 上托管的每一个主流软件——比如 MySQL、Kafka 和 elastic search——都有一个或多个运营商。如果您是一个不托管数据库的普通 Kube 用户，您可能会考虑构建一个不相关的操作符，这是完全错误的。

本文讨论了什么是 Kubernetes 操作符，以及为什么所有 Kube 用户都应该考虑为他们的应用程序建立一个操作符。

# Kubernetes 操作简单来说

Kubernetes 操作符是一个遵循 Kube 操作符模式的程序。根据官方 [Kubernetes 网站](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/):

> 操作员是 Kubernetes 的软件扩展，它利用**定制资源**来管理应用程序及其组件。操作员遵循 Kubernetes 原则，特别是**控制回路**

正如您所看到的，操作符模式由两个主要部分组成:

*   **定制资源(CR):**CR 是一个 Kube API 对象，类似于现成的 Kube 资源，如 Pod、部署和服务，但是 Kube 用户定义了它的大部分模式。CR 有规格和状态部分，由 Kube 维护，因此用户可以通过 Kube APIs 创建、读取、更新和删除(CRUD)它。规格部分允许用户用他们各自的语言描述他们应用程序的期望状态。例如，用户可以在 CR 中创建一个名为“replicasPerRegion”的键，它存储每个区域应该有多少个应用程序实例。
*   **控制循环:**控制循环是 Kube 用户实现的一段代码。该循环的工作是不断地与 Kube APIs 交互，并将现实与期望状态(CR)进行比较和协调。因此，它的另一个名字是:调和循环。该循环还将比较和协调的结果放入 CR 的 status 部分。

任何遵循这种模式的程序都是 Kube 操作符。你可能也听说过叫做 [Operator SDK](https://github.com/operator-framework/operator-sdk) 的东西，但是它只不过是一个构建操作符的框架，因此没有必要。

# 我为什么要构建一个运算符

## **面向 Kubernetes 的 OOP**

运算符模式优化和重构容器，就像面向对象编程(OOP)对软件系统所做的那样。它通过提高基于容器的应用程序的模块化、可读性和可重用性来管理复杂性。

像 Java 这样的编程语言带有内置的原语和数据结构，如 integer、map 和 list，允许用户在其上构建复杂的自定义类。正如所有经验丰富的程序员所知道的，根据 OOP 原则，与其使用内置类型`Map<String, String>`来存储学生信息，不如创建如下的类:

```
public class Student {
  private String firstName;
  private String lastName;
  private String id;
}
```

类似地，Kube 为运行容器提供了一组“原语”,如 Pod、副本集和部署。虽然使用它们来快速运行一些容器很方便，但是随着系统复杂性的增加，维护所有容器变得更具挑战性。通过定义您自己的 CR，并让控制循环逻辑创建更多的 Kube“原语”，您可以归档以下内容

*   为了更好的可读性，在 Kube 清单文件中使用更有意义的名称。
*   特定于您的用例的验证。例如，您可以让 Kube 拒绝一个 CR，如果它将一个应用程序的实例数设置为小于 3。
*   Kube“原语”之上的定制部署逻辑例如，强制一个部署与另一个部署具有相同数量的副本。
*   操作人员看不到复杂性，因此他们不需要学习几十个 YAML 文件，只需要学习几个门面 CRs。

## 轻便

使用 Kubernetes 的一个原因是可移植性。它提供了一个通用层，基于容器的应用可以在其上运行，而不管底层基础设施、AWS、IBM Cloud 或您自己的数据中心。运营商进一步增强了便携性。

部署复杂的应用程序并不简单:一些容器可能需要等待其他容器，或者在某些情况下可能需要禁用部分部署逻辑。人们发明了像 Spinnaker 和 Jenkins 这样的连续交付(CD)框架，以及像 Helm 和 Kustomize 这样的工具，来处理这样的复杂性。因此，以一组图像、Kube 清单和部署脚本/图表/配置文件的形式交付应用程序是很常见的。但是让我们面对现实:不同的团队选择不同的工具，不同的客户有不同的 CD 设置。操作员可以将您从为不同设置复制和定制相同部署逻辑的困境中解救出来。

使用 Operator 模式，Kube Operator 的控制循环处理订单和条件的部署逻辑。操作符本身是一个可以在任何 Kube 集群上运行的容器，这使得 Kube 之外的部署依赖性需求最小化。要部署一个应用程序，所有用户需要做的就是创建操作员容器和一个或几个 CRs，其余的由操作员负责。仍然需要 CD 和其他部署工具，但是它们的逻辑归结为几行代码，只需要几分钟就可以从一个移植到另一个。

## 更好的操作

操作员模式旨在模拟和自动化操作员的行为。人类操作员:

1.  根据文档(如操作手册)建立系统的期望状态。例如，有多少实例应该是健康的或者通过了某些检查。
2.  检查当前情况是否符合期望的状态，如果不符合，则继续步骤 3。
3.  采取行动解决问题。
4.  重复步骤 2 和 3，直到达到所需的状态。

对于人工操作员来说，无论他们是 sre 还是购买你的应用程序在现场运行的客户，操作员模式都会将他们的注意力引导到正确的地方。CRs 的规格部分为操作员详细说明了期望的状态，而 CRs 的状态部分显示了现实如何与所述期望的状态相一致。它们的声明式风格也使得状态易于阅读和维护，同时也隐藏了所需状态如何存档的复杂性。最后但同样重要的是，您的应用程序获得了“自我修复”的能力，因为控制循环不会停止，直到达到所需的状态。

## 享受 Kube 生态系统

由于 CRs 也是 Kube API 对象，如 Deployment 和 Pod，它们本质上与 Kube 生态系统中的工具兼容。与其投资你自己的工具来可视化、管理和监控你的应用，不如使用像`kubectl`和 Kube dashboard 这样的工具。与内部操作工具相比，Kube 社区提供了更多的功能、错误修复和教程。此外，人类操作员可以避免花费时间学习新工具，并更快地登船。

## 这很简单

最后，[运营商 SDK](https://sdk.operatorframework.io/build/) 让打造运营商变得简单。它支持建立一个带有 Golang，Ansible 或 Helm 的操作符。我强烈建议从基于 Ansible 的操作符开始，即使你已经熟悉 Golang。在 Ansible 的`k8s`和`template`模块的帮助下，您可以用一个相当简单的 YAML 文件来描述您的操作员的任务，并在几个小时内让一个不平凡的操作员开始工作。

# 结论

如果你在 Kubernetes 上发布复杂的应用程序，Kube Operator 应该是你的首要任务。它使用起来很简单，但从长远来看，它可以改善你的应用程序的结构、可移植性和操作体验，同时节省开发时间。

如果你喜欢这篇文章，[在 Medium 上跟随我](https://medium.com/@nealhu)！我撰写关于分布式系统和软件架构的文章，例如:

*   [在使用 Kustomize](/before-you-use-kustomize-eaa9529cdd19) 之前。流行的 Kubernetes 配置管理工具 Kustomize 的优缺点
*   [类日志记录](/logging-like-a-pro-8cc6ad09e415)。向更好的自动化方向设计软件系统的技巧
*   [自动化友好的软件系统以及如何构建它们](/automation-friendly-software-systems-and-how-to-build-them-7a7c5e3c1a15)。向更好的自动化方向设计软件系统的技巧
*   [极简软件架构](/minimalist-software-architecture-426888684e60)。构建大规模多区域分布式系统的经验教训
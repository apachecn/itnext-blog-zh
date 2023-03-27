# 用地形构建 EKS 集群

> 原文：<https://itnext.io/build-an-eks-cluster-with-terraform-d35db8005963?source=collection_archive---------0----------------------->

![](img/6926c09b389eaa2066df3abfa6f9d333.png)

[更新版本](https://nicosingh.gitlab.io/medium-deploy-eks-cluster-using-terraform/) | [源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/-/blob/master/components.py) |用一种叫做[的奇妙工具制作的图表](https://diagrams.mingrammer.com/)

**🚀2022 年 5 月更新**:鉴于我个人使用下面解释的代码的经验，以及在过去两年中收到的大量反馈，我决定将 Terraform 定义分为两个不同的模块:

*   [Base](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/-/tree/master/base):AWS 中**创建** VPC & EKS 资源的地形模块。
*   [配置](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/-/tree/master/config) : terraform 模块，**配置**EKS 集群中的 Kubernetes 组件(入口控制器、名称空间、点终止处理程序等)。

这一决定背后的原因是将基础设施的*创建*和*配置*拆分为“不同的项目”(将 infra-as-code 视为分布式单元，而不是整体)，以及避免与 Terraform 提供者对位于同一项目中的其他资源创建的一些模块的依赖性相关的一些问题。这种方法为将来使用新工具/平台以更灵活的方式配置基础设施提供了可能性。

## 介绍

这篇文章描述了在 AWS 中使用 Terraform 和一些 AWS 模块创建一个多区域 Kubernetes 集群。具体来说，我们将使用[基础设施作为代码](https://www.hashicorp.com/blog/infrastructure-as-code-in-a-private-or-public-cloud/)来创建:

*   新的 **VPC** 具有多区域公共&私有子网和单个 NAT 网关。
*   一个 **Kubernetes 集群**，基于在私有子网中运行的 Spot EC2 实例，带有一个基于平均 CPU 使用率的自动扩展组。
*   一个**应用负载均衡器(ALB)** 来接受公共 HTTP 调用并将它们路由到 Kubernetes 节点，以及运行健康检查来扩展 Kubernetes 服务(如果需要的话)。
*   集群内部的一个 **AWS 负载平衡器控制器**，用于接收&将来自外部世界的 HTTP 请求转发到 Kubernetes pods。
*   带有 **SSL 证书**的 **DNS 区域**为每个 Kubernetes 服务提供 HTTPS。这个区域将由一个叫做**外部 DNS** 的服务从 Kubernetes 管理。
*   一个**示例应用程序**将部署到我们的集群中，使用一个小的 Helm 图表。

官方 Terraform 模块的使用为我们带来了 AWS 组件编码的简单性，遵循了经验证的提供商的最佳实践(又名*不要重新发明轮子*)，如专用网络或 Kubernetes 集群。

## 要求

*   AWS 帐户，可编程访问。稍后我们将使用这些凭证来配置一些环境变量。
*   Terraform CLI 或 Terraform Cloud。在本文档中，我们使用 1.1.9 版本，但是如果您愿意，也可以随意使用更新的版本。我的建议是使用 [docker 镜像](https://hub.docker.com/r/hashicorp/terraform)或 [tfenv](https://github.com/tfutils/tfenv) ，以简化特定版本的安装和使用。
*   运行 Terraform CLI 的终端，如果使用 Terraform Cloud，则为源代码管理报告。就我个人而言，我使用 CI 管道来实现这一点，以打破计算机运行 Terraform 命令的依赖性，并应用过去部署的历史记录。

💰**预算标注**:创建 EKS VPC&DNS 资源可能会给你的 AWS 月度账单带来一些成本，因为有些资源可能会超出免费等级。因此，在应用任何地形图之前，请注意这一点！。

## 地形结构

在简短的介绍之后，让我们以代码的形式进入我们的基础设施！我们将看到每个步骤所需的地形配置的小片段；随意复制它们，并尝试在自己身上应用这些计划。但是，如果您对完成这项工作感到好奇或不耐烦，请查看由 CI 管道管理的所有 Terraform 配置的存储库[以应用它们。](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform)

Terraform 的第一步是定义与版本、提供商和状态文件后端相关的 Terraform 配置:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/terraform.tf)

✅ **建议**:在对我们的基础设施进行编码时，声明要使用的 Terraform 版本是一个好主意，这样可以避免在将来运行 terraform 时，如果我们使用较新/较旧的版本，可能会影响我们代码的任何重大更改。

✅ **推荐**:在运行 *init* 命令的同时，资源提供者可以被 Terraform 自动处理。然而，使用版本号(如上所示)显式地定义它们是一个好主意，以避免未来版本引起的数据源/资源中断更改。

✅ **推荐** : AWS Terraform provider 配置包括一个 *default_tags* 定义，这是一个很好的选项，可以自动为 AWS 中创建的所有资源创建标签。这些标签不是强制性的，但是对于以有组织的方式[跟踪成本](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)是一个很好的建议。

✅ **推荐**:后台配置几乎是空的，那是有目的的。如果需要的话，建议将这个设置外部化到几个文件中(例如，每个环境有一个配置)。在这种情况下，我们将使用单个 S3 后端，每个 terraform 工作空间有几个状态文件:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/backend.tfvars)

这意味着我们将使用一个名为“我的充满活力的应用程序基础架构”的 S3 存储桶，看起来像这样:

```
s3://my-vibrant-and-nifty-app-infra/
|_environment/
  |_development/
  | |_infra.json
  |_staging/
  | |_infra.json
  |_production/
  | |_infra.json
```

其中每个文件夹代表一个托管 Terraform 远程状态文件的环境(开发、暂存和生产)。

另一方面，建议**避免**在*提供者*块中定义 AWS 凭证。相反，我们可以为此使用环境变量，Terraform 会自动使用这些环境变量来验证 AWS APIs:

```
AWS_ACCESS_KEY_ID=AKIAXXXXXXXXXXXXXXXX
AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
AWS_DEFAULT_REGION=us-west-2
```

⚠️ **重要提示**:这里定义的 S3 桶如果在 AWS 中不存在，Terraform 不会创建。这个 bucket 是通过手动操作或使用 CI/CD 工具运行如下命令在外部创建的:

```
aws s3 mb s3://my-vibrant-and-nifty-app-infra --region us-west-2
```

⚠️ **重要提示**:请记住，S3 存储桶名称必须在全球范围内**唯一**，跨 AWS 客户和地区。运行 *aws s3 mb* 命令时，以及定义 *backend.tfvars* 文件时，尝试为您的 bucket 使用自定义名称。这就是为什么我选择了一个非常个性化的名字“我的充满活力和漂亮的应用基础设施”。

要初始化每个工作区，例如“development ”,我们应该运行以下命令:

```
terraform init -backend-config=backend.tfvars
terraform workspace new development
```

在将来的执行中，我们可以使用以下命令选择现有的工作区:

```
terraform init -backend-config=backend.tfvars
terraform workspace select development
```

现在，我们准备开始将我们的基础设施写成代码了！。

# 局部模块结构

如前所述，文章中描述的所有 Terraform 资源都分布在单个 repo 中托管的两个模块中，称为 **Base** 和 **Config** 。它们将从根文件夹中调用，如下所示:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/main.tf)

模块的这种使用将通过使用 *module.base.cluster_id* 作为第二个模块的输入来创建 Terraform 中的隐式依赖关系，第二个模块是第一个模块的输出，并且依赖于 EKS 集群的创建。换句话说，在 Config 模块中定义的所有资源都将依赖于在 Base 模块中创建的 Kubernetes 集群，而不使用令人厌烦的 [depends_on](https://www.terraform.io/language/meta-arguments/depends_on) 选项。

上面使用的所有变量都在一个名为 [variables.tf](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/-/blob/master/variables.tf) 的文件中定义，该文件位于项目的根目录下。它们也在每个模块文件中定义，并将在下一节中详细解释。

## [基础模块] VPC 创作

让我们从创建一个新的 VPC 开始，在我们的基础设施 terraform 项目中使用 AWS 发布的[官方 VPC terraform 模块](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws),将我们的 EKS 相关资源隔离在一个安全的地方:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/base/network.tf)

正如在前面的代码块中所评论的，我们将创建一个新的 VPC，每个可用区域上有子网，有一个 NAT 网关以节省一些成本，添加一些 EKS 要求的标签[。记住还要为前一个块定义一些变量值文件(例如，每个环境一个):](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html#vpc-subnet-tagging)

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/base-network-development.tfvars)

现在，我们应该准备好使用 Terraform 创建这个 VPC 资源了。如果我们已经运行了 *init* 命令，我们可以使用 *plan* 命令检查 Terraform 要创建或更新的资源:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars
```

然后，在用户确认后，我们可以使用*应用*命令应用这些更改:

```
terraform apply development.tfplan
```

## [基本模块] EKS 集群

下一步是使用[官方 EKS Terraform 模块](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws)创建一个新的 Kubernetes 集群:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/base/eks.tf)

如前面的代码块所示，我们正在创建一个 EKS 集群，该集群使用一个针对 Kubernetes 的 EC2 自动伸缩组，该组由基于 CPU 平均使用率自动伸缩的 Spot 实例组成。

请记住，此 Terraform 配置块使用了在之前的 Terraform 块中定义的一些变量，因此需要将其作为新文件存储在与 VPC 定义文件相同的文件夹中。但是，有些变量是新的，所以我们需要在一个新文件中定义它们相应的值:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/-/blob/master/base-eks-development.tfvars)

正如我们在上面看到的，我们正在定义两个 EKS 节点组(它们将在后台处理 Kubernetes 的 EC2 实例)。一个具有 X86 spot 实例(Intel 和 AMD)，一个具有按需 ARM 实例( [Graviton](https://aws.amazon.com/ec2/graviton/) )。这只是因为我想这样做；**根据您自己的基础设施要求/预算/限制，随时更改它**。

✅ **建议:**为了方便代码阅读和变量文件的使用，最好创建一个单独的 Terraform 配置文件来一次定义所有变量(例如 *variables.tf* )，然后将几个变量值文件定义为:

*   一个单独的 *terraform.tfvars* 文件(由 terraform 命令自动加载),包含所有通用变量值，没有定制值或环境特定值。
*   特定环境或情况 **。带有所有变量值的 tfvars* 文件，这些变量值将特定于特定情况或环境，并将在运行 *terraform plan* 命令时明确使用。

然而，为了简单起见，我们将跳过这些规则，以逐步简化对 AWS 资源创建的每个部分的理解。这意味着我们将运行 *terraform plan* 命令添加每个变量值文件，同时我们编写新的配置块:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars \
  -var-file=base-eks-development.tfvarsterraform apply development.tfplan
```

一旦计划得以实施，我们在 AWS 中就有了一个全新的 EKS 集群！让我们继续这个 Kubernetes 集群的配置，使用另一个 Terraform 模块。

## [配置模块] EKS 集群

由于我们现在处于另一个 Terraform 模块中，并且 EKS 集群已经存在，我们将不得不使用数据源来获取它们的数据。让我们做这件事和其他一些东西:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config/eks.tf)

如你所见，我们在这里:

*   获取我们现有的 EKS 集群作为数据源，以便配置 Kubernetes & Helm Terraform 提供商。
*   为 [EC2 Kubernetes Spot 终止处理程序](https://github.com/aws/aws-node-termination-handler)部署一个舵图，当 Spot 实例被 AWS 自动终止时，它负责重新分配 Kubernetes 对象。

使用两件事所需的变量:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config-eks-development.tfvars)

我们可以像往常一样规划和应用:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars \
  -var-file=base-eks-development.tfvars \
  -var-file=config-eks-development.tfvarsterraform apply development.tfplan
```

Terraform 将负责为我们运行“头盔安装”命令。

## [配置模块] IAM 访问

下一步是配置 AWS 用户进入 EKS 集群所需的任何访问权限。为此，我们将使用 aws-auth ConfigMap:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config/iam.tf)

如果我们仔细阅读 aws-auth [文档](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)，我们还需要在我们的 Kubernetes 集群中配置 RBAC 访问。这就是为什么我们为我们的开发人员用户创建一个 ClusterRoleBinding 对象，自定义对 Kubernetes pods 的 *get* 和 *port-forward* 访问。

这些资源的变量文件如下所示:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config-iam-development.tfvars)

⚠️ **注意**:上面显示的用户 id 是虚构的，当然它们必须根据你的 AWS 账户中的用户组进行定制。请记住，这些用户名 **do** **not** 在创建 EKS 集群或分配 RBAC 访问权限时必须作为 AWS IAM 身份存在，因为它们将只存在于 Kubernetes 集群中。IAM/Kubernetes 用户名关联由 AWS CLI 在[向 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html#create-kubeconfig-automatically)进行身份验证时处理。

为了计划和应用这些定义，我们应该像这样运行:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars \
  -var-file=base-eks-development.tfvars \
  -var-file=config-eks-development.tfvars \
  -var-file=config-iam-development.tfvarsterraform apply development.tfplan
```

## [配置模块]负载平衡器

现在我们可以继续创建一个应用程序负载平衡器(ALB ),来处理对我们服务的 HTTP 请求。ALB 的创建将由 [AWS 负载平衡器控制器](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest)服务负责，该服务将使用 Helm:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config/ingress.tf)

正如你在上面看到的，入口定义使用了一个新的 AWS 颁发的 SSL 证书来提供我们的 ALB 中的 HTTPS，该证书将放在我们的 Kubernetes pods 前面，并且还定义了一些[负载平衡器控制器](https://github.com/aws/eks-charts/tree/master/stable/aws-load-balancer-controller#installing-the-chart)所需的注释。

这个舵图**安装的 Kubernetes 服务不会**在 AWS 中创建任何 ALBs 或 DNS 记录。一旦在 Kubernetes 中使用所需的注释创建了新的入口对象，这些资源将分别由负载平衡器控制器和外部 DNS 服务创建。我们将在文章的最后深入讨论这些注释。

在创建 EKS 群集期间，管理 ALBs 和 Route53 记录所需的 IAM 权限已经授予 LB 控制器和外部 DNS 服务。

⚠️ **注意**:此处显示的 terraform 项目重复使用了在此 Terraform 工作空间之外创建的 DNS 区域(在“dns_base_domain”变量中定义)。这就是为什么我们使用*数据源*来获取现有的 Route53 区域，而不是创建新的*资源*的原因。如果需要，请随意更改，如果您还没有 DNS 资源，请创建新的 DNS 资源。

和其他 Terraform 配置文件一样，这个文件也使用了一些新的变量。因此，让我们为我们的“开发”环境定义它们:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config-ingress-development.tfvars)

然后运行*地形图* & *应用*:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars \
  -var-file=base-eks-development.tfvars \
  -var-file=config-eks-development.tfvars \
  -var-file=config-iam-development.tfvars \
  -var-file=config-ingress-development.tfvarsterraform apply development.tfplan
```

## [配置模块]外部 DNS

下一步是部署 [ExternalDNS](https://github.com/kubernetes-sigs/external-dns) 服务，它将负责管理 Kubernetes 中每个入口定义所请求的 Route53 记录:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config/external-dns.tf)

正如我们在上面看到的，external-dns Helm 图表需要一些与新生成的 ACM 证书相关的注释来提供 SSL 连接，还需要 Route53 基域来创建/修改/删除 Kubernetes 中的记录。

terraform 定义需要一些其他变量来设置日志级别和同步频率:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config-external-dns-development.tfvars)

在用户确认后，将按如下方式应用:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars \
  -var-file=base-eks-development.tfvars \
  -var-file=config-eks-development.tfvars \
  -var-file=config-iam-development.tfvars \
  -var-file=config-ingress-development.tfvars \
  -var-file=config-external-dns-development.tfvarsterraform apply development.tfplan
```

## [配置模块] Kubernetes 名称空间

最后一步——实际上不是强制性的，但建议这样做——是定义一些 Kubernetes 名称空间来分隔我们的部署，并在我们的集群中实现更好的应用程序管理和可见性:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config/namespaces.tf)

这个配置文件期望在我们的 EKS 集群中创建一个名称空间列表:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform/blob/master/config-namespaces-development.tfvars)

这可以应用为:

```
terraform plan -out=development.tfplan \
  -var-file=base-network-development.tfvars \
  -var-file=base-eks-development.tfvars \
  -var-file=config-eks-development.tfvars \
  -var-file=config-iam-development.tfvars \
  -var-file=config-ingress-development.tfvars \
  -var-file=config-external-dns-development.tfvars \
  -var-file=config-namespaces-development.tfvarsterraform apply development.tfplan
```

就是这样！我们终于有了一个生产就绪的 EKS 集群，可以通过公共 IP 访问来托管应用程序🎉。记得访问下面的&[Config](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform-config)库，全面了解所有这些 Terraform 配置，以及在 AWS 中应用它们的示例 CI 管道。

## [奖励]示例应用程序部署

作为奖励，我将为[留下一个样例应用程序](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform-sample-app)的链接，它基于这个 [docker 映像](https://github.com/nginxinc/NGINX-Demos/tree/master/nginx-hello)，使用 Helm 将一个非常小的容器部署到我们新的 Kubernetes 集群中。它还包含一些 CI 作业，可以帮助您熟悉 *aws eks* 和 *helm* 命令。

![](img/3a3dc151b1ecbbf97adfeb179d940076.png)

如前所述，每个应用程序内部的入口对象拥有创建 ALB 和管理 Route53 记录的魔力:

[源代码](https://gitlab.com/nicosingh/medium-deploy-eks-cluster-using-terraform-sample-app/blob/master/helm/templates/ingress.yaml)

以*Alb . ingress . kubernetes . io*开头的注释负责通过 LB 控制器([引用](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/))管理 AWS 中的 alb，以*external-DNSαkubernetes . io*开头的注释负责通过 ExternalDNS ( [引用](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#verify-externaldns-works-service-example))管理 Route53 记录。

## 包扎

暂时就这样吧！我希望这个页面能够帮助您理解 AWS 中基本 Kubernetes 集群背后的一些关键概念，并掌握一些关于 Terraform 配置文件的良好实践。

我真的很感激任何类型的反馈，问题或评论。在这里随意 ping 我[，或者在这个帖子里发表任何评论。](https://nico.singh.cl)

[](https://github.com/sponsors/nicosingh) [## GitHub 赞助商上的赞助商@nicosingh

### 非常感谢你的支持😊我感谢你对做开源软件和分享知识的认可…

github.com](https://github.com/sponsors/nicosingh)
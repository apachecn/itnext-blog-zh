# 使用 aws-ssm-operator 和 terraform 在 AWS EKS 进行简单和精益的机密管理

> 原文：<https://itnext.io/simple-and-lean-secrets-management-in-aws-eks-using-aws-ssm-operator-and-terraform-585f149c221a?source=collection_archive---------2----------------------->

## 如何使用 AWS SSM 参数存储作为中央机密存储，并使用 aws-ssm-operator 生成 Kubernetes 机密

![](img/a1a4e6cc7561458b23cca9ffaa769fd3.png)

由[格伦·卡斯滕斯-彼得斯](https://unsplash.com/@glenncarstenspeters?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

## 介绍

在 Kubernetes 中管理秘密可能是一项艰巨的任务。一般来说，**绝不推荐将未加密的秘密存储在任何 git-repository** 中。这通常会导致不同开发人员或 devops 工程师之间分发秘密文件，这些文件不受 git 的版本控制，但需要以某种方式进行交换和管理。 [**Hashicorp Vault**](https://www.vaultproject.io/) 已经演变为解决这个问题的**事实上的标准解决方案**。然而，有时建立、维护和操作一个健壮的、高可用性的和适当配置的存储集群是不可行的。

在这篇文章中，我将使用“ [aws-ssm-operator](https://github.com/toVersus/aws-ssm-operator) ”展示一个用于 AWS 环境的**精简而简单的解决方案，这是一个开源的 Github 项目([https://github.com/toVersus/aws-ssm-operator](https://github.com/toVersus/aws-ssm-operator))。**

## SSM 算子

AWS-SSM 操作器是一个 Kubernetes 操作器，**自动将保存在 AWS SSM 参数库中的参数映射到 Kubernetes 秘密**中。与 AWS Secrets Manager 不同，AWS Secrets Manager 引入了每个秘密每月 0.4 美元的额外费用，而 AWS SSM 参数存储是免费的。

让我们看看如何部署操作员。基于位于[https://github.com/toVersus/aws-ssm-operator#installation](https://github.com/toVersus/aws-ssm-operator#installation)的操作员库的安装说明，我们使用`[kustomize](https://kustomize.io/)`定义 k8s 对象。查看位于[https://github.com/yandok/aws-ssm-operator-demo.git](https://github.com/yandok/aws-ssm-operator-demo.git)的演示库——`k8s`目录包含部署的基本和覆盖库定制定义:

```
k8s
├── base
│   ├── 200-serviceaccount.yaml
│   ├── 201-clusterrole.yaml
│   ├── 202-clusterrolebinding.yaml
│   ├── 300-ssm-parameterstore-v1alpha1-crd.yaml
│   ├── kustomization.yaml
│   └── operator.yaml
├── dev
│   ├── 200-serviceaccount-patch.yaml
│   ├── kustomization.yaml
```

`deployment` —对象与`ServiceAccount` `aws-ssm-operator`一起运行，后者是用`dev` —覆盖中的覆盖定义定制的，如:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-ssm-operator
  namespace: kube-system
  annotations:
    *# Substitute your account ID and IAM service role name below.* eks.amazonaws.com/role-arn: arn:aws:iam::<aws-account-number>:role/<iam-role-name>
```

用`kustomize build k8s/dev > k8s/dev/out/dev.yaml`构建 k8s 定义，用`kubectl apply -f k8s/dev/out/dev.yaml`部署它们。

## EKS-先决条件

为了让运营商服务账户承担 kustomize-overlay 中定义的 AWS IAM 角色，EKS 集群必须提供 **IAM OIDC 身份提供者。**

如果您正在使用[eks-terra form-module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)([https://registry . terra form . io/modules/terra form-AWS-modules/eks/AWS/latest](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest))这可以通过将`enable_irsa`-属性设置为`true`来完成。

```
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  version         = "13.1.0"
  cluster_version = "1.18"
  cluster_name    = local.cluster_name  
  ... **enable_irsa = true**
  ...
}
```

对于 IAM-setup，我们定义了一个`[aws_iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)` [—资源](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)，它允许`aws-ssm-operator` — `ServiceAccount`承担角色-

```
resource "aws_iam_role" "aws-ssm-role" {

  name = "aws-ssm-role"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
       ** "Federated": "${module.eks.oidc_provider_arn}"**
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        **"StringEquals": {
          "${local.cluster_oidc_issuer_url_short}:sub": "system:serviceaccount:kube-system:aws-ssm-operator"
        }**
      }
    }
  ]
}
EOF

}
```

一个`[aws_iam_policy_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy_attachment)` [—资源](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy_attachment)将一个相关的策略附加到角色上

```
resource "aws_iam_policy_attachment" "aws-ssm-policy-attachment" {
  name       = "aws-ssm-policy-attachment"
  roles      = [aws_iam_role.aws-ssm-role.name]
  policy_arn = aws_iam_policy.aws-ssm-policy.arn
}
```

和一个`[aws_iam_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)` [—资源](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)，它允许角色访问 AWS SSM 参数存储中的参数。

```
resource "aws_iam_policy" "aws-ssm-policy" {

  # ... other configuration ...
  name   = "aws-ssm-policy"
  policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:Describe*",
                "ssm:Get*",
                "ssm:List*"
            ],
            "Resource": "*"
        }
    ]
}
EOF
}
```

## 创建 k8s 秘密，由 aws-ssm-operator 生成

现在，我们已经使用 **IAM OIDC 身份提供者**创建了一个 EKS 集群，并提供了所需的 IAM 设置，我们可以定义一个 k8s `ParameterStore`，如下所示:

```
apiVersion: ssm.aws/v1alpha1
kind: ParameterStore
metadata:
  name: demo-parameter-store
spec:
  valueFrom:
    parameterStoreRef:
      path: /path/to/parameters
```

aws-ssm-operator 将获取该路径下管理的参数，并创建一个 k8s `secret` —包含 key = parameter-name 参数的对象，可以用`kubectl get secret demo-parameter-store -o yaml`来检查。

## 资源

[](https://github.com/toVersus/aws-ssm-operator) [## GitHub-toVersus/AWS-SSM-operator:AWS SSM 参数存储库的 Kubernetes 操作员

### 一个 Kubernetes 操作器，它自动将存储在 AWS SSM 参数存储中的内容映射到 Kubernetes 机密…

github.com](https://github.com/toVersus/aws-ssm-operator)  [## 将 IAM 角色与服务帐户相关联

### 在 Kubernetes 中，您可以通过添加以下内容来定义与集群中的服务帐户相关联的 IAM 角色…

docs.aws.amazon.com](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html) 

[https://registry . terra form . io/modules/terra form-AWS-modules/eks/AWS/latest](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)

[](https://github.com/mumoshu/aws-secret-operator) [## GitHub-mumo Shu/AWS-secret-operator:一个 Kubernetes 操作器，可以自动创建和更新…

### 一个 Kubernetes 操作员，它根据 AWS 中存储的内容自动创建和更新 Kubernetes 秘密…

github.com](https://github.com/mumoshu/aws-secret-operator) [](https://kustomize.io/) [## Kubernetes 本地配置管理

### 纯声明式的配置定制方法，内置于 kubectl 中，可管理任意数量的…

kustomize.io](https://kustomize.io/)
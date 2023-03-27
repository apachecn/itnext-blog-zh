# 如何创建多堆栈 AWS CDK (Python)基础设施即代码(IaC)项目

> 原文：<https://itnext.io/how-to-create-a-multiple-stacks-aws-cdk-python-infrastructure-as-code-iac-project-e8a8defcd209?source=collection_archive---------3----------------------->

![](img/162ebb769f83ffe13c5b5049a1bddc43.png)

AWS CDK

这篇文章是关于创建一个多栈 AWS CDK IaC 项目。我将使用 Python，因为我对它更熟悉，但是如果你对另一种语言很熟悉，也可以使用 Typescript、Javascript、Java 或 C#

我还创建了一个简单的 lambda 应用程序和一个简单的容器化应用程序，它们将使用这个基础设施。这些应用程序都有用 Github 操作创建的简单 CICD。

**为什么在 AWS CDK 中需要多个堆栈？**

让我们想象一下，您需要将相同的应用程序部署到不同的区域，采用不同的配置，或者您有一个大型基础架构，并且希望将每个应用程序/服务堆栈分开，保留在自己的定义文件中，以便多个开发人员或团队可以在没有冲突的情况下一起工作。或者您有不同的环境开发、测试、生产前、生产等。多个堆栈使管理这种类型的基础架构变得容易。

# **无服务器应用:**

将数据从 CSV 文件导入 DynamoDB，然后上传到 S3 存储桶。AWS Lambda 将在新文件上传时自动触发。

*Github 库:**[https://github.com/omerkarabacak/serverless-csv-to-dynamodb](https://github.com/omerkarabacak/serverless-csv-to-dynamodb)*

***基础设施** : S3 + Lambda + DynamoDB*

*[https://github . com/omerkarabacak/multi-stack-AWS-CDK-python-IAC-project/blob/main/lambda _ stack/lambda _ stack . py](https://github.com/omerkarabacak/multi-stack-aws-cdk-python-iac-project/blob/main/lambda_stack/lambda_stack.py)*

# ***集装箱化应用:***

*应用程序将简单地计算一个给定数字的阶乘并返回结果。我使用 Python Flask，只是为了创建一个简单的容器化应用程序。*

****Github 资源库:****[https://Github . com/omerkarabacak/container ised-python-flask-app](https://github.com/omerkarabacak/containerised-python-flask-app)**

**基础设施:ECR + ECS + ELB**

**[https://github . com/omerkarabacak/multi-stack-AWS-CDK-python-IAC-project/blob/main/container _ stack/container _ stack . py](https://github.com/omerkarabacak/multi-stack-aws-cdk-python-iac-project/blob/main/container_stack/container_stack.py)**

# **基础设施代码项目:**

**这个项目有两个不同的堆栈，一个用于无服务器应用程序，另一个用于容器化应用程序。**

# **如何运行这个 AWS CDK 项目**

## **安装 CDK CLI 工具(如果尚未安装，请安装)**

```
**npm install -g aws-cdk**
```

## **如何为 Python 创建虚拟环境并安装依赖项**

```
**python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt**
```

## **如何运行测试**

```
**pytest**
```

## **显示堆栈**

```
**cdk ls**
```

## **如何部署选定的堆栈**

```
**cdk deploy lambda-stack --profile <AWS_PROFILE_NAME>**
```

## **如何部署所有堆栈**

```
**cdk deploy --all --profile <AWS_PROFILE_NAME>**
```

## **有用的命令**

*   **`cdk ls`列出应用程序中的所有堆栈**
*   **`cdk synth`发出合成的云生成模板**
*   **`cdk deploy`将此堆栈部署到您的默认 AWS 帐户/地区**
*   **`cdk diff`将部署的堆栈与当前状态进行比较**
*   **`cdk docs`打开 CDK 文档**

*****Github 资源库:***[https://Github . com/omerkarabacak/multi-stack-AWS-CDK-python-IAC-project](https://github.com/omerkarabacak/multi-stack-aws-cdk-python-iac-project)**

**如果你喜欢这篇文章，我想你会想看看我的另一篇也与这个话题相关的文章:**

**[AWS EKS 101:创建集群和部署应用| mer KARABACAK | 2022 年 12 月| ITNEXT](/aws-eks-101-creating-a-cluster-and-deploying-an-app-9608a1cac016)**

****支持媒介和我:)**[https://omerkarabacak.medium.com/membership](https://omerkarabacak.medium.com/membership)**
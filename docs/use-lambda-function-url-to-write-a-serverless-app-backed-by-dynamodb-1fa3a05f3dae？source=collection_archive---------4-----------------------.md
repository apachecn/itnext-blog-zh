# 使用 Lambda 函数 URL 编写一个由 DynamoDB 支持的无服务器应用程序

> 原文：<https://itnext.io/use-lambda-function-url-to-write-a-serverless-app-backed-by-dynamodb-1fa3a05f3dae?source=collection_archive---------4----------------------->

## *这是一个使用 AWS SAM* 打包和部署的 Go Lambda 函数

[Lambda 函数 URL](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html) 是一个相对较新的特性(在撰写这篇博客时)，它为您的 Lambda 函数提供了一个专用的 HTTP(S)端点。当您只需要一个功能端点(例如作为一个 webhook)并且不想设置和配置 API 网关时，它真的很有用。看起来我似乎永远也看不够！我已经写了几篇关于这个主题的博客文章，其中包括一个使用它来[构建一个无服务器后台](/using-aws-lambda-function-url-to-build-a-serverless-backend-for-slack-a292ef355a5d)然后[使用 AWS CDK](/package-and-deploy-a-lambda-function-as-a-docker-container-with-aws-cdk-fd0df5e37de7) 部署那个解决方案的实际例子。

![](img/cdc3a945c3182c4c9c8117c5d1e1b4ac.png)

Gabriel Heinzer 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

这又是一篇博文(很短！)演示了如何使用 Lambda 函数 URL 编写一个由 DynamoDB 支持的简单应用程序。您将能够调用由 Lambda 函数 URL 公开的 API 端点，它将依次在 DynamoDB 上执行操作(`GetItem`、`PutItem`、`Scan`)。使用 [AWS Go SDK](https://aws.amazon.com/sdk-for-go/) 中的 [DynamoDB 包](https://pkg.go.dev/github.com/aws/aws-sdk-go-v2/service/dynamodb)在 Go 中编写函数，并使用 [AWS 无服务器应用模型(SAM)](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) 快速构建和部署解决方案。

> *代码是* [*可在 GitHub*](https://github.com/abhirockzz/lambda-functionurl-dynamodb-sam-go) *上找到供你参考*

让我们开始吧！在您继续之前，请确保您已准备好以下事项:

# 先决条件

*   [如果您还没有 AWS 帐户，请创建一个帐户](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html)并登录。您使用的 IAM 用户必须有足够的权限进行必要的 AWS 服务调用和管理 AWS 资源。
*   安装和配置 AWS CLI
*   [Go](https://go.dev/dl/) ( `1.16`或以上)安装
*   [安装了 AWS 无服务器应用模型](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) (AWS SAM)
*   [Git 已安装](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

# 部署

首先克隆 Github repo，然后切换到正确的目录:

```
git clone https://github.com/abhirockzz/lambda-functionurl-dynamodb-sam-go
cd lambda-functionurl-dynamodb-sam-go
```

使用 AWS SAM 构建应用程序:

```
sam build
```

现在您已经准备好部署应用程序了:

```
sam deploy --guided
```

出现提示时，输入所需信息，如堆栈名称等。请参见下面的示例:

```
Configuring SAM deploy
====================== Looking for config file [samconfig.toml] :  Not found Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [sam-app]: 
        AWS Region [us-east-1]: 
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [y/N]: y
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: y
        #Preserves the state of previously provisioned resources when an operation fails
        Disable rollback [y/N]: n
        DemoFunction Function Url may not have authorization defined, Is this okay? [y/N]: y
        Save arguments to configuration file [Y/n]: 
        SAM configuration file [samconfig.toml]: 
        SAM configuration environment [default]:
```

> *一旦您运行了一次* `*sam deploy --guided*` *模式并将参数保存到配置文件(samconfig.toml)中，您可以在以后使用* `*sam deploy*` *来使用这些默认值。*

如果成功，您现在应该准备好以下内容:

*   带有 HTTP(S) URL 的 Lambda 函数
*   DynamoDB 表(称为`users`)
*   具有 Lambda 和 DynamoDB 所需的最低权限的 IAM 角色(`PutItem`、`GetItem`和`Scan`)

> *注意 SAM 部署流程的输出。这包含测试所需的函数的 HTTP URL 端点*

# 测试应用程序

测试集成包括向 Lambda 函数 URL 发送 HTTP 请求。这个例子使用了 curl CLI，但是你也可以使用其他选项。

将 Lambda 函数 URL 端点导出为环境变量。

```
export LAMBDA_FUNCTION_URL_ENDPOINT=<SAM deployment output>e.g. export LAMBDA_FUNCTION_URL_ENDPOINT=https://qfmpu2n74ik7m2m3ipv3m6yw5a0qiwmp.lambda-url.us-east-1.on.aws/
```

调用端点在 DynamoDB 表中创建新条目:

```
curl -i -X POST -H "Content-Type: application/json" -d '{"email":"user1@foo.com", "username":"user-1"}' $LAMBDA_FUNCTION_URL_ENDPOINTcurl -i -X POST -H "Content-Type: application/json" -d '{"email":"user2@foo.com", "username":"user-2"}' $LAMBDA_FUNCTION_URL_ENDPOINT
```

在这两种情况下，您都应该得到 HTTP `201 Created`响应。这表明物品已经添加到 DynamoDB 中的`users`表中。

让我们检索刚刚创建的记录的信息(我们通过向 URL 添加一个`query`参数(email)来实现):

```
curl -i $LAMBDA_FUNCTION_URL_ENDPOINT?email=user2@foo.com
```

要检索您刚刚添加的所有项目:

```
curl -i $LAMBDA_FUNCTION_URL_ENDPOINT
```

尝试搜索尚未添加的项目:

```
curl -i $LAMBDA_FUNCTION_URL_ENDPOINT?email=notthere@foo.com
```

在这种情况下，您将得到一个`HTTP 404 Not Found`响应。

最后，尝试插入一个重复的记录:

```
curl -i -X POST -d '{"email":"user2@foo.com", "username":"user-2"}' $LAMBDA_FUNCTION_URL_ENDPOINT
```

在这种情况下，Lambda 函数返回一个 HTTP `409 Conflict`，因为一个[条件表达式](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html) ( `attribute_not_exists(email)`)防止一个现有的项目(具有相同的`email`)被`PutItem`调用覆盖。

就是这样！您已经尝试了 Lambda 函数公开的所有操作。

# 清除

完成后，请删除堆栈:

```
sam delete --stack-name STACK_NAME
```

确认堆栈已被删除

```
aws cloudformation list-stacks --query "StackSummaries[?contains(StackName,'STACK_NAME')].StackStatus"
```

# 总结一下！

这是一个简短(但希望有用)的教程！您可以使用它来快速引导一个无服务器应用程序，该应用程序具有 DynamoDB 后端，其功能由 Lambda 函数 URL 公开。

编码快乐！
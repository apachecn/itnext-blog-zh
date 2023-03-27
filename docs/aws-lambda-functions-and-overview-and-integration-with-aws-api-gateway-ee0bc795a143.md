# AWS: Lambda 函数—概述，以及与 AWS API 网关的集成

> 原文：<https://itnext.io/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway-ee0bc795a143?source=collection_archive---------2----------------------->

![](img/4116eb2906df88b0d6fcf1c7806264e7.png)

[AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) 允许在不需要创建和管理服务器的情况下运行代码，也被称为*无服务器方法。*

AWS Lambda 将确定运行一个函数需要多少 CPU 和内存，必要时它会执行自动缩放。

要运行的代码被组织在*λ函数*中，并且可以用*触发器*来触发。可以使用 CloudWatch 日志来检查结果。

作为触发器，您可以使用几乎任何 AWS 服务，如 API Gateway、SQS、应用负载平衡器、CloudFront、Kinesis 或外部事件，例如 Github 的 webhook。

在本文中，我们将创建一个简单的 AWS Lambda 函数，检查它的控制面板、可用选项和功能，然后创建一个 AWS API 网关，将 HTTP 请求转发给 AWS Lambda 函数。

# 内容

*   [AWS Lambda —使用案例](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#AWS_Lambda_-_use_cases)
*   [组件和概念](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Components_and_concepts)
*   [创建“Hello，World”Lambda 函数](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Creating_a_Hello_World_Lambda_function)
*   [创建功能](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Create_a_function)
*   [监控](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Monitoring)
*   [配置](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Configuration)
*   [总体配置](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#General_configuration)
*   [触发器](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Triggers)
*   [权限](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Permissions)
*   [目的地](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Destinations)
*   [环境变量](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Environment_variables)
*   [VPC](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#VPC)
*   [监控和操作工具](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Monitoring_and_operations_tools)
*   [并发](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Concurrency)
*   [异步调用](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Asynchronous_invocation)
*   [数据库代理](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Database_proxies)
*   [文件系统](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#File_system)
*   [别名](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Aliases)
*   [版本](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Versions)
*   [AWS Lambda 和 AWS API 网关——集成示例](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#AWS_Lambda_and_AWS_API_Gateway_-_an_integration_example)
*   [创建一个 Lambda 函数](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Create_a_Lambda_function)
*   [创建一个 AWS API 网关](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/#Create_an_AWS_API_Gateway)

# AWS Lambda —使用案例

一般来说，AWS 中的 Lambda 可以做很多事情。这样的“银弹”，允许做 AWS 控制台本身无法实现的事情。

示例用例可能是:

*   一个网站:一个基于 javascript 的静态托管的 AWS S3 前端，前端将通过一个 Lambda 函数接收通过 AWS API 网关到数据库的请求
*   日志分析:一个很好的例子是 [AWS WAF 安全自动化](https://aws.amazon.com/ru/solutions/implementations/aws-waf-security-automations/)，当所有传入的 HTTP 请求被发送到 AWS Kinesis 时，它会将它们转发到 Lambda，它会执行一些检查，如果需要，会阻止客户端的 IP
*   备份自动化:AWS SNS 可以向 Lambda 函数发送一个事件，例如当 AWS S3 存储桶中使用了太多的磁盘空间时，这将删除一些旧的备份
*   数据处理:例如，当一个新文件上传到 AWS S3 时，它将生成一个事件，该事件将触发一个 Lambda 函数，该函数将执行视频文件编码
*   无服务器 cronjobs:使用 CloudWatch Events 按时间表生成一个事件，这将触发一个 Lambda 函数

# 组件和概念

让我们简要概述一下 AWS Lambda 的主要概念:

*   ***函数*** *:* 是运行在函数中的代码。可以作为 zip 文件或 Docker 镜像(*部署包)。*参见[配置 AWS Lambda 功能](https://docs.aws.amazon.com/lambda/latest/dg/lambda-functions.html)。
*   ***触发器*** :将触发一个函数的 AWS 资源。这种事件包括 AWS 服务和*事件映射*。参见[调用 AWS Lambda 函数](https://docs.aws.amazon.com/lambda/latest/dg/lambda-invocation.html) и [使用 AWS Lambda 和其他服务](https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html)。
*   ***事件*** :一个 JSON 对象，包含 Lambda 函数的数据进行处理
*   ***执行环境*** *:* 执行功能的安全环境。参见 [AWS Lambda 执行环境](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-context.html)。
*   ***部署包*** *:* 要运行的 Lambda 功能代码。可以是 zip 存档或 Docker 图像。参见 [Lambda 部署包](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html)。
*   ***运行时*** :运行一个功能的工作环境。参见[λ运行时间](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
*   ***层*** :一个 zip 存档文件，其中包含运行某个函数的附加代码，例如——外部库。参见[创建和共享 Lambda 层](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)。
*   ***扩展*** : AWS Lambda 允许使用扩展，这些扩展可用于将功能与外部服务(如监控)集成。参见[使用 Lambda 扩展](https://docs.aws.amazon.com/lambda/latest/dg/using-extensions.html)。
*   ***并发*** :多个 Lambda 函数实例同时运行，处理传入的数据。参见[管理 Lambda 函数的并发性](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)。
*   ***限定词*** :指向版本或别名的“指针”。参见[λ函数版本](https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html)。
*   ***目的地*** :一个 AWS 资源，函数将在其中发送处理后的数据。参见[配置异步调用的目的地](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html#invocation-async-destinations)。

# 创建“Hello，World”Lambda 函数

首先，让我们创建一个最简单的 Lambda 函数，看看它是如何工作的，我们有什么。

## 创建一个函数

转到 AWS Lambda，点击*创建功能*:

![](img/92ac09d6c86068536d3a0b65cb106814.png)

现在，让我们使用一个现有的模板。选择*使用蓝图*，找到 *hello-world-python* :

![](img/77eac64ae20b38de0d5c34e145f95a1f.png)

点击*配置*:

![](img/fee46a77a72c862f3cd3371cc52ae5d0.png)

设置一个函数的名称，例如 *example-hello* ，保留一个默认的 IAM 角色——它将允许我们的函数使用 CloudWatch 日志，并检查将要使用的代码:

![](img/2d4606646c4c29c6839b3e8d234c87c1.png)

点击*创建功能*:

![](img/cb852942dc8c1666d52812871354753f.png)

切换到*测试*标签:

![](img/fc273e870aa2daaee3d5c366c8882d73.png)

这里，我们可以传递一个 JSON，其中包含要由我们的函数处理的数据。

运行它:

![](img/4452e554066ce6b363bf06df3d509a0f.png)

现在，让我们来看看 AWS 控制台建议使用什么来进行 Lambda 函数管理。

## 监视

首先是监控。在这里，我们可以使用 AWS CloudWatch 指标和日志，通过 AWS X-Ray、Lambda insights 和 AWS CodeGuru 进行调用跟踪:

![](img/b7ddf05ea1bc9db10bacab69560ec401.png)

## 配置

## 一般配置

*   内存设置:运行一个函数时允许使用的最大内存。此外，根据内存设置，Lambda 将提供一个 CPU 限制:对于每 1769 MB，将有一个 vCPU。参见[配置功能存储器(控制台)](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-common.html#configuration-memory-console)。
*   执行超时:最大值可以设置为 900 秒，在此之后，函数执行将被停止。请记住，这将影响成本。参见[超时](https://docs.aws.amazon.com/whitepapers/latest/serverless-architectures-lambda/timeout.html)。
*   IAM 角色:包括具有 AWS 资源权限的 IAM 策略

![](img/c6954fa0222b6fb6d9d4c749a19c6a01.png)

## 扳机

触发我们功能的触发器。

几乎可以是任何 AWS 服务:

![](img/1b7641423546639f2d252e03ccb098d5.png)

例如，我们可以从 AWS 应用程序负载平衡器创建一个触发器，该触发器将接受到特定 URI 的连接，并将其转发给 Lambda 函数:

![](img/4695646c816dac14a7e8fba512e9be11.png)

## 许可

在这里，您可以查看和调整将配置功能权限的 IAM 角色和策略:

![](img/95be974e6deba7499d39f87712833903.png)

## 目的地

向何处发送函数的执行结果。

例如，可以是一个 AWS SNS 主题，该主题会将其转发给 Opsgenie，ops genie 会将消息发送到一个松弛通道:

![](img/dbfe7c31c2f9b83995af185a8957b652.png)

## 环境变量

可以在函数中使用的变量。敏感数据可以使用 AWS 密钥管理服务(KMS)进行加密:

![](img/e15d8eb8adb9c5db7e1a5eada395ff06.png)

## VPC

可以在专用 AWS 虚拟私有云上放置一个功能来限制其网络访问:

![](img/bd37485989ee8f0d9c7cc5499b64f30c.png)

## 监控和操作工具

监控设置，您可以在其中启用或禁用附加服务，如 AWS X-Ray、CloudWatch Lambda Insights 和 Amazon CodeGuru Profiler:

![](img/7318771b8b9aa5b6ecc8f7aee526bef9.png)

在*扩展中，*您可以从现有解决方案列表中选择，也可以创建自己的解决方案:

![](img/3b67b3864618cfcfee7e832fd11f5a87.png)

## 并发

可以同时运行的函数实例的最大数量。参见[管理 Lambda 函数的并发性](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)。

可以是以下两种类型之一:

****![](img/791026198958c9dc048be40ab2b3ec18.png)****

## ****异步调用****

****事件队列的设置—生存期、出错时的重试次数、错误通知等。****

****参见[异步调用](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html?icmpid=docs_lambda_help):****

****![](img/07ef0fa1d097207d04f306981991cea8.png)****

****参见[同步调用](https://docs.aws.amazon.com/lambda/latest/dg/invocation-sync.html)和[异步调用](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html)。****

## ****数据库代理****

****一个 [Amazon RDS 代理](https://aws.amazon.com/ru/rds/proxy/)配置用于您的功能。用于减少数据库服务器连接数量的 RDS 代理:****

****![](img/96b0c1198d2f2741b6952823f936b777.png)****

## ****文件系统****

****您可以在函数中挂载一个 AWS 弹性文件系统目录:****

****![](img/487c37cfef75c92ded6d4164cc10fcc3.png)****

## ****别名****

****别名是一种指针，指向函数代码的特定版本，以后可以在它的 ARN 中使用。****

****此外，您可以有几个别名，并在它们之间分配请求。参见[λ函数别名](https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html):****

****![](img/5fd8d0f1bef1736890c55b37c4eaee7a.png)****

## ****版本****

****AWS Lambda 允许使用代码和部分设置版本。在开发环境中测试新代码时会很有用，例如通过创建专用的*别名*:****

****![](img/551bdba7f6d7ac591a513202bb6b5ac9.png)****

# ****AWS Lambda 和 AWS API 网关—集成示例****

****因此，我们已经检查了什么是 AWS Lambda 以及它的设置中有什么。****

****现在，让我们创建一个 AWS API 网关，它将请求转发给 AWS Lambda 函数。****

****API 网关将接受对 */test* URI 的请求，并使用其事件将请求发送给我们的 Lambda。****

## ****创建 Lambda 函数****

****这时，从头选择*作者*，在运行时使用 *Python* :****

****![](img/d7f7f39cab128a44f5932c80ebda5b6b.png)****

****保留默认代码:****

****![](img/6e02a0e288203d19dfe49a64ae4e5992.png)****

****这里，`lambda_handler()`是调用函数时要调用的默认函数。它接受两个参数:****

1.  ****`event`:API 网关事件，参见[使用 AWS Lambda 和其他服务](https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html)****
2.  ****`context`:运行函数所允许的方法和参数，参见 [AWS Lambda 上下文对象中的 Python](https://docs.aws.amazon.com/lambda/latest/dg/python-context.html)****

## ****创建 AWS API 网关****

****创建一个新的网关，将其类型设置为 *HTTP API* :****

****![](img/a2a927efa835e320bf5e32ecbc96146c.png)****

****添加集成:****

****![](img/041a78231a9ea72c491c1696bb331244.png)****

****选择 Lambda、AWS 区域和要调用的函数:****

****![](img/f9ba8bb519509d6e52c2499dc5376893.png)****

****将 URI 设置为*/测试*:****

****![](img/756b5004e2b0c8b40fdb5206fd24075a.png)****

****离开默认[阶段](https://docs.aws.amazon.com/apigateway/latest/developerguide/stages.html):****

****![](img/fb229260e0b3f29eb82314e5c7f6c845.png)****

****不到一分钟，您的网关就准备好了。复制其 URL:****

****![](img/2d37c0344641fff5f13d4393ad6860c5.png)****

****用`curl`试试:****

```
**$ curl [https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test](https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test)
“Hello from Lambda!”**
```

****要检查事件的全部内容，用`[json.dumps()](https://docs.python.org/3/library/json.html)`打印出来:****

```
**import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps(event)
    }**
```

****更改密码后，按下*展开*:****

****![](img/d8d21f35c01186abd6b1cc451806f24a.png)****

****试试看:****

```
**$ curl [https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test](https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test)
{“version”: “2.0”, “routeKey”: “ANY /test”, “rawPath”: “/test”, “rawQueryString”: “”, “headers”: {“accept”: “*/*”, “content-length”: “0”, “host”: “fwu399qo70.execute-api.us-east-2.amazonaws.com”, “user-agent”: “curl/7.78.0”, “x-amzn-trace-id”: “Root=1–611bbaa4–3cb7c28e4e3181dd647f1030”, “x-forwarded-for”: “194.***.***.29”, “x-forwarded-port”: “443”, “x-forwarded-proto”: “https”}, “requestContext”: {“accountId”: “534***385”, “apiId”: “fwu399qo70”, “domainName”: “fwu399qo70.execute-api.us-east-2.amazonaws.com”, “domainPrefix”: “fwu399qo70”, “http”: {“method”: “GET”, “path”: “/test”, “protocol”: “HTTP/1.1”, “sourceIp”: “194.***.***.29”, “userAgent”: “curl/7.78.0”}, “requestId”: “ENoZriIrCYcEPWg=”, “routeKey”: “ANY /test”, “stage”: “$default”, “time”: “17/Aug/2021:13:33:24 +0000”, “timeEpoch”: 1629207204179}, “isBase64Encoded”: false}**
```

****现在，让我们看看如何在函数中使用环境变量。****

****添加一个新的:****

****![](img/d7ab492f3253d3c92d3c45328b9644cc.png)****

****并用`os.getenv()`打印其值:****

```
**import os
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': os.getenv('Env')
    }**
```

****检查一下:****

```
**$ curl [https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test](https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test)
test**
```

****此外，您可以更改默认的处理程序。****

****将`lambda_handler`更名为`main_handler`:****

```
**import os
import json

def main_handler(event, context):
    return {
        'statusCode': 200,
        'body': os.getenv('Env')
    }**
```

****如果您现在尝试访问该函数，您将得到 ***内部服务器错误*** :****

```
**$ curl [https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test](https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test)
{“message”:”Internal Server Error”}**
```

****向下滚动到*运行时设置*:****

****![](img/5b6db36feb45e9c5d4339c672f5ab82d.png)****

****并更换*手柄*:****

****![](img/cb3efa864c466092053d4f940622620d.png)****

****再次运行:****

```
**$ curl [https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test](https://fwu399qo70.execute-api.us-east-2.amazonaws.com/test)
test**
```

****完成了。****

*****最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/aws-lambda-functions-and-overview-and-integration-with-aws-api-gateway/) *。*****
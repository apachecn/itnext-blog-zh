# AWS Lambda 通过无服务器框架得到简化

> 原文：<https://itnext.io/aws-lambda-simplified-with-serverless-framework-8fbc01418422?source=collection_archive---------1----------------------->

![](img/15971174c45beed2defeaa6cfb7be540.png)

潘卡杰·帕特尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

如果你以前使用过 [AWS Lambda](https://aws.amazon.com/lambda/) 来创建一个使用 [API Gateway](https://aws.amazon.com/api-gateway/) 的 API，那么你一定知道在频繁更新的情况下创建和管理它们的痛苦。对我来说也是如此，尤其是许多功能很难管理，很难调试和频繁更新。由于有许多管理 Lambda 的框架，我尝试了[无服务器框架](https://serverless.com/)，一切都为我改变了。看到管理和跟踪一切是如此的强大和简单，我真是大吃一惊。这有助于开发和部署 Lambda 功能，以及管理 AWS 基础设施资源。这有助于我将注意力集中在编写代码上，而不用担心基础设施管理。

简单介绍一下无服务器计算，它是一个云计算模型，旨在抽象基础架构管理，提供一个具有按使用付费模式的内在可扩展设计。这有助于提供可用性和容错性，这有助于开发人员专注于编写业务逻辑，而不用担心其他事情。

在无服务器中，我们有一个功能，它是一个独立的部署单元，只是在云上执行的代码。任何触发这些功能的事情都被称为事件。执行这些功能所需的资源(基础设施)由云提供商管理。就无服务器框架而言，我们需要创建一个类似项目的服务，包含功能、触发这些功能的事件以及它们所需的资源。

在本文中，我们将创建一个 NodeJs 函数，它可以使用无服务器框架部署在 AWS Lambda 上。为此，我们首先需要使用' [npm](https://www.npmjs.com/) '安装无服务器框架。无服务器为我们提供了两个彼此等价的命令‘无服务器’和‘SLS’。我们将在本文中使用“无服务器”命令。

```
npm install -g serverless
```

为了开始在无服务器上工作，我们需要创建一个服务，在这里我们需要用一个服务名来传递 NodeJs 运行时，这个服务名充当我们的代码所在的目录的名称。

```
serverless create --template aws-nodejs --path lambdaService**Response of this command**:Serverless: Generating boilerplate...
Serverless: Generating boilerplate in "/path/to/lambda/lambdaService"
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                           serverless.com, v1.37.1
 -------'Serverless: Successfully generated boilerplate for template: "aws-nodejs"
```

我们在服务目录中创建了两个文件，分别名为 **serverless.yml** 和 **handler.js** 。yml 包含我们的无服务器代码的配置，它定义了部署服务的提供者、功能细节、使用的任何自定义插件、触发每个功能的事件、功能所需的资源等。

```
# Welcome to Serverless!
#
# This file is the main config file for your service.
# It's very minimal at this point and uses default values.
# You can always add more config options for more control.
# We've included some commented out config examples here.
# Just uncomment any of them to get that config option.
#
# For full config options, check the docs:
#    docs.serverless.com
#
# Happy Coding!service: lambdaServiceprovider:
  name: aws
  runtime: nodejs8.10
  stage: dev
  region: us-east-1
  memorySize: 512 plugins:
  - serverless-offlinecustom:
    serverless-offline:
        port: 4000functions:
  ourFunction:
    handler: handler.ourFunction
    description: Used to as dummy lambda function
    events:
     - http:
         path: /
         method: get
```

handler.js 文件包含项目的 lambda 函数代码。在 serverless.yml 中定义的每个函数都将指向 handler.js 中的一个函数。我们将把这个文件中的所有代码都写在一个将要被调用的函数中。

```
'use strict';module.exports.ourFunction = async (event, context) => {
 return {
  statusCode: 200,
  body: JSON.stringify({
   message: 'Function executed successfully!',
   input: event,
  }),
 };
};
```

我们将创建一个 API 来调用我们的函数。这将触发 API 网关路由的创建，该路由将根据请求调用 Lambda。我们刚刚在我们的 serverless.yml 文件中添加了 API 相关的细节，我们准备好了。

```
ourFunction:
    handler: handler.ourFunction
    description: Used to as dummy lambda function
    events:
     - http:
         path: /
         method: get
```

一旦我们完成了在处理函数中编写业务逻辑的工作，我们需要将这些代码部署到 AWS Lambda 中。当我们运行命令来部署所有功能、事件和资源时，它们是使用 AWS CloudFormation 模板创建的。

```
serverless deploy**Response of this command**:Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
.....
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service lambdaService.zip file to S3 (1.36 MB)...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
...........................
Serverless: Stack update finished...
Service Information
service: lambdaService
stage: dev
region: us-east-1
stack: lambdaService-dev
resources: 9
api keys:
  None
endpoints:
  GET - [https://1hntg29euyg.execute-api.us-east-1.amazonaws.com/dev/](https://1wntg29ej5.execute-api.us-east-1.amazonaws.com/dev/)
functions:
  ourFunction: lambdaService-dev-ourFunction
layers:
  None
```

在部署此服务之前，我们需要确保的一件事是，我们创建并存储所需的 AWS 凭证。详情可在[这里](https://serverless.com/framework/docs/providers/aws/guide/credentials/)找到。您甚至可以使用以下命令为无服务器设置 AWS 凭据:

```
serverless config credentials --provider aws --key AWS_ACCESS_KEY --secret AWS_SECRET
```

我们可以从本地系统使用“invoke”命令调用我们的 lambda 函数，并从 lambda 获得日志响应。

```
serverless invoke -f ourFunction -l
```

无服务器框架最好的一点是它的插件，可以通过 npm 轻松下载并使用 serverless.yml 添加。这里我们将使用 serverless-offline 创建一个本地 API 路由，可用于本地调试。

```
npm install serverless-offline
```

我们可以通过更改 serverless.yml 文件中的配置来更改脱机应用程序的端口。要离线运行 lambda，我们只需要运行下面的命令，就可以进行本地调试了。

```
serverless offline start**Response of command**:Serverless: Starting Offline: dev/us-east-1.Serverless: Routes for ourFunction:
Serverless: GET /Serverless: Offline listening on [http://localhost:4000](http://localhost:4000)
```

一旦一切都完成了，我们不需要这个函数，那么我们可以通过执行 remove 命令简单地删除它。

```
serverless remove**Response of this command**:Serverless: Getting all objects in S3 bucket...
Serverless: Removing objects in S3 bucket...
Serverless: Removing Stack...
Serverless: Checking Stack removal progress...
........
Serverless: Stack removal finished...
```

所有这些让我们的生活变得足够简单，使用 Lambda 不再是痛苦的经历。

***PS:如果你喜欢这篇文章，请用掌声支持它*** 👏 ***。欢呼***
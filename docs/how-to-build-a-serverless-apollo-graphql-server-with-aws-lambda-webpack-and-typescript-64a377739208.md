# 如何用 AWS Lambda、Webpack 和 TypeScript 构建一个无服务器的 Apollo GraphQL 服务器

> 原文：<https://itnext.io/how-to-build-a-serverless-apollo-graphql-server-with-aws-lambda-webpack-and-typescript-64a377739208?source=collection_archive---------0----------------------->

![](img/d81e6d03d1f9d29c21af8db793614824.png)

Hawaiʻi 莫纳克亚山的星空(图片来源:[方志伟](http://http://derekfong.me)

在我之前的文章中，我已经展示了[如何用](https://medium.com/free-code-camp/build-an-apollo-graphql-server-with-typescript-and-webpack-hot-module-replacement-hmr-3c339d05184f)[一些我在开发阶段发现有用的常用工具集](https://medium.com/better-programming/how-to-integrate-an-apollo-graphql-server-with-mongodb-and-typescript-code-generator-b029d821591)构建一个“Hello World”Apollo graph QL 服务器。我希望您已经了解了我们的 GraphQL 之旅，现在到了您的 Apollo 服务器可以部署的时候了！

# 服务器或无服务器—交易是什么？

![](img/375746cab89aeef2f0dfcbcfcb426985.png)

为什么没有服务器？这可能是您公司的解决方案架构师为您即将到来的项目做出的决定，或者您听到了关于“现代无服务器方法”的[好(或坏)](https://hackernoon.com/what-is-serverless-architecture-what-are-its-pros-and-cons-cc4b804022e9)的事情，并且只是想尝试一下……无论您的理由是什么，我都会为您提供帮助😉

# 步骤 1:设置 AWS 环境

在开始动手编写代码之前，您需要在您的计算机上进行一些配置。我不得不说这一步并不有趣。也就是说，正确设置我们的部署配置至关重要，如果您遵循下面的关键步骤，完成配置应该不会花费太长时间。

我们将在[亚马逊网络服务(AWS)](https://aws.amazon.com/) 上部署我们的 Apollo 服务器。[创建一个 AWS 账户](https://portal.aws.amazon.com/billing/signup)，如果你还没有的话。虽然 AWS 是一个提供[付费服务](https://aws.amazon.com/pricing)的平台，但它也在 [AWS 免费层](https://aws.amazon.com/free)下提供大量的免费使用，这应该足以涵盖本文中的使用。也就是说，您仍然需要了解[定价](https://aws.amazon.com/pricing)，并考虑[制定计划来管理服务和消耗的资源](https://aws.amazon.com/premiumsupport/knowledge-center/free-tier-charges)以[避免意外费用](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/checklistforunwantedcharges)。

一旦你设置好 AWS 账户，你需要[在你的电脑上安装最新版本的 AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install) 。安装完成后，您可以通过运行`aws`命令来验证安装。例如，以下命令返回您计算机上安装的当前 AWS CLI 版本:

```
$ aws --versionaws-cli/2.0.23 Python/3.7.4 Darwin/18.7.0 botocore/2.0.0
```

接下来，您需要[使用用户凭证配置 AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)。基本上，您需要做的就是运行`aws configure`命令，它会提示您输入四条信息:

*   [访问密钥 ID](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds)
*   [秘密访问密钥](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds)
*   [AWS 区域](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-region)
*   [输出格式](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-format)

按照本[快速入门指南](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart)获取 AWS CLI 所需的值。**作为** [**的最佳实践**](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#lock-away-credentials) **，请务必在您** [**创建管理员 IAM 用户和组**](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) **时创建访问键。**

```
$ aws configureAWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

AWS CLI 将此信息存储在凭证文件中名为`default`的[配置文件](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles)(一组设置)中。默认情况下，当您运行一个没有明确指定要使用的配置文件的`aws` CLI 命令时，会使用该配置文件中的信息。您可以通过运行`aws configure list`命令来检查当前的配置数据:

```
$ aws configure listName                    Value             Type    Location----                    -----             ----    --------profile                <not set>             None    Noneaccess_key     ****************ABCD      config_file    ~/.aws/configsecret_key     ****************ABCD      config_file    ~/.aws/configregion                us-west-2              env    AWS_DEFAULT_REGION
```

![](img/aab9d087363efc2d8990169fe12b5be3.png)

唷~就是这样！您的计算机现在已设置为与 AWS 交互。

# 第二步:安装`apollo-server-lambda`

![](img/0f45761b95b3b741e6132967c7831242.png)

创建一个名为`demo-apollo-server-lambda`的新文件夹，初始化项目并安装`apollo-server-lambda`。

```
$ mkdir demo-apollo-server-lambda && cd demo-apollo-server-lambda
$ npm init --yes
$ npm install --save apollo-server-lambda graphql
```

因为我们使用 TypeScript，我们需要安装`typescript`作为开发依赖:

```
$ npm install --save-dev typescript
```

并为您的项目根创建一个`[tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)`。

📄文件:`tsconfig.json`

接下来，在项目根目录下创建一个`/src`文件夹。将 [GraphQL 模式的类型定义](https://www.apollographql.com/docs/tutorial/schema) ( `type-defs.ts`)和[解析器](https://www.apollographql.com/docs/tutorial/resolvers/) ( `resolvers.ts`)添加到`/src`文件夹中。

📄文件:`src/type-defs.ts`

📄文件:`src/resolvers.ts`

接下来，在`/src`文件夹中创建一个`apollo-server.ts`。你可能会注意到这与我们用“传统”服务器方法设置 Apollo 服务器的[非常相似，除了我们没有用`listen()`运行服务器，而是创建了一个处理程序并将其导出为`graphqlHandler`。](https://medium.com/free-code-camp/build-an-apollo-graphql-server-with-typescript-and-webpack-hot-module-replacement-hmr-3c339d05184f)

📄文件:`src/apollo-server.ts`

# 步骤 3:安装无服务器框架

现在我们已经用 TypeScript 实现了 Apollo 服务器，但是我们如何“运行”服务器呢？与“传统”服务器不同，你不能简单地运行`npm start`并期望我们的“无服务器”Apollo 服务器神奇地响应任何传入的请求。我们需要将我们的 Apollo 服务器“转换”成我们的平台提供商(在我们的例子中是 AWS)理解的无服务器功能(Lambdas)。为了实现这一点，你可以试着用强硬的方式去做，并自己去实现；或者，如果你像我一样是个懒人，我们不妨利用其他聪明人已经建立的框架。

## 介绍“[无服务器框架](https://www.serverless.com/)

[无服务器框架](https://www.serverless.com/)提供工具和组件，帮助*开发、部署、故障排除和保护无服务器应用，同时大幅降低开销和成本*。它支持多种平台(例如 AWS、Azure、Google Cloud 等。)以及不同的编程语言和框架(如 NodeJS、Python、Go、Java 等。).[官方无服务器文档](https://www.serverless.com/framework/docs)提供了一些不错的教程和例子，所以如果你想了解更多关于无服务器框架的知识，一定要去看看。

够了，没有服务器的谈话😝。现在我们需要安装来自 NPM 的*无服务器框架*:

```
$ npm install -g serverless
```

并在您的项目根中创建一个`serverless.yml`。

📄文件:`serverless.yml`

`serverless.yml`非常简单:很可能你只需要替换`services`和`region`的值来满足你的需要。如果您需要配置的更多细节，请参考`[serverless.yml](https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml)` [参考](https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml)。

在`serverless.yml`中，你可能会注意到包含了一个`serverless-webpack`插件，你可能会想为什么我们使用 [Webpack](https://webpack.js.org/) 来编写服务器端代码？

不像长期运行的服务器只启动一次，然后运行几天，无服务器 Lambdas 可能会在每次被调用时触发冷启动，因此较小的捆绑模块或多或少会有助于提高性能。克里斯·阿姆斯特朗写的这篇文章详细地解释了这个话题。

要将`[serverless-webpack](https://github.com/serverless-heaven/serverless-webpack)`插件添加到我们的项目中，我们需要安装一些开发依赖包:

```
$ npm install --save-dev serverless-webpack webpack webpack-node-externals ts-loader
```

并在您的项目根目录中创建一个`[webpack.config.js](https://webpack.js.org/configuration)`。

📄文件:`webpack.config.js`

现在是时候部署我们的阿波罗服务器了！如果您正确地遵循了这些步骤并运行:

```
$ serverless deploy --stage prod
```

`serverless`应该输出类似于这个例子的东西:

```
$ serverless deploy --stage prodServerless: Bundling with Webpack...
Time: 3894ms
Built at: 08/04/2020 9:53:38 PM
Asset      Size  Chunks             Chunk Names
src/apollo-server.js  10.6 KiB       0  [emitted]  src/apollo-server
Entrypoint src/apollo-server = src/apollo-server.js
[0] external "apollo-server-lambda" 42 bytes {0} [built]
[1] ./src/apollo-server.ts 449 bytes {0} [built]
[2] ./src/resolvers.ts 193 bytes {0} [built]
[3] ./src/type-defs.ts 294 bytes {0} [built]
Serverless: Package lock found - Using locked versions
Serverless: Packing external modules: apollo-server-lambda@^2.16.1, graphql@^15.3.0
Serverless: Packaging service...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
........
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service demo-apollo-server-lambda.zip file to S3 (6.07 MB)...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
....................................
Serverless: Stack update finished...
Service Information
service: demo-apollo-server-lambda
stage: prod
region: ap-southeast-2
stack: demo-apollo-server-lambda-prod
resources: 13
api keys:
None
endpoints:
POST - [https://abcde12345.execute-api.ap-southeast-2.amazonaws.com/prod/graphql](https://abcde12345.execute-api.ap-southeast-2.amazonaws.com/prod/graphql)
GET - [https://abcde12345.execute-api.ap-southeast-2.amazonaws.com/prod/graphql](https://abcde12345.execute-api.ap-southeast-2.amazonaws.com/prod/graphql)
functions:
graphql: demo-apollo-server-lambda-prod-graphql
layers:
None
```

注意输出中的`POST`端点。这是你的阿波罗服务器的网址。现在你可以尝试用诸如 [Postman](https://learning.postman.com/docs/sending-requests/supported-api-frameworks/graphql) 或 [cURL](https://developer.github.com/v4/guides/forming-calls/#communicating-with-graphql) 之类的工具查询你的 Apollo 服务器:

```
# POST Request.
query GetTestMessage {
  testMessage
}
```

它应该用一个`Hello World!`消息来响应。

```
{
  "data": {
    "testMessage": "Hello World!"
  }
}
```

恭喜你！您已经成功部署了您的第一台无服务器 Apollo 服务器！

# 第四步:地方发展

现在我们已经用 AWS Lambda 部署了我们的 Apollo 服务器…但是如果我们需要更新或增强我们的服务器呢？“无服务器”是否意味着我们需要将每个代码变更都部署到 AWS Lambda 上以便进行测试？幸运的是，有一个无服务器插件可以在本地模拟 AWS Lambda 和 API Gateway，以加快开发周期。

启用这个插件是显而易见的。首先，安装 NPM 软件包:

```
$ npm install --save-dev serverless-offline
```

并将`serverless-offline`插件添加到`serverless.yml`，在`serverless-webpack`之后:

📄文件:`serverless.yml`

就是这样！现在如果你跑:

```
$ serverless offline start
```

您应该会看到类似如下的输出:

```
POST  [http://localhost:3000/dev/graphql](http://localhost:3000/dev/graphql)
POST  [http://localhost:3000/2015-03-31/functions/graphql/invocations](http://localhost:3000/2015-03-31/functions/graphql/invocations)
GET   [http://localhost:3000/dev/graphql](http://localhost:3000/dev/graphql)
POST  [http://localhost:3000/2015-03-31/functions/graphql/invocations](http://localhost:3000/2015-03-31/functions/graphql/invocations) offline: [HTTP] server ready: http://localhost:3000 🚀
offline:
offline: Enter "rp" to replay the last request
```

现在，如果您尝试对`http://localhost:3000/dev/graphql`、
运行 GraphQL `testMessage`查询，您应该会得到相同的`Hello World!`响应。

# 步骤 5:管理环境变量

在构建真实世界的应用程序时，您很可能必须处理特定于环境的配置，例如`DEV` / `PROD`环境的数据库连接字符串，或者不应该签入源代码控制的敏感信息，如数据库凭证。

![](img/8f69bcbe240922f76fb4c35116b9b5c5.png)

如果您一直在从事 Node.js 项目，我打赌您一直在使用(或者至少听说过)NPM 包。对于`serverless`项目，有一个类似于`.env`的插件，帮助管理不同环境中的环境变量并将其预加载到无服务器中。所以让我们继续安装这个插件:

```
$ npm install --save-dev serverless-dotenv-plugin
```

接下来，将插件添加到`serverless.yml`中，作为`plugins`下的第一项。

📄文件:`serverless.yml`

现在，为了演示它如何跨不同的环境工作，在项目根目录下创建两个`.env`文件:一个用于`development`，另一个用于`production`。

📄文件:`.env.development`

📄文件:`.env.production`

在`src`下创建一个`environment.ts`文件，类似于我们在上一篇文章中对[的操作:](https://medium.com/free-code-camp/build-an-apollo-graphql-server-with-typescript-and-webpack-hot-module-replacement-hmr-3c339d05184f)

📄文件:`src/environment.ts`

并更新`src/resolvers.ts`以从`environment.ts`获取值。

📄文件:`src/resolvers.ts`

`serverless-dotenv-plugin` [根据您的目标环境](https://github.com/colynb/serverless-dotenv-plugin#automatic-env-file-resolution-as-of-verson-30-thanks-to-danilofuchs)自动解析正确的 `[.env](https://github.com/colynb/serverless-dotenv-plugin#automatic-env-file-resolution-as-of-verson-30-thanks-to-danilofuchs)` [文件。要查看实际情况，请尝试在本地启动服务器:](https://github.com/colynb/serverless-dotenv-plugin#automatic-env-file-resolution-as-of-verson-30-thanks-to-danilofuchs)

```
$ serverless offline start
```

并尝试在`[http://localhost:3000/dev/graphql](http://localhost:3000/dev/graphql:)` [:](http://localhost:3000/dev/graphql:) 上运行 GraphQL `testMessage`查询

```
query GetTestMessage {
  testMessage
}
```

您应该会得到以下响应:

```
{
  "data": {
    "testMessage": "Secret message for DEV!"
  }
}
```

现在，如果您尝试将更改部署到 AWS Lambda 上:

```
$ NODE_ENV=production serverless deploy --stage prod
```

并尝试在您的 Lambda URL 上运行相同的 GraphQL `testMessage`查询，您应该得到以下响应:

```
{
  "data": {
    "testMessage": "Secret message for PROD!"
  }
}
```

如果您登录到 [AWS 管理控制台](https://aws.amazon.com/console)并查找您刚刚部署的 Apollo Server Lambda，您应该看到来自`.env.production`的环境变量被注入到您的 [Apollo Server Lambda 的环境变量](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars)。

# 最后:清理

如本文开头所述，尽管 AWS 为其大部分服务提供免费等级使用，但如果您的应用程序使用量超过免费等级，仍会涉及定价。作为一个经验法则，总是积极主动地管理你的应用程序的使用，并移除任何不再使用的资源，以避免意外的费用。

上述`serverless remove`命令应删除已部署的服务。

```
$ serverless remove --stage prod
```

并且您可以验证在 [AWS 管理控制台](https://aws.amazon.com/console)中不应该留下部署足迹。

# 包扎

就这样了。我希望你喜欢读这篇文章:)

源代码可以在 GitHub 上找到:
[https://github.com/derek-fong/demo-apollo-server-lambda](https://github.com/derek-fong/demo-apollo-server-lambda)

我相信无服务器应用程序有很大的潜力。当无服务器方法开始流行起来的时候，看看未来几年在开发领域会发生什么会很有趣。然而，就像一般的软件开发一样，无服务器方法不是一个万灵药，在下一个项目采用无服务器之前，当您权衡每种部署策略的利弊时，您应该知道好的、坏的和不好的。

喜欢读我的故事吗？请留下一些掌声👏🏻下面是我的一些其他关于 GraphQL 的文章，你可能会感兴趣:

*   [如何使用 TypeScript 和 Webpack 热模块替换构建 Apollo GraphQL 服务器](https://medium.com/free-code-camp/build-an-apollo-graphql-server-with-typescript-and-webpack-hot-module-replacement-hmr-3c339d05184f)
*   [如何将 Apollo GraphQL 服务器与 MongoDB 和 TypeScript 代码生成器集成](https://medium.com/better-programming/how-to-integrate-an-apollo-graphql-server-with-mongodb-and-typescript-code-generator-b029d821591)
*   …很快会有更多的选择！

对我下一个主题的问题、反馈或建议？请在下面留言💁🏻‍♂️
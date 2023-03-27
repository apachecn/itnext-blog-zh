# 让我们用 GraphQL 在 AWS Lambda 上测试一下 Prisma 2🏎️

> 原文：<https://itnext.io/lets-take-prisma-2-for-a-test-drive-on-aws-lambda-with-graphql-%EF%B8%8F-f4be711e93cc?source=collection_archive---------1----------------------->

![](img/8b45dd0ee0c65a159fbfd7dd0926a941.png)

[Prisma](https://www.prisma.io/) 是针对[节点](https://nodejs.org/)的开源数据库工具包。意在取代 [ORMs](https://blog.bitsrc.io/what-is-an-orm-and-why-you-should-use-it-b2b6f75f5e2a) 像[type ORMs](https://typeorm.io/)、 [Knex.js](https://knexjs.org/) 等。Prisma 也是在后端 API 中编写原始 SQL 的一种选择。最初它是用 [GraphQL](http://graphql.org) 来吹捧的，然而，你也可以用 [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) 来使用它。Prisma 通过创建与数据库模型相匹配的查询，使您更容易使用数据库。

> 注意 Prisma 目前只支持 [MySQL](https://www.mysql.com/) 、 [PostgreSQL](https://www.postgresql.org/) 和 [SQLite](https://www.sqlite.org/) 。

我以前在一个项目[中使用过 Prisma 1，并经历了一段糟糕的部署时间。这一次，有关于如何部署它的清晰文档。因此，我将在这篇文章的剩余部分回顾我在](https://github.com/AmoDinho/formulaone-graphql) [AWS Lambda](https://aws.amazon.com/lambda/) 上试驾 Prisma 2 时的经历。

GitHub 上有一个回购协议，其中的最终代码将在本文中使用，在我看来，你可以开始运行并构建一些重要的东西。

在这篇文章中，我假设你以前使用过 [AWS](https://www.serverless.com/framework/docs/providers/aws/guide/credentials/) 和[无服务器框架](https://www.serverless.com/framework/docs/providers/aws/guide/installation/)。🤰如果不是的话，[无服务器堆栈](https://serverless-stack.com/)教程是一个很棒的免费资源！🌠

# 如何着手设置一切？

Prisma CLI 将是你用来建立你的 Prisma 项目，并告诉它你正在使用什么数据库。

而 Prisma 客户端将查看您的数据库并为您进行所有查询。在这个特殊的例子中，我使用的是 GraphQL，Prisma 通常提倡在制作 GraphQL 模式时使用[代码优先的方法](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3)。这将要求你使用他们的 [Nexus 库](https://nexus.js.org/)。在这个特别的项目中，我打算采用 [SDL 优先](https://www.apollographql.com/docs/graphql-tools/generate-schema/)的方法。🤠

所以我首先用[无服务器 GraphQL 节点启动器](https://github.com/pimp-my-book/serverless-graphql-nodejs-starter)搭建了一个[无服务器](https://serverless.com/)项目。然后，我在我的本地机器上用 MySQL 创建了一个数据库，模式如下:

![](img/29a8bfa70e0506c649c5b452fbe86c32.png)

我使用 [MySQL workbench](https://dev.mysql.com/downloads/workbench/) 来创建表格。

接下来，我安装了 Prisma CLI 并初始化了项目。完成后，我将我的本地数据库 URL 添加到`schema.prisma`文件中:

![](img/c97d16f6bbcf52b4e7b17a7ef9c6ab31.png)

在这之后，我们需要自省数据库，这样 Prisma 就可以用下面的命令为我们生成模型:

```
$ prisma introspect
```

这将为您创建一个数据模型，如下所示:

![](img/92711969be6e1c1a264df9c287ad3e98.png)

现在请记住，我们需要使用`[@prisma/client](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/api)`来允许我们连接到数据模型，以便能够针对我们的数据库运行查询。如果您在您的终端中运行`***yarn prisma generate***`,您应该得到以下内容:

![](img/e93e4696ee2dd9f2baad8cf3c22628fd.png)

有了所有的设置，您应该能够针对您的数据库编写查询。因为我使用的是 GraphQL，所以我设置了一个简单的模式:

![](img/1a398ca24f9e16905e27c4efb14debae.png)

所以在我的例子中，我只是写了一个突变来创建一个 F1 车队:

![](img/d963f5128de716e87e654120f1df113c.png)

简单吧？如果我在 Playgroud 中执行这个，在运行`sls offline`之后，它会从新创建的变异中返回数据:

![](img/7fdbb91e96e1ea8d24ab6f6a71bbffe9.png)

所以现在一切都在本地工作，但是如果你在云中开发应用足够长的时间，你就会知道它在云上工作之前是不工作的。

关键是确保您的`package.json`中有必要的 Prisma 命令，并且正确配置了您的 [Webpack](https://webpack.js.org/) 。很多功劳都要归功于 [Salvatore](https://dested.com/) (查看他的 [fullstack 项目](https://github.com/dested/OrbitalGa.me)的类似堆栈)，他在部署到 AWS 时遇到了类似的问题。CloudWatch 给出以下错误:

![](img/73aeaeb493d291f6986d36868f05e4c1.png)

为了避免这个错误，请确保这些命令在您的`***package.json:***`中

![](img/6083c118a56b40a749dbaf361701df97.png)

现在您需要添加 [copy-webpack-plugin](https://webpack.js.org/plugins/copy-webpack-plugin/) ，它将使 webpack 能够访问我们的代码绑定后所需的二进制文件。

接下来，确保您在 Webpack 部分下的`***serverless.yml***`的定制部分引用了`prisma generate` 命令。这样做是为了确保您的 Prisma 客户端库是在`***node_modules:***`中创建的

![](img/3ae3006368b8791e1481ce6c539d8e47.png)

最后，我们只需要相应地更新我们的 Webpack 配置，告诉插件在哪里可以找到二进制文件(`***schema.prisma***`)并将其复制到我们的构建中:

![](img/019fe236f1d37e45908a7c033157f5ad.png)

一切都应该在代码方面完成🚀。

现在我们需要一个[无服务器 RDS Aurora 集群](https://aws.amazon.com/rds/aurora/serverless/)的基础设施。确保你有一个 AWS 账户，并为你的 Lambda 配置了一个 [VPC](https://aws.amazon.com/vpc/) 。你可以按照这篇[文章](https://hackernoon.com/a-crash-course-on-serverless-with-aws-building-apis-with-lambda-and-aurora-serverless-49885c46e37a)的第一部分在 RDS 上配置 Aurora Serverless。如果您有时间，您可以通过查看来自无服务器框架团队的这个[示例报告](https://github.com/serverless/examples/tree/master/aws-node-graphql-and-rds)来尝试将其配置为 [IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_code) 。

完成所有这些后，确保您在`serverless.yml.`中引用了您的 VPC，然后确保您已经将连接 URL 添加到您的`**.env**`文件中，以便 Prisma 能够连接到云中的数据库。

这里有一个`.env`文件的例子(由 Prisma 提供):

```
DATABASE_URL="mysql://johndoe:randompassword@localhost:3306/mydb"
```

下面是带有 VPC 端点的`***serverless.yml***`的样子:

![](img/98fb152c6b0262a1328257f82090a20a.png)

现在我们可以部署这个功能:

```
$ sls deploy
```

如果一切顺利，您的控制台输出应该如下所示:

![](img/682a9417386e7daff44dd1ea558df853.png)

我们应该能够在点击提供的端点后创建一个突变:

![](img/4adc03233476c7468fec3608e0bb69b4.png)

请确保“/dev/”位于 Lambda 终结点和“graphql”之间

这个例子没有考虑到`NODE_ENV`阶段中的变化，在这个阶段中，基于您的环境(开发、暂存等)，您会有不同的数据库 URL。为此，我会尝试以下方法:

```
 process.env.database_url = process.env.NODE_ENV = DEVELOPEMENT ? ‘dev_url’ : ‘prod_url’
```

# 太酷了，那么我们为什么要在目前的野外使用这个呢🤷‍♂️?

## 原始 SQL

就我个人而言，我不想再写原始 SQL，这是我在之前的项目中写的:

![](img/077a06a5483662f7bbd74e77b01fd2d0.png)

这是因为 Knex 和 [Sequelize](https://sequelize.org/) 对 Lambda 不起作用，这最终让我别无选择，只能编写原始 SQL。

使用 Prisma 的开发者体验是无缝的！一个小时后，我有了一些在本地运行的东西，这将使它成为向您的 CTO/技术主管宣传它的有力案例。

## Prisma 工作室

如果您在终端中运行以下程序:

```
$ prisma studio --experimental
```

它会在你的浏览器中打开一个新窗口😯：

![](img/1b8918d48048bf4342d77ac8262cdd28.png)

Prisma 工作室

Prisma Studio 允许我在一个漂亮的 GUI 中查看与我创建的模型相关的所有数据，并可以进行基本的 CRUD 操作。这完全消除了为这些类型的工作流安装其他第三方工具的需要。🎉

然而，Prisma 2 仍处于测试阶段，因此您可能希望不要将其用于生产工作负载。一旦它普遍可用，并且有了这篇文章的内容，你应该能够用 GraphQL 在 Lambda 上制造一个领先的、生产就绪的 API！💰

现在你已经有了一个前沿的开发者体验，去取悦你的用户吧！
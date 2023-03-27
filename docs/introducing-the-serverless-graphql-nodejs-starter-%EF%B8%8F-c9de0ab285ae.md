# 无服务器 GraphQL NodeJS Starter 简介🤩👷🏾‍♂️

> 原文：<https://itnext.io/introducing-the-serverless-graphql-nodejs-starter-%EF%B8%8F-c9de0ab285ae?source=collection_archive---------5----------------------->

![](img/5981f5976a39e25b24130d821f3c8432.png)

我发现自己在年中过于频繁地手工编写无服务器的 GraphQL APIs。所以我从制作了[无服务器堆栈教程](https://serverless-stack.com/)的可爱的人们那里拿到了[无服务器 NodeJS 初学者工具包](https://github.com/AnomalyInnovations/serverless-nodejs-starter)，并对其进行了修改以支持 [GraphQL 查询和变异](https://graphql.org/learn/queries/)。因此推出了[无服务器 GraphQL 入门套件](https://github.com/pimp-my-book/serverless-graphql-nodejs-starter)！

该套件专门针对 [AWS Lambda](https://aws.amazon.com/lambda/) 和 [Apollo 服务器](https://www.apollographql.com/docs/apollo-server/)。然而，我很确定你可以用 [GCP](https://cloud.google.com/) 和 [Azure](https://azure.microsoft.com/en-us/) 来配置它，因为 Apollo 确实有它们的实现。一旦安装好，你就可以部署这个功能，通过链接其他资源来扩展它，比如 [DynamoDB](https://aws.amazon.com/dynamodb/) 表、 [S3 桶](https://aws.amazon.com/s3/)等等。

如果你想采用微服务的方法，你需要尝试阿波罗联盟 T21，并做好动手的准备。此外，如果您正在寻求使用[订阅](https://www.apollographql.com/docs/apollo-server/data/subscriptions/)进行实时用例，我会推荐使用 [AWS AppSync](https://aws.amazon.com/appsync/) ，这是一个无服务器的 GraphQL 管理的后端。然后你只需要在你的前端配置你的客户端来接受[多个 API 端点](https://medium.com/open-graphql/apollo-multiple-clients-with-react-b34b571210a5)，那么 Bob 就是你的叔叔。

## 集成不同类型的数据库怎么样？🤔

这里有一个关于 DynamoDB 的[例子，如果你想使用](https://github.com/pimp-my-book/pmb-plus-backend-poc)[极光](https://aws.amazon.com/rds/aurora/?nc2=h_ql_prod_db_aa)，那么你可以看看这个来自[无服务器](https://serverless.com/)的 Gareth 的[精彩帖子](https://serverless.com/blog/graphql-api-mysql-postgres-aurora/)。如果你担心冷启动，因为你的 lambda 在 VPC(虚拟私有云)中，你不应该太担心。在我目前从事的一个项目中，我发现查询和突变的加载时间大约为 3-6 秒。

我的建议是在本地开发中使用常规的 RDS MySQL/ PostgreSQL 实例，然后在开发和生产阶段为 MySQL/ PostgreSQL 构建两个无服务器的 Aurora 集群。这是因为 Lambda 需要与 Aurora 在同一个 VPC 中，并且当在本地工作时，如果您试图访问您的无服务器 Aurora 集群，您会得到可怕的“ ***意外令牌 S in JSON in position 0***”错误消息。除非有人有可行的替代方案，请分享🙃。

此外，我无法使用 [Knex](http://knexjs.org/) 和 [Sequelize](https://sequelize.org/) 作为 ORM 让 lambda 工作。一旦我选定了无服务器的 MySQL 模块，我就万事大吉了。我得到了太多的内部服务器错误，并乐于妥协编写原始 SQL 来启动和运行项目。

如果你是无服务器或 GraphQL 的新手，在[皮条客我的书](https://pimpmybook.co.za/)我们创建了一个资源来提高工程师的技能，以跟上这些令人敬畏的技术，称为 [#CodeJavascript](https://github.com/pimp-my-book/CodeJavaScript) 。

不维护任何生产 REST APIs 的感觉。

> 画出所有的东西！！！🎉🌟🎈
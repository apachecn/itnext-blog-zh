# 使用 TypeORM 和 MySQL 的具有多个数据库连接的 NestJS 项目启动程序

> 原文：<https://itnext.io/nestjs-project-starter-with-multiple-database-connection-using-typeorm-and-mysql-47150e282ed5?source=collection_archive---------0----------------------->

![](img/aa671eff9091cf38a35db0089ced4c96.png)

在这个简短的教程中，我将使用 MySQL。第一步是创建一个新的 Nestjs 项目。你可以遵循官方的[文档](https://docs.nestjs.com/)或者使用下面的命令。

## 设置 NestJS

首先，让我们安装 nest CLI 工具。

```
$ npm i -g @nestjs/cli
```

接下来，让我们使用 CLI 初始化一个空项目

```
$ nest new project-name
```

接下来，安装所需的 npm 软件包

```
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```

## 让我们使用 Nest CLI 生成一个模块

为此，我们可以使用内置的 CLI。生成比创建自己的控制器和服务要容易得多。使用以下命令生成所需的模块、服务和控制器。我将在 *src/modules/power-search* 文件夹中生成一个名为 power-search 的新模块

```
$ nest g module modules/power-search
$ nest g controller modules/power-search
$ nest g service modules/power-search
```

创建模块后，下一步是设置开发环境和数据库连接。

## MySQL 设置

让我们首先设置我们的开发环境。我们将使用 Docker 来安装 MySQL 并在本地机器上运行它。复制下面的 docker 文件并使用`docker-compose up`运行它。如果你想了解更多关于 docker 的信息，请参考[这篇文章](/lets-dockerize-a-nodejs-express-api-22700b4105e4)。

*因为我们需要另一个数据库，所以我使用 init.sql 脚本创建了它。将脚本复制到 docker-compose 位置。*

现在运行 docker-compose，使用:

```
docker-compose up
```

一些重要的 docker 命令

```
docker-compose down # Removes the container
docker-compose down -v # Remove the created volumes as well
docker-compose up -d # -d will run the compose file in background
docker-compose build # This will just build an image
```

因为我们已经完成了本地环境的设置。让我们安装 TypeORM 和相关的包。

## NestJs 上的 Mysql

对此，也可以参考官方[文档](https://docs.nestjs.com/techniques/database)。

```
$ npm install --save @nestjs/typeorm typeorm mysql2
```

我们将使用一种简单的方法。首先，让我们在 app.module.ts 文件中将连接字符串创建为 defaultOptions 变量。

我知道在 API 中硬编码密码或任何类型的凭证都不是一个好的做法。所以要去掉这个，使用一个 **dotenv** 文件。NestJS 有一个内置的配置管理包。

如果您检查上面的代码，有 2 个连接对象。如果没有为连接提供名称，则名称将默认为' *default* '。**如果我们使用多个连接，必须为辅助连接提供一个名称！**

## 不那么混乱的方法

我们可以使用 ormconfig.json 并将所有连接指定为一个数组，并使用该名称导入每个配置。但是截至目前，**这个不行。希望在未来的版本中可以修复。**

## NestJS 应用程序设置

接下来，我们必须设置电源搜索模块。为此，请使用下面的代码示例。

请注意，您必须使用 import 提供连接名称。如果使用默认连接，则无需指定名称。

```
imports: [TypeOrmModule.forFeature([power_search], 'w')]
```

尽管如此，我们还没有完成。如果我们要使用它，我们必须在 power-search.service.ts 中指定名称。否则，它不会识别 power_search 模型是' **w** '连接的一部分。

现在我们完成了。快乐编码，感谢你阅读这篇文章❤
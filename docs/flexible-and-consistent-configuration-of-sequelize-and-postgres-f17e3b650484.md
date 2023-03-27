# Sequelize 和 Postgres 灵活一致的配置

> 原文：<https://itnext.io/flexible-and-consistent-configuration-of-sequelize-and-postgres-f17e3b650484?source=collection_archive---------2----------------------->

![](img/26d9ddd518cb3771a3fb2d2537e7d42a.png)

一只快乐的狗(图片来源于作者的照片)

`[Postgresql](https://www.postgresql.org)`是一个非常受欢迎的`SQL`服务器，理由很充分；它做得很棒，它处理`JSON`和`HSTORE`数据结构，因此给你`MongoDB`的能力和健壮的`SQL`服务器的所有相关好处，它是免费和开源的，它得到很好的支持，并且它是由`[Heroku](https://heroku.com)`提供的默认数据库。

`[Sequelize](http://docs.sequelizejs.com)`是最流行的`NodeJS`对象关系映射(`ORM`)库，也是有原因的。它是强大的，富有表现力的，成熟的。

当开发使用`Sequelize`和`Postgres`的应用程序时，我发现自己一遍又一遍地使用相同的基本模式来确保数据库连接可用。我已经发布了`[sequelize-pg-utilities](https://www.npmjs.com/package/sequelize-pg-utilities)`来将这些模式整合到一个单独的库中。非常简单，它唯一的依赖项是`[pgtools](https://www.npmjs.com/package/pgtools)`,如果需要的话，它使用这个依赖项来自动创建应用程序的数据库。

# 配置

首先，依赖数据库的应用程序需要一些配置。`Sequelize`方式是在`config/config.json`中坚持配置，这是相当有限的。为了允许定制配置，您需要将环境变量合并到标准配置文件选项中。此外，对于`config/config.json`文件中不存在的配置选项，您需要合理的默认值。

`configure`函数接受解析后的`config/config.json`文件，并返回一个配置对象，其值包括适当的环境变量值和合理的默认值。

因此，要创建一个已配置的`sequelize`实例，您可以:

```
const { configure } = require('sequelize-pg-utilities')
const config = require('path/to/config/config.json')**const { name, user, password, options } = configure(config)**const sequelize = new Sequelize(name, user, password, options)
```

以下环境变量优先于`config/config.json`中定义的变量。

*   `DATABASE_URL`如果提供了数据库 url，它将覆盖下面的许多`DB`设置。
*   `DB_NAME`数据库名称——您也可以提供一个默认值(见下文)
*   `DB_USER`数据库用户—无默认值
*   `DB_PASS`数据库密码—无默认值
*   `DB_POOL_MAX`数据库连接的最大数量——默认为`5`
*   `DB_POOL_MIN`数据库连接的最小数量—默认为`1`
*   `DB_POOL_IDLE`数据库空闲时间—默认为`10000`毫秒
*   `DB_HOST`数据库主机—默认为`‘localhost’`
*   `DB_PORT`数据库端口—默认为`5432`
*   `DB_TYPE`数据库类型——默认为`‘postgres’`——这个库是用 Postgres 编写的，所以除非你知道你在做什么，否则请不要改变它。

如果你提供`DATABASE_URL`环境变量，就像`[Heroku](https://heroku.com)`和其他`paas`系统通常做的那样，那么`configure`函数将从中提取它所需要的大部分。提取的值将优先于后续值。

# 初始化数据库

通常，您会希望应用程序在第一次运行时创建数据库，尤其是在`development`和`test`环境中。为此，您可以使用`makeInitialiser`功能对`initialiser`进行适当配置。

```
const { makeInitialiser } = require('sequelize-pg-utilities')
const config = require('path/to/config/config.json')**const initialise = makeInitialiser(config)**const start = async () => {
  try {
    await initialise() // now do whatever else is needed
    // to start your server
  } catch (err) {
    console.error(err)
    process.exit(1)
  }
}
```

您可以通过将重试次数作为参数传递给`initialise`来设置重试次数。默认是`5`。

```
const result = await initialise(10)
```

# 配置序列 CLI

`Sequelize CLI`要求您在项目的根目录下定义一个`.sequelizerc`文件，该文件导出诸如`config`、`migrations-path`和`models-path`之类的数据。

该配置的格式如下:

```
{
  [env]: { username, password, database, options }
}
```

您可以使用`migrationConfig`函数来生成配置细节，以满足`Sequelize CLI`的需求。

在你的代码中，创建一个`migrationConfig.js`文件，如下所示:

```
const { migrationConfig } = require('sequelize-pg-utilities')
const config = require('path/to/config/config.json')module.exports = migrationConfig(config)
```

您的`.sequelizerc`文件变成:

```
const path = require('path')module.exports = {
  config: path.resolve(__dirname, 'path', 'to', 'migrationConfig.js'),
  'migrations-path': path.resolve(__dirname, 'migrations'),
  'models-path': path.resolve(__dirname, 'src', 'models')
}
```

# 选择

`configure`、`makeInitialiser`和`migrationConfig`功能都具有相同的签名。它们接受以下参数。

*   `config`:解析后的`config/config.json`文件。必需，无默认值。
*   `defaultDbName`:如果数据库名称没有在环境变量中设置，并且如果`config`文件没有定义数据库名称，那么使用这个作为数据库名称。可选，无默认值。
*   `operatorsAliases` : Sequelize 建议你不要使用`[operators aliases](http://docs.sequelizejs.com/manual/tutorial/querying.html#operators-aliases)`，但是如果你想的话可以在这里设置。可选，默认为`false`。
*   `logger`:你可以在这里传递一个 logger 函数给 Sequelize 使用。可选，默认为`false`，表示不记录任何内容。

# 摘要

您可以使用`sequelize-pg-utilities`简化并保持`Sequelize` / `Postgres`配置和初始化的一致性，这将公开以下功能:

*   `configure`:使用序列配置文件中的数据以及环境变量和合理的默认值来创建标准配置对象。
*   创建一个配置好的、异步的、重试的数据库`initialise`函数，如果它不存在，你可以用它来创建你的数据库。
*   `migrationConfig`:产生`.sequelizerc`文件所需的正确格式配置选项的功能。

# 链接

*   一个非常流行和强大的 SQL 服务器。
*   `[Sequelize](http://docs.sequelizejs.com)`一个非常流行和强大的 ORM 工具
*   `[Heroku](https://heroku.com)`一个非常流行的`platform-as-a-service`工具
*   `[pgtools](https://www.npmjs.com/package/pgtools)`一个纯`NodeJS`实现的`Postgresql’s`和`[createdb](http://www.postgresql.org/docs/9.4/static/app-createdb.html)`实用程序。
*   `[sequelize-pg-utilities](https://www.npmjs.com/package/sequelize-pg-utilities)`在 NPM
*   GitHub 中的`[sequelize-pg-utilities](https://github.com/davesag/sequelize-pg-utilities)`源代码

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。
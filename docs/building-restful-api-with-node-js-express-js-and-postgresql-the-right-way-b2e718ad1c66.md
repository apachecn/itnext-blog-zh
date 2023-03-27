# 用 Node.js 构建 RESTful Api，Express。正确使用 Js 和 PostgreSQL

> 原文：<https://itnext.io/building-restful-api-with-node-js-express-js-and-postgresql-the-right-way-b2e718ad1c66?source=collection_archive---------0----------------------->

![](img/c5e5271a940439463b6fe51ac070846d.png)

在我的文章[“用 NodeJs 和 Cloudinary 生成 Pdf”](https://medium.com/@effectblessing/generating-a-pdf-with-node-js-express-cloudinary-8a9b9160207)之后，我收到了一些人的反馈，他们要求我写一篇关于用 Nodejs 构建 RESTful Api 的文章。所以我决定把这篇文章放在这里，希望它能帮助到一些人。

# 我们将建造什么？

您将了解如何构建一个公共汽车交通预订 API——这样用户就可以预订他们的旅程，可以看到他们所有的预订，并有权取消任何预订。

> *唯一不可能的旅程是你从未开始的旅程。🙌—托尼·罗宾斯*

# 目录

*   入门指南
*   先决条件
*   项目设置
*   项目结构
*   设置数据库(PostgrelSQL)
*   服务器助手
*   JWT 认证和授权
*   设置控制器
*   设置路线
*   服务器设置
*   邮递员测试
*   结论

# 入门指南

在这篇文章中，我将解释如何连接和使用 [PostgreSQL](https://www.postgresql.org/) 数据库来存储数据。我们还将学习如何编写一些基本的[SQL](结构化查询语言)查询。SQL 是一种在数据库中存储、操作和检索数据的标准语言。为了简单起见，我们可以更好地理解 SQL 是如何工作的，在这篇文章中我不打算使用 ORM(对象关系映射)。查看 [stackoverflow](https://stackoverflow.com/a/1279678) 上的[这个](https://stackoverflow.com/a/1279678)答案，更好地理解 ORM。ORM 基本上是一种技术，它允许我们在不了解 SQL 的情况下查询和操作数据库中的数据。ORM 包公开了查询和操作数据库中的数据所需的一些方法。我们有几个 ORM 包可以用于我们的数据库，例如 [Sequelize](http://docs.sequelizejs.com/) 。不使用 ORM，我们将直接使用 [PG](https://node-postgres.com/) NodeJS 包——PG 是一个 NodeJS 包，用于与 PostgreSQL 数据库接口。单独使用 [PG](https://node-postgres.com/) 也将使我们有机会理解一些基本的 SQL 查询，因为我们将使用原始 SQL 查询来查询和操作数据库中的数据。

# 先决条件

虽然我会尽量把事情简单化。然而，在深入本教程之前，掌握一些关于 Nodejs 的基本知识是很重要的，如果你不友好地遵循这个[指南](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)，这篇文章假设你已经在你的 PC 上安装了 Nodejs 和 Npm。

# 项目设置

*   在你的系统上创建一个新的项目目录，你可以把它叫做`**transportApi**`
*   将工作直接转换到项目上，如果你使用的是窗口系统，在你的终端或命令提示符下运行`npm init`——运行`npm init`会提示你一些问题来帮助设置你的项目

![](img/1df98db5597edef8f539c33dc79ecf8d.png)

完成后，您应该会在项目中看到`package.json`文件，它包含了项目的基本信息。

```
{"name": "transportapi","version": "1.0.0","description": "","main": "server.js","scripts": {"test": "echo \"Error: no test specified\" && exit 1"},"author": "Beveloper","license": "ISC"}
```

**安装项目的所有依赖项:**

我们将安装这个项目所需的依赖没有花哨的外部软件包，如摩根等。

*   [expressjs](https://expressjs.com/) — Expressjs 是一个 nodejs web 应用程序框架，它使我们能够创建快速简单的 API。
*   PG— PG 是一个 NodeJs 包，用于连接 PostgreSQL 数据库。
*   [JSON Web Token](https://jwt.io/)(JWT)——是一个[开放标准](https://tools.ietf.org/html/rfc7519)，用于作为 JSON 对象在各方之间安全地传输信息。它的发音是 *jot* ，或者像我们的荷兰朋友说的那样， [*yaywaytay*](https://news.ycombinator.com/item?id=14293602) 。JWT 通常用于授权。jwt 可以使用秘密或公钥/私钥对进行签名。用户登录后，每个后续请求都需要 JWT，从而允许用户访问该令牌允许的路由、服务和资源。
*   主体解析器——在处理程序之前解析中间件中的传入请求主体，在`req.body`属性下提供。
*   moment——是一个轻量级的 JavaScript 日期库，用于解析、验证、操作和格式化日期。
*   dotenv——是一个零依赖模块，将环境变量从`.env`文件加载到`[process.env](https://nodejs.org/docs/latest/api/process.html#process_process_env)`
*   CORS(跨源资源共享)—是一个 node.js 包，允许请求跳过同源策略并从远程主机访问资源。
*   Bcryptjs —是一种帮助您散列密码的安全方法。
*   @babel/polyfill —是一个工具链，主要是**用来**将 ECMAScript 2015+代码转换成当前和旧版本浏览器或环境中的 JavaScript 向后兼容版本。
*   [巴别塔手表](https://www.npmjs.com/package/babel-watch) —这是开发需要的。babel watch package 做的一件事是编译我们的代码，并在每次我们对代码进行更改时重新加载服务器。

运行以下命令来安装上述所有软件包

```
**$** npm install --save express pg moment body-parser dotenv jsonwebtoken cors make-runnable bcryptjs [@babel/polyfill](http://twitter.com/babel/polyfill) npm-run-all $ npm **install** --save-dev babel-core **babel-cli** babel-preset-env **babel-watch babel-preset-es2015 babel-register**
```

如果一切顺利，您应该会看到类似这样的内容

![](img/70bed9d81a1cb6f4476ea761a6d04693.png)

您会注意到`express and moment` etc 在`dependencies`下面，这是因为在我们将代码部署到生产环境中时，这些是必需的。`babel-cli, babel-preset-env and babel-watch etc`仅在开发期间需要。

# 项目结构

在您的系统上安装 [PostgreSQL](https://www.postgresql.org/) 。如果您的电脑上没有安装，请浏览本[指南](https://www.guru99.com/download-install-postgresql.html)

使用以下格式设置项目结构；

```
TransportApi |-app
   |-controllers
   |-db
   |-helpers
   |-middlewares
   |-routes
 |-node_modules
 |-.babelrc
 |-.env
 |-env.js
 |-package-lock.json
 |-package.json
 |-server.js
```

![](img/87c49cefd65694ae7eac16f05748fbb4.png)

# 设置数据库

让我们创建我们的数据库并将其命名为`transportApi`。为了创建 DB，我们可以使用任何 PostgreSQL 客户端，例如针对 Mac 用户的 [POSTICO](https://eggerapps.at/postico/) 或针对 window 用户的 [PgAdmin4](https://www.pgadmin.org/download/) 。
如果你需要一个在你的电脑上设置 PostgrelSQL 的指南，看看这个简单易懂的[指南](https://www.guru99.com/download-install-postgresql.html)

## 创建 TransportApi 表

首先，让我们在系统环境中保存我们的数据库 URL。你的数据库网址应该是以下格式
`postgres://{db_username}:{db_password}@{host}:{port}/{db_name}`
例如`postgres://transportUser:develop@localhost:5235/transport`

如果您无法在本地设置 DB，简单的方法是使用任何 PostgreSQL 云 DB，比如 [ElephantSQL](https://www.elephantsql.com/) 。如果使用 cloud PostgreSQL DB，则复制 URL 并使用它。我的端口可能与你的不同，所以请确保使用正确的 p。

**环境变量设置**

打开根目录下的`.env`文件，将代码复制粘贴到其中。

```
*#.env*DATABASE_URL=postgres://transportUser:develop@*localhost:5235*/transportPORT=5253
```

*   我们定义了几个环境变量(`***DATABASE_URL*** AND ***PORT***`)，这样我们可以在以后的项目中使用它们。

接下来，我们需要在数据库中创建`transportApi`表。在这里，我们将开始使用带有基本 SQL 查询的`pg`包来创建表。

在我们的`db folder`中创建一个名为`**dev**` 的文件夹，并分别创建三个名为 dbConnection.js、dbQuery.js 和 pool.js 的文件。

文件应该是这样的。

代码步骤:

*   从`pg`导入`Pool`对象。查看第[页文档](https://node-postgres.com/features/pooling)以了解更多关于`pooling`的信息。我们用它来连接我们的 PostgreSQL 数据库。Pool 是使用 pg 最简单、最常见的方式。您还可以利用他们的客户端 API 来连接到数据库。
*   从`dotenv`导入`dotenv`并使用`dotenv.config()`加载它——这是为了在我们的项目中搜索`.env`文件并将其内容加载到系统环境中，这样我们就可以使用节点`process.env`来访问这些变量。
*   我们创建了一个新的`Pool`实例，并将`connectionString`传递给它的构造函数。我们使用`process.env.DATABASE_URL`从系统环境中获取`DATABASE_URL`变量。
*   我们监听了池`connect`事件和控制台日志`connected to the db`

代码步骤:

*   我们创建了一个新方法`query`，它接受两个参数`text`——查询文本和`params`—`text`所需的值。这两个参数是查询数据库所需要的。该方法返回一个承诺，我们将在控制器中调用它。点击[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)阅读更多关于 JavaScript Promise 的内容。

代码步骤:

*   从`pg`导入`Pool`对象。查看 pg [文档](https://node-postgres.com/features/pooling)以了解更多关于`pooling`的信息。我们用它来连接我们的 PostgreSQL 数据库。Pool 是使用 pg 最简单、最常见的方式。您还可以利用他们的客户端 API 来连接到数据库。
*   我们听了池`connect`事件和控制台日志`connected to the db`
*   我们设置了一个`createTables()`函数，函数内部是一个查询，该查询使用唯一的字段分别创建`userTbale, busTable, tripTable and bookingTable` 。

```
const *createUserTable* =()=>{ const *userCreateQuery* =`*CREATE TABLE IF NOT EXISTS users* *(id SERIAL PRIMARY KEY,* *email VARCHAR(100) UNIQUE NOT NULL,* *first_name VARCHAR(100),* *last_name VARCHAR(100),* *password VARCHAR(100) NOT NULL,* *created_on DATE NOT NULL)*`;
```

上面所做的是告诉 PostgreSQL DB 创建`users`表，如果没有列出字段的`users`表，其余的表也是如此。

使用第一个表(用户表)作为案例研究，我们观察到以下情况；

*   我们用`userCreateQuery`作为参数调用`pool`查询方法，它返回一个`promise`。
    我们调用`pool.end()`来关闭`pool`到数据库的连接。
*   我们创建了另一个名为`dropTables()`的函数——它删除`users`表。
    我们设置了一个新的查询`DROP TABLE IF EXISTS users`，如果数据库中存在 users 表，它将删除该表。
*   我们使用`pool.on('remove')`监听`pool` `remove`事件，使用`process.exit(0)`退出节点进程。
*   最后，我们需要`make-runnable`包——我们需要它能够从终端调用我们的两个功能中的任何一个。
    注意:你要在最后要求`make-runnable`。还记得我们之前安装了`make-runnable`作为项目开发依赖。
    你也会注意到我们使用了`require`而不是`import`，这是因为我们只想单独从终端运行`db.js`文件，它不是我们项目的直接组成部分，所以编译它没有意义。最后，让我们运行`createTables`函数来创建我们的表。为此，我们需要在我们的`Package.json`文件中添加一个命令。

在您的启动脚本中，删除测试命令并添加以下内容

```
"scripts": {"create-dev-tables": "babel-node ./app/db/dev/dbConnection createAllTables","start": "nodemon --watch . --exec babel-node -- server","setup": "npm-run-all -p start create-dev-tables"},
```

您的 package.json 应该知道如下所示

![](img/eb9018b4cd7ad68546e590cf126bf751.png)

接下来，我们运行命令`$ **npm run setup**`

![](img/814ca75669eaf531d022c97dbba080f4.png)

运行完上面的，应该会看到下面类似的东西；

![](img/f11328d04bc761833710e71c6dc3ba1f.png)

注意:除了完成上述过程，您还可以使用任何 PostgreSQL 客户端直接创建表，例如 [pgAdmin](https://www.guru99.com/create-drop-table-postgresql.html) ， [POSTICO](https://eggerapps.at/postico/docs/v1.4.1/table-ddl.html) ，或者在终端上使用 PostgreSQL 命令。

## **设置助手**

在进入核心项目之前，需要创建一些辅助方法，这将促进整个项目的可重用性。

复制以下内容并粘贴到`***app/helpers/validations.js***`中

代码步骤:

*   `isValidEmail()`方法使用 regex(正则表达式)验证用户提供的电子邮件地址。
    查看本指南，了解更多关于 [**Regex**](https://scrimba.com/g/gregularexpressions) 的信息
*   `validatePassword` —该方法检查用户在注册或登录时提供的密码长度是否小于或等于 5 个字符。
*   `isEmpty`方法检查用户提供的任何输入是否未定义或为空。
*   我们导出这个方法，这样我们就可以在我们的项目中使用它们。

复制以下内容并粘贴到`***app/helpers/status.js***`中

**代码步骤:**

*   创建了一个对象**状态**并定义了一些带有值的属性。
*   成功—当用户执行 post 请求时，返回状态代码 200
*   错误—当用户遇到故障时，返回状态代码 500
*   not found——顾名思义，当用户试图执行未定义的请求时，会返回状态代码 404，这是禁止的另一个名称😜。
*   未授权—当用户未经授权访问路径/页面时会出现这种状态，在这种情况下会返回状态代码 401。
*   已创建—创建数据时，返回状态代码 201。
*   Bad —一个很好的用例是当提供一个无效的电子邮件时，返回一个状态代码 400。
*   Nocontent —当用户提交一个空表单时，将返回状态代码 400。
*   冲突——一个很好的用例是当用户输入一个已经存在的电子邮件。应该会返回 409 状态代码。
*   我们导出这个方法，这样我们就可以在我们的项目中使用它们。

**授权和认证 *(JWT)***

![](img/35eda7bf72d713d6e12dd791eb461e30.png)

JWT 的快乐

当我们继续学习本教程时，我们的项目会有一些需要保护的路由和一些需要授权的用户。就像我一样，你可能想知道这是如何实现的。幸运的是，我们有 JSON Web 令牌(JWT)来解决这个问题。

JSON Web Token (JWT)是一个开放标准，它定义了一种紧凑的自包含方式，以 JSON 对象的形式在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

***授权*** 是使用 JWT 最常见的场景。用户登录后，每个后续请求都将包含 JWT，允许用户访问该令牌允许的路由、服务和资源。单点登录是目前广泛使用 JWT 的一个特性，因为它的开销很小，并且能够很容易地跨不同的域使用。

使用 JWT 进行授权有一些优点:

1.  纯粹的无国籍。不需要额外的服务器或基础设施来存储会话信息。
2.  它可以很容易地在服务之间共享。

在其紧凑的形式中，JSON Web 令牌由点(`.`)分隔的三个部分组成，它们是:

*   页眉
*   有效载荷
*   签名

因此，JWT 通常如下所示。

`xxxxx.yyyyy.zzzzz`

# 现在，是写代码的时候了。

![](img/6abf32142c704f7cf6e6b63a46be545b.png)

# ***认证*** ***jwt . sign()***

***jwt.sign(payload，secretkey，[选项，回调])***

第一个函数`jwt.sign()`将生成一个 JWT 令牌，把它分配给一个用户对象，然后返回那个 JWT 令牌，这样我们就可以把它传递到任何我们需要的地方。它可以是异步的，也可以是同步的，这取决于是否提供了回调。 ***payload*** 参数在我们这里会是用户对象， ***secretkey*** 是你自己编的，可以是任何东西。回调参数是我们处理发送令牌的地方，选项参数是可以设置过期时间的地方。

复制以下代码并粘贴`*app/helpers/validations.js*`

代码步骤:

在 validation.js 里面我们有一个函数***generateUserToken***

![](img/6c6828cfc42f70fea2241e61623bdaf9.png)

*   我们从它们各自的模块中导入了 jsonwebtoken(JWT)和 dotenv。
*   我们创建了一个函数`generateUserToken`，我们使用`jwt.sign(..)`对用户的令牌进行签名，通过发送电子邮件、user_id、is_admin(稍后讨论)、first_name 和 last_name 作为有效载荷，我们还传递了我们的 **secret** 并将令牌设置为 3 天后过期。*。*

# ***授权
jwt.verify()***

***jwt.verify(token，secretkey，[options，callback])***

![](img/c9bbbe0ba952fae1739fc52c612dbc32.png)

`jwt.verify()`当访问受保护的路由时，将验证用户令牌。它将令牌作为一个参数，即您在`jwt.sign()`函数中定义的密钥，然后您有选项和回调参数。回调将是我们可以访问和发送受保护数据的地方。

复制以下内容并粘贴到`*app/middleware/verifyAuth.js*`

代码步骤:

这里，我们用`verifyToken()`方法创建一个新的 Auth 对象。方法所做的基本上是使用我们在签名令牌时使用的相同密钥来验证和解码用户请求令牌。我们使用`**const{token} = req.headers**`从请求头中获取令牌，并将其与我们在签名令牌时使用的秘密一起发送给`jwt.verify(..)`。如果令牌有效，我们从令牌有效负载中检索`***email, user_id, is_admin,first_name, last_name***`，并查询 DB 以确保用户存在于 DB 中。如果用户存在于数据库中，我们在`req`对象中创建了一个新的对象属性。我们将使用它来处理其他请求处理程序。最后，由于这个方法是一个中间件，我们使用了`next()`来移动到下一个请求处理器。如果这里发生了任何错误，我们会向用户返回一条错误消息，而不必转到下一个请求处理程序。

# **设置控制器**

接下来，我们需要为我们的项目设置控制器，首先是我们的管理员和用户控制器。

## ***管理员控制器***

复制以下内容并粘贴到`***app/controllers/adminController.js***`中

代码步骤:

注意:有一些路由需要管理员权限才能访问

*   这里，我们创建了两个方法— `createAdmin`和`updateUserToAdmin`。我们还利用了`async/await`。
*   我们导入 moment，这样做的原因是因为我们将使用它来创建日期。
*   我们从`app/db/dev/dbQuery.js`导入了 dbQuery。
*   我们还从`app/helpers/status.js`中导入了我们的 ***状态*** 助手方法`empty`和`errorMessage,successMessage, status`。
*   我们还从`app/helpers/validations.js`中导入了我们的 ***验证*** 助手方法`hashPassword`和`comparePassword, isValidEmail, validPassword, isEmpty, generateUserToken`。
*   我们创建了一个函数`createAdmin`，在这个函数中，我们从请求体中解构了我们的值`email, first_name, last_name, is_admin, password`。
*   接下来，我们创建了一个名为 created_on 的变量，它保存我们创建的用户表的日期。
*   接下来，我们使用我们的助手方法`***isValidEmail.***`检查我们的电子邮件是否有效
*   此外，我们检查我们的密码字符是否超过 5 个。
*   使用我们的助手方法`hashPassword`，我们使用 bcryptjs 模块散列我们的密码。

## ***用户控制器***

复制以下内容并粘贴到`***app/controllers/usersController.js***`中

代码步骤:

*   这里，我们创建了两个方法— `createUser`和`signUser`。我们还利用了`async/await`。
*   我们导入 moment，这样做的原因是因为我们将使用它来创建日期。
*   我们从`app/db/dev/dbQuery.js`导入了 dbQuery。
*   我们还从`app/helpers/status.js`中导入了我们的 ***状态*** 助手方法`empty`和`errorMessage,successMessage, status`。
*   我们还从`app/helpers/validations.js`导入了我们的 ***验证*** 助手方法`hashPassword`和`comparePassword, isValidEmail, validPassword, isEmpty, generateUserToken`。
*   我们创建了一个函数`createUser`，在这个函数中，我们从请求体中解构了我们的值`email, first_name, last_name, password`。
*   接下来，我们创建了一个名为 created_on 的变量，它保存我们创建的用户表的日期。
*   接下来，我们使用我们的助手方法`***isValidEmail.***`检查我们的电子邮件是否有效
*   此外，我们检查我们的密码字符是否超过 5 个。
*   使用我们的助手方法`hashPassword`，我们使用 bcryptjs 模块散列我们的密码。
*   接下来，我们创建了一个名为 created_on 的变量，它保存我们创建的总线表的日期。
*   对于我们的 SQL 查询，我们使用了`INSERT INTO users(list_columns_here..) VALUES($1, $2, $3, $4, $5...)`——它的作用是在`users`表中创建一个新行，并将提供的值插入到它的字段中。`values`是一个数值数组，包含我们要插入到表格中的内容。`values`数组中的元素必须与`$1, $2, $3, $4, $5`的顺序相同。我们使用`returning *`返回创建的行。记得我们创建了`query`方法，它在`app/db/dev/dbQuery.js`中接受两个参数`quertText`和`params`，这是我们将使用它的地方。我们调用了该方法，并将`createUserQuery`和值作为参数发送进来。
*   我们利用尝试和捕捉，我💕它。
*   使用 generateUserToken helper 方法，我们生成了我们的用户令牌。
    ***JWT*** 将在本文稍后讨论。
*   `***_bt_check_unique***` —检查重复值并返回错误。

因为`dbQuery.query`返回一个承诺，我们利用`async/await`让我们的代码看起来性感`😘.`

*   `***signUser***` -我们设置`SELECT * FROM users WHERE email = $1`来获取一个有特定电子邮件 ***的用户。***
*   类似于 ***createUser*** 函数，它检查电子邮件和密码是否不正确，它还比较密码，确保用户输入的密码和电子邮件都正确无误，然后对它们进行签名。
*   此外，用户在登录时会获得一个 ***令牌*** 。这是由我们的一代人创造的。
*   最后，我们分别导出了我们的方法`createUser and signUser`，因此我们可以在我们的项目中使用它们。

## ***总线控制器***

复制以下内容并粘贴到`***app/controllers/busController.js***`中

代码步骤:

*   在这里，我们创建了两个方法- `addBusDetails()`和`getAllBuses()`。我们还利用了`async/await`。
*   我们导入 moment，这样做的原因是因为我们要用它来创建我们的约会。
*   `addbusDetails()`——我们从`app/db/dev/dbQuery.js`导入了我们的 dbQuery。
*   我们还从`app/helpers/validations.js`中导入了我们的助手方法`empty`和`errorMessage,successMessage, status`。和`app/helpers/status.js`分别表示。
*   我们创建了一个函数`addBusDetails()`，在其中我们从请求体中解构了我们的值`number_plate, manufacturer, model, year, capacity`。
*   接下来，我们创建了一个名为 created_on 的变量，它保存我们创建的用户表的日期。
*   我们还使用我们的`***empty***`方法检查输入字段是否为空，如果为空，它将发送一个错误 ***“所有字段都是必需的”*** ，并返回一个状态代码 400。
*   对于我们的 SQL 查询，我们使用了`INSERT INTO bus(list_columns_here..) VALUES($1, $2, $3, $4, $5, $6 ...)`——这是在`bus`表中创建一个新行，并将提供的值插入到它的字段中。`values`是一个数值数组，包含我们要插入到表格中的内容。`values`数组中的元素必须与`$1, $2, $3, $4, $5, $6`的顺序相同。我们使用`returning *`返回创建的行。记得我们创建了接受两个参数`quertText`和`app/db/dev/dbQuery.js`中的`params`的`query`方法，这是我们将使用它的地方。我们调用该方法，并将`createBusQuery`和值作为参数发送进来。由于`dbQuery.query`返回一个承诺，我们利用`async/await`让我们的代码看起来更性感`😘.`

## ***行程控制器***

接下来，我们将创建我们的 tripController，类似于总线控制器。

复制以下内容并粘贴到`***app/controllers/tripController.js***`中

代码步骤:

tripController 与之前创建的 BusController 非常相似，但是增加了一些东西

*   在这里，我们创建了两个方法— `createTrip(), getAllTrips()cancelTrips, filterTripByOrigin, filterTripByDestination`。
*   ***取消行程*** —我们设置`UPDATE trip SET status=$1 WHERE id=$2 returning*`根据行程 ***ID 取消行程。***
*   ***filterTripByOrigin***—我们设置`SELECT * FROM trip WHERE ***origin***=$1 ORDER BY id DESC` 从降序 ***中过滤用户旅行的起点。***
*   ***filterTripByDestination***—我们设置了`SELECT * FROM trip WHERE destination=$1 ORDER BY id DESC` 从降序 ***中过滤用户旅行的目的地。***

## ***预订控制器***

接下来，我们将考虑 bookingController，它似乎是我们的 ***transportApi 的重要组成部分。***

复制以下内容并粘贴到`***app/controllers/bookingController.js***`中

代码步骤:

bookingController 与之前创建的 BusController 非常相似，但是增加了一些东西

*   在这里，我们创建了四个方法— `createBooking(), getAllBooking, deleteBooking, and updateBookingSeat`。
    前两种方法***create booking***和***getAllBooking***与我们 ***busController 中前面的解释相同。***
*   ***删除预订*** —我们设置`UPDATE trip SET status=$1 WHERE id=$2 returning*`根据行程 ***ID 取消行程。***
*   ***更新预订座位*** —我们设置`UPDATE booking SET seat_number=$1 WHERE user_id=$2 returning*`在预订过程中根据用户 ID 和用户座位号更新预订。

如果你已经走到这一步，你应该得到一个大拇指！

# **设置路线**

接下来，我们将考虑为创建的每个控制器设置路径。

## ***公交路线***

复制以下内容并粘贴到`***app/routes/busRoute.js.***`

代码步骤:

*   我们进口快递。
*   接下来，我们将在我们的总线控制器中创建的所有 ***方法*** 导入到`***app/controllers/busController.js.***`中
*   我们声明了一个变量( ***router*** )，它保存了我们的快速路由器。
*   我们使用我们的路由器变量和传递请求方法( ***POST，PUT，PATCH，GET*** )定义了我们的路由和匹配路由，例如
    `***router.post('/buses', addBusDetails), router.get('/buses', getAllBuses)etc***`。

## **注:**

*   虽然路线相同但请求方法不同，所以处理不同的请求。
*   相同的模式适用于本教程中的所有路线
*   **对任何端点的每个请求都将通过** `**verifyAuth**` **中间件处理实际请求。使用这种新的设置，没有有效令牌的用户将会得到一个错误。**

## ***预定路线***

## ***行程路线***

# ***设置我们的服务器***

现在我们已经有了控制器和路线设置，让我们看看如何把一切放在一起。

复制以下代码并粘贴到`***app/server.js.***`

代码步骤:

**。**我们安装了 babel-polyfill npm 包并导入了它——我们在这里需要它，以便节点运行时能够识别`async/await`和`Promise`。

***。*** 我们从各自的模块中导入必要的包

***。*** 接下来，我们从 routes 文件夹中导入所有路线

***。*** 添加 **cors** 中间件用于*解析 URL 编码体(通常由浏览器发送)*

**T58。** 表达 ***。*** *添加 urlencoded 作为中间件，用于解析 JSON 和 urlencoded 数据，并填充* **`req.body`.**

***。*** 接下来，我们对我们的 **Api** `***'/api/v1', userRoute***`进行了版本化，并追加了各自的路由。

**T1。** 最后我们监听了我们的端口，这样我们就可以启动服务器了。

# 邮递员测试

![](img/b5f9d300803d46e854d2c24dfb234720.png)

## 创建用户-发布 API/v1/注册

![](img/2bcf241a20897731a43159774cfe783c.png)

## 创建管理员-发布 API/v1/管理员/注册

![](img/9a6e810bce5135988c7fa8e6b4f9f567.png)

复制令牌并将其放入标头中

![](img/c1f368426729b64798e972927a09d0a0.png)

## 创建总线—发布 API/v1/总线

![](img/9d8692d37ed32b38c1862c89e037ec45.png)

## 获取总线—获取 API/v1/bus/

![](img/a33634f0caba65c111855660cb221cdd.png)

## 创建行程-过帐 API/v1/行程

![](img/8856aa88faef055ba59650950fccc51f.png)

## 获取行程—获取 API/v1/行程

![](img/2b702d45bfbb80c9658e3bd7d4d442b3.png)

## 取消行程—修补 api/v1/trips/:tripId

![](img/3be715913f0db58b118ee663780e6e9e.png)

## 创建预订—发布 API/v1/预订

![](img/2534cca34684686b28b633552a510065.png)

## 获取预订—获取 API/v1/预订

![](img/f96ad10a742d6414324cb9ba6f0b903e.png)

# 结论

我们刚刚成功构建了我们的运输后端应用程序。如果你走了这么远，恭喜你，✌.
当然，在学习本教程的过程中，如果你有任何疑问或问题，请留下你的评论，我们会尽快回复你。
再次感谢阅读、鼓掌和分享！👌

在 [Twitter 上关注我](https://twitter.com/beveloper)我们来讨论一下。

点击查看完整代码

# 资源

1.  [作为开发人员，你为什么应该使用 CMS](https://agilitycms.com/resources/posts/why-you-as-a-developer-should-be-using-a-cms)
2.  [无头商务解释:定义、用例、障碍|敏捷 CMS](https://agilitycms.com/resources/posts/headless-commerce-explained-definitions-use-cases-and-roadblocks)
3.  [API vs . SDK:有什么区别？|敏捷 CMS](https://agilitycms.com/resources/posts/apis-vs-sdks-what-s-the-difference)
4.  [2021 年最值得关注的静态站点发电机](https://agilitycms.com/resources/posts/static-site-generators)
5.  [https://agility CMS . com/resources/posts/what-s-the-difference-between-headless-CMS-and-static-site-generator/](https://agilitycms.com/resources/posts/what-s-the-difference-between-headless-cms-and-static-site-generator/)
6.  [什么是网络生活，作为一名编辑，你为什么要关心它？|敏捷 CMS](https://agilitycms.com/resources/posts/what-is-netlify-and-why-should-you-care-as-an-editor)
7.  [什么是无头 CMS？|敏捷 CMS](https://agilitycms.com/resources/guide/what-is-a-headless-cms)
8.  [无头 CMS 相对于传统 CMS 的优势|敏捷 CMS](https://agilitycms.com/resources/guide/top-10-reasons-why-you-should-choose-a-headless-cms-over-a-traditional-cms)
9.  [内容架构:组织和管理内容和团队的关键](https://agilitycms.com/resources/posts/content-architecture)
10.  [WordPress 及其替代品:无头 CMS |敏捷性 CMS](https://agilitycms.com/resources/posts/wordpress-vs-agility-cms)
11.  [开源 CMS 与专有 CMS:您能两全其美吗？敏捷性| CMS](https://agilitycms.com/resources/whitepaper/proprietary-vs-open-source-content-management-systems)
12.  [选择反应型细胞质雄性不育系:寻找什么？敏捷性| CMS](https://agilitycms.com/resources/posts/react-cms)
13.  [掌控您的内容架构:内容建模](https://agilitycms.com/resources/posts/content-modelling)
14.  [DXP vs 无头 CMS:现代 DXP 建筑](https://agilitycms.com/resources/posts/the-benefits-of-a-hybrid-dxp)
15.  Jamstack 开发:[2021 年您需要了解的 10 位 JAMstack 先驱](https://agilitycms.com/resources/posts/top-jamstack-pioneers)
# 在 Node.js/Electron 应用中保存数据的 5 种以上方法

> 原文：<https://itnext.io/5-ways-to-persist-data-in-node-js-electron-apps-87e2aa7251f5?source=collection_archive---------0----------------------->

![](img/eb1afbb6967cfd77c896c2828c155c4f.png)

在本教程中，我们将探索在使用电子平台和 web 技术构建的跨平台桌面应用程序中保存和检索数据的不同方法，我们还将介绍如何安装所需的工具，编写一些示例代码，然后了解每种方法的优缺点。但是首先，让我们从电子入门开始，对于那些还不熟悉它的人。

# 电子导论

早在 2013 年，Github 就发布了 Electron，允许 web 开发人员使用他们现有的 JavaScript、CSS 和 HTML 前端技能，通过单一代码库为 Windows、MAC 和 Linux 构建类似原生的跨平台桌面应用程序。

Electron 基于谷歌 Chromium 项目和 Node.js 平台，这意味着你拥有一个最新网络标准的嵌入式浏览器(多亏了 Chromium)和整个 Node.js 模块，可从 NPM 获得，供你使用。

由于 electronic 基本上是一个 web 浏览器，你可以利用最现代的客户端框架和库，比如 Angular 和 React，来构建不仅适用于 web，也适用于桌面的应用程序。

如果你对电子有任何疑问，如果它真的改变了我们开发桌面应用程序的方式，忽略大公司甚至微软都在使用它的事实(你不应该)，你可以简单地从这个[链接](https://npm-stat.com/charts.html?package=electron)查看与电子使用相关的 NPM 统计数据。还有正在进行的通过这个[链接](https://electron.atom.io/apps/)使用电子创建的应用程序列表

# 在电子应用程序中保存数据

首先，你为什么要这样做，也就是持久化数据？
在大多数情况下，应用程序需要持久化数据，原因有很多，例如:

*   保存用户定义的设置:存在许多应用范围的设置或参数，这些设置或参数因用户而异，因此，它们需要由最终用户来设置，并通过某种机制来保存，以避免在应用重启之间丢失它们。
*   保存特定领域的数据:在企业应用程序等数据密集型应用程序中，数据是核心元素，因此持久化和检索是一个至关重要的特性。这种应用程序需要数据存储来跟踪信息或进行进一步的处理和分析。
*   添加离线支持:如果您的应用程序依赖于在线数据，并且您可能希望添加离线支持，那么您还需要集成一些机制来本地持久化数据。
*   实现缓存:在某些情况下，您可能需要为您的应用程序添加某种类型的网络数据磁盘缓存，例如，为了优化应用程序加载时间，使用 SQLite 等嵌入式数据库。

在开发应用程序时，考虑如何持久化数据是关键的一步，桌面应用程序也不例外。您在应用程序中处理数据的方式会影响应用程序的性能，从而影响最终的用户体验。

有太多可用的机制来持久化数据，从平面文件、键值存储到 SQLite 和成熟的数据库系统，如 MySQL、PostgreSQL 和 Microsoft SQL Server 等。

您可以使用简单的经典存储介质，如平面文件、嵌入式数据库、关系数据库系统或 NoSQL 数据库。这完全由您决定，但您需要考虑您的应用程序要求，以及每种方法的利弊，以做出正确的决定。

# 如何选择在电子应用程序中保存数据的最佳方式？

几乎所有类型的应用都需要某种数据存储和检索，但并不是每种方法都适合所有用例，例如，如果您只需要存储一些配置数据或用户定义的设置，那么平面文件或键值存储(如 HTML5 localStorage 或 HTML5 NoSQL IndexedDB)对于这种任务来说可能是非常有用和简单的机制。

localStorage API 非常容易使用。对于 IndexedDB 来说，API 有点复杂，但是您可以将它与包装库一起使用，比如 Dexie.js，它有一个简单明了的 API。

如果您需要使用简单的结构(即不同数据实体之间的关系很少，属性很少)来持久存储特定于领域的数据，那么 SQLite 或类似的嵌入式数据库解决方案是这些用例的正确选择。

如果您正在构建一个复杂的数据库应用程序或数据密集型应用程序，其中包含多个表和关系，那么使用原始 SQL 可能会令人生畏并且容易出错。因此，ORM 将对您有很大的帮助，它允许您轻松地表达您的数据库需求，并让您专注于应用程序的业务领域方面。有许多 JavaScript/TypeScript 表单可以很好地与电子生态系统集成，如 Sequelize、Bookshelf.js 和相对较新的 TypeORM(基于 TypeScript)。

有两种主要类型的数据库:关系数据库和 NoSQL 数据库。要使用的正确数据库类型在很大程度上取决于这两点:数据库的性质和结构，以及同步要求。但是使用一种数据库类型优于另一种数据库类型的标准是什么呢？为了回答这个问题，让我们想象这样一个场景:你正在为一个需要跟踪制造商、客户、产品、订单和发票等数据的企业构建一个应用程序。这些数据以不同的关系相互关联。例如，一个订单属于一个客户并有许多相关产品，一张发票与某个订单相关，等等。除了存储数据，您还需要能够执行涉及一个实体的简单查询或连接多个实体的复杂查询。

根据这些要求，我们可以得出一些要点来帮助我们决定需要使用什么样的数据库系统，例如:

*   数据实体可以存储在包含列和行的表格结构中。
*   在开发应用程序时，可以提前预测数据实体列(模式)。
*   要求数据完整性。

对于这个用例，关系数据库是一个理想的选择。

如果模式是不可预测的，不断发展，需要随着时间的推移而改变，数据是不相关的，性能是主要关注点，那么 NoSQL 数据库是什么使用。

你也可以阅读[这篇文章，了解关于 SQL 和 NoSQL 比较](https://www.sitepoint.com/sql-vs-nosql-differences/)的更多细节。

除此之外，还有许多因素你也需要考虑，例如:

*   性能，
*   数据可移植性，
*   在基于内存和磁盘 IO 的数据库中，
*   同步，
*   先离线与否，
*   不管有没有服务器，
*   不管有没有嵌入，
*   使用许可证，

因此，为持久化数据选择正确的方法取决于您的应用程序需求。在这篇文章中，我们将看到一些可能的，简单的和高级的，在你的电子应用中持久化数据的机制。

如果你想测试代码，当你继续阅读教程时，你需要创建一个新的电子项目，你可以简单地用 GitHub 快速启动项目来做，所以使用你的终端或命令提示符开始克隆它们的库:

```
git clone https://github.com/electron/electron-quick-start cd electron-quick-start
```

然后安装需求并启动应用程序:

你也可以遵循这个[教程](https://www.sitepoint.com/desktop-node-apps-with-electron/)，它会带你安装电子设备，构建和打包一个简单的桌面应用程序。

# 用普通的 SQL 和 MySQL 数据库持久化数据

MySQL 是由 Michael Widenius 创建的开源关系数据库系统。为了能够在您的系统中使用 MySQL，您需要首先安装它，这取决于您的目标操作系统，但它通常是一个容易遵循的过程。因此，在使用本教程中的 MySQL 或测试 MySQL 演示之前，请确保为您的系统安装 MySQL 版本。

MySQL 可用于大多数主流操作系统，包括 Linux、Mac 和 Windows。

# 为 Node.js 安装 MySQL 客户端

首先，我们需要从 NPM 安装 Node.js 的 mysql 模块，继续运行:

# 引导模块

下面是我们使用 node-mysql 模块对 MySQL 数据库执行 SQL 查询需要遵循的步骤:

*   导入或需要 mysql 模块
*   创建连接对象
*   连接到数据库
*   查询数据库
*   关闭连接

我们首先需要 mysql 模块:

```
var mysql = require('mysql');
```

然后创建一个数据库连接，您需要输入 MySQL 数据库系统凭证和您的数据库名称。

```
var connection = mysql.createConnection({ host : 'localhost', user : 'root', password : 'YOUR_PASSWORD', database : 'demodb' });
```

创建连接对象后，我们使用 connect()方法实际连接到数据库

此时，我们已经准备好对数据库执行 SQL 查询。让我们举一个简单的例子

```
connection.query('SELECT 1 + 1 AS result', function (error, results, fields) { if (error) throw error; console.log('1 + 1 = ', results[0].result); });
```

如果您使用完一个连接，您需要使用 end()方法关闭它

正如你所看到的，API 是清晰的，不言自明的，很容易连接到 MySQL 数据库，然后执行 SQL 查询。要获得一个简单完整的使用 MySQL 的电子演示，一定要查看这个 Github 库。

# 使用 MySQL +普通 SQL 的利弊

在 Electron apps 中使用带 MySQL 的普通 SQL 有一些优点和缺点，所以让我们从优点开始:* MySQL 不限制 electronic 跨平台特性，因为它可用于 electronic 支持的所有操作系统(Windows、Linux 和 MAC)。* MySQL 可以很容易地与 Electron 安装和集成(见上面的代码)。* MySQL Node.js client 是一个本机模块，因此无需重新构建即可用于所有面向电子的架构。

现在让我们看看缺点:

*   MySQL 不与电子应用程序捆绑在一起，因此最终用户需要从您的应用程序中单独安装 MySQL(但是您可以使用一些方法在您的应用程序中自动安装 MySQL)。
*   MySQL DBMS 对于小型电子项目来说是大材小用。

# 用顺序化 ORM (+MySQL)持久化数据

ORM 是使用 SQL 数据库的智能软件解决方案或抽象，即使您对 SQL(关系数据库的本地编程语言)一无所知，它们对于创建和使用具有复杂结构和关系的数据库尤其有用，因此尽管您很欣赏它们，但您并不需要在每个用例中都使用它们。编写原始 SQL 可能比使用 ORM 更好，特别是在性能方面，并且不了解 SQL 不能成为性能下降的借口。

# 什么是顺序化？

Sequelize 是一个对象关系映射器，它使 JavaScript (Node.js)开发人员能够在不实际编写 SQL 的情况下使用 SQL 数据库，而是使用 JavaScript 对象。ORM 消除了为执行 CRUD(创建、读取、更新和删除)操作而编写原始 SQL 的需要，并允许您使用一种编程语言编写应用程序代码，还可以访问和使用数据库。

您可以将 Sequelize 与许多数据库管理系统和引擎一起使用，例如:PostgreSQL、MySQL、SQLite 和 MSSQL。

# 安装序列

安装 Sequelize ORM 也是使用 npm install 命令的问题，因此打开您的终端或命令提示符，然后运行:

```
npm install --save sequelize
```

对于我们的用例，我们将使用 Sequelize 和 MySQL DBMS，因此您还需要使用以下命令安装 MySQL client for Node.js:

```
npm install --save mysql2
```

# 使用序列

安装 Sequelize 后，您可以通过首先创建一个到所选数据库的连接来开始使用它，例如:

```
const Sequelize = require("sequelize"); const sequelize = new Sequelize('mysql://YOUR_USERNAME:[[email protected]](https://www.techiediaries.com/cdn-cgi/l/email-protection):5432/YOUR_DATABASE_NAME');
```

你可以通过这个[链接](http://docs.sequelizejs.com/class/lib/sequelize.js%7ESequelize.html)传递许多选项给 Sequelize 构造函数，检查它们。

现在您已经准备好连接到数据库，您可以简单地使用 authenticate()方法:

```
sequelize .authenticate() .then(() => { 
console.log('Connection successfully made.'); }) .catch(err => { console.error('Error connecting to database', err); });
```

接下来，我们需要使用 sequelize.define('name '，{attributes}，{options})创建一个模型:

```
const Product = sequelize.define('product', { name: { type: Sequelize.STRING }, quantity: { type: Sequelize.FLOAT }, cost: { type: Sequelize.FLOAT }, price: { type: Sequelize.FLOAT }, unit: { type: Sequelize.STRING } });
```

关于在顺序表单[中创建模型的更多信息，请查看此](http://docs.sequelizejs.com/manual/tutorial/models-definition.html)。

接下来，我们可以创建 SQL 表并保存一些数据:

```
Product.sync().then(() => { return Product.create({ name: 'Product 001', quantity: 10, cost: 100, price: 120, unit: 'kg' }); });
```

现在让我们进行一个查询，检索数据库中的所有产品:

```
Product.findAll().then(products => { console.log(products) })
```

你可以通过这个[链接](http://docs.sequelizejs.com/manual/tutorial/querying.html)找到更多关于使用 Sequelize 查询数据库的信息。

您可以从这个链接在 GitHub 上找到这种方法的工作演示。

# MySQL 和 Sequelize ORM 方法的优缺点

就像前面的方法一样，带有 Sequelize ORM 的 MySQL 也有一些优点和缺点。对于专业人士:

*   Sequelize 支持的 DBMS 和引擎可用于所有电子平台(Windows、Linux 和 MAC)。
*   电子应用程序可以很容易地实现，以支持多数据库管理系统(通过应用程序设置)。
*   更适合大型电子项目。
*   红杉很容易与电子结合。
*   js 模块不是本机的，所以它不需要任何重建目标电子 ABI(应用程序二进制接口)。
*   用户可以通过 SQLite 使用你的电子应用程序，而无需安装任何数据库管理系统。

对反对者来说:

*   用户需要安装底层 DBMS(SQLite 除外)。
*   ORMs 在你的应用程序和数据库之间增加了另一层，因此产生的电子应用程序可能会有性能降级。

我们在示例中使用了 MySQL，但是您也可以使用任何 Sequelize 支持的 DBMS/引擎，例如 PosgreSQL、MSSQL、Oracle 甚至 SQLite，只需安装相应的 Node.js 客户端，并在实例化 Sequelize 时指定 DBMS(参见上面的代码)。

更好的是，您可以通过应用程序中的配置选项，轻松地为应用程序用户提供设置 DBMS/引擎的选择。

# 其他类似的形式

你也可以使用其他流行的电子表格。为了简洁起见，我们将在没有进一步细节的情况下提及它们:

*   [node-orm2](https://github.com/dresende/node-orm2) :类似于 Sequelize ORM 的 Node.js ORM，可以和 MySQL/MariaDB、PostgreSQL、MSSQL、Oracle、SQLite、MongoDB、Amazon Redshift 一起使用。
*   [Bookshelf.js](http://bookshelfjs.org/) :基于 [Knex query builder](http://knexjs.org/) 的另一个 Node.js ORM，可以和 PostgreSQL、MySQL、MariaDB、SQLite3 一起使用。
*   [TypeORM](https://github.com/typeorm/typeorm) :类似于 Java Hibernate 或者 PHP 主义的类型脚本 ORM。它也可以与电子集成。

# 用 SQLite 持久化数据

SQLite 是一个嵌入式(无服务器)单文件 SQL 数据库引擎。由于 SQLite 代码是公开的，所以你可以免费使用它，无论是在免费项目还是商业项目中，没有任何限制。SQLite 数据库表、索引、触发器和视图包含在一个磁盘文件中。

SQLite 是世界上使用最多的数据库引擎，有很多好的理由，所以让我们看看如何在 electronic 中使用它。

另外，你也可以把 SQLite 和 SQL ORM 一起使用，比如 Sequelize 或者 TypeORM。这是通过将数据库引擎指定为 sqlite 来更改一个参数的问题。

# 安装和重建 SQLite3

SQLite3 是一个本地 Node.js 模块，因此如果不将其重新构建为目标电子，就不能直接用于电子。

有许多方法可以做到这一点。你可以从[电子文档](https://electron.atom.io/docs/tutorial/using-native-node-modules/)中找到它们。

首次安装电子-重建:

```
npm install --save-dev electron-rebuild
```

然后安装 sqlite3 模块:

```
npm install sqlite3 --save
```

接下来，使用以下内容为电子重建 sqlite3:

```
./node_modules/.bin/electron-rebuild -f -w sqlite3
```

如果过程顺利，你就可以开始了。

# 使用 SQLite3

现在让我们看一个如何使用 sqlite3 的小例子，首先我们需要导入 SQLite3 模块:

```
const sqlite3 = require('sqlite3');
```

然后创建一个数据库对象来连接内存中的数据库:

```
var db = new sqlite3.Database(':memory:');
```

您也可以通过指定文件路径来使用基于文件的数据库:

```
var db = new sqlite3.Database('/path/to/database/file.db');
```

接下来，让我们创建一个表，插入一些值，查询它们，然后在控制台上打印结果:

```
db.serialize(function () { db.run("CREATE TABLE Products (name, barcode, quantity)"); db.run("INSERT INTO Products VALUES (?, ?, ?)", ['product001', 'xxxxx', 20]); db.run("INSERT INTO Products VALUES (?, ?, ?)", ['product002', 'xxxxx', 40]); db.run("INSERT INTO Products VALUES (?, ?, ?)", ['product003', 'xxxxx', 60]); db.each("SELECT * FROM Products", function (err, row) { console.log(row); }); });
```

最后，在完成我们的数据库后，我们可以用以下命令关闭它:

# 电子版 SQLite 的利与弊

就像前面的方法一样，将 SQLite 与 Electron 结合使用有一些优点和缺点。让我们从优点开始:

*   SQLite 可以与您的电子应用程序捆绑在一起，因此最终用户不需要在您的应用程序旁边安装任何东西。
*   SQLite 不会妨碍 electronic 的跨平台特性，因为它适用于 electronic 的所有目标平台(Windows、Linux 和 MAC)。
*   适合中小型项目。
*   除了重新构建 gotchas 之外，SQLite 可以很容易地与 electronic 集成和使用。

现在让我们看看缺点:

*   SQLite 是一个原生模块，所以需要重新编译才能与 electronic 一起使用。
*   为了针对不同的架构，您需要为每个可能的架构重新构建 SQLite。

# 用 MongoDB 持久化数据

MongoDB 是一个免费的开源 NoSQL 数据库系统，可用于 Windows、Linux、OS X、Solaris 和 FreeBSD。

# 安装 MongoDB

MongoDB 与 Electron 是分开的，但是要使用它，我们需要集成一个 MongoDB 客户端，它可以用来连接到 MongoDB 数据库并与之通信。Mongoose 是 MongoDB 的 Node.js 客户端，因此在使用与数据库通信之前，您需要先安装它。

使用终端或命令提示符运行以下命令:

```
npm install mongoose --save
```

接下来你需要确保为你的目标电子 ABI 重建猫鼬。

```
npm install electron-rebuild --save ./node_modules/.bin/electron-rebuild -f -w mongoose
```

# 使用 MongoDB

安装 Mongoose 客户端后，您首先需要要求它:

```
var mongoose = require('mongoose') ,
Schema = mongoose.Schema ,
ObjectId = Schema.ObjectId;
```

接下来使用以下命令连接到您的数据库:

```
mongoose.connect('mongodb://localhost/my_database',{useMongoClient:true});
```

其中 my_database 是您的数据库的名称。

然后创建您的 Mongoose 模式:

```
var productSchema = new Schema({ location: ObjectId, date: {type: Date, default: Date.now}, family: {name: String}, name: String, quantity: Number });var locationSchema = new Schema({ name: String });
```

接下来，从模式中创建实际的模型:

```
var Product = mongoose.model('Product', productSchema); var Location = mongoose.model('Location', locationSchema);
```

现在，您可以使用不同的 API 对数据库进行 CRUD 操作。您可以通过创建这些对象的实例并调用它们的 save()方法来添加数据库行:

让我们创建一些位置:

```
var loc001 = new Location({name:'Location001'}).save(); var loc002 = new Location({name:'Location002'}).save(); var loc003 = new Location({name:'Location003'}).save();
```

让我们创造一些产品:

```
var product001 = new Product({location:loc001._id, family:{name:"family001"},name:"Product001",quantity:10}).save(); var product002 = new Product({location:loc001._id, family:{name:"family002"},name:"Product002",quantity:20}).save();var product003 = new Product({location:loc002._id, family:{name:"family001"},name:"Product002",quantity:30}).save();
```

我们还可以使用 find()方法查询位置和产品:

```
Location.find(function(err, locations) { console.log(locations); }); Product.find({location: loc001._id}, function(error, products) { console.log(products); });
```

您可以从这个[链接](http://mongoosejs.com/index.html)中找到更多关于可用 mongoose APIs 的文档。

# 使用 MongoDB 的利与弊

对于使用 MongoDB 和电子应用程序的优点:

*   适用于所有电子支持的平台，如 Windows、Linux 和 MAC。所以它不限制电子的跨平台特性。
*   可轻松安装并与电子集成。

也有一些缺点:

*   不能与 Electron 捆绑在一起，所以最终用户需要将它与您的应用程序分开安装。
*   对于小应用程序来说是过度的。

# 用 NeDB 保存数据

NeDB 代表 Node.js 嵌入式数据库，它是一个纯 JavaScript 模块，没有二进制依赖，其 API 是 MongoDB 的子集。NeDB 可以用作内存或持久数据库。

# 安装 NeDB

您可以通过 NPM 安装 NeDB，包括:

您也可以使用 bower 软件包管理器安装为浏览器设计的版本

# 使用 NeDB

在使用 NeDB API 之前，您需要 NeDB 模块使用:

```
var Datastore = require('nedb');
```

接下来，您可以使用以下命令创建内存数据库:

```
var db = new Datastore();
```

或通过指定数据文件(将被创建)的路径的持久数据存储:

```
var db = new Datastore({ filename: 'path/to/datafile' });
```

在对我们创建的数据存储区应用任何操作之前，您需要使用以下命令将其加载到内存中:

```
db.loadDatabase(function (err) { // You can now execute your commands });
```

还可以在创建数据存储时将 autoload 参数设置为 true，以便在不显式调用 loadDatabase()方法的情况下将其自动加载到内存中。

```
db = new Datastore({ filename: 'path/to/datafile', autoload: true });
```

您可以使用 insert 方法插入新文档:

```
var doc = { name: 'product001', quantity : 100 }; db.insert(doc, function (err, newDoc) { // newDoc is the newly inserted document });
```

要获取插入的文档，请使用 find()方法:

```
db.find({ name: 'product001' }, function (err, docs) { // results });
```

# 使用 NeDB 和电子的利与弊

就像其他方式一样，将 NeDB 与 Electron 结合使用有一些优点和缺点，所以让我们从优点开始:

*   NeDB 可以和电子捆绑。
*   由于它是纯 JavaScript，所以可以与 Electron 一起使用，无需重新构建。
*   NeDB 可以很容易地与电子集成和使用。
*   NeDB 适合中小型项目。

对于缺点，没有太多关于使用 NeDB 与电子，除了:

# 结论

在 Electron 中有很多数据持久化的方法，选择正确的方法主要取决于您的用例。如果只是保存应用程序设置，那么你可以使用简单的机制，如平面文件或 HTML5 存储 API，对于高级数据需求，你应该选择大型数据库解决方案，如 MySQL 或 MongoDB(有或没有 ORM)

[在 Twitter 上分享](http://twitter.com/intent/tweet?text=5+%20Ways%20to%20Persit%20Data%20in%20Node.js/Electron%20Apps&url=https://www.techiediaries.com/electron-data-persistence/&via=) [在脸书分享](http://www.facebook.com/sharer/sharer.php?u=https://www.techiediaries.com/electron-data-persistence/) [在 Google+上分享](http://plus.google.com/share?url=https://www.techiediaries.com/electron-data-persistence/)

*原载于*[*www.techiediaries.com*](https://www.techiediaries.com/electron-data-persistence/)*。*
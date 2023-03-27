# NodeJS 应用程序中的自索引功能

> 原文：<https://itnext.io/self-indexing-functions-in-nodejs-applications-1c1ca88f599c?source=collection_archive---------4----------------------->

![](img/7ddbf476e22e450fc5ccf8e10a479331.png)

图片来源于作者的照片。

当编写`NodeJS`应用程序时，经常会希望能够将合适的文件添加到文件夹中，并让根级 `index.js`文件*自动*找到新文件，并因此公开一个新特性。手动这样做意味着你需要确保`index.js` `requires`文件，然后`exports`它。这是一个很容易忘记的步骤，如果忘记了，调试起来会很麻烦。

为了简单灵活，我编写了一个名为`[traverse-folders](https://www.npmjs.com/package/traverse-folders)`的`NodeJS`实用程序。注意，这在客户端不起作用，因为它依赖于对文件系统的访问。

# 例子

你正在编写一个 API 服务器，你的路由控制器都在`src/api`中。在`src/api/index.js`中，您可以这样做:

```
const path = require('path')
const traverse = require('traverse-folders')const pathSeparator = new RegExp(path.sep, 'g')const apis = {}
const base = __dirnameconst processor = file => {
  const name = file
    .slice(base.length + 1, -3)
    .replace(pathSeparator, '_')
  apis[name] = require(file)
}traverse(base, processor)module.exports = apis
```

所以如果你的`src/api`文件夹看起来像这样:

```
src/
  api/
    index.js
    ping.js
    version.js
    /things
      /createThing.js
      /deleteThings.js
      /getThing.js
      /listThings.js
      /updateThings.js
```

结果将是`src/index.js`现在将导出

```
{
  ping,
  version,
  things_createThing,
  things_deleteThing,
  things_getThing,
  things_listThings,
  things_updateThing,
}
```

每个命名函数都将正确地与控制器代码相关联。

# 让模拟 API 与真实 API 保持同步

通常路由控制器会调用某种数据库连接，或者做一些使单元测试无用的事情。与集成测试不同，单元测试决不能依赖于外部服务，相反，为了保持隔离和快速运行，需要模拟它们。

为了解决这个问题，需要`src/api`的单元测试需要换入一个`mockAPI`对象。

为了构建一个`mockAPI`并确保它与真正的 API 保持同步，创建一个文件`test/utils/mockAPI.js`，如下所示:

```
const path = require('path')
const { stub } = require('sinon')
const traverse = require('traverse-folders')const pathSeparator = new RegExp(path.sep, 'g')const mockApi = {}
const apiPath = 'src/api'
const processor = file => {
  const name = file
    .slice(apiPath.length + 1, -3)
    .replace(pathSeparator, '_')
 **names[name] = stub()** }traverse(apiPath, processor)module.exports = mockApi
```

正如你所看到的，它与`src/api/index.js`几乎相同(我用粗体显示了不同的一行)，但是`processor`函数不需要底层控制器，它只是分配了一个`stub`。

当然，你可以通过将`const name = file.slice(apiPath.length + 1, -3).replace(pathSeparator, ‘_’)`部分外部化到一个单独的`makeName`函数中，并将`pathSeparator`定义为一个外部常量，因为它们在两种情况下是相同的。

# 加载模型

`[Sequelize](https://sequelizejs.com)`的用户通常使用这种技术来加载他们的`model`文件。因此，如果你有一个文件夹层次结构的模型，比如:

```
src/
  models/
    index.js
    someSharedLogic.js
    User.model.js
    Company.model.js
```

您的`src/models/index.js`文件应该是这样的:

```
const Sequelize = require('sequelize')
const traverse = require('traverse-folders')const {
  dbName,
  dbUser,
  dbPass,
  options
} = require('../utils/db/dbConfig')const sequelize = new Sequelize(dbName, dbUser, dbPass, options)const db = { sequelize, Sequelize }const processor = file => {
  const model = sequelize.import(file)
  db[model.name] = model
}const associate = modelName => {
  if (typeof db[modelName].associate === 'function')
    db[modelName].associate(db)
}traverse(__dirname, processor, { suffix: '.model.js' })Object.keys(db).forEach(associate)module.exports = db
```

瞧，任何对`require('src/models')`的引用都将返回您定义和关联的模型，并且不会被`someSharedLogic`文件弄糊涂。

# 选择

你可以定制`suffix`和`ignore`文件，让遍历图像文件夹或任何你喜欢的东西变得非常简单。默认选项包括:

```
{
  ignore: 'index.js',
  suffix: '.js'
}
```

# 链接

*   `[Sequelize](https://sequelizejs.com)`
*   `[traverse-folders](https://www.npmjs.com/package/traverse-folders)`在 NPM
*   `[traverse-folders](https://github.com/davesag/traverse-folders)`GitHub 中的源代码

## 请参见

*   [用 Express 和 Swagger 连接 API 服务器](/wiring-up-an-api-server-with-express-and-swagger-9bffe0a0d6bd)
*   [单元测试使序列模型变得容易](/unit-testing-sequelize-models-made-easy-108f079f1e38)
*   [使用](/using-async-routes-with-express-bcde8ead1de8) `[async](/using-async-routes-with-express-bcde8ead1de8)` [路线带快车](/using-async-routes-with-express-bcde8ead1de8)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。
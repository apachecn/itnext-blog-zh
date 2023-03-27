# 单元测试变得简单

> 原文：<https://itnext.io/unit-testing-sequelize-models-made-easy-108f079f1e38?source=collection_archive---------3----------------------->

![](img/4d56c60d547dad9661dac3677403d710.png)

图片改编自 Jaro Larnos 的[微型城市和铁路，并在](https://www.flickr.com/photos/jlarnos/40950601550)[许可](https://creativecommons.org/licenses/by/2.0/)条款下使用。

对于使用关系数据库的`NodeJS`开发人员来说，`[Sequelize](http://docs.sequelizejs.com)`是首选的对象关系映射工具。然而，对`Sequelize`模型进行有效的单元测试，不需要调用底层数据库的连接，会带来一些挑战。

为了帮助解决这个问题，我写了一个名为`[sequelize-test-helpers](https://www.npmjs.com/package/sequelize-test-helpers)`的小工具套件。

```
npm i -D sequelize-test-helpers
```

![](img/f379f81c1709f6192a1760be805c8c81.png)

# 定义和导入模型

定义和导入序列模型遵循一个非常可预测的模式。

你写一个`Model`定义文件比如:

`src/models/Simple.js`

```
const model = (sequelize, DataTypes) => {
  const Simple = sequelize.define('Simple', {
    name: DataTypes.STRING,
    email: DataTypes.STRING
  }) return Simple
}module.exports = model
```

并添加一个索引文件`src/models/index.js`

```
const fs = require('fs')
const path = require('path')
const Sequelize = require('sequelize')const {
  dbName,
  dbUser,
  dbPass,
  options
} = require('../utils/dbConfig')const basename = path.basename(module.filename)
const sequelize = new Sequelize(dbName, dbUser, dbPass, options)
const db = { Sequelize, sequelize }const onlyModels = file =>
  file.indexOf('.') !== 0 &&
  file !== basename &&
  file.slice(-3) === '.js'const importModel = file => {
  const modelPath = path.join(__dirname, file)
  const model = sequelize.import(modelPath)
  db[model.name] = model
}const associate = modelName => {
  if (typeof db[modelName].associate === 'function')
    db[modelName].associate(db)
}fs.readdirSync(__dirname)
  .filter(onlyModels)
  .forEach(importModel)Object.keys(db).forEach(associate)module.exports = db
```

这个`index.js`文件:

1.  创建到数据库的连接，
2.  遍历本地文件夹并在每个模型文件上调用`[sequelize.import](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#import)`,
3.  通过其`name`存储相关模型，
4.  然后，对于每个带有`associate`函数的模型，调用该函数将模型连接在一起。

这几乎是所有使用 Sequelize 的项目从单个模型文件中预加载模型的方式。

# 单元测试模型

只要您的代码第一次调用`require('src/models')`该行

```
const sequelize = new Sequelize(dbName, dbUser, dbPass, options)
```

会被调用，并试图连接到您的数据库。这不仅很慢，而且，如果您实际上没有运行数据库，很可能会导致连接错误。

因此，在我们的测试中，我们希望完全避免`require('src/models')`。我们可以将测试集中在我们想要测试的特定模型上:

```
const SimpleModel = require('src/models/Simple')
```

但是为了使用它，我们需要向它传递一个实例化的`sequelize`对象和一个`Sequelize` `DataTypes`对象。

`sequelize-test-helpers`库提供了一个*模拟*序列实例和一整套序列数据类型。

## 单元测试模型的名称和属性

`test/unit/models/Simple.spec.js`

```
const {
  sequelize,
  dataTypes,
  checkModelName,
  checkPropertyExists
} = require('sequelize-test-helpers')const SimpleModel = require('../../../src/models/Simple')describe('src/models/Simple', () => {
  const Model = SimpleModel(sequelize, dataTypes)
  const instance = new Model() checkModelName(Model)('Simple') context('properties', () => {
    ;['name', 'email'].forEach(checkPropertyExists(instance))
  })
})
```

这测试了模型的`name`是我们期望的名字，并且模型具有我们期望的属性。

## 单元测试关联

`sequelize-test-helpers`库还提供了允许我们测试关联的工具。

让我们假设上面的`Simple`模型与一个`OtherModel`相关联。

mock `sequelize`对象使用`[sinon](http://sinonjs.org)`将`[spies](http://sinonjs.org/releases/v6.0.0/spies/)`附加到 Sequelize 的每个关联类型，这意味着您可以通过在`before`块中调用`Model.associate`来测试它们，然后使用`sinon`的`calledWith`函数来检查它是否被正确调用。

```
context('check associations', () => {
  const OtherModel = 'some other model' // it doesn't matter what before(() => {
    Model.associate({ OtherModel })
  } it('defined a belongsTo association with OtherModel', () => {
    expect(Simple.belongsTo).to.have.been.calledWith(OtherModel)
  })
}
```

您可以对所有的`belongsTo`、`hasOne`、`hasMany`和`belongsToMany`执行此操作。

## 单元测试挂钩

通常你会在一个模型中定义[钩子](http://docs.sequelizejs.com/manual/tutorial/hooks.html)，比如`beforeValidate`或者`afterCreate`，并且`Sequelize`提供了三种不同的方法来定义这些钩子。这里有一个简单而荒谬的模型，它定义了三个钩子，每个钩子都有不同的定义。

`src/models/HasHooks.js`

```
const model = (sequelize, DataTypes) => {
  const HasHooks = sequelize.define(
    'HasHooks',
    {
      name: DataTypes.STRING
    },
    {
      hooks: {
        beforeValidate: hooker => {
          hooker.name = 'Alice'
        }
      }
    }
  ) HasHooks.hook('afterValidate', hooker => {
    hooker.name = 'Bob'
  }) HasHooks.addHook('afterCreate', 'removeMe', hooker => {
    hooker.name = 'Carla'
  }) return HasHooks
}module.exports = model
```

`sequelize-test-helpers`库提供了一个助手，允许我们测试钩子，不管它们是如何定义的(或者它们有多愚蠢)。)

```
const {
  sequelize,
  dataTypes,
  checkHookDefined
} = require('sequelize-test-helpers')const HasHooksModel = require('../../src/models/HasHooks')describe('src/models/HasHooks', () => {
  const Model = HasHooksModel(sequelize, dataTypes)
  const instance = new Model() context('hooks', () => {
    ;[
      'beforeValidate',
      'afterValidate',
      'afterCreate'
    ].forEach(checkHookDefined(instance))
  })
})
```

## 单元测试索引

您可以使用各种`check*Index`功能来检查一系列不同的`index`类型。

```
context('indexes', () => {
  ;['email', 'token'].forEach(checkUniqueIndex(instance))
})
```

您可以检查以下`index`类型:

*   `checkUniqueIndex`:检查是否定义了特定的`unique index`，
*   `checkNonUniqueIndex`:检查是否定义了特定的`non-unique index`，以及
*   `checkUniqueCompoundIndex`:检查是否定义了特定的`unique compound index`。

这些`check*`函数一起允许你独立地对大多数种类的序列模型进行全面的单元测试。

# 依赖于模型的单元测试代码

除了对模型本身进行单元测试之外，当然也有必要对使用这些模型的功能进行单元测试。同样，我们并不想真的接近我们的数据库，我们只是想测试单独使用模型的代码。为此，我们需要一组`mockModels`。

`sequelize-test-helpers`库为此提供了一个名为`makeMockModels`的工具。

## 简单的保存功能

让我们测试以下代码:

`src/utils/save.js`

```
const { User } = require('../models')const save = async ({ id, ...data }) => {
  const user = await User.findOne({ where: { id } })
  if (user) return await user.update(data)
  return null
}module.exports = save
```

`save`函数接受一些`data`，使用提供的`id`查找一个`User`，如果找到了，用提供的`data`更新找到的`user`，并返回更新的`user`模型。如果没有找到，函数返回`null`。

## 对保存功能进行单元测试

为了单独测试这一点，我们可以使用几个实用程序。

*   `[sinon](http://sinonjs.org)`:剔除功能，
*   `[proxyquire](https://github.com/thlorenz/proxyquire)`:注入我们自己的模拟模型来代替真实模型的工具，以及
*   `[sequelize-test-helpers](https://www.npmjs.com/package/sequelize-test-helpers)`:提供`makeMockModels`实用程序。

`test/unit/utils/save.spec.js`

```
const { expect } = require('chai')
const sinon = require('sinon')
const proxyquire = require('proxyquire')const { makeMockModels } = require('sequelize-test-helpers')const mockModels = makeMockModels({ User: { findOne: sinon.stub() } })const save = proxyquire('../../../src/utils/save', {
  '../models': mockModels
})const fakeUser = { update: sinon.stub() }describe('src/utils/save', () => {
  const data = {
    firstname: 'Testy',
    lastname: 'McTestface',
    email: 'testy.mctestface@test.tes',
    token: 'some-token'
  } const resetStubs = () => {
    mockModels.User.findOne.resetHistory()
    fakeUser.update.resetHistory()
  } let result context('user does not exist', () => {
    before(async () => {
      mockModels.User.findOne.resolves(undefined)
      result = await save(data)
    }) after(resetStubs) it('called User.findOne', () => {
      expect(mockModels.User.findOne).to.have.been.called
    }) it("didn't call user.update", () => {
      expect(fakeUser.update).not.to.have.been.called
    }) it('returned null', () => {
      expect(result).to.be.null
    })
  }) context('user exists', () => {
    before(async () => {
      fakeUser.update.resolves(fakeUser)
      mockModels.User.findOne.resolves(fakeUser)
      result = await save(data)
    }) after(resetStubs) it('called User.findOne', () => {
      expect(mockModels.User.findOne).to.have.been.called
    }) it('called user.update', () => {
      expect(fakeUser.update).to.have.been.calledWith(
        sinon.match(data))
    }) it('returned the user', () => {
      expect(result).to.deep.equal(fakeUser)
    })
  })
})
```

`makeMockModels`函数为我们做了一些事情。

1.  首先，它在项目的根目录下检查一个`.sequelizerc`文件，如果找到了，它使用在那里定义的`models-path`来查找模型所在的文件夹。
2.  如果它找不到一个`.sequelizerc`文件，那么它就在`src/models`中查找。
3.  如果在那里找不到任何模型，它将抛出一个错误。在这种情况下，您还可以选择将 models 文件夹的路径传递给`makeMockModels`。
4.  一旦找到模型，它就会模拟通常在`src/models/index.js`中的 sequelize `import`代码，将所有的模型名加载到一个对象中。
5.  最后，它将`@noCallThru: true`附加到末尾，告诉`proxyquire`根本不要导入真实的模型文件。这很重要，因为否则真实的模型文件仍然会尝试连接到数据库。
6.  一旦构建了这个对象，`makeMockModels`就会用您提供的任何模型存根覆盖任何默认模型。

所以在上面的例子中，`mockModels`最终包含了

```
{
  User: { update: sinon.stub() },
  Company: 'Company',
  '@noCallThru': true
}
```

然后我们使用 proxyquire 在真实模型的位置插入`mockModels`。

```
const save = proxyquire('../../../src/utils/save', {
  '../models': mockModels
})
```

然后就用西农的存根照常测试。

通过使用`makeMockModels`,你可以对所有依赖于你的模型的代码进行单元测试，而不需要接近你的数据库。这使您的测试保持隔离和快速。

# 结论

使用诸如`sequelize-test-helpers`、`sinon`和`proxyquire`这样的实用程序，您可以全面地测试序列化模型和使用这些模型的代码，而不必连接到实际的数据库。

目前`sequelize-test-helpers`被设计为与`Chai`和`Sinon`一起工作，并假设您正在使用`Node version 10`或更好的版本。

当然，单元测试不能代替集成测试。

当编写依赖于各种数据库模型之间的交互的代码时，重要的是你也要编写打开数据库连接的测试，并检查你的代码*确实做了你期望的*。

也就是说，我喜欢先写我的单元测试，因为我用这种方式抓住了大多数最愚蠢的错误，并且它迫使我用奖励干燥和简单的方式写我的代码。

正确完成的单元测试的优点是运行速度非常快，通常在几毫秒内，并且您不需要任何外部基础设施。需要外部依赖的测试，如数据库、消息队列等，必然需要更长的运行时间。

# 链接

*   `[Sequelize](http://docs.sequelizejs.com)`:最广泛用于`NodeJS`项目的 ORM。
*   `[sequelize-test-helpers](https://www.npmjs.com/package/sequelize-test-helpers)`:一组帮助单元测试`Sequelize`模型的工具。
*   `[sinon](http://sinonjs.org)`:广泛使用的`NodeJS`存根库。
*   `[proxyquire](https://github.com/thlorenz/proxyquire)`:一个广泛使用的`NodeJS`模仿库，允许你将你自己的模仿注入到你正在测试的代码中。
*   `[chai](http://www.chaijs.com)`:一种用于编写测试的非常有表现力的语法。适用于大多数`NodeJS`测试框架。

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。
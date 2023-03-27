# 为您的应用程序配置使用 JSON 模式

> 原文：<https://itnext.io/use-json-schema-for-your-app-config-3c1f4773560f?source=collection_archive---------6----------------------->

![](img/08be4162624bec3198ad48573f1720bd.png)

[JSON 模式](https://json-schema.org/)是一种组织应用程序配置变量的便捷方式。你的应用程序的配置毕竟是一个模型。以下是我认为在应用程序的配置设置中应该满足的要求。

*   变量/属性(那个是 gimmy)
*   为未来的开发人员或部署您的开源应用程序的人员提供这些变量的用户友好描述。
*   有意义的地方默认。
*   一种根据环境(如开发或生产)对配置值组合进行分组的方法。
*   将环境变量合并到您的配置中。这包括用环境中的值覆盖配置值，还包括将那些环境变量从字符串转换为数字和布尔值。不再有`if (process.env.myBoolVariable === 'true')`。
*   (可选)验证您的配置数据结构。得到一个配置验证错误比在代码中找到它要好得多。
*   (可选)能够在应用运行时更新配置。这应该伴随着监听应用配置变化的能力。
*   (可选)嵌套数据结构。

在我开发的应用程序中，我继承并启动了大量的配置设置。我用过`dotenv`和`nconf`。我还使用了像导出一个对象的 JS 文件这样简单的设置。上面的列表是我通过目睹别人做过的我喜欢的事情，以及从艰难的道路中学到的教训而积累起来的。我用过的大多数设置，包括`dotenv`和`nconf`，都没有达到上面的一个或多个要点。`nconf`勾选了最多的框，但是`dotenv`更受欢迎，可能是因为它的简单。不过我喜欢这些库中的一点是 config 是全局可访问的。使用`dotenv` config 就是`process.env`，使用`nconf`你只需要模块，不用担心配置文件的路径。我今天提出的设置也可以这样工作，只是需要打包。

为您的配置使用 JSON Schema 使您能够满足许多需求，而其他需求可以用最少的代码来满足。下面是一个 JSON 模式中配置变量的例子。

```
"port": {
  "type": "integer",
  "description": "The binding port for the server",
  "default": 8080,
  "minimum": 0
}
```

让我们看看我们满足了哪些要求。我们有一个变量及其类型、描述和默认值。除此之外，我们可以添加验证，比如最小值。这里的类型使我们能够转换在环境变量中得到的字符串。其余的要求就不那么容易了，但是让我们看看如何尝试满足它们。

# 故事的其余部分

本文的其余部分将包含 JavaScript 代码示例，说明如何从 JSON 模式配置模型进入工作配置设置。

下面是这个例子的文件结构。

```
- config
  - schema.json  // config model
  - development.json     // config settings for development
- src
  - lib
    - observable.js // observable class to hold config values
  - config.js    // build and export a singleton config observable
  - server.js    // create http server
```

## (计划或理论的)纲要

`/config/schema.json`

```
{
  "$id": "https://github.com/mygithub-handle/my-app-repo/blob/master/config/schema.json",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "My app config",
  "type": "object",
  "required": [
    "port"
  ],
  "properties": {
    "port": {
      "type": "integer",
      "description": "The binding port for the server",
      "default": 8080,
      "minimum": 0
    }
  }
}
```

## 配置文件

`/config/dev.json`

为了便于讨论，让我们假设您需要使用一个不同于默认端口的端口进行开发。

```
{
  "port": 3200
}
```

## 可观察量

`/src/lib/observable.js`

我将使用一个简单的自定义可观察类来保存值。我已经把[藏在 github gist](https://gist.github.com/rhythnic/a84f49a0f5e5440e2cdc30cbb30f404e) 里了。它由节点的`EventEmitter`和`ramda`组成。它允许设置和获取任何路径，深度合并多个值，监听变化，并在变异前进行验证。使用这种观察将允许我们在应用程序运行时更新配置，并监听配置的变化。不要太担心可观察到的，只是如何使用它。

## 属国

`src/config.js`

```
// ajv is a json-schema validator
const Ajv = require('ajv')
// observable for holding the config values
const Observable = require('./lib/observable')
// schema shown above
const schema = require('../config/schema.json')
// utilities
const R = require('ramda')
const path = require('path')
```

## 环境变量

`/src/config.js`

使用模式中的根级键，构建一个从`process.env`中读取这些键的对象。还要将字符串转换为模式中的类型。

```
// build an object of config values taken from process.env
function buildEnvironmentVariablesConfig (schema) {
  const trueRx = /^true$/i
  const configKeys = Object.keys(schema.properties)
  let env = R.pick(configKeys, process.env)
  return Object.keys(env).reduce((acc, key) => {
    const { type } = schema.properties[key]
    switch (type) {
      case 'integer':
        return R.assoc(key, parseInt(env[key], 10), acc)
      case 'boolean':
        return R.assoc(key, trueRx.test(env[key]), acc)
      default:
        return R.assoc(key, env[key], acc)
    }
  }, {})
}
```

## 默认

`/src/config.js`

从架构中的默认值构建一个对象。

```
// build an object using the defaults in the schema
function buildDefaults (schema, definitions) {
  return Object.keys(schema.properties).reduce((acc, prop) => {
    let spec = schema.properties[prop]
    if (spec.$ref) {
      spec = definitions[spec.$ref.replace('#/definitions/', '')]
      if (spec && spec.type === 'object') {
        return R.assoc(prop, buildDefaults(spec, definitions), acc)
      }
    }
    return R.assoc(prop, spec.default, acc)
  }, {})
}
```

## 导入配置设置文件

`/src/config.js`

导入带有该环境设置的文件，如`development.json`、`production.json`、`staging.json`等。你可以用一个环境变量指向这个文件，比如 CONFIG_FILE。该函数为您提供了传递绝对路径或相对于保存模式的`config`文件夹的路径的选项。

```
// import the config file
function buildConfigFromFile (filePath) {
  if (!filePath) return {}
  const isAbsolutePath = filePath.charAt(0) === '/'
  return isAbsolutePath
    ? require(filePath)
    : require(path.join(__dirname, '../config', filePath))
}
```

## 将三者合并在一起

`/src/config.js`

现在我们有了 3 个表示环境变量、一个配置文件和默认值的对象，我们可以将它们合并在一起。`R.mergeDeepRight`将对默认值和配置文件值进行深度合并。`R.merge`将对结果和环境变量值进行浅层合并。在两个合并函数中，最右边的对象优先。

```
// merge the environment variables, config file values, and defaults
let configValues = R.merge(
  R.mergeDeepRight(
    buildDefaults(schema, schema.definitions),
    buildConfigFromFile(process.env.CONFIG_FILE)
  ),
  buildEnvironmentVariablesConfig(schema)
)
```

## 验证功能

`/src/config.js`

如果用户试图设置一个无效的配置值，我们的验证函数将抛出一个错误。另一种选择是只返回 false 并记录错误。

```
const ajv = new Ajv()
const ajvValidate = ajv.compile(schema)function validate (data) {
  const valid = ajvValidate(data)
  if (valid) return true
  throw new Error(ajv.errorsText())
}
```

## 初始化可观察对象并导出

`/src/config.js`

使用上面的 validate 函数创建可观察实例，并用我们刚刚构建的配置值初始化它。导出要在应用程序中使用的单例配置对象。在应用任何突变之前，可观察对象将需要 validate 返回`true`。

```
const config = new Observable({ validate })
config.setAllValues(configValues)module.exports = config
```

## 使用可观察配置

`/src/server.js`

这里有一个使用端口变量的小例子。

```
const config = require('./config')
const http = require('http')const server = http.createServer(/* some middleware callback */)
server.listen(config.get('port'))
```

# 结论

在 JSON Schema 和 observable 的基础上构建应用程序的配置会产生一个非常有特色的配置设置。
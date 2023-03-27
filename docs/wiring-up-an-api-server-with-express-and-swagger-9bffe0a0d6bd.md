# 用 Express 和 Swagger 连接 API 服务器

> 原文：<https://itnext.io/wiring-up-an-api-server-with-express-and-swagger-9bffe0a0d6bd?source=collection_archive---------1----------------------->

![](img/8d4b780bb448e030e57d225f48d21108.png)

我给我的家用电脑编程。把自己传送到未来。(作者图片。)

`[ExpressJS](https://expressjs.com)`是使用`NodeJS`编写 API 服务器的首选框架，`[Swagger](https://swagger.io)`是一种出色的方式来指定 API 的细节，允许您使用一系列方便的工具来确保 API 的一致性、文档化和可测试性。

为了避免重复，最好也使用 API 的 Swagger 定义来告诉 Express 如何将传入路径映射到路由控制器函数。

为了让这变得简单，我写了一个名为`[swagger-routes-express](https://www.npmjs.com/package/swagger-routes-express)`的小软件包(最近更新，除了`Swagger 2`之外，还支持 `OpenAPI 3`)。

# 例子

下面是一个用 Swagger 在文件`my-api.yml`中定义的简单 API，该文件位于项目的根目录下。

```
swagger: "2.0"
info:
  description: Something about the API
  version: "1.0.0"
  title: My Cool API
basePath: "/api/v1"
schemes:
  - "https"
paths:
  /ping:
    get:
      tags:
        - "root"
      summary: "Get Server Information"
      operationId: "ping"
      produces:
      - "application/json"
      responses:
        200:
          description: "success"
          schema:
            $ref: "#/definitions/ServerInfo"
definitions:
  ServerInfo:
    type: "object"
    properties:
      name:
        type: "string"
      description:
        type: "string"
      version:
        type: "string"
      uptime:
        type: "number"
```

有一条路径`GET /ping`，它返回一个`JSON`响应:

```
{
  "name": "My Cool API",
  "description": "Something about the API",
  "version": "1.0.0",
  "uptime": 101
}
```

Swagger doc 为名为`‘ping’`的`GET /ping`路线定义了一个`operationId`。

在我们服务器的`src`文件夹中，我们将如下定义`ping`控制器

`src/api/ping.js`

```
const {
  name,
  version,
  description
} = require('../../package.json')const ping = (req, res) => {
  res.json({
    name,
    description,
    version,
    uptime: process.uptime()
  })
}module.exports = ping
```

在`src/api/index.js`中，我们将确保它被导出:

```
const ping = require('./ping')
module.exports = { ping }
```

定义一个`async`函数来创建和配置 Express 应用程序本身。

这将使用`[swagger-parser](https://www.npmjs.com/package/swagger-parser)`库来解析我们的 swagger api `YAML`文件(因此需要成为`async`)，并将解析后的信息传递给`swaggerRouter`进行配置。

`src/makeApp.js`

```
const express = require('express')
const SwaggerParser = require('swagger-parser')
const swaggerRoutes = require('swagger-routes-express')
const api = require('./api')const makeApp = async () => {
  const parser = new SwaggerParser()
  const apiDescription = await parser.validate('my-api.yml')
  const connect = swaggerRoutes(api, apiDescription) const app = express()
  // do any other app stuff,
  // such as wire in passport, use cors etc. // then connect the routes
  **connect**(app) // add any error handlers last
  return app
}module.exports = makeApp
```

在`src/index.js`中，我们启动服务器:

```
const makeApp = require('./makeApp')makeApp()
  .then(app => app.listen(3000))
  .then(() => {
    console.log('Server started')
  })
  .catch(err => {
    console.error('caught error', err)
  })
```

瞧——运行它，对 `GET /ping`的调用将调用`ping`控制器。

# 添加特定于路由的安全中间件

大多数 API 需要为特定的路由添加某种级别的身份验证和访问控制。在 Swagger 文档中，您可以逐个路径地添加安全信息。

向 Swagger 文档添加一个`GET /api/v1/things`路径:

```
/things:
  get:
    summary: "Find things"
    description: "Returns a list of things"
    operationId: "v1/things/listThings"
    produces:
    - "application/json"
    responses:
      200:
        description: "success"
        schema:
          $ref: "#/definitions/ArrayOfThings"
    security:
    - slack:
      - "identity.basic"
      - "identity.email"
```

在实际的应用程序中，您需要添加`ArrayOfThings`的定义和安全定义，但是为了简洁起见，忽略它们。

给`src/api/index.js`增加一个`listThings`控制器，将`v1/things/listThings`映射到`v1_things_listThings`(你可以通过`option`改变`pathSeparator`，见下文)。

```
const ping = require('./ping')
const v1_things_listThings = require('./v1/things/listThings')module.exports = { ping, v1_things_listThings }
```

您需要告诉`swaggerRouter`如何将文档中定义的安全范围与实际的认证中间件相关联。通常你会使用 passport 来建立基于标准的认证中间件，但是最终中间件只是一个带有`req, res, next`签名的函数。让我们在`src/auth/isAllowed.js`中定义一个，它只是在标准的`Authorization`请求头中查找文本`'LET THE RIGHT ONE IN'`。

```
module.exports = (req, res, next) => {
  if (req.get('authorization') === 'LET THE RIGHT ONE IN') {
    return next()
  }
  return res.status(401).end()
}
```

现在，在我们的 makeApp 函数中，我们需要向`swaggerRouter`传递一些选项。

```
const connect = swaggerRoutes(api, apiDescription, {
  scopes: {
    'identity.basic,identity.email': isAllowed
  }
})
```

现在`connect`函数将知道如何将路径与范围`identity.basic`相关联，并将`identity.email`与`isAllowed`中间件相关联。

## 基本路径呢？

与上面定义的`ping`路径不同，`/things`与 API 的`basePath`、`'api/v1'`连接在一起。由于`root`标签的缘故，`swaggerRouter`知道`ping`路径不在基本路径下。您可以在下面列出的选项中对此进行更改。

## 缺少控制器怎么办？

如果 Swagger doc 定义了一个不能映射到现有路线控制器函数的`operationId`，那么`connector`将在它的位置插入一个默认的`notImplemented`函数，返回`res.status(501).end()`。

如果没有`operationId`与路径相关联，那么`connector`将插入一个默认的`notFound`函数，该函数返回`res.status(404).end()`。

您可以通过传递给`swaggerRouter`的选项覆盖默认功能，如下所述。

# 选择

`swaggerRouter`功能的全套允许选项如下:

```
{
  notImplemented, // a default function is supplied,
  notFound, // a default function is supplied
  scopes: {},
  apiSeparator: '_',
  rootTag: 'root'
}
```

# 其他工具

`[swagger-express-validator](https://github.com/gargol/swagger-express-validator)`根据它们的 Swagger 定义验证 API 路由控制器的输入和输出。当结合全面的集成测试时，您可以断言您的服务器的正确性。

`[swagger-ui-express](https://github.com/scottie1984/swagger-ui-express)`实用程序将您的 Swagger 定义文档作为一个交互式 web 用户界面，确保访问您的 API 的第三方能够访问与其代码一致的文档。

# 链接

*   [快递](https://expressjs.com)
*   [大摇大摆](https://swagger.io)
*   `[swagger-routes-express](https://www.npmjs.com/package/swagger-routes-express)`在 NPM
*   `[swagger-routes-express](https://github.com/davesag/swagger-routes-express)`GitHub 中的源代码
*   `[swagger-express-validator](https://github.com/gargol/swagger-express-validator)`
*   `[swagger-ui-express](https://github.com/scottie1984/swagger-ui-express)`

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。
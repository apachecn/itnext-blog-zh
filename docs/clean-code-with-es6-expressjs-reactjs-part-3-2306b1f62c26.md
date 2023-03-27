# 使用 ES6 清理代码:ExpressJS + ReactJS:第 3 部分

> 原文：<https://itnext.io/clean-code-with-es6-expressjs-reactjs-part-3-2306b1f62c26?source=collection_archive---------3----------------------->

![](img/d7930344e8dd5c427ecfe11b931965ab.png)

由[托马斯·克维斯霍尔特](https://unsplash.com/photos/oZPwn40zCK4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/server?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fclean-code-with-es6-expressjs-reactjs-part-3–2306b1f62c26%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

这是一个简短系列文章的第三部分，重点是使用 ES6+ Javascript、React、Redux、ExpressJS 和 MongoDB 创建和部署一个准系统的全栈博客站点。

在这些文章中，我真的试图把重点放在我发现的使用 ES6+ JavaScript 和我称之为“Redux 风格”的项目结构来设置和组织我的代码的一些方法上。目标是编写可读、可扩展且易于测试的代码。

这个系列将分为几个部分:

1.  [设置 ExpressJS 使用 ES6+语法](/clean-code-with-react-expressjs-and-mongodb-part-1-aa6b1a4aef82)
2.  [Redux 风格的文件结构](/clean-code-with-react-expressjs-and-mongodb-part-2-89d20e684820)
3.  创建简单快速服务器👈🏽*你在这里*
4.  用我们的博客设置 Mongoose(即将推出！)
5.  用 CMS 创建一个 React 博客(即将推出！)
6.  将 Redux 添加到您的前端(即将推出！)
7.  包装您的应用:部署(即将推出！)

# 创建简单快速服务器

现在我们已经设置好了 ES6+语法，并且知道如何“Redux-style”地导入/导出我们的模块，让我们开始用 ExpressJS 创建一个非常简单但是健壮的服务器。

如果您遵循上一篇文章，您的文件结构应该如下所示。不要担心`*.js`文件是空的，因为我们很快就会处理它们。

```
📁/
 📂configs/
  📄index.js
  📄server.js
 📂database/
 📂routes/
 📄index.js
 📄.babelrc
 📄package.json
 📄yarn.lock
```

首先清除我们的**根**文件中的所有内容。复制并粘贴以下代码:

*。/index.js:*

```
// APP
import express from 'express'
const app = express()
// CONFIGS
// ROUTES
// RUN
app.listen(8000, err => {
  if (err) {
    console.log(err)
  }
  console.log('EXPRESS SERVER Running on port: 8000')
})
```

🔬我们正在做三件事:

1.  导入 express(不使用“Redux-style”)
2.  快速应用程序的创建和实例
3.  告诉我们的 express 应用程序在端口 8000 运行并报告(或报告任何错误)

# 配置服务器中间件

接下来，让我们设置我们的中间件。我们将使用以下包:[头盔](https://www.npmjs.com/package/helmet)、[压缩](https://www.npmjs.com/package/compression)、[身体解析器](https://www.npmjs.com/package/body-parser)、 [cors](https://www.npmjs.com/package/cors) 和[摩根](https://www.npmjs.com/package/morgan)。

🔬**中间件**是在 HTTP 请求路由逻辑之前运行的代码的术语

用纱线将它们安装在我们的终端:

`yarn add helmet compression body-parser cors -S`

和

`yarn add morgan -D`

在`📂configs/`目录中修改`server.js`和`index.js`如下:

*。/configs/server.js:*

```
import helmet from 'helmet'
import compression from 'compression'
import cors from 'cors'
import morgan from 'morgan'
import bodyParser from 'body-parser'
//
export const configureServer = app => {
  app.use(helmet())
  app.use(compression())
  app.use(bodyParser.urlencoded({ extended: 'true' }))
  app.use(bodyParser.json())
  app.use(bodyParser.json({ type: 'application/vnd.api+json' }))
  app.use(morgan('dev'))
  app.use(cors())
  return app
}
```

*。/configs/index.js*

```
export * from './server'
```

现在，在我们的**根**T5 处，添加以下代码:

*。/index.js:*

```
// APP
import express from 'express'
const app = express()
// CONFIGS
import {configureServer} from './configs
configureServer(app)
// ROUTES
// RUN
app.listen(8000, err => {
  if (err) {
    console.log(err)
  }
  console.log('EXPRESS SERVER Running on port: 8000')
})
```

🔬让我们来分析一下这个复制意大利面:

1.  我们正在导入我们的中间件，以便稍后可以通过它运行我们的应用程序:

*   [**头盔**](https://www.npmjs.com/package/helmet) 为我们的 HTTP 头提供了基本的安全性
*   [**压缩**](https://www.npmjs.com/package/compression) 减小我们发送回客户端的响应的大小
*   [**cors**](https://www.npmjs.com/package/cors) 保护我们免受可疑的[跨来源请求](https://w3c.github.io/webappsec-cors-for-developers/)
*   [**摩根**](https://www.npmjs.com/package/morgan) 是一个伟大的，基本的工具，让我们监视我们的控制台中的 HTTP 活动
*   [**body-parser**](https://www.npmjs.com/package/body-parser) 转换传入的请求，以便我们可以在 Express 中处理它们

2.接下来，我们创建一个接受 Express 应用程序实例的函数，并通过中间件运行该应用程序。

*(注意:这些中间件中的每一个都可以彻底配置。****CORS****尤其如此。因为我们正在设置一个最小的服务器，所以我不会在这里讨论这些细节，但是我们过一会儿再回到 cors。)*

# 设置路线

在`📂routes/`目录下，创建以下两个文件:`public.js`和`index.js`。我们将在这里用`index.js`做一些稍微不同的事情，但是不要担心，它非常符合一直指导我们的组织精神。

将以下代码添加到各自的文件中:

*。/routes/public.js:*

```
export const publicRoutes = app => {
  app.get('/', (req, res) => {
    res.json({ message: 'Hello GET from ROOT' })
  })app.post('/', (req, res) => {
    res.json({ message: 'Hello POST from ROOT' })
  })
}
```

*。/routes/index.js:*

```
import { publicRoutes } from './public'export const configureRoutes = app => {
  publicRoutes(app)
  return app
}
```

现在，让我们使用我们的主要路线`index.js`。我们的整个`index.js`应该如下所示:

*。/index.js:*

```
// APP
import express from 'express'
const app = express()
// CONFIGS
import { configureServer, configureDb } from './configs'
configureServer(app)
// ROUTES
import { configureRoutes } from './routes'
configureRoutes(app)
// RUN
app.listen(8000, err => {
  if (err) {
    console.log(err)
  }
  console.log('EXPRESS SERVER Running on port: 8000')
})
```

如果您现在运行应用程序，您应该能够从我们指定的两条路线获得回复。使用 Postman，我们可以看到我们从`[http://localhost:8000/](http://localhost:8000/:)` [:](http://localhost:8000/:)

```
{ "message": "Hello GET from ROOT" }
```

不错！

# 进一步组织

但是为什么就此打住呢？

浏览我们的代码，我们可以注意到两件事:

1.  我们重复代码的某些部分，其中一些我们可以称之为“常量”
2.  我们可能想把逻辑从我们的路线中分离出来

将我们的常量分离到一个单独的文件中是一个特别好的主意。我们可以在我们的`📂configs`目录中创建一个`constants.js`文件。它将保存与我们的工作环境相关的所有逻辑(比如在`.env`文件中指定的端口和 API 键:参见这篇 [Twilio 文章](https://www.twilio.com/blog/2017/08/working-with-environment-variables-in-node-js.html)了解更多信息。

## 清洁和粉笔索引. js

在我们开始工作之前，让我们进一步清理我们的主`index.js`并清理我们的控制台输出！这并不是真正必要的，但是为什么不竭尽全力让我们的代码看起来超级干净和易读呢？

使用纱线添加[粉笔](https://www.npmjs.com/package/chalk):

```
yarn add chalk -D
```

现在，在我们的`📂configs`目录中创建一个名为`runServer.js`的文件。添加以下代码:

*。/configs/runServer.js:*

```
import { constants } from './'
import chalk from 'chalk'export const runServer = app => {
  app.listen(constants.PORT, err => {
    let date = new Date()
    if (err) {
      console.log(chalk.redBright(err))
    }
    console.log(
      chalk.bgBlueBright('Express Server\n') +
        chalk.blueBright(
          'Port: ' + constants.PORT + '\nStarted: ' + date.toLocaleTimeString()
        )
    )
  })
}
```

🔬Chalk 允许我们指定控制台输出的颜色，如果你整天盯着你的控制台，这是很好的。我们还添加了时间，这样我们就可以看到服务器上次刷新的时间。

注意，我们已经从同一个`📂configs`目录中导入了`constants`。让我们来指定这些常数。创建一个名为`constants.js`的文件，在同一个目录下通过`index.js`导出两个新函数:

*。/configs/constants.js:*

```
export const constants = {
  PORT: 8000
}
```

*。/configs/index.js:*

```
export * from './constants'
export * from './runServer'
export * from './server'
```

现在我们可以用下面的代码更新我们的`📁backend`目录中的`index.js`，删除`app.listen(8000)...`。既然这样，让我们重新安排一下`import`语句。如果我们拿它穿过棉绒，它会心脏病发作。整个文件应该如下所示:

*。/index.js:*

```
// IMPORTS
import express from 'express'
import { configureServer, runServer } from './configs'
import { configureRoutes } from './routes'
// INITALIZE APP
const app = express()
// CONFIGS
configureServer(app)
// ROUTES
configureRoutes(app)
// RUN
runServer(app)
```

不仅可读性更好，而且，当我们需要修改某些东西时，我们知道去哪里！

# 清理路线

虽然我们还没有建立我们的 MongoDB 数据库(这是下一篇文章)，但是我们仍然可以通过将逻辑(或者将要成为的逻辑)移动到`📁database`目录来做准备。但是让我们从`📁routes`目录开始。

用以下代码修改`📁routes`目录中的`public.js`:

*。/routes/public.js:*

```
import { postsGetAll, userLogin } from '../database'export const publicRoutes = app => {
  app.get('/api/get_all_posts', postsGetAll)
}
```

让我们保持逻辑的语义分离。没有必要，但仍然是一个好主意。我们将创建另一个路由文件:

*。/routes/userManagement.js:*

```
import { postsGetAll, userLogin } from '../database'export const userManagementRoutes = app => {
  app.post('/api/login_user', userLogin)
}
```

现在我们需要更新`📁routes`目录中的`index.js`:

*。/routes/index.js:*

```
import { publicRoutes } from './public'
import { userManagementRoutes } from './userManagement'export const configureRoutes = app => {
  publicRoutes(app)
  userManagementRoutes(app_
  return app
}
```

🔬请注意，我们更改了路由文件中的 URL。如果我们使用带有内置路由器的 ExpressJS，URL 将取决于您在路由过程中的位置。有了这种类型的架构，我们可以清楚地知道我们要把客户送到哪里。

# 数据库逻辑的位置

即使我们正在做一些甜蜜的路由，如果我们试图运行我们的应用程序，它会崩溃。这是因为我们需要在我们的`📁database`目录中设置一些东西。现在，让我们通过创建以下文件来实现这一点:

*。/database/posts.js*

```
// Blog Post Logic
export const postsGetAll = (req, res) => {
         res.json({ posts: 'ALL POSTS!' })
}
```

*。/database/user.js*

```
// User Management Logicexport const userLogin = (req, res) => {
         res.json({ username: 'User Login Username: ' +         
                    req.body.username })
}
```

*。/database/index.js*

```
export * from './posts'
export * from './user'
```

现在我们可以用`yarn start`运行我们的服务器并测试路线了！我将使用 Postman 来完成这项工作:

# 接下来呢？

在本文中，我们学习了如何设置一个非常简单的服务器。以这种方式分离我们的逻辑不仅有助于可读性，还有助于重用。(如果*真的*我们想要，我们可以创建另一个 Express 实例，使用不同的路由，但是使用相同的中间件！)在下一篇文章中，我们将探讨如何使用 Mongoose 设置 MongoDB。敬请期待！
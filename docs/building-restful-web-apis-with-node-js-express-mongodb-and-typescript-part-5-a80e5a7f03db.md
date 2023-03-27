# 用 Node.js、Express、MongoDB 和 TypeScript 构建 RESTful Web APIs 第 5 部分

> 原文：<https://itnext.io/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-5-a80e5a7f03db?source=collection_archive---------0----------------------->

![](img/e448896648e7a16f3be0c2707c5a95ea.png)

(图片来自 OctoPerf)

有一个关于[如何在 Lynda](https://www.lynda.com/Node-js-tutorials/Next-steps/633869/671263-4.html) 上构建 Web APIs 的课程，但是他们没有使用 TypeScript。所以我决定用 TypeScript 做一个。这个项目中有许多需要改进的地方。如果你找到了，请留下评论。我很感激。)

[**第一部分:设置项目**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-1-2-195bdaf129cf)

[**第二部分:实现路由和 CRUD**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-2-98c34e3513a2)

[**第 3 部分:为 Web APIs 使用控制器和模型**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-3-d545b243541e)

[**第 4 部分:将 Web APIs 连接到 MongoDB 或其他**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-4-954c8c059cd4)

[**第 5 部分:我们的 Web APIs 的安全性**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-5-a80e5a7f03db)

[**奖励:用云函数、Firestore 和 Express**](/building-a-serverless-restful-api-with-cloud-functions-firestore-and-express-f917a305d4e6) 构建“无服务器”RESTful API

[**奖励:在 Nodejs**](/handling-long-running-api-requests-in-nodejs-403bd566d47) 中处理长时间运行的 API 请求

在这一部分，我将向您展示保护 RESTful Web APIs 的各种方法。对于更安全的 API 应用程序，您应该至少使用一种或组合使用这些方法。

如果你想使用像 [mLab](https://mlab.com/) 、[compose](https://www.compose.com)……这样的服务，他们已经在他们的终端实现了一个安全的系统。你需要做的就是按照他们的指示将数据库连接到你的应用程序。

**方法 1** : **首先也是最重要的是，你应该始终使用 HTTP 上的 HTTPS**

对于本地测试，我将在 Windows 上使用 [OpenSSL](https://slproweb.com/products/Win32OpenSSL.html) 来为 HTTPS 配置生成密钥和证书。这个过程在 Mac 或 Linux 上是相似的。

安装 OpenSSL 后，我将打开 OpenSSL 并开始生成密钥和证书文件。

```
OpenSSL> req -newkey rsa:2048 -nodes -keyout keytemp.pem -x509 -days 365 -out cert.pem
OpenSSL> rsa -in keytemp.pem -out key.pem
```

之后，我们将把 **key.pem** 和 **cert.pem** 文件移到我们的项目中。它们将位于 config 文件夹中。

然后我们将编辑 server.ts 文件来启用 https。

```
import app from './app';
**import * as https from 'https';
import * as fs from 'fs';**
const PORT = 3000;**const httpsOptions = {
    key: fs.readFileSync('./config/key.pem'),
    cert: fs.readFileSync('./config/cert.pem')
}****https.createServer(httpsOptions, app).listen(PORT, () => {
    console.log('Express server listening on port ' + PORT);
})**
```

为了测试服务器，我们将运行

```
ts-node .\lib\server.ts
```

从现在开始，我们的应用程序将始终在 HTTPS 上运行。

![](img/de4e3cf3906d76a807041cb7ff766982.png)

通过 HTTPS(邮递员)获取数据

![](img/f28f2bd79ff76652e82098e917c54704.png)

如果试图通过 HTTP 访问，您将得不到任何响应并出现错误

**方法 2:使用密钥进行认证**

这个方法使用一个惟一的键来传递 URL，因此您可以访问数据库。您可以使用 crypto 从命令行创建密钥。

```
node -e "console.log(require('crypto').randomBytes(20).toString('hex'))"
```

![](img/40b18c14c30eeea67c5d7eeaf610bf8a.png)

现在，我们将使用中间件在响应请求之前检查密钥。例如，如果您想要获取所有联系人，您需要传递一个键。

```
// GET request
https://127.0.0.1:3000?key=**78942ef2c1c98bf10fca09c808d718fa3734703e**
```

我们将在发送请求之前编辑**/lib/routes/CRM routers . ts**。*请记住，在生产中，您应该在环境中传递密钥，而不是像示例中那样直接传递*。

```
// lib/routes/crmRouters.ts
// get all contactsapp.route('/contact')
.get((req: Request, res: Response, **next: NextFunction**) => {
 // middleware          
 if(req.query.key !== '**78942ef2c1c98bf10fca09c808d718fa3734703e**'){
  **res.status(401).send('You shall not pass!');**
 } else {
  **next();**
 }                        
}, this.contactController.getContacts)
```

![](img/b57a2242a6463e7456863f9a4b11a3ff.png)

我们被允许用密钥获取数据

![](img/ecc9d50af08f79183e2b6d12f108f69b.png)

没有钥匙你不能进入

**方法 3:保护您的 MongoDB**

很悲哀的是，默认情况下，MongoDB 根本就没有像这样的安全性。如果您想检查您当前的配置。转到您的 mongo 安装目录并键入 mongo。

![](img/55f42124c8f3c90591b88366b31d86ff.png)

如您所见，对数据库没有访问控制，任何人都可以对数据库做任何事情。因此，我们将为 MongoDB 启用身份验证特性。

首先，我们需要创建一个帐户，以便通过 Mongodb 进行身份验证。

![](img/89379515c68c13b9e1b58b780f660a69.png)

之后，我们将停止并重启带身份验证的 MongoDB。记得检查你的 dbpath。

```
// Stop MongoDB (Windows)
net stop MongoDB// Start mongodb with authentication
mongod --auth --port 27017 --dbpath C:\your-data\path
```

现在，如果我们登录到 mongo shell，就不会有关于访问控制的警告。

![](img/701d0a22b633f7ef831d2cd506a44e80.png)

或者您可以使用刚刚创建的用户名和密码连接到 mongo shell。

```
mongo --port 27017 -u dalenguyen -p 123123  --authenticationDatabase CRMdb
```

现在，如果我们试图访问数据库，即使有密钥，我们也不能。

![](img/8e40b95e57a8fcd7ca5dccaacf58e1a0.png)

这就是为什么我们需要编辑 mongodb 的 URL，以便应用程序能够工作。*同样，您应该将 mongodb URI 放到环境中。*

```
// lib/app.tsclass App {
...public mongoUrl: string = '**mongodb://dalenguyen:123123@localhost:27017/CRMdb**';
```

然后你重启 RESTful API，一切将再次开始正常工作，但现在你有一个更安全和可控的 API 应用程序。我们可以实现更多的安全方法来改进我们的应用程序。我会试着在其他帖子中更新它们。

在这之后，现在我们有了一个完全安全的工作 RESTful Web APIs 应用程序，它带有 TypeScript 和 Nodejs。如果你想检查所有代码，请访问我的 [github 库](https://github.com/dalenguyen/rest-api-node-typescript)获取完整代码。

[https://github.com/dalenguyen/rest-api-node-typescript](https://github.com/dalenguyen/rest-api-node-typescript)
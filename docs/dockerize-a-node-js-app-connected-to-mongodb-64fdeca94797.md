# 将一个 Node.js 应用程序连接到 MongoDb

> 原文：<https://itnext.io/dockerize-a-node-js-app-connected-to-mongodb-64fdeca94797?source=collection_archive---------0----------------------->

你好亲爱的程序员，欢迎来到我的技术文章系列，献给 **Node.js** 和 **Docker。**希望你喜欢！

请找到系列的第一部分:
1。[“用 VS 代码 Dockerize 一个 node . js app”](/dockerize-a-node-js-app-with-vs-code-bd471710dc22)

![](img/eddc731dcdf0e33adbb03cc5698512ed.png)

[https://unsplash.com/photos/PBctAzztru0](https://unsplash.com/photos/PBctAzztru0)

# 问题:

您已经从本系列前一篇文章中了解了如何将 Docker 与 Node 结合使用。我知道我们都爱 MERN。我们的下一步是理解 Node 和 Mongo 如何在容器内相互连接。我们走吧！

# 1.在本地安装 MongoDb

是时候进入一些文档数据库的东西了。首先[从这里下载 MongoDb 服务器。](https://www.mongodb.com/dr/fastdl.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-4.0.10-signed.msi/download)

如果你在安装过程中没有改变任何东西，它还应该安装一个叫做[MongoDb Compass Community](https://www.mongodb.com/products/compass)的东西。

这是一个很好的工具，可以在 MongoDb 中检查、更改、添加或删除集合中的数据。您可以使用默认地址和端口连接到本地实例，如下图所示，或者连接到任何其他服务器。

![](img/0c163ed2a74bbea6e9846e684050a186.png)

要本地连接，只需按“连接”。在里面你可以看到一些默认的集合，你可以玩。稍后我们将需要 **MongoDb 指南针**。

# 2.通过 Express app 连接到 MongoDb

在本教程中，我将使用我最喜欢的编辑器 Visual Studio 代码。你还需要安装 Nodejs 和 Docker。在我的例子中，我使用的是 Windows，所以我从这里得到了 Windows 的 [Docker。](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

现在运行以下命令:

```
mkdir test-mongo-app && cd test-mongo-app && npm init -y && code .
```

安装依赖项的时间。我们需要快递和猫鼬包装。

```
npm i express mongoose
```

在根文件夹中创建名为`server.js`的文件。

另外，不要忘记在开始时改变你的`package.json`来运行`server.js`文件。

```
{
 "name": "test-mongo-app",
 "version": "1.0.0",
 "description": "",
 "main": "server.js",
 "scripts": {
 "start": "node server.js"
 },
 "keywords": [],
 "author": "",
 "license": "ISC",
 "dependencies": {
 "express": "4.17.1",
 "mongoose": "5.6.1"
 }
}
```

很好。让我们创建一个有两条路线的基本快递应用程序。一个用于从数据库中读取*用户*，另一个用于向其中添加虚拟*用户*数据。

首先，检查是否一切都与刚才的快速服务器。

```
// server.js
const express = require("express");
const app = express();const PORT = 8080;app.get("/", (req, res) => {
 res.send("Hello from Node.js app \n");
});app.listen(PORT, function() {
 console.log(`Listening on ${PORT}`);
});
```

可以运行`npm start`来测试一下。如果您看到消息“*在 8080* 上监听”，则一切正常。同样打开 [http://localhost:8080](http://localhost:8080) 查看是否能看到 hello 消息。

有个好东西叫 [nodemon](https://www.npmjs.com/package/nodemon) 。当源代码发生任何变化时，它会自动重建我们的项目。让我们使用它！😀

```
npm install — save-dev nodemon
```

在`package.json`中添加新命令。所以我们用它来发展。

```
 "scripts": {
 "start": "node server.js",
 "dev": "nodemon server.js"
 }, 
```

现在在开发时使用 run `npm run dev `而不是` npm start'。

```
npm run dev
```

您将注意到控制台中的不同，因为现在 nodemon 正在监视项目中的任何更改，如果需要，可以重新构建它。在`server.js`改变一些东西，你会注意到😉

现在在项目的根目录下创建文件夹`src`。在这里，我们将添加所有其余的文件。

让我们为 mongoose 创建一个用户模型。创建文件名`User.model.js`

```
// User.model.js
const mongoose = require("mongoose");
const userSchema = new mongoose.Schema({
 username: {
 type: String
 }
});const User = mongoose.model("User", userSchema);module.exports = User;
```

很好！这里我们为我们的文档数据库定义了一个模型。用户模型只有一个字符串字段 *username* 。暂时够了:)

让我们添加一个名为`connection.js`的文件来连接数据库。

```
 // connection.js
const mongoose = require("mongoose");
const User = require("./User.model");const connection = "mongodb://localhost:27017/mongo-test";const connectDb = () => {
 return mongoose.connect(connection);
};module.exports = connectDb;
```

请注意，mongo-test 将是我们数据库(集群)的名称！

现在修改一点`server.js`启动 app。您应该在控制台中看到 MongoDb 已连接的消息。

```
 // server.js
const express = require("express");
const app = express();
const connectDb = require("./src/connection");const PORT = 8080;app.get("/users", (req, res) => {
 res.send(“Get users \n”);
});app.get("/user-create", (req, res) => {
 res.send(“User created \n”);
});app.listen(PORT, function() {
 console.log(`Listening on ${PORT}`);connectDb().then(() => {
 console.log("MongoDb connected");
 });
});
```

耶！🎉我们将 Express 应用程序与本地 MongoDb 实例连接起来！

# 3.实现对 MongoDb 的读写

我们应该实现两条读取和添加新用户的路线。
打开“server.js”文件，首先在顶部导入我们的模型:

```
 // server.js
const User = require("./src/User.model");
// …
```

然后像这样实现下面的两条路线:

```
// server.js
app.get("/users", async (req, res) => {
 const users = await User.find(); res.json(users);
});app.get("/user-create", async (req, res) => {
 const user = new User({ username: "userTest" }); await user.save().then(() => console.log(“User created”)); res.send("User created \n");
});
// …
```

注意这里我们使用的是异步/等待模式。如果你对此感到好奇，请在这里找到它。

基本上，我们实现了两条路径“/users”和“/user-create”。✋:是的，是的，我知道创建应该通过 POST http 动词来完成，但只是为了使测试更容易，避免为数据库配置种子方法。

现在是考验的时候了！🔍在浏览器中打开这个链接[http://localhost:8080/user-create](http://localhost:8080/user-create)在 db 中创建一个虚拟用户记录。打开此链接[http://localhost:8080/users](http://localhost:8080/users)在浏览器中获取所有用户为 JSON。

这样做之后，你可以回到 MongoDb Compass 并在这里检查用户集合。你应该看看这个

![](img/de1984823a17cfe9f414bd4bf9180971.png)

# 4.Dockerize 节点和 MongoDb

将 Docker 文件添加到根文件夹。

```
touch Dockerfile
```

在其中粘贴以下内容:

```
FROM node:8
# Create app directory
WORKDIR /usr/src/app
# Install app dependencies
COPY package*.json ./RUN npm install# Copy app source code
COPY . .#Expose port and start application
EXPOSE 8080
CMD [ "npm", "start" ]
```

我们可以用这个命令简单地构建我们的 express 应用程序

```
docker build -t mongo-app .
```

但是..这只会运行我们的 express 应用程序，但不会与 MongoDb 一起运行。这就是为什么我们需要一个“docker-compose”文件。🐳

现在创建另一个名为`docker-compose.yml`的文件并粘贴它:

```
 version: "2"
services:
 web:
 build: .
 ports:
 — "8080:8080"
 depends_on:
 — mongo
 mongo:
 image: mongo
 ports:
 — "27017:27017"
```

我们在这个文件中定义了两个服务。一个是我们在端口 8080 上运行的节点应用，另一个是 mongodb 实例。

⚠️在您运行下一个命令之前，**请确保**您已经在`connection.js`文件中将连接字符串更改为 mongo db。

```
const connection = "mongodb://mongo:27017/mongo-test";
```

我们用 **mongo** 替换了 **localhost** ，这非常重要。因为我们应该告诉应用程序，我们希望从 docker 内部虚拟网络而不是本地虚拟网络访问 MongoDb。

现在运行魔法命令🔮

```
docker-compose up
```

在[http://localhost:8080/users](http://localhost:8080/users)和[http://localhost:8080/user-create](http://localhost:8080/user-create)上打开浏览器，查看我们在 Docker 中运行的应用。

*(如果出现任何问题，请尝试停止/删除映像和容器，通过再次破坏 docker 合成来重建映像，如果 mongo 映像未从 hub 中取出，请尝试重新登录 docker hub 或重启 docker for Windows)*

在这里看到[源代码](https://github.com/vguleaev/Express-Mongo-Docker-tutorial/tree/master/test-mongo-app)。尽情享受吧！

🚀如果你从那篇文章中读到了一些有趣的东西，请喜欢并关注我的更多帖子。谢谢你亲爱的编码员！😏
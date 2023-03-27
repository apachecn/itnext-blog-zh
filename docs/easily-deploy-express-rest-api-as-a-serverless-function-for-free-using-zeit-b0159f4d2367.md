# 使用 ZEIT，轻松地将 express API 部署为免费的无服务器功能

> 原文：<https://itnext.io/easily-deploy-express-rest-api-as-a-serverless-function-for-free-using-zeit-b0159f4d2367?source=collection_archive---------6----------------------->

使用 ZEIT 免费轻松部署 express API 作为无服务器功能的指南。

![](img/510d86abdf27962776b3498f3684a8f2.png)

[Benjamin Voros](https://unsplash.com/@vorosbenisop?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片(只是美图，与内容无关)

# 介绍

无服务器计算(或简称为无服务器计算)是一种执行模型，云提供商(AWS、Azure 或 Google Cloud)负责通过动态分配资源来执行一段代码。并且只对用于运行代码的资源数量收费。代码通常在无状态容器中运行，这些容器可以由各种事件触发，包括 HTTP 请求、数据库事件、队列服务、监控警报、文件上传、预定事件(Cron 作业)等。发送给云提供商执行的代码通常以函数的形式出现。因此，无服务器有时被称为*【功能即服务】*或*【FaaS】*。

# 初始化

## Windows 操作系统

如果您使用的是 windows，我强烈建议您下载并安装 git bash。它与 git 捆绑在一起，可以从这里下载:[https://git-scm.com/downloads](https://git-scm.com/downloads)

## 项目设置

转到项目文件夹，打开终端或 git bash(如果在 windows 上)。我们现在将创建一个新文件夹，并在其中进行更改。复制并粘贴下面的命令来完成。

```
mkdir express-serverless-example && cd express-serverless-example 
```

现在我们将安装项目所需的所有依赖项，确保 nodejs(你可以在这里获得:[https://nodejs.org/en/download/](https://nodejs.org/en/download/))已经安装在你的系统中。要确保安装了 nodejs 和 npm，您可以运行下面的命令来检查每个版本。

```
node -v && npm -v
```

我们现在将初始化一个新项目，为此，您可以运行下面的命令。这将使用默认选项创建一个新的 package.json 文件。

```
npm init -y
```

我们现在将安装 express。

```
npm i express
```

我们现在将安装 now(用于 ZEIT 的 CLI 工具)作为全局依赖项。

```
npm i -g now
```

现在，在你最喜欢的代码编辑器中打开新创建的文件夹，如果你正在使用 vscode(你可以在这里得到它:[https://code.visualstudio.com/](https://code.visualstudio.com/))，我强烈推荐，你可以键入下面的命令，它将打开 vscode。

```
code .
```

# 快速代码

我们现在将创建一个“index.js”文件来创建一个 express 应用程序(文件名**必须**为“index.js”才能与 ZEIT 一起使用)，并将以下内容粘贴到其中。

```
const express = require("express");const app = express();const port = 3000; // Body parserapp.use(express.urlencoded({ extended: false }));// Home routeapp.get("/", (req, res) => {res.send("Welcome to a basic express App");});// Users routeapp.get("/users", (req, res) => {res.json([{ name: "Akash", location: "India" },{ name: "Abhishek", location: "India" },]);});// Listen on port 5000app.listen(port, () => {console.log(`Server is running on port 3000Visit http://localhost:3000`);});
```

就这样，我们完成了基本的 express 应用程序。

# 主办；主持

如果你还没有，在[https://zeit.co/](https://zeit.co/)上创建一个账户，在你的终端或 git bash 中输入下面的命令，然后按照提示登录。

```
now login
```

在 ZEIT 中托管我们的应用程序之前，我们必须为它创建一个配置文件。为此，创建一个名为“now.json”的新文件，并粘贴以下内容。

```
{"version": 2,"builds": [{ "src": "index.js", "use": "@now/node-server" }],"routes": [{"src": "/","dest": "/index.js","methods": ["GET"]},{"src": "/users","dest": "/index.js","methods": ["GET"]}]}
```

我现在将解释上面的每个字段，[版本](https://zeit.co/docs/configuration#project/version)字段将指定 ZEIT Now 平台版本，[构建](https://zeit.co/docs/configuration#project/builds)字段将指定使用哪个构建和哪个文件作为源，[路由](https://zeit.co/docs/configuration#project/routes)字段将指定所有要使用的路由，这是最重要的部分，因此**如果您添加新路由，不要忘记在此处包含它**。

你可以在这里找到更多关于配置的信息[https://zeit . co/docs/configuration # introduction/configuration-reference](https://zeit.co/docs/configuration#introduction/configuration-reference)

现在，我们都完成了，现在您可以运行下面的命令来在 ZEIT 上托管您的 API。

```
now
```

该函数应该成功上传，并且您应该获得一个访问它的链接。

*如果你觉得这篇文章很有帮助，别忘了* ***拍拍*** *和* ***把它*** *分享给你的朋友，谢谢你的时间。*
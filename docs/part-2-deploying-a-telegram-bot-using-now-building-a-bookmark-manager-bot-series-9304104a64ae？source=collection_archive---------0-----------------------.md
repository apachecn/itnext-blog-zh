# [第 2 部分]如何使用 Now 部署电报机器人(构建书签管理器机器人系列)

> 原文：<https://itnext.io/part-2-deploying-a-telegram-bot-using-now-building-a-bookmark-manager-bot-series-9304104a64ae?source=collection_archive---------0----------------------->

![](img/73fe8e3fd972b8e4621b1cae3c22be53.png)

这篇文章是一个系列的一部分，在第一部分[中，我们已经构建了自己的书签管理器 bot，在这一部分中，我们将看看如何部署我们的 bot，以便我们最终可以开始使用它！正如你将看到的，这一步其实很容易！](https://medium.com/p/972bb7198def/edit)

**注意:**尽管本文是系列文章的一部分，但是如果您已经自己构建了一个电报机器人(使用 Node.js ),并且正在考虑如何部署它，那么您也可以跟着做！

## 首先..

我们需要调整我们的`package.json`文件。为了使运行我们的脚本更容易，我们需要包含`scripts`参数并指定我们的*开始*脚本。此外，我们需要指定我们想要使用的节点的版本，为此您还需要包含`engines`参数。让我们调整`package.json`，使它看起来像这样:

```
{
  "name": "bookmark-bot",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "firebase": "^5.0.4",
    "node-telegram-bot-api": "^0.30.0",
    "open-graph-scraper": "^3.3.0"
  },
  "engines": {
    "node": "10.3.0"
  }
}
```

在`start`脚本中，我们告诉 Node 运行`node index.js`，这是我们在上一部分中用来测试 bot 的命令。下次你想测试你的机器人时，你可以简单地使用`yarn start`(如果你使用 Yarn 作为你的包管理器)或者`npm start`(如果你使用 npm)。

要检查您的节点版本，请运行`node -v`(或`node --version`)并将其用作您的引擎。

现在，让我们让机器人运行起来！

# 部署我们的机器人

**网址:**[https://zeit.co/now](https://zeit.co/now)
**价格:**免费，最多 3 个实例

*现在*是 Zeit 提供的一项服务，可以让你轻松部署 Node.js 应用。*现在*是我最喜欢的部署聊天机器人的服务。他们有一个很好的免费层计划，他们的服务非常方便用户。

首先，前往[https://zeit.co/signup](https://zeit.co/signup)创建一个账户。如果您愿意，可以将您的帐户连接到 GitHub 帐户，但这不是必需的。对于这篇文章，我使用的是一个没有连接的帐户。

现在有两种方法可以在你的机器上获得*，你可以下载[桌面 app](https://zeit.co/download) 或者只安装命令行界面。使用这两种方式部署都很容易，桌面应用程序会给你一个漂亮的界面，而不是运行命令。*

## ***使用 now-cli 部署***

*要安装 *Now* ，只需在终端中运行以下命令即可全局安装 *Now* 包:*

```
*yarn global add now// or if you are using npm instead of Yarn
npm install -g now*
```

*超级简单，不是吗？要部署你的机器人，你所要做的就是运行`now`。*

```
*cd your-project-folder
now*
```

*因为这是你第一次使用*现在*你将被要求你的电子邮件，然后你将被要求打开一个电子邮件进行验证。确认电子邮件地址后，再次运行`now`进行部署。*

*如果你使用的是免费账户，会有一个提示说你的日志将会公开，并询问你是否想继续。为了避免将来出现这种提示，您可以运行`now --public`而不是`now`。*

**现在*将生成一个`.now.sh` URL，您可以用它来访问您的应用程序，但是在我们的例子中，它什么也不显示，因为没有前端。*

***恭喜你！你的机器人现在已经部署好了，你可以随时使用它，而不必在本地运行它。***

## *防止冻结状态*

*如果机器人没有接收到任何流量，它将自动进入*冻结*状态。如果发生了这种情况，或者您想防止它发生，您可以运行`now scale your-deployment-url 1`，其中 *your-deployment-url* 是您的部署 url，以`.now.sh`结尾，您可以从您的 zeit.co 仪表板中获得。*

***如果出了问题..你可以检查你的日志，看看哪里出了问题。你可以在[你的仪表盘](https://zeit.co/dashboard/deployments)点击你的应用程序的网址来访问你的日志。***

## *使用 Now 桌面应用程序*

*桌面会让你更容易部署项目，并给你一些额外的功能。一旦你安装了桌面应用程序，你需要做的就是把你的项目文件夹放到应用程序中！*

# *那都是乡亲们！*

*我们的机器人启动并运行，我们现在可以随时保存我们的书签！我们的书签仍然保存在 Firebase 数据库中，这使得实际查找它们有点不方便。在下一部分，我们将使用 Vue.js 创建一个漂亮的界面，这样我们就可以轻松地访问和管理我们的书签！*

***系列索引**
[【第一部分】如何使用 Node.js](/part-1-building-a-bookmark-manager-telegram-bot-node-js-vue-js-972bb7198def)
[【第二部分】如何使用 Now](/part-2-deploying-a-telegram-bot-using-now-building-a-bookmark-manager-bot-series-9304104a64ae)
*部署电报机器人下一部分即将推出！**
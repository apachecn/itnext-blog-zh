# 将 Next.js 应用程序部署到 Firebase 函数

> 原文：<https://itnext.io/deploying-next-js-app-to-firebase-functions-eb473791d79e?source=collection_archive---------0----------------------->

![](img/465efe4813f81520355c0365b175b79e.png)

先说重点:什么是 [Next.js](https://nextjs.org) ？

> 基于 React、Webpack 和 Babel 构建的服务器端通用 JavaScript webapps 的小型框架。

它开箱即用，具有预先配置的功能，如路由、静态优化、服务器端呈现、代码分割、CSS-in-JS 支持、热重装和 API routes à la [Express.js](https://expressjs.com) ，

部署到 [ZEIT Now](https://zeit.co) 非常容易，但我想探索将我的应用程序部署到 Firebase 功能的可能性，以便为数据库、分析、广告和托管提供一个中心共同点。

鉴于 Next.js 和 Firebase 的受欢迎程度，并没有人们想象的那么多来源。我找到了以下几个很棒的:

*   [https://github . com/jthegedus/firebase-GCP-examples/tree/master/functions-nextjs](https://github.com/jthegedus/firebase-gcp-examples/tree/master/functions-nextjs)
*   [https://code burst . io/next-js-on-cloud-functions-for-firebase-with-firebase-hosting-7911465298 F2](https://codeburst.io/next-js-on-cloud-functions-for-firebase-with-firebase-hosting-7911465298f2)
*   [https://github . com/zeit/next . js/tree/canary/examples/with-firebase-hosting](https://github.com/zeit/next.js/tree/canary/examples/with-firebase-hosting)

我之所以想写这篇文章，是因为我想提供一种快速部署现有项目的方法，跳过深入的解释。

## 步骤 1:配置 Firebase

转到 Firebase 控制台，创建一个新项目。请记住，您需要采用 *Blaze* 计划，以便能够从您的云功能加载外部网络资源。

然后打开终端，转到您的项目文件夹，并在其中初始化 Firebase:

只选择*托管*功能，然后继续使用默认值。一旦`firebase`完成其操作，删除`public/index.html`:它会干扰路由。

然后，编辑`firebase.json`到此:

它告诉 Firebase hosting 重写所有对我们将要定义的`nextjs-server`函数的请求。

## 步骤 2:配置服务器代码

首先，让我们安装依赖项:

因此，您可以在`src/server/index.js`中创建服务器代码:

然后，您告诉 Babel 如何通过创建`src/server/.babelrc`文件从`index.js`源生成 Node.js 服务器代码:

## 步骤 3:移动 Next.js 应用程序代码

把你所有的应用程序代码放在`src/client`文件夹里。你还必须移动`next.config.js`和`tsconfig.json`，如果你有的话。

编辑`src/client=next.config.js`以更改分发路径:

更新`package.json`以便在 Firebase 服务器上使用正确的入口点和正确的 Node.js 版本:

## 步骤 4:配置构建脚本

在`package.json`中插入一些有用的脚本来开发你的应用程序:

如果您启动“npm run dev ”,您将获得一个带有热重装的本地部署服务器。

另一批脚本可以专门用于构建:

如果您启动“npm run build ”,您将在 dist/中获得一个产品包。

您可以启动一个 Firebase 本地服务器来调试您的云功能:

如果你启动“npm run serve ”,你将得到一个本地 Firebase 服务器。

最后，将发布版本上传到 Firebase 的部署脚本:

如果您启动“npm 运行部署”,您将构建并上传到 Firebase。

## 笔记

你可以在 [Github](https://github.com/muccy/next-js-to-firebase-article) 上看到整个项目。
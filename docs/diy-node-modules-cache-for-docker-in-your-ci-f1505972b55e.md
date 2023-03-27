# 在配置项中为 DIY node _ modules 缓存

> 原文：<https://itnext.io/diy-node-modules-cache-for-docker-in-your-ci-f1505972b55e?source=collection_archive---------4----------------------->

![](img/513cd6bb8c090dbc18e8036329063575.png)

# 背景

虽然我不是 DevOps 专家，但我使用 CI 工具已经有一段时间了，在我的职业生涯中，我一直致力于在我的工作流和我构建的产品/web 应用程序中实现最佳性能和效率。

虽然这绝不是一个完美的解决方案，老实说，它可能不是最好的，但在我的测试中，它确实工作得很好。

# 问题

我们今天构建的大多数应用程序都尽可能地利用了自动化工作流的优势。从我们的测试到部署，以及最近在某种程度上我们的代码编写…

我看到的一个问题是，当涉及到为基于 JS 的 web 应用程序构建图像时，比如 Vue 和 React，就我个人而言，我已经与 React 合作多年，以前在 Azure 上工作时，我们的 web 应用程序的构建时间大约为 12 分钟，最近我一直在与 Google Cloud 合作，我看到的时间大约为 10 分钟。

现在，这可能与 CI 工具无关，而是与应用程序的规模和复杂性有关，因为大部分时间是由一个常见步骤`npm install`占用的，并且考虑到这是一个在线操作，许多因素都会影响该步骤需要多长时间。

# 解决方案(？)

最近在遛狗的时候，我有了一个为 node 创建自己的缓存容器的疯狂想法，我非常喜欢使用多阶段构建，并且刚刚更新了项目以解决这个问题，在更新之前，我们将构建的基本节点映像提升到大约 1.6GB，切换到多阶段，并且将 alpine 容器提升到 140mb。

虽然这个想法可能不太可行，或者至少对新的项目有益，但是旧的更成熟和稳定的项目可以看到这个想法的合理改进。

首先创建一个缓存映像，这是一个用所需的基本节点映像构建的简单映像，只需安装节点模块，然后将它们复制到 alpine 映像，我们就完成了。

```
FROM node:18 as buildCOPY package*.json ./RUN npm install --no-audit --progress=falseFROM alpine as releaseCOPY --from=build /node_modules ./node_modules
```

这个映像成为我们的“缓存”映像，当在一个更稳定的项目中时，可以每周甚至每月重建一次，因为这些包是相当稳定的。

从那以后，您只需将它作为构建阶段的一部分，正如您将从第一行`FROM node-cache as cache`中看到的，其中`node-cache`是您提供给映像的任何名称，它可能需要包含对容器注册表的引用。

不要忘记，在 CI 上使用它之前，需要构建缓存映像并将其推送到容器注册中心。

```
FROM node-cache as cache
 ​
 *# Build Stage*
 FROM node:18 as build
 COPY --from=cache /node_modules ./node_modules
 COPY package*.json ./
 COPY . ./
 RUN npm install --no-audit --progress=false --prefer-offline
 RUN npm run build
 ​
 *# Release stage*
 FROM node:18-alpine as release
 *# Copy files over from build stage*
 COPY --from=build /build ./build
 COPY --from=build package*.json ./
 COPY --from=build /server.js ./server.js
 ​
 RUN npm install --only=production
 ​
 CMD [ "npm", "run", "prod" ]
```

# 构建阶段

这是我们利用缓存的地方，在这一步中，我们使用了`node-18`映像，这是构建原始缓存映像时使用的同一映像，其中的关键部分是第`COPY --from=cache /node_modules ./node_modules`行，这一行将节点模块文件夹从我们的缓存复制到我们的构建阶段。

这样做意味着我们现在可以在一个类似的环境中，在我们的构建阶段访问相关的安装包。然后，我们复制包文件，特别是目录中的剩余文件。

还应该注意，您的项目应该包含一个`dockerignore`文件，并且应该在该文件中指定`node_modules`，否则`COPY . ./`步骤将覆盖容器中的 node_modules 文件夹。

接下来我们运行`npm install`步骤，额外的参数可以加快一点速度，但也指定 npm 需要在联机检查之前进行本地检查，这将确保只有添加或升级的包，因为缓存映像最后一次构建后才会被下载。

# 发布阶段

如果我们在发布阶段再往下看一点，前几个步骤是复制构建目录(我们编译的 web 应用程序)、package.json 文件以及`server.js`。

`server.js`是一个小的`express`服务器，允许我们从 web 访问 docker 容器中的应用程序。

```
 const http = require('http');
 const Express = require("express");
 const path = require('path');
 ​
 const port = process.env.PORT || 7010;
 ​
 const app = Express();
 const server = http.createServer(app);
 ​
 server.listen(port, function () {
     console.log(`Server listening on port ${port}`);
 });
 ​
 app.get('/', function(req, res) {
     res.sendFile(path.join(__dirname, "build", "index.html"));
 });
 ​
 app.use(Express.static(path.join(__dirname, "build")));
 ​
 module.exports = server;
```

倒数第二个命令是`RUN npm install --only=production`，包含的标志指示节点只安装在`package.json`的“dependencies”键中列出的包，忽略“devDependencies”中的任何内容，因此对于这个特定的例子，只有`express`被安装到`alpine`映像中。

为了更好地工作，你需要确保你的`package.json`被正确地分割，以确保只有必需的包被列为依赖项，其余的都应该是 devDependencies。

在我的本地测试中，这导致构建时间缩短了 60%以上，平均构建时间从更新前的至少 150 秒缩短到更新后的不到 50 秒。

在管道中，我们看到构建时间缩短了 40–45 %,这是因为需要先下载图像。

对于那些想要进一步了解，甚至测试这个解决方案的人，我已经使用标准的 [CRA](http://create-react-app.dev/) 创建了一个[回购](https://github.com/RemeJuan/docker-cache-example)，在这里你会找到类似的 Docker 文件，你可以按照自述文件中的步骤开始工作。

我希望您对此感兴趣，如果您有任何问题、评论或改进，请随时发表评论。如果你有更好的解决方案，也可以分享给大家:微笑:

如果你喜欢，一个喜欢会很棒。

感谢阅读。

[](/up-your-testing-game-ae40cb5d4449) [## 升级您的测试游戏

### 今天我们来看看 Flutter 测试提供的一个很棒的工具。

itnext.io](/up-your-testing-game-ae40cb5d4449) [](/widget-testing-dealing-with-renderflex-overflow-errors-9488f9cf9a29) [## 小部件测试:处理 Renderflex 溢出错误

### 在单元测试中处理“一个 RenderFlex 被…溢出”的简单方法…

itnext.io](/widget-testing-dealing-with-renderflex-overflow-errors-9488f9cf9a29) 

照片由 [Timelab Pro](https://unsplash.com/@timelabpro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/container?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

*最初发布于 2022 年 8 月 1 日*[*https://remelehane . dev*](https://remelehane.dev/posts/diy-node-cache-for-docker-ci/)*。*
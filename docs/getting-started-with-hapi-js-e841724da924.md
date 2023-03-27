# 哈比神 JS 入门

> 原文：<https://itnext.io/getting-started-with-hapi-js-e841724da924?source=collection_archive---------1----------------------->

## 第 1 部分:启动服务器和基本路由

![](img/8910495eb4137d1ee442ae96fee0bf94.png)

我知道似乎 Node 的唯一框架是 [Express](https://expressjs.com/) ，但是我认为让新程序员尝试一下 [hapi](https://hapijs.com/) 是值得的。哈比神非常轻量级，围绕一个插件系统构建，保持一切模块化。另外，它的[文档很棒](https://hapijs.com/api) ，而且它们有一个不错的小[教程部分](https://hapijs.com/tutorials)。在这个系列中，我们将回顾基础知识，从启动服务器和添加一些路由开始。

> 这里有一个 GitHub 可以看到完成的项目

## 哈比神 16 对 17

关于你应该使用哪个版本的快速提示。从 16 到 17，框架发生了变化，从使用回调到异步/等待。本教程，以及大部分专业世界，将使用 hapi 17 及以上。如果你正在处理需要更新的旧代码，我强烈推荐使用[这篇 hapi 转换文章](https://futurestud.io/tutorials/hapi-v17-upgrade-guide-your-move-to-async-await)作为指南。那也是一个很棒的网站，我喜欢浏览他们的教程。

# 入门:安装

跑步，你就可以出发了。您还应该全局安装 [nodemon](https://nodemon.io/) 。这将负责在文件更改时重启服务器。或者，你可以克隆我的 [GitHub](https://github.com/MostlyFocusedMike/hapi-notes-1) ，它使用 [npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b) 的魔力来运行一个本地安装的 nodemon。

# 使用 server.js 文件启动和运行

到目前为止，我们的`server.js`文件是这样的:

```
const Hapi = require(‘hapi’); const server = new Hapi.Server({ 
    host: ‘localhost’, 
    port: 3101, 
}); const launch = async () => {
    try { 
        await server.start(); 
    } catch (err) { 
        console.error(err); 
        process.exit(1); 
    }; 
    console.log(`Server running at ${server.info.uri}`); 
}launch();
```

如果你运行`npm start`并转到 [http://localhost:3101/](http://localhost:3101/) ，你会看到我们的服务器*已经启动并正在运行*，你只是得到一个 404，因为我们还没有做一个路由处理器。但在此之前，我们先来谈谈这两个主要部分:

## 服务器方法及其配置对象

如您所见，我们通过调用 hapi 的`[Server method](https://hapijs.com/api#server)`来创建我们的服务器。它接受一个[配置对象](https://hapijs.com/api#server.options)并返回一个我们可以使用的`server`对象。这么多东西*可以*放入 config 对象，但是现在你需要的是一个主机和端口，一个字符串和一个整数。此外，如果你只是闲逛，`‘localhost’`很好，但是如果你使用 Docker，把[设置为`‘0.0.0.0’`会让你自己更加烦恼。](https://stackoverflow.com/questions/47025647/localhost-vs-0-0-0-0-with-docker-on-mac-os)

## “发射”方法

哈比神在名为`start()`的服务器上有一个内置的异步方法，你会认为这就是你所需要的。但是很快，我们将在服务器真正启动之前对它做很多事情，把它们都放在一个地方会很有帮助。这就是为什么创建自己的函数很常见。我喜欢把我的命名为“发射”，但你可以叫它什么。它还将通过将错误打印到控制台并退出节点进程来处理任何错误。

在我们定义了函数之后，我们在文件的末尾调用它，这最终启动了我们的服务器。现在，让我们添加一些路线！

# 添加基本路线

您通过使用服务器对象的`[route()](https://hapijs.com/api#-serverrouteroute)` [方法](https://hapijs.com/api#-serverrouteroute)来添加路由。该方法采用配置对象或对象数组。下面是最基本的路线:

```
server.route({   
    method: 'GET',   
    path: '/',   
    handler: (request, h) => {     
        return 'I am the home route'   
    } 
});
```

现在，如果你进入 [http://localhost:3101/](http://localhost:3101/) ，屏幕上会显示一个字符串，而不是 404。进步！至于配置对象，我们现在给它最少三个属性:`method`、`path`和`handler`。

## 方法

这是触发路由的 http 动词，它可以是一个字符串或字符串数组(更常见的是字符串)。大小写无所谓，但是我个人觉得全大写看起来比较整洁。

## 小路

这是您将放入浏览器的实际路由字符串路径，您将在此指定您的[参数](https://hapijs.com/api#path-parameters)。

## 处理函数

这是一个决定什么被发送回客户端的函数。它既可以访问初始的`[request](https://hapijs.com/api#request)` [对象](https://hapijs.com/api#request)，也可以访问名为`[h response toolkit](https://hapijs.com/api#response-toolkit)`的对象。它们非常适合更复杂的响应，但是只需要发送一个简单的`return`就可以了。json 或者 strings。

# 在上下文中向服务器添加路由

所有这些加在一起，这就是你最终的`server.js`文件的样子:

```
const Hapi = require('hapi');// create a server with a host and port
const server = new Hapi.Server({
    host: 'localhost',
    port: 3101
});// add each route
server.route([
    {
        method: 'GET',
        path: '/',
        handler: (request, h) => {
            return 'I am the home route';
        },
    },
    {
        method: 'GET',
        path: '/example',
        handler: (request, h) => {
            return { msg: 'I am json' };
        },
    }
]);// define server start function
const launch = async () => {
    try {
        await server.start(); 
    } catch (err) {
        console.error(err);
        process.exit(1);
    };
    console.log(`Server running at ${server.info.uri}`);
}// start your server
launch();
```

恭喜你！你已经用 hapi 建立了一个微型服务器！我们将在下一期讨论[路径参数](https://hapijs.com/api#path-parameters)以及如何使用[响应工具包](https://hapijs.com/api#response-toolkit)，但我鼓励您同时查看教程和文档。

大家编码快乐，

迈克
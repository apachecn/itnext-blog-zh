# 通过 MERN 堆栈使用 Socket.io

> 原文：<https://itnext.io/using-socket-io-with-a-mern-stack-93f4245e86e7?source=collection_archive---------2----------------------->

在这篇文章中，我将为你提供一套在 MERN 堆栈上设置 [Socket.io](https://socket.io/) 的说明。这个想法是你可以用它作为你未来项目的起点。

> 插座。IO 支持基于事件的实时双向通信。

Socket.io 是在网站中创建实时事件的常用工具。这是一个用户订阅的想法，你可以推送到他们的应用程序实例。这方面的例子包括实时消息、实时反馈、即时更新数据等等。

## 设置

像我的大多数 MERN 项目一样，我们将从设置[这个样板代码](https://github.com/keithweaver/MERN-boilerplate)开始。下载或者分叉。您需要安装 MongoDB、Node.js 等。在文本编辑器中打开代码，并将`/config/config.example.js`重命名为`/config/config.js`。您还需要为开发环境创建一个到 MongoDB 的连接，因此在该文件中将其更改为:

```
module.exports = {
  db: 'mongodb://username:password@url:port/db',
  db_dev: 'mongodb://localhost:27017/db_name_demo',
};
```

`db_name_demo`是我的数据库名。设置样板代码的下一步是安装所有需要的包。打开您的终端，导航到文件夹并运行:

```
npm install
```

几分钟后，这应该完成。运行:

```
npm run start:dev
```

如果您在浏览器中打开 [http://localhost:8080](http://localhost:8080) ，您应该会看到下图。样板代码的设置已经完成。

![](img/75fd8635859bb0ff94b342d6c3853832.png)

## 添加 Socket.io 包

您需要两个包来使用 Socket.io，所以运行:

```
npm install socket.io --save
npm install socket.io-client --save
```

很明显，一个包是前端的，另一个是后端的。如果你不像我一样把所有东西都放在一个仓库里，你需要把它安装在特定的地方。

打开`/server/server.js`，我们将添加所需的 Socket.io 代码。在这些线下面:

```
const app = express();
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
```

你会把:

```
// Socket.io
const http = require('http').Server(app);
const io = require('socket.io')(http);
io.on('connection', function(socket){
  console.log('a user connected');
  socket.on('disconnect', function(){
    console.log('User Disconnected');
  });socket.on('example_message', function(msg){
    console.log('message: ' + msg);
  });
});
io.listen(8000);
```

在上面的代码片段中，我们可以看到三个事件:`connection`、`disconnect`、&一个名为`example_message`的自定义事件。如果您有许多自定义消息，我建议将它们移动到类似于 routes 的自己的文件中。

当浏览器打开我们的应用程序时，就会调用连接。断开就是 app 关闭的时候。我们将设置一个按钮来触发自定义消息。

## 前端的 Socket.io

在前端，我们将在 React 中添加一个按钮来触发 Socket.io 客户端，该客户端将发送带有数据的自定义消息。打开`/client/app/components/Home/Home.js`，你会看到我们项目的索引页面(加载的第一个东西)。在这里，我们将导入客户端，因此在导入下面:

```
import React, { Component } from 'react';
import 'whatwg-fetch';
import openSocket from 'socket.io-client';
const socket = openSocket('[http://localhost:8000'](http://localhost:8000'));
```

我们还将添加一个被绑定的函数。在构造函数中，将这一行添加到底部:

```
this.sendSocketIO = this.sendSocketIO.bind(this);
```

渲染之上(在组件类内部，但在渲染外部):

```
sendSocketIO() {
    socket.emit('example_message', 'demo');
}
```

在新计数器按钮下方的渲染中，添加一个新按钮:

```
<div>
  <button onClick={this.sendSocketIO}>Send Socket.io</button>
</div>
```

就是这样。前端设置完毕。如果你在添加前端文件时迷路了，这里有[一个完整文件的要点](https://gist.github.com/keithweaver/06fb43df85f956eccaa7ef18b277ed00)。

让我提一下，`socket.emit`是我们用来向后端发送消息的。`demo`是字符串数据，所以如果你想传递更多，它可以是一个字符串化的对象。`example_message`与后台`.on`消息完全相同。

## 结果

现在再运行一次:

```
npm run start:dev
```

您应该会看到:

![](img/0b8db768047fa70303cfaf4055abb8fe.png)

如果您查看您的终端窗口，请单击发送 Socket.io 按钮，然后再查看您的终端窗口。您应该看到:

![](img/120d394f2e3995bfb4ae2e14818d8b48.png)

这就是 MERN 的设置。将来我可能会发布更多的帖子，用 Socket.io 做更多的事情。

请给我一个👏如果你喜欢这篇文章。我尽可能多地发布类似的文章，并且每周发布一个 Youtube 视频。 [*原帖*](https://medium.com/@Keithweaver_/using-socket-io-with-a-mern-stack-2a7049f94b85) *。*
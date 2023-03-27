# [阅读 Koa]第二天—阅读并理解 Koa 核心

> 原文：<https://itnext.io/readingkoa-day-two-read-and-understand-koa-core-a6afd24e012c?source=collection_archive---------4----------------------->

![](img/8ed2c4e799b2bb9557b0ce77802eea08.png)

# 介绍

我们将阅读来自 **Koa** 的结构和所有源代码——一个带有异步中间件的新 web 框架。如果你不知道 Koa 的中间件是如何工作的，你可以先看看这篇文章:

[](https://medium.com/@alickwong/how-koa-middleware-works-f4386b5573c) [## Koa 中间件如何工作

### Koa 中的中间件不同于 Express，Koa 使用洋葱模型。这个惊人的框架 Koa 只包含四个…

medium.com](https://medium.com/@alickwong/how-koa-middleware-works-f4386b5573c) 

我们将涵盖 Koa 中的所有文件，其中仅包含四个文件(令人惊讶):

*   应用程序. js
*   上下文. js
*   请求. js
*   响应. js

# 文件 1:应用程序文件(application.js)

这是 Koa 的切入点。这是我们初始化 koa 服务器的方法:

```
const Koa = require('koa');
const app = new Koa();
app.listen(3000);
```

`new Koa()`实际实例化一个新的应用对象，这里是`application.js`中的构造函数:

```
module.exports = class Application extends Emitter {
  constructor() {
    super(); this.proxy = false;
    this.middleware = [];
    this.subdomainOffset = 2;
    this.env = process.env.NODE_ENV || 'development';
    this.context = Object.create(context); // from File 2: context.js
    this.request = Object.create(request); // from File 3: request.js
    this.response = Object.create(response); // from File 4: response.js
    if (util.inspect.custom) {
      this[util.inspect.custom] = this.inspect;
    }
  }
```

## 关于发射器

`new Koa()`初始化一个`Application`对象，该对象扩展了`Emitter`。在扩展了`Emitter`类之后，它将公开一个`eventEmitterObject.on()`函数，该函数允许将一个或多个函数附加到由对象发出的命名事件上。这意味着我们可以像这样将事件附加到 Koa:

```
const app = new Koa();app.on('event', (data) => {
  console.log('an event occurred! ' + data); // an event occurred! 123
});
app.emit('event', 123);
```

当 EventEmitter 对象发出一个事件时，所有附加到该特定事件的函数都会被同步调用。被调用的侦听器返回的任何值都将被忽略并将被丢弃。

[事件| Node.js v12.4.0 文档](https://nodejs.org/api/events.html)

## 关于 Object.create()

我们还可以在构造函数中看到`Object.create()`，它只是创建了一个新的对象，使用一个现有的对象作为新创建对象的原型。以下是一些例子:

```
const person = {
  isHuman: false,
  printIntroduction: function () {
    console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
  }
};const me = Object.create(person);me.name = "Matthew"; // "name" is a property set on "me", but not on "person"
me.isHuman = true; // inherited properties can be overwrittenme.printIntroduction(); // "My name is Matthew. Am I human? true"
```

[Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

## 启动服务器

说完了`new Koa()`，我们可以调查一下`app.listen(300)`。如果我们使用`app.listen(3000);`启动服务器，下面的代码将被执行:

```
listen(...args) {
  debug('listen');
  // Step 1: call callback(), create a http server
  const server = http.createServer(this.callback());
  // Step 5: http server created, start listen to port
  return server.listen(...args);
}callback() {
  // Step 2: prepare middlewares
  const fn = compose(this.middleware); if (!this.listenerCount('error')) this.on('error', this.onerror); const handleRequest = (req, res) => {
    // Step 3: createContext, we will talk more about this
    const ctx = this.createContext(req, res);
    // Step 4: handleRequest, we will talk more about this
    return this.handleRequest(ctx, fn);
  }; return handleRequest;
}
```

如果您想知道如何在没有 Koa 的情况下启动 http 服务器，这里有一个我们直接使用 http 包创建服务器的正常方法:

```
const http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.write('Hello World!');
  res.end();
}).listen(8080);
```

## 关于 createContext(向代码添加了注释)

```
createContext(req, res) {
    // create new object by using this.context as prototype
    const context = Object.create(this.context);
    // create new object, make sure request and response object can be access inside context object
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    // make sure context, request, response, app object can access each other
    context.app = request.app = response.app = this;
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.ctx = response.ctx = context;
    // again make sure response object can be accessed inside request object
    request.response = response;
    response.request = request;
    context.originalUrl = request.originalUrl = req.url;
    context.state = {};
    // return context object, this is the ctx object we can use in middleware
    return context;
  }
```

## 关于 handleRequest(在代码中添加了注释)

```
handleRequest(ctx, fnMiddleware) {
    const res = ctx.res;
    res.statusCode = 404;
    const onerror = err => ctx.onerror(err);
    // when all middleware have been finish, call respond()
    const handleResponse = () => respond(ctx);
    // if res from http package throw error, call onerror function
    onFinished(res, onerror);
    // middleware part have been covered in the last post, we dont discuss here
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
```

## 响应(在代码中添加了注释)

```
// Just attach ctx.body to res, not really special here
function respond(ctx) {
  if (false === ctx.respond) return; if (!ctx.writable) return; const res = ctx.res;
  let body = ctx.body;
  const code = ctx.status; // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null;
    return res.end();
  } if ('HEAD' == ctx.method) {
    if (!res.headersSent && isJSON(body)) {
      ctx.length = Buffer.byteLength(JSON.stringify(body));
    }
    return res.end();
  } // if body does not exist, return 
  if (null == body) {
    if (ctx.req.httpVersionMajor >= 2) {
      body = String(code);
    } else {
      body = ctx.message || String(code);
    }
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  } // If body is buffer, return body directly
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res); // JSON encrypt the body
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}
```

# 文件 2:上下文(context.js)

这个文件使用了一个名为 delegate 的包来导出 context.js 中的方法，我已经写了一篇文章来理解这个包是如何工作的:

[](https://medium.com/@alickwong/npm-package-study-delegates-6-million-weekly-download-a-single-js-file-package-6d53a2795d15) [## NPM 软件包研究—代表(每周 600 万次下载，一个 js 文件包)

### 介绍

medium.com](https://medium.com/@alickwong/npm-package-study-delegates-6-million-weekly-download-a-single-js-file-package-6d53a2795d15) 

下面是`context.js`的底部:

```
delegate(proto, 'response')
  .method('attachment')
  .method('redirect')
.....delegate(proto, 'request')
  .method('acceptsLanguages')
  .method('acceptsEncodings')
  .access('querystring')
```

这意味着当你访问`ctx.querystring`时，它实际上是在访问`ctx.request.querystring`，而`ctx.request`是在`createContext`被调用时分配的。

所以这个`delegate`主要是通过在中间件中使用`ctx`让你轻松访问`response`和`request`里面的方法(因为所有的中间件都有`ctx`作为输入)。这里有一个在 [day one](https://medium.com/@alickwong/how-koa-middleware-works-f4386b5573c) 帖子中提到的中间件的例子:

```
// Here is the ctx
app.use(async (ctx, next) => {
  console.log(3);
  ctx.body = 'Hello World';
  await next();
  console.log(4);
});
```

# 文件 3:请求(request.js)

这是`ctx.request`的原型。这个文件主要让你访问来自`this.req`的 http 请求的所有数据，比如头、ip、主机、url 等....以下是一些例子:

```
get(field) {
    const req = this.req;
    switch (field = field.toLowerCase()) {
      case 'referer':
      case 'referrer':
        return req.headers.referrer || req.headers.referer || '';
      default:
        return req.headers[field] || '';
    }
  },
```

# 文件 4:响应(response.js)

这是`ctx.response`的原型。这个文件主要让你访问`this.res`中的数据，比如响应头和体，下面是部分源代码:

```
set(field, val) {
    if (this.headerSent) return; if (2 == arguments.length) {
      if (Array.isArray(val)) val = val.map(v => typeof v === 'string' ? v : String(v));
      else if (typeof val !== 'string') val = String(val);
      this.res.setHeader(field, val);
    } else {
      for (const key in field) {
        this.set(key, field[key]);
      }
    }
  },
```

# 顺便说一下

感谢阅读！如果这篇文章可以帮助你理解 Koa，请给我一些掌声=]这对我是一个很大的支持。

# 参考

[Koa—node . js 的下一代 web 框架](https://koajs.com/#introduction)
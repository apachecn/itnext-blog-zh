# 用 RxJS 处理 WebSocket 错误

> 原文：<https://itnext.io/websocket-error-handling-with-rxjs-17125c6f2159?source=collection_archive---------0----------------------->

我爱 RxJS！这篇博文展示了如何使用 RxJS websocket 客户端在出错时自动重新连接。

**服务器**

让我们创建一个简单的 websocket 服务器，每 10 条消息终止一次连接。这是使用 [ws](https://github.com/websockets/ws) 包的简单节点服务器。

```
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 7777 });

wss.on('connection', function connection(ws) {
  let count = 0;
  ws.on('message', function incoming(message) {
    if (count === 10) {
      ws.terminate();
    } else {
      ws.send(message);
      count++;
    }
  });
});
```

**RxJS 客户端**

所以现在是时候看看 RxJS 提供了什么来处理 websockets 了。RxJS 为我们提供了 WebSocket object，它是浏览器提供的 w3c 兼容 webSocket 对象的包装器。这是:

```
[webSocket](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#static-method-webSocket)(urlConfigOrSource: string | WebSocketSubjectConfig): [WebSocketSubject](http://reactivex.io/rxjs/class/es6/observable/dom/WebSocketSubject.js~WebSocketSubject.html)
```

我们可以用一行简单的代码创建 websocket 连接:

```
const subject = *webSocket*('ws://localhost:7777');
```

创建连接后，为了向服务器发送消息，只需像这样对主题执行 next()即可:

```
subject.next('hi there');
```

要从 websocket 接收消息，我们可以订阅主题:

```
**const** subscription = subject
  .asObservable()
  .subscribe(data => observer.next(data), 
             error => observer.error(error), 
             () => observer.complete());
```

所以我们有了创建 websocket 所需的所有块。Observable.create 用于将逻辑包装成可观察对象。这里的示例代码还模拟了使用 setInterval 每秒发送一次消息。

```
const *createWebSocket* = uri => {
  return Observable.*create*(observer => {
    try {
      const subject = *webSocket*(uri);

      const handler = *setInterval*(() => {
        subject.next('hi there');
      }, 1000);

      const subscription = subject.asObservable()
        .subscribe(data => 
                  observer.next(data), 
                  error => observer.error(error), 
                  () => observer.complete());

      return () => {
        *clearInterval*(handler);
        if (!subscription.closed) {
          subscription.unsubscribe();
        }
      };
    } catch (error) {
      observer.error(error);
    }
  });
};
```

**使用 retryWhen 的错误处理**

现在 RxJS websocket 已经可以使用了。因为这只是一个可观察值，所以我们可以使用任何处理错误的 RxJS 操作符。 [**retryWhen**](https://www.learnrxjs.io/operators/error_handling/retrywhen.html) 对我们来说可能是个不错的选择——在服务器终止 websocket 连接的情况下，重新尝试打开它。

下面是使用 websocket observable 并用 [retryWhen](https://www.learnrxjs.io/operators/error_handling/retrywhen.html) 处理错误的代码。

```
*createWebSocket*('ws://localhost:7777')
  .pipe(
    *retryWhen*(errors =>
      errors.pipe(
        *tap*(err => {
          *console*.error('Got error', err);
        }),
        *delay*(1000)
      )
    )
  )
  .subscribe(data => *console*.log(data), err => *console*.error(err));
```
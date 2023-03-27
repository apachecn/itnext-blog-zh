# 负载平衡器中节点的正常启动和关闭

> 原文：<https://itnext.io/nodejs-graceful-start-shutdown-3ed16418ede3?source=collection_archive---------4----------------------->

![](img/401d474463ebf145c714f5ef67fdd9b3.png)

为了避免部署、关闭、启动和重新启动应用程序时出现问题，确保现有请求完成并在服务器关闭前防止更多请求到达服务器非常重要。这称为正常关机。

同样重要的是，在接收到客户机请求之前，要确保服务器能够向数据库发出请求，以便能够满足这些请求，否则可能会发生错误和竞争情况，这就是我们所说的平稳开始。

## 优雅的开始

在接受请求之前，您的应用程序必须成功地建立与数据库的连接。

1.  App 启动 *(npm 启动)*
2.  应用程序打开数据库连接
3.  应用程序监听端口
4.  应用程序告诉负载平衡器它已准备好接受请求

让我们看看如何在代码中实现这一点。

```
const server = http.createServer(app)
mongoose.connect(process.env.MONGO_URL)
  .then(db => {
    console.log('connected to mongo successfully')
    server.listen(port, err => {
      if (err) {
        console.log('something bad happened', err)
      } else {
        process.send = process.send || function () { }
        process.send('ready')
        console.log(`server is listening on ${port}`)
      }
    })
  })
  .catch(err => console.log('Mongo connection error', err))
```

## 正常关机

在正常关闭时，您的应用程序必须经过 5 个步骤:

1.  收到停止的通知
2.  要求负载平衡器停止接收请求
3.  完成所有正在进行的请求
4.  释放所有资源(数据库、队列……)
5.  出口

让我们看看这在代码中是什么样子的

```
// If the Node process ends, close the Mongoose connection
const sigs = ['SIGINT', 'SIGTERM', 'SIGQUIT']
sigs.forEach(sig => {
  process.on(sig, () => {
    // Stops the server from accepting new connections and finishes existing connections.
    server.close(function (err) {
      if (err) {
        console.error(err)
        process.exit(1)
      }
      // close your database connection and exit with success 
      // for example with mongoose
      mongoose.connection.close(function () {
        console.log('Mongoose connection disconnected')
        process.exit(0)
      })
    })
  })
})
```
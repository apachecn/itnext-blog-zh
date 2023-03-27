# 用 Go 构建(Web) HTTP 服务器

> 原文：<https://itnext.io/building-a-web-http-server-with-go-6554029b4079?source=collection_archive---------2----------------------->

![](img/722275fa55ca481e79dcbe2b2f65d37f.png)

Go 是一种通用编程语言，使用清晰的语法和高级功能进行编译和静态类型化。它的性能和设计使它成为金融科技公司最喜欢的语言。尽管有这么多好处，Go lang 对整个社区的许多开发者来说仍然是陌生的，所以我们将在这里尝试改变这一点。

如果你还没有在围棋上“破冰”，请花 5 分钟来演示围棋的简单和容易。你不会后悔的。

# 假设 Docker 中有一个环境

我们将从以前的帖子中获取一个已经准备好的环境，因此如果您还没有阅读它，请参考它:

[](https://medium.com/@rrgarciach/bootstrapping-a-go-application-with-docker-47f1d9071a2a) [## 用 Docker 引导 Go 应用程序

### 如果您正在使用 Go 创建一个 Web 应用程序(针对 HTTP 或其他类型的服务),那么您可能…

medium.com](https://medium.com/@rrgarciach/bootstrapping-a-go-application-with-docker-47f1d9071a2a) 

# 规格

我们的 HTTP 服务器必须能够完成三个关键任务:

1.  处理动态请求:处理来自任何类型客户端(用户或其他服务)的符合 HTTP 标准的传入请求。
2.  服务静态资产:向客户端(通常是 Web 浏览器)提供 Javascript 文件、CSS 文件和图像。
3.  接受连接:HTTP 服务器应该监听特定端口上的传入连接。

也就是说，让我们从代码开始。

# 主文件

为了遵循 Go 惯例，我们将创建一个`main.go`文件，声明“主”包，导入我们将需要的两个必需的包(`fmt`和`net/http`)，并声明标准的`main`方法，这将是我们的应用程序的入口点:

```
package mainimport (
 "fmt"
 "net/http"
)func main() {
}
```

我们所有的业务逻辑都将放入`main`功能中。现在让我们开始实施我们的规范。

# 处理动态请求

我们将依赖一个名为`net/http`的 Go 核心库/包，它包含了我们动态处理请求所需的所有功能。首先要做的是使用`http.HandleFunc`函数为我们想要的每条路径注册一个新的处理程序。该函数接受两个参数:第一个参数是表示匹配路径的字符串，第二个参数是将被执行来处理请求的函数。

```
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "This is a website server by a Go HTTP server.")
 })
```

这是所有 REST API 框架中非常常见的模式。请注意所提供的函数作为第二个参数是如何接收两个服务作为参数的(通过依赖注入)；一个处理响应(`http.ResponseWriter`)，另一个(`http.Request`)从请求中获取任何数据或值。对于第二个服务，我们可以用`r.URL.Query().Get("keyword")`读取 GET 参数，或者用`r.Body`在请求体中发送参数。

# 服务静态资产

接下来要做的是提供静态资产，通常是 Javascript 或 CSS 文件以及图像。为此，我们将使用`net/http`包中的`http.FileServer`,并通过向其传递要提供的文件所在的目录，将它指向一个 URL 路径:

```
fs := http.FileServer(http.Dir("static/"))
```

现在变量`fs`保存了静态资产内部服务器的实例，所以现在我们只需要将它映射到一个 URL 路径，就像我们之前处理动态请求一样:

```
http.Handle("/static/", http.StripPrefix("/static/", fs))
```

请注意，我们使用了一个额外的方法`http.StripPrefix`来去除部分 URL 路径，以便正确地从系统中提供文件。

# 接受连接

我们就要完成了，这里要做的最后一部分是监听和接受来自网络/互联网上的客户端的连接。为此，我们将再次依赖 Go 的内置`net/http`包:

```
http.ListenAndServe(":3001", nil)
```

通过这种方式，我们指示我们 Go 应用程序监听端口 3001 上的任何传入请求(`gin`包默认使用这个服务器来监听)。

# 试试看！

现在运行 docker 来启动我们的服务器容器:

```
docker-compose up
```

一旦服务器开始运行，转到浏览器并导航到`localhost:3030`。你会看到一个文本“这是一个网站服务器由一个去 HTTP 服务器。”这意味着您的服务器已经启动并正常运行。

# 其他路线

最后，让我们添加一条额外的路线，看看会发生什么:

```
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello World! I'm a HTTP server!")
 })
```

正如你所看到的，这里我们添加了一个稍微大一点的路径，而不仅仅是“/”，然后提供一个不同的文本。切记在`http.ListenAndServe(“:3001”, nil)`之前注册该功能。

添加完这段代码后，在容器的 stdout 控制台中等待自动重新加载完成:

```
go-expense-api    | [gin] Building...
go-expense-api    | [gin] Build finished
```

现在转到您的浏览器，现在导航到`localhost:3030/hello`以查看“hello world”。

# 代码

有了所有这些，我们得到的 HTTP 服务器应用程序项目将如下所示:

# 摘要

在这个版本中，我们通过创建一个能够处理动态请求、服务静态资产和监听来自客户端的传入请求的超级最小(但功能齐全)HTTP 服务器来“打破僵局”。从这一点来看，很容易理解围棋是如何发展的。

你现在可能意识到使用构建的包`net/http`来声明和映射路由是多么容易，然而我们将在接下来的文章中看到如何使用一个更加容易和流行的包`gorilla/mux`来创建路由。所以请保持联系，在[媒体](https://medium.com/@rrgarciach)、 [Github](https://github.com/rrgarciach) 或[推特](https://twitter.com/rrgarciach)上关注我，以便从我这里接收更新和新闻。你也可以访问我的[网络博客空间开发者猫](http://spacedevelopercat.wordpress.com)来找到我的这篇和其他帖子。
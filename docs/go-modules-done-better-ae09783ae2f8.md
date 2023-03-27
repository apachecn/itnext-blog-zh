# 围棋模块做得更好

> 原文：<https://itnext.io/go-modules-done-better-ae09783ae2f8?source=collection_archive---------3----------------------->

![](img/c10ff6ceec54067ae1f52bd17fd970df.png)

> 我之前的文章[现实生活中的围棋模块](https://medium.com/@ckeyes88/go-modules-in-real-life-87a21fb4d8aa)的后续

一年多前，我发表了一篇文章，详细介绍了我和我在[的团队如何使用 go 模块建立我们的项目。我们使用了一个多模块 repo，并使用了`replace`指令将 Go 工具链指向相对路径模块。如果你对我们是如何做到的感兴趣，你可以回去看看上面链接的那个帖子。](https://helpfulhuman.com)

## 我们最初的方法

如果你读过我的前一篇文章，你会记得我们的项目由一个不断增长的 webhook 处理程序集合组成。这些 webhooks 都是单独的 AWS Lambda 函数，作为独立的可执行文件。他们还共享了许多功能，因为他们都来自同一个应用程序。因此，我们包含了一些共享数据结构、数据库访问库等。在这个项目中，我们建立了一个多模块存储库。我们在`go.mod`文件中使用 Go 的`replace`指令来告诉任何模块在哪里可以找到本地依赖。

这使得我们可以在不同的 webhooks 上共享模块。它还允许我们将所有代码保存在同一个存储库中。我们不必跨 7 个以上的 webhooks 复制代码，也不必为每个 web hooks 建立单独的 CI/CD 管道。我们可以根据你认为微服务应该有多微来讨论最后一个目标的优点，但这是另一篇文章的主题。

老实说，让这些网络钩子工作起来，我也有点时间紧张。所以当它起作用时，我非常兴奋，没有考虑其他的解决方案。

我在之前的文章中已经列出了这些细节。也就是说，自从写了那篇文章后，我的心和思想最近发生了变化。

## 那种挥之不去的感觉

> 如果你从来没有写过让你想在一个月甚至一个星期后回去重写的代码，那么你做代码的时间还不够长。

自从我们设置了 webhooks 并按预期工作后，我就有一种挥之不去的感觉，我的解决方案有点粗糙。*我肯定你们没有人知道这是什么感觉，因为你们从一开始就写出了完美的代码，但是请你迁就我一下*

Russ Cox(核心 Go 贡献者)证实了我的怀疑，他说你应该谨慎使用多模块回购。我知道我需要找到一个更好的方法来解决这个问题，但是直到最近我才有时间去做这件事。

让我们进入一些代码，看看在过去的一年里，我们的回购发生了怎样的变化。

## 一个模块来管理它们

如果你跟随我之前的介绍，那么你会认出一些代码。

为了让您跟上我们的进度，我们有三个模块。其中两个是独立的可执行文件，作为简单的 web 服务器工作。这些可以是任何有主要功能的东西。第三个是表示共享逻辑的日志库。它可以是一个数据库连接、中间件或者任何不作为主可执行文件运行的东西。原始文件结构应该与您在下面看到的相匹配:

```
/my_project
    /web_server
        go.mod
        go.sum
        main.go
    /logger
        go.mod
        go.sum
        logger.go
    /web_server_two
        go.mod
        go.sum
        main.go
```

## 步骤 1:组合成一个模块

我们要做的第一件事是删除三个子模块。

为此，删除所有子目录中的`go.mod`和`go.sum`文件。如果你刚刚加入我们，创建一个顶级工作目录和三个子目录，就像你在上面看到的那样。

接下来，我们将在顶层目录中创建一个新的单个模块。为此，请运行以下命令:

```
go mod init github.com/<GH USERNAME>/myproject
```

> 关于模块名的旁注。实际上你可以给它起任何你想起的名字，但是有一些理由说明为什么坚持使用 github url 是好的。只有当你打算把它作为一个公共的 go 模块发布时，这个命名约定才是重要的。如果它是一个私有模块，那么你想怎么命名就怎么命名。这也有助于 Go<1.11\. To my knowledge this only matters if you’re building a module for use with <1.11 so if you’re not then don’t worry about it.

At the end of this, your file structure should match what’s below.

```
/my_project
    go.mod
    /web_server
        main.go
    /logger
        logger.go
    /web_server_two
        main.go
```

We now have a single go module (or the start of one) that contains two executables and a shared logging library. The `go.mod` file will be empty except for the two lines below:

```
module github.com//my_project

go 1.12
```

## Step 2: Update the Logger

I’ve switched up the logger a little from the last post to show a 3rd party dependency. I’ll be using the logging library `logrus` which I wrapped with my own logger package. In a real project, you might do this so that you can report a log to multiple outputs, create a middleware function and to make it easy to mock for testing.

In our case, we’re going to wrap the logrus library and expose a log function for each of our severity levels, `error`, `warn`, `info`, and `debug`. Your logger should match my logger file below:

```
package logger

import (
    log "github.com/sirupsen/logrus"
)

// Logger is a base struct that could eventually maintain connections to something like bugsnag or logging tools
type Logger struct {
    serviceName string
}

// NewLogger creates a new instance of the custom logger struct and returns it
func NewLogger(serviceName string) *Logger {
    var l = new(Logger)
    l.serviceName = serviceName

    return l
}

// LogDebug is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogDebug(messages ...interface{}) {
    log.WithField("service-name", l.serviceName).Debug(messages)
}

// LogInfo is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogInfo(messages ...interface{}) {
    log.WithField("service-name", l.serviceName).Info(messages)
}

// LogWarning is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogWarning(messages ...interface{}) {
    log.WithField("service-name", l.serviceName).Warn(messages)
}

// LogError is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogError(messages ...interface{}) {
    log.WithField("service-name", l.serviceName).Error(messages)
}
```

Once you get the logger in place you can run `go get -u` to fetch the `logrus` dependency. Note that this is the package `logger` which is inside the module `github.com/<gh username="">/my _ project’的向后兼容性。</gh>

现在，让我们通过将它导入到我们的一个主要应用程序中来测试一下。

## 步骤 3:导入您的子包

现在我们已经设置了这个子包，我们可以使用“import”github . com/<gh username="">/my _ project/[" '将它导入到这个项目中的任何其他地方。]</gh>

让我们通过将它导入到我们的 web 服务器来看看它的运行情况。

```
package main

import (
    "net/http"

    "github.com/ckeyes88/my_project/logger"
)

func main() {
    // Create a new logger initialized with the service name
    l := logger.NewLogger("Web Server 1")

    // wrap our hello handler function
    http.Handle("/hello", loggerware(l, http.HandlerFunc(handle)))
    http.ListenAndServe(":5600", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("world"))
}

// loggerware can wrap any handler function and will print out the datetime of the request
// as well as the path that the request was made to.
func loggerware(l *logger.Logger, next http.HandlerFunc) http.HandlerFunc {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestPath := r.URL
        l.LogInfo("Request Made To: ", requestPath)
    })
}
```

正如你所看到的，我们几乎不需要改变我们的 web 服务器。我们所做的唯一更改是对 logger 构造函数的更改，因为我们添加了那个初始化函数。

## 步骤 4:更新 Web 服务器 2

最后，为了证明它的可共享性和对多模块版本的模仿，让我们添加第二台 web 服务器，巧妙地命名为“web_server_two”。

将此代码添加到您的“web_server_two/main.go”文件中:

```
package main

import (
    "net/http"

    "github.com/ckeyes88/my_project/logger"
)

func main() {
    l := logger.NewLogger("Web Server 2")

    http.Handle("/valar-morghulis", loggerware(l, http.HandlerFunc(handle)))
    http.ListenAndServe(":5500", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("valar dohaeris"))
}

func loggerware(l *logger.Logger, next http.HandlerFunc) http.HandlerFunc {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestPath := r.URL
        l.LogInfo("Request Made To Server Two: ", requestPath)
    })
}
```

正如您所看到的，在模块的任何地方导入共享日志库都没有问题。当您运行它时，您会看到与相应的服务名称相对应的特定消息。这演示了一种更简单的方法来在许多主要的可执行文件之间共享代码。

## 为什么我们应该避免替换，什么时候应该使用它？

最后，我想提出一些相对于原始解决方案的改进。我还想提供一两个你可能仍然使用`replace`指令的理由。

如您所见，对于相同的功能，该解决方案的维护量要少得多。当您试图实现我们的初始目标时，多模块方法没有任何真正的好处:

*   单个共享子包
*   多个主可执行文件和一个 repo
*   单个 CI/CD 管道

多模块方法增加了不必要的复杂性，因为它迫使您管理所有子模块之间的独立依赖关系。

没有任何理由按照我在上一篇文章中的方式使用多模块存储库。多模块方法的一个更合理的用例是隔离您的模块的一个版本，该版本在您的存储库中总是可用的。这允许您在主控级别对主版本进行潜在的重大更改。Go 团队提供了这个例外，因为他们知道人们可能希望在进行重大更改之前锁定当前的稳定版本。

`replace`的另一个好的用例是当你对另一个项目中使用的 Go 模块进行局部修改时。这是在本地测试您的开发变更的好方法，这样您就不必在每次想要测试时发布新版本的 Go 模块。

## 结论

希望这能给你更多的工具来构建你的 Go 模块。谢谢你读到这里。如果您有任何问题或意见，请随时使用下面的表格联系我们！

## 更多资源

这里有一些继续学习围棋模块的好资源。

*   [官方文件](https://blog.golang.org/using-go-modules)
*   [更多来自围棋模块作者](https://research.swtch.com/vgo)

非常感谢你读到这里！如果你喜欢这篇文章，一定要与他人分享。如果你有任何想法或问题，请留下评论。如果你想知道新文章发布的时间，你可以在我的网站上注册我的邮件列表。

*原载于* [*芯片博客*](https://www.chipkeyes.me/go-modules-done-right/) *。*
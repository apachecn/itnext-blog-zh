# 微在行动:跨服务边界的错误处理

> 原文：<https://itnext.io/micro-in-action-error-handling-across-services-boundary-2f9f27821bd5?source=collection_archive---------1----------------------->

如何在 Go Micro v2 中的服务之间传递结构错误信息

![](img/48b8d6082418acfcc69d464c1a215fac.png)

照片由[塞普·鲁兹](https://unsplash.com/@rutzsepp?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/backgrounds?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

在分布式系统中，一个服务产生错误然后返回给另一个服务是很常见的情况。如果我们能够在不丢失细节信息的情况下跨服务传递所有结构性错误，那就太好了。

Micro v2 的`github.com/micro/go-micro/v2/errors`包正是为此服务的。让我引用官方文档中的介绍:

> Go Micro 为分布式系统中发生的大多数事情(包括错误)提供了抽象和类型。通过提供一组核心错误和定义详细错误类型的能力，我们可以始终如一地了解典型 Go 错误字符串之外的情况

它在整个 Micro v2 中使用，并在基于 Micro 的所有服务中提供一致的编组/解编能力。

# 定义

错误类型由 ProtoBuf 定义:

```
syntax = "proto3";package errors;message Error {
  string id = 1;
  int32 code = 2;
  string detail = 3;
  string status = 4;
};
```

Golang 代码就是从这个 ProtoBuf 文件中生成的。在 Go Micro 中，每个字段都有常规含义:

*   **Id** 用于识别错误。在 Micro 中，服务名通常被分配给这个字段。比如“go.micro.web”、“com.foo.service.hello”。
*   **代码**是表示错误类型的数字。我们知道 Micro 使用 gRPC 作为默认传输协议。简单地说，gRPC 是基于 HTTP/2 的。所以框架继承 HTTP 状态码约定是很自然的，取 500 表示“内部服务器错误”，403 表示“禁止”，等等。
*   **详细信息**是底层错误字符串。
*   **状态**表示代码字段对应的简短描述。因为一般来说，code 字段是一个有效的 HTTP 状态代码，所以这个字段的值只是从标准库中返回的值`http.StatusText(code)`。

# 助手功能

除了类型定义，这个包还提供了一些帮助函数来简化日常开发。

*   `func New(id, detail string, code int32) error`:创建一个新的结构错误，并提供 id、详细信息和代码。(如前所述，**状态**字段将自动填充)。
*   `func Parse(err string) *Error`:试图将一个 JSON 字符串解析成一个`errors.Error`。如果失败，输入字符串将被分配给返回错误的**细节**字段。

您可能会注意到上面两个函数的返回值是不同的。我猜这是一个深思熟虑的设计决定。这表明当您在服务处理程序中返回一个新错误时，您并不关心实际的错误类型。另一方面，当您试图解析来自服务客户端的错误时，错误的类型变得至关重要。

此外，还有另外八个函数包装了最常用的代码。它们是`BadRequest`、`Unauthorized`、`Forbidden`、`NotFound`、`MethodNotAllowed`、`Timeout`、`Conflict`、`InternalServerError**.**`如果你熟悉 HTTP 状态码，你应该马上就能明白这些函数的含义。

# 应用场景

这个包的应用场景是在服务边界附近。

首先，在处理程序中返回一个错误:

```
func (e *Hello) Call(ctx context.Context, req *hello.Request, rsp *hello.Response) error {
   if len(req.Name) == 0 {
      **return** **errors.BadRequest("com.foo.service.hello", "name is required")**
   }
   rsp.Msg = "Hello " + req.Name
   return nil
}
```

然后，解析来自客户端的错误:

```
// create hello service client
helloClient := hello.NewHelloService("com.foo.service.hello", service.Client())

// invoke hello service method
resp, err := helloClient.Call(context.TODO(), &hello.Request{Name: "Bill 4"})
if err != nil {
   e := **errors.Parse(err.Error())**
   if **e.Code == http.*StatusBadRequest***{
      // deal with bad request
   }
}
```

## 包扎

Micro v2 中的`errors`包，像这个框架的其他组件一样，为分布式系统中的常见任务提供了抽象。

在这个包的帮助下，开发人员可以用一致的方式处理服务间的错误。任何熟悉 HTTP 状态代码的人都会发现适应起来非常容易，因为这个包重用了众所周知的 HTTP 状态代码集。

它将帮助我们建立一个易于理解和维护的分布式系统。你越早将它引入你的项目，你就越能从中受益。

干杯。

有关 Micro v2 的逐步教程和其他有关 Micro 的文章，请访问 [Micro In Action](https://medium.com/@dche423/micro-in-action-1be29b057f2d) 系列的索引页面。
# ASP.NET 核心 3 中的日志请求和响应

> 原文：<https://itnext.io/log-requests-and-responses-in-asp-net-core-3-a1bebd49c996?source=collection_archive---------0----------------------->

这篇文章将是 ASP.NET 核心文章中[日志请求和响应的更新，后者不再适用于更现代版本的 ASP.NET 核心。在很大程度上，这篇文章将完全符合原来的，但代码位更新。](https://elanderson.net/2017/02/log-requests-and-responses-in-asp-net-core/)

![](img/ab72ccf3c26a336eefbf6954c4614ef1.png)

作为调试工作的一部分，我需要一种方法来记录请求和响应。编写一个中间件似乎是解决这个问题的好方法。处理请求和响应机构也比我预期的要复杂。

## 中间件

在 ASP.NET 核心[中间件](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware)是组成 HTTP 管道的组件，处理应用程序的请求和响应。每个被调用的中间件都可以选择在调用下一个中间件之前对请求进行一些处理。在执行从对下一个中间件的调用返回之后，有机会对响应进行处理。

应用程序的 HTTP 管道在 Startup 类的 Configure 函数中设置。运行、映射和使用是三种可用的中间件类型。Run 应仅用于终止管道。Map 用于管道分支。Use 似乎是最常见的中间件类型，它执行一些处理并调用队列中的下一个中间件。更多细节见官方[文件](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware)。

## 创建中间件

中间件可以作为 lambda 直接在 Configure 函数中实现，但更常见的是，它是作为一个类实现的，该类使用 IApplicationBuilder 上的扩展方法添加到管道中。这个例子将使用类路由。

这个例子是一个中间件，它使用 ASP.NET 核心的内置日志记录来记录请求和响应。创建一个名为**requestresponselogginmiddleware**的类。

该类需要一个带两个参数的构造函数，这两个参数都将由 ASP.NET 核心的依赖注入系统提供。第一个是 RequestDelegate，它将是管道中的下一个中间件。第二个是 ILoggerFactory 的实例，它将用于创建记录器。RequestDelegate 存储在类 level _next 变量中，loggerFactory 用于创建存储在类 level _logger 变量中的记录器。

```
public class RequestResponseLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;

    public RequestResponseLoggingMiddleware(RequestDelegate next,
                                            ILoggerFactory loggerFactory)
    {
        _next = next;
        _logger = loggerFactory
                  .CreateLogger<RequestResponseLoggingMiddleware>();
    }
}
```

添加一个 Invoke 函数，当您的中间件由管道运行时，这个函数将被调用。下面的函数除了调用管道中的下一个中间件之外什么也不做。

```
public async Task Invoke(HttpContext context)
{
     //code dealing with the request

     await _next(context);

     //code dealing with the response
}
```

接下来，添加一个静态类来简化将中间件添加到应用程序管道的过程。这与内置中间件使用的模式相同。

```
public static class RequestResponseLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestResponseLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestResponseLoggingMiddleware>();
    }
}
```

## 添加到管道中

要将新的中间件添加到管道中，打开 **Startup.cs** 文件，并将下面一行添加到**配置**函数中。

```
app.UseRequestResponseLogging();
```

请记住，中间件的添加顺序会影响应用程序的行为。因为这篇文章讨论的中间件是日志，所以我把它放在管道的开始处。

## 记录请求和响应

现在，我们的新中间件的设置工作已经完成，我们将回到它的调用函数。正如我上面所说的，这比我想象的要复杂得多，但是谢天谢地我找到了[这个](http://www.sulhome.com/blog/10/log-asp-net-core-request-and-response-using-middleware)作者 [Sul Aga](https://twitter.com/sulhome) 的文章，它真的帮助我解决了我遇到的问题，还有很多关于这篇文章最初版本的反馈。

这篇文章最初版本的反馈之一是关于[潜在的内存泄漏和使用可回收内存流](https://stackoverflow.com/a/52328142/1291838)。首先，添加一个对微软的 NuGet 引用。IO .可回收内存流包。接下来，我们将添加一个类级变量来保存我们将在构造函数中创建的一个**recycllablememorystreammanager**的实例。以下是更新后的类视图，其中包含这些更改以及对日志记录方法的 **Invoke** 函数和存根的更改。

```
public class RequestResponseLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;
    private readonly RecyclableMemoryStreamManager _recyclableMemoryStreamManager;

    public RequestResponseLoggingMiddleware(RequestDelegate next,
                                            ILoggerFactory loggerFactory)
    {
        _next = next;
        _logger = loggerFactory
                  .CreateLogger<RequestResponseLoggingMiddleware>();
        _recyclableMemoryStreamManager = new RecyclableMemoryStreamManager();
    }

    public async Task Invoke(HttpContext context)
    {
        await LogRequest(context);
        await LogResponse(context);
    }

    private async Task LogRequest(HttpContext context) {}
    private async Task LogResponse(HttpContext context) {}
}
```

首先，我们要看一下 **LogRequest** 函数，以及它使用的一个助手函数。

```
private async Task LogRequest(HttpContext context)
{
    context.Request.EnableBuffering();

    await using var requestStream = _recyclableMemoryStreamManager.GetStream();
    await context.Request.Body.CopyToAsync(requestStream);
    _logger.LogInformation($"Http Request Information:{Environment.NewLine}" +
                           $"Schema:{context.Request.Scheme} " +
                           $"Host: {context.Request.Host} " +
                           $"Path: {context.Request.Path} " +
                           $"QueryString: {context.Request.QueryString} " +
                           $"Request Body: {ReadStreamInChunks(requestStream)}");
    context.Request.Body.Position = 0;
}

private static string ReadStreamInChunks(Stream stream)
{
    const int readChunkBufferLength = 4096;

    stream.Seek(0, SeekOrigin.Begin);

    using var textWriter = new StringWriter();
    using var reader = new StreamReader(stream);

    var readChunk = new char[readChunkBufferLength];
    int readChunkLength;

    do
    {
        readChunkLength = reader.ReadBlock(readChunk, 
                                           0, 
                                           readChunkBufferLength);
        textWriter.Write(readChunk, 0, readChunkLength);
    } while (readChunkLength > 0);

    return textWriter.ToString();
}
```

让这个函数工作并允许读取请求体的关键是**上下文。Request.EnableBuffering()** 允许我们从流的开头开始读取。函数的其余部分非常简单。

下一个函数是 **LogResponse** ，它用于执行管道中中间件的下一位，使用 **await _next(context)** ，然后在管道的其余部分运行后记录响应体。

```
private async Task LogResponse(HttpContext context)
{
    var originalBodyStream = context.Response.Body;

    await using var responseBody = _recyclableMemoryStreamManager.GetStream();
    context.Response.Body = responseBody;

    await _next(context);

    context.Response.Body.Seek(0, SeekOrigin.Begin);
    var text = await new StreamReader(context.Response.Body).ReadToEndAsync();
    context.Response.Body.Seek(0, SeekOrigin.Begin);

    _logger.LogInformation($"Http Response Information:{Environment.NewLine}" +
                           $"Schema:{context.Request.Scheme} " +
                           $"Host: {context.Request.Host} " +
                           $"Path: {context.Request.Path} " +
                           $"QueryString: {context.Request.QueryString} " +
                           $"Response Body: {text}");

    await responseBody.CopyToAsync(originalBodyStream);
}
```

如您所见，读取响应正文的技巧是用新的内存流替换正在使用的流，然后将数据复制回原始正文流。我不知道这会对性能产生多大影响，在生产环境中使用它之前，我会确保研究它是如何扩展的。

## 包扎

我希望这篇更新的文章能像原来的一样有用。这一轮我在 GitHub repo 中有代码，可以在[这里](https://github.com/elanderson/ASP.NET-Core-Basics-Refresh/commit/c1b24de0d44dfc45d379b91d721842656c4ba3d8)找到相关变更的提交。

*原载于* [*安德森*](https://elanderson.net/2019/12/log-requests-and-responses-in-asp-net-core-3/) *。*
# 生产中的角度通用

> 原文：<https://itnext.io/angular-universal-in-production-8be56b750e92?source=collection_archive---------3----------------------->

我目前正在做的一个项目需要使用 [Angular Universal](https://universal.angular.io/) 服务器端渲染进行 SEO 优化。虽然有很多很棒的文章可以帮助你开始使用 Angular Universal(如果你正在使用 Angular CLI，我推荐[这篇](https://github.com/angular/angular-cli/wiki/stories-universal-rendering))，但是没有太多关于在实际生产环境中使用它的信息。在这篇文章中，我将讨论在生产环境中运行 Universal 时需要考虑的一些额外的事情。四个主要考虑因素是:

*   从 CDN 而不是通过节点服务器提供静态资产
*   将请求代理到后端 API 以避免对 CORS 的需求
*   贮藏
*   记录

首先要考虑的是，在生产环境中，我们不希望通过通用节点服务器提供静态资产，如图像或 CSS 和 JavaScript 文件。我们还希望将请求代理到后端 API，以便可以使用相对路径进行 API 调用，并且可以避免 CORS 配置。在开发中，我们使用这样的设置通过节点服务器提供静态文件，并代理对后端 API 的请求，以避免对 CORS 的需求:

```
// proxy api requests and serve static files if we're in the development environment
if (environment === 'development') {

    // Serve static files from /browser
    app.get('*.*', express.static(join(DIST_FOLDER, 'browser')));

    const proxy = httpProxy.createProxyServer();
    app.all('/api/*', function (req, res) {
        proxy.web(req, res, {
        target: 'test-docker.learning.com/catalogservice',
        secure: false
        });
    });
}
```

“*”。* '模式意味着任何文件扩展名为(.css，。js，。png 等。)将由“浏览器”目录提供，该目录是在编译通用应用程序时创建的，包含客户端 Angular 应用程序的所有静态文件。 [http-proxy](https://www.npmjs.com/package/http-proxy) NPM 包用于代理 API 对后端服务的请求。

在生产中，我们使用 Amazon Cloudfront CDN 来提供相同的行为。首先，我们在 Cloudfront 发行版中为这个通用应用程序设置了三个来源。

1.  一个自定义原点，指向运行通用节点服务器的主机。
2.  一个自定义原点，指向运行我们后端 API 的主机。
3.  一个 S3 源，在那里部署了我们的客户端通用应用程序的静态资产。

第二，我们用规则设置了行为，这些规则将请求转发到上面设置的每个源，优先级如下

1.  路径模式为“api/*”的行为，指向我们的后端 api 服务源
2.  路径模式为“*”的行为。*，它指向包含客户端 Angular 应用程序的 S3 原点
3.  指向通用节点服务器源的总括“*”模式。

通过这种设置，对应用程序的初始请求将转到通用服务器，并将返回服务器呈现的 index.html 页面。这种行为还设置了 30 分钟的 TTL，以便对特定页面的任何后续请求都将直接从边缘缓存得到服务，而不是到达我们的通用服务器。一旦页面加载到浏览器中，所有对静态资产和加载客户端应用程序的请求都将转到 S3 源。最后，来自客户端应用程序的任何 API 请求都将被代理到我们的后端 API 服务。这样，所有通过节点服务器的都是初始页面加载请求，所有静态资产都从 S3 桶提供服务，并缓存在 Cloudfront edge 服务器上。

最后，生产环境中的任何应用程序都需要考虑一个问题，那就是为故障排除提供可靠的日志记录。我们使用 ELK 栈(Elasticsearch、Logstash、Kibana)进行所有的日志聚合和监控。在我们的万能服务器中，我们为此使用了[班扬](https://www.npmjs.com/package/bunyan)和[班扬-弹性搜索](https://www.npmjs.com/package/bunyan-elasticsearch)包。该设置如下所示:

```
// Set up logger
let logger;
const appName = `Catalog Universal Server - ${environment}`;
if (environment !== 'development') {
const esStream = new elasticsearch({
    indexPattern: '[catalog-universal-]YYYY.MM.DD',
    type: environment,
    host: 'http://elk.ldc.aws'
});
esStream.on('error', function (err) {
    console.log('Elasticsearch Stream Error:', err.stack);
});

logger = bunyan.createLogger({
    name: appName,
    streams: [
    { stream: process.stdout },
    { stream: esStream }
    ],
    serializers: bunyan.stdSerializers
});
} else {
    logger = bunyan.createLogger({ name: appName });
}
```

然后，我们设置 bunyan 中间件来记录基本的请求数据，如请求 IP 地址、请求长度、请求持续时间等。

```
// request logging
app.use(bunyanMiddleware(
    {
        headerName: 'X-Request-Id',
        propertyName: 'reqId',
        logName: 'req_id',
        obscureHeaders: [],
        logger: logger,
        additionalRequestFinishData: function (req, res) {
            return { example: true };
        }
    }
));
```

最后，我们设置了全局错误处理程序来将任何错误记录到 bunyan

```
// Handle errors
app.use((err, req, res, next) => {
    // log the error
    logger.error(err);
    // pass the error to the default express error handler
    next(err);
});
```

感谢阅读，希望这已经给了你一些关于如何设置一个用于生产的 Angular 通用应用程序的想法。

**类别:** [角度](https://tommooney.net/categories/#angular)，[编程](https://tommooney.net/categories/#programming)

**更新:**2018 年 04 月 01 日

[推特](https://twitter.com/intent/tweet?text=Angular+Universal+in+Production%20https%3A%2F%2Ftommooney.net%2Fprogramming%2Fangular%2Funiversal-in-production%2F) [脸书](https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Ftommooney.net%2Fprogramming%2Fangular%2Funiversal-in-production%2F) [谷歌+](https://plus.google.com/share?url=https%3A%2F%2Ftommooney.net%2Fprogramming%2Fangular%2Funiversal-in-production%2F) [领英](https://www.linkedin.com/shareArticle?mini=true&url=https%3A%2F%2Ftommooney.net%2Fprogramming%2Fangular%2Funiversal-in-production%2F)

*原载于 2018 年 4 月 1 日*[*【tommooney.net】*](https://tommooney.net/programming/angular/universal-in-production/)*。*
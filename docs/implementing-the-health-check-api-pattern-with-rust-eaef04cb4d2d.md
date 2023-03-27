# 用 Rust 实现健康检查 API 模式

> 原文：<https://itnext.io/implementing-the-health-check-api-pattern-with-rust-eaef04cb4d2d?source=collection_archive---------5----------------------->

![](img/7a045b0d9a1e3c0a53d72f5c5005d69e.png)

照片由 [SpaceX](https://unsplash.com/@spacex?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/rocket?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

本周，当我准备将一个基于 Rust 的后端服务部署到 Kubernetes 集群时，我意识到我还没有将我的后端服务配置为由 [**kubelet**](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) 探测，以进行[](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes)**和 [**就绪**](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes) 检查。我能够通过添加一个`/health` API 端点来完成这个需求，这个端点根据服务的当前状态用一个`Ok`或`ServiceUnavailable` HTTP 状态来响应。**

**这个`/health` API 端点解决方案是 [**健康检查 API 模式**](https://microservices.io/patterns/observability/health-check-api.html) 的一个实现，该模式用于检查您的 API 服务的健康状况。在像 [**Spring**](https://spring.io/) 这样的 web 框架中，像[**Spring Actuator**](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)这样的嵌入式解决方案可供您集成到您的 Spring 项目中。然而，在许多 web 框架中，您必须自己构建这种健康检查 API 行为。**

**在这篇博文中，我们将使用[**actix-web**](https://actix.rs/)web 框架实现健康检查 API 模式，该框架使用 [**sqlx**](https://github.com/launchbadge/sqlx) 连接到本地 PostgreSQL 数据库实例。**

# **先决条件**

**开始之前，确保您的机器上安装了 [**Cargo**](https://doc.rust-lang.org/cargo/getting-started/installation.html) 和 [**Rust**](https://www.rust-lang.org/) 。*安装这些工具最简单的方法就是使用*[***rust up***](https://rustup.rs/)*。***

**还要在您的机器上安装 [**Docker**](https://docs.docker.com/get-docker/) ，这样我们就可以轻松地创建并连接到 PostgreSQL 数据库实例。**

**如果这是你第一次看到 [**Rust**](https://rust-lang.org/) 编程语言，我希望这篇博文能启发你更深入地了解一种有趣的静态类型语言和生态系统。**

***如果你想跟随你身边的代码，我已经在*[***GitHub***](https://github.com/tjmaynes/gists/tree/main/rust/health-check-rust)*上提供了这个项目的源代码。***

# **创建新的 Actix-Web 项目**

**让我们打开您最喜欢的 [**命令行终端**](https://github.com/alacritty/alacritty) 并通过`cargo new my-service --bin`创建一个新的 Cargo 项目。**

> ***`*--bin*`*标志会告诉 Cargo 自动创建一个* `*main.rs*` *文件，让 Cargo 知道这个项目不是一个库而是会产生一个可执行文件。****

***接下来，让我们通过运行命令`cargo run`来检查我们是否能够运行项目。运行这个命令后，您应该会得到如下所示的输出。***

```
*Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/health-endpoint`
Hello, world!*
```

***Easy-peezee，对吗？***

***接下来，让我们启动并运行 PostgreSQL 实例。***

# ***运行 PostgreSQL***

***在使用 Docker Compose 创建 PostgreSQL 实例之前，我们需要创建一个初始 SQL 脚本来创建数据库。让我们将下面的代码添加到项目根目录下名为`db`的目录中一个名为`init.sql`的 SQL 文件中。***

***这个脚本将检查名为“member”的数据库是否已经存在，如果不存在，它将为我们创建数据库。接下来，让我们将下面的 YAML 复制到一个`docker-compose.yml`文件中并运行`docker compose up`。***

***经过一些丰富多彩的文字🌈在控制台窗口中，您应该已经启动并运行了 PostgreSQL。***

***好了，现在我们已经确认了我们的服务运行，并且我们有一个 PostgreSQL 实例在本地运行，让我们打开您最喜欢的 [**文本编辑器**](https://code.visualstudio.com/) 或 [**IDE**](https://www.jetbrains.com/idea/) 并将我们的项目依赖项添加到我们的`Cargo.toml`文件中。***

***对于`sqlx`，我们希望确保在编译期间包含“postgres”特性，因此我们有 PostgreSQL 驱动程序来连接到我们的 PostgreSQL 数据库。接下来，我们要确保包含了`runtime-actix-native-tls`特性，这样 sqlx 就可以支持使用 [**tokio**](https://tokio.rs/) *运行时*的`actix-web`框架。最后，让我们包括`serde`和`serde_json`来序列化我们的健康检查 API 响应体，以便稍后在帖子中使用。***

> *****注:**对于初来乍到的铁锈，你可能会在心里想，“管它呢？Actix 运行时？我以为 actix-web 只是 Rust 的一个 web 框架。”是的，而且还不止这些。由于 Rust 的设计没有考虑任何特定的 [**运行时**](https://en.wikipedia.org/wiki/Runtime_system) ，所以您当前所处的问题领域需要一个特定的运行时。有专门用于处理客户机/服务器通信需求的运行时，例如 [**Tokio**](https://docs.rs/tokio/1.12.0/tokio/) ，这是一个流行的事件驱动的非阻塞 I/O 运行时。 [**Actix**](https://docs.rs/actix/) ，actix-web 背后的底层运行时，是一个基于 [**actor 的**](https://en.wikipedia.org/wiki/Actor_model) 消息传递框架，构建在 tokio 运行时之上。***

***所以，现在我们已经添加了依赖项，让我们继续创建我们的`actix-web`服务。为此，让我们用以下 Rust 代码替换`src/main.rs`文件中的内容:***

***上面的代码给了我们一个运行在端口`8080`上的 HTTP 服务器和一个总是返回`Ok` HTTP 响应状态代码的`/health`端点。***

***回到您的终端，运行`cargo run`查看服务启动并运行。在一个新标签页中，继续运行`curl -i localhost:8080/health`并看到您收到如下响应:***

```
*$ curl -i localhost:8080/health
HTTP/1.1 200 OK
content-length: 8
content-type: application/json
date: Wed, 22 Sep 2021 17:16:47 GMTHealthy!%*
```

***既然我们已经启动并运行了基本的健康 API 端点，那么让我们更改健康 API 的行为，以便在到 PostgreSQL 数据库的连接处于活动状态时返回一个`Ok` HTTP 响应状态代码。为此，我们首先需要使用`sqlx`建立一个数据库连接。***

# ***创建数据库连接***

***在使用 sqlx 的 [**connect**](https://docs.rs/sqlx/0.5.7/sqlx/postgres/struct.PgConnection.html#method.connect) 方法建立数据库连接之前，我们需要创建一个数据库连接字符串，格式为`<database-type>://<user>:<password>@<db-host>:<db-port>/<db-name>`，它与我们的本地 PostgreSQL 设置相匹配。***

***同样，不要硬编码我们的数据库连接字符串，让我们通过一个名为`DATABASE_URL`的环境变量使 [**可配置**](https://12factor.net/config) ，并在我们的每个`cargo run`调用之前预先考虑该变量，如下所示:***

```
*DATABASE_URL=postgres://root:postgres@localhost:5432/member?sslmode=disable cargo run*
```

***有了可用的`DATABASE_URL`环境变量，让我们在`main`函数中添加一行来获取新导出的环境变量。***

***接下来，让我们再写一些代码，在我们的`main`函数中创建一个数据库连接。***

***在将数据库连接传递给健康端点处理程序之前，我们首先需要创建一个代表服务共享可变状态的`struct`。Actix-web 使我们能够在路由之间共享我们的数据库连接，这样我们就不会在每个请求上创建一个新的数据库连接，这是一个昂贵的操作，会降低我们服务的性能。***

***为了实现这一点，我们需要创建一个 Rust `struct`(在我们的`main`函数之上)，我们称之为包含我们的`db_conn`引用的`AppState`。***

***现在，在我们的`db_conn`实例化下面，我们将创建一个 AppState dat 对象，它被包装在一个`web::Data`包装器中。`web::Data`包装器将允许我们在请求处理程序中访问 AppState 引用。***

***最后，让我们将应用程序的`app_data`设置为我们克隆的`app_state`变量，并用`move`语句更新我们的`HttpServer::new`闭包。***

***如果我们不克隆`app_state`变量，Rust 会抱怨说`app_state`变量不是在我们的闭包里创建的，Rust 也没有办法保证`app_state`在被调用时不会被破坏。关于这方面的更多信息，请查看 [**铁锈所有权**](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html) 和 [**复制特征**](https://hashrust.com/blog/moves-copies-and-clones-in-rust/) 文档。***

***到目前为止，我们的服务代码应该如下所示:***

***现在我们已经将包含数据库连接的`app_state`对象传递到了`App`实例中，让我们继续更新`get_health_status`函数来检查我们的数据库连接是否有效。***

# ***数据库连接检查***

***为了从我们的`get_health_status`函数中捕获我们的`AppState`数据，我们需要给我们的`get_health_status`函数添加一个`Data<AppState>`参数。***

***接下来，让我们编写一个轻量级 PostgreSQL 查询，使用`SELECT 1`查询来检查我们的数据库连接。***

***接下来，让我们更新`HttpResponse`响应，当我们的数据库连接时返回一个`Ok`，当没有连接时返回`ServiceUnavailable`。同样，为了调试的目的，让我们包含一个比`healthy`或`not healthy`更有用的响应体，通过序列化一个 Rust `struct`，使用`serde_json`，描述*为什么*我们的健康检查成功或失败。***

***最后，让我们用`cargo run`命令运行我们的服务:***

```
*DATABASE_URL=postgres://root:postgres@localhost:5432/member?sslmode=disable cargo run*
```

***接下来，打开另一个终端选项卡，运行下面的`curl`命令:***

```
*curl -i localhost:8080/health*
```

***这将返回以下响应:***

```
*HTTP/1.1 200 OK
content-length: 27
content-type: application/json
date: Tue, 12 Oct 2021 15:56:00 GMT{"database_connected":true}%*
```

***如果我们通过`docker compose stop`关闭我们的数据库，那么两秒钟后，当您再次调用前面的`curl`命令时，您应该会看到一个`ServiceUnavailable` HTTP 响应。***

```
*HTTP/1.1 503 Service Unavailable
content-length: 28
content-type: application/json
date: Tue, 12 Oct 2021 16:07:03 GMT{"database_connected":false}%*
```

# ***结论***

***我希望这篇博文可以作为实现健康检查 API 模式的有用指南。您可以将更多信息应用到您的`/health` API 端点，例如，如果适用的话，当前用户的数量、缓存连接检查等。确保后端服务看起来“健康”所需的任何信息。这因服务而异。***

***如果你愿意支持我成为一名作家，可以考虑注册成为一名灵媒。只要每月 5 美元，你就可以无限制地使用媒体。感谢你阅读我的博文！***
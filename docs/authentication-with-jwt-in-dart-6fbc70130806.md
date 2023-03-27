# Dart 中的 JWT 认证

> 原文：<https://itnext.io/authentication-with-jwt-in-dart-6fbc70130806?source=collection_archive---------1----------------------->

## 如何在 Dart 中实现 JSON Web 令牌认证

![](img/fcdd5cfa00ebc779054302a7e4426dcd.png)

## 介绍

W 随着安全问题[稳步上升](https://www.securitymagazine.com/articles/90376-concerns-about-physical-security-and-cybersecurity-are-increasing)，组织正在寻找更好、更快的方法来实施强大、可靠的安全措施，以保护关键基础设施和重要客户数据。

在本文中，我们将了解如何使用 [JSON Web 令牌](https://jwt.io)在 [Dart](https://dart.dev) 中启动并运行基本身份验证，这是一种越来越流行且易于使用的方法，用于保护网络上两台机器之间的身份验证。

Dart 是一种富有表现力的高效语言，适合处理任何标准的客户端或服务器任务，这使得它成为基于现代概念(如[固体设计原则](https://www.google.com/search?client=safari&rls=en&q=solid+design+principles&ie=UTF-8&oe=UTF-8)和[十二因素应用程序方法学](https://12factor.net))构建端到端客户端-服务器解决方案的绝佳选择。

示例项目的源代码可以在 GitHub 上的[处获得。如果你还没有 Dart 环境，你可以](https://github.com/kenreilly/dart-jwt-example)[在这里](https://dart.dev/get-dart)获得。

## 项目设置

这个项目是用命令`$ stagehand package-shelf`创建的，它为 [shelf web 服务器](https://github.com/dart-lang/shelf)初始化一个样板项目。

首先，让我们检查一下 **pubspec.yaml** 中的项目定义:

在这个文件中有名称、描述、环境和依赖关系的标准定义。添加 [jaguar_jwt](https://pub.dev/packages/jaguar_jwt) 包是为了处理创建和验证令牌的内部 jwt 任务。还添加了 [dotenv](https://pub.dev/packages/dotenv) 包，以符合 12 因素原则的[因素 III(配置)](https://12factor.net/config)，并允许使用本地环境变量进行原型制作和测试。

为这个项目创建了一个默认的配置文件 **.env.example** :

`PORT`变量表示服务器将在端口 3000 上运行，而`JWT_AUTH_SECRET`是一个特定于应用程序的秘密，用于生成令牌，并验证它们以确保它们来自我们的网络。

## **申请入口点**

我们要看的下一个文件是 **bin/server.dart** :

**Server** 类处理示例 API 服务器的初始化，包括环境变量加载、命令行参数解析和服务器管道/中间件设置。定义为参数的*调试*模式在本例中未实现，仅用于说明。

*_echo* 方法是一个通用的 HTTP 请求处理程序，它总是返回 OK，这使得它成为一个很好的测试点，确保我们是否获得 OK 完全取决于底层中间件。

使用名副其实的 *createMiddleware、*创建 *auth* 中间件，然后在请求记录之后和 *_echo* 处理程序之前将其放入管道中。

一旦服务器启动，管道将接收请求，通过 auth 中间件处理它们，然后或者在那里停止(如果响应是由 auth 处理程序发送的),或者继续到 _echo 处理程序(如果 auth 处理程序验证了请求并在管道中进一步发送它)。

## 配置

接下来是第一个实用程序类，在 **lib/config.dart** 中:

**Config** 类在内部使用 dotenv 包，从文件中加载一个环境图，然后将属性 *port* 和 *secret* 暴露给程序的其余部分。这种方法使得升级 dotenv 包变得容易，甚至可以用另一个工作解决方案完全替换它，而对系统的其余部分没有影响。它还允许设置故障保护逻辑，以便在变量丢失或值不正确时进行捕捉。

## 生成哈希

第二个实用程序类用于生成散列，位于 **lib/hash.dart** 中:

这个简单的散列类有一个目的:生成用来代替明文密码的散列。虽然这只是一个示例应用程序，但通常最好是不惜一切代价避免使用明文密码，*即使只是原型制作*。花五分钟时间使用哈希。

## 身份验证提供者

我们要看的下一个文件是 **lib/auth-provider.dart** :

**AuthProvider** 类处理这个 API 的实际认证。创建一个 [JsonDecoder](https://api.dartlang.org/stable/2.4.1/dart-convert/JsonDecoder-class.html) 的实例来处理来自 JSON 的登录请求体的反序列化。 *_check* 方法是一个实用程序，如果用户数据映射对象与从硬编码到类中的*用户列表*中传递的对象相匹配，则该实用程序返回 true。在一个真实的应用程序中，这个用户列表将被存储在某个数据库中和/或由一个服务管理，但是为了测试的目的，快速模拟一个 100%工作的东西是很方便的。

在请求处理期间， *handle* 方法由中间件处理程序调用，如果请求可以在内部处理(例如处理登录或拒绝 API 调用的坏令牌),它将返回**响应**,或者如果不需要任何操作，它将返回`null`,允许请求继续到管道中的下一个中间件或处理程序(在我们的例子中，回显请求)。

为了实现这一点，处理程序检查请求 url 字符串，如果 url 是`'login'`，将调用 *auth* 方法，否则调用 *verify* 方法。

*auth* 方法首先反序列化请求 JSON 并从中提取所需的属性，将*用户名*和*密码*的散列变体存储在 *creds* 中，然后在硬编码的用户列表中搜索一个实体，当使用 *_check* 方法进行测试时，该实体返回一个匹配。如果存在一个令牌，则使用用户名以及颁发者和受众声明创建一个令牌。有关 JWT 声明和实施的更多信息，[参见本页](https://jwt.io/introduction/)。

*验证*方法反向工作，从`Authorization`头中检索一个 JWT(在删除`'Bearer: '`组件之后)，或者验证请求并允许它继续发送，或者如果令牌本身无效或者它的声明与硬编码到这个应用程序中的不匹配，则直接拒绝它(`‘Acme Widgets Corp’`和`‘example.com’`)。

## 测试

为了测试 API，从项目目录中用`$ dart bin/server.dart`启动服务器。应该会返回消息`Serving at [http://localhost:3000](http://localhost:3000)`,表明已经准备好测试了。首先，让我们检查一下我们未经身份验证的请求是否被拒绝:

```
$ curl localhost:3000/hello
```

这应该会返回响应`Authorization rejected`。接下来，使用硬编码列表中的用户名/密码测试登录:

```
$ curl localhost:3000/login -d \ '{"username":"test","password":"insecure"}'
```

如果一切顺利，这将返回一个 JSON web 令牌(截图如下)。复制该字符串并将其粘贴到下面的`MY_JWT`处:

```
$ curl localhost:3000/hello -H "Authorization: Bearer MY_JWT"
```

这将返回`Authorization OK for “hello”`,表明我们之前被拒绝的请求现在被批准了，这意味着 API 正在按预期工作，请求正在被正确处理。

## 结论

本文展示了使 Dart 成为服务器端开发的最佳选择的一些特性，例如简洁而富于表现力的语法和维护良好且易于使用的包系统。这个示例项目([在这里可用](https://github.com/kenreilly/dart-jwt-example))为构建更复杂的 API 解决方案以支持任何类型的桌面、移动和 web 客户端提供了一个很好的起点。

感谢您的阅读，祝您的下一个 DevOps 项目好运！

![](img/9d9cc3a18d955d14a61e02ec4a2881ff.png)

JWT 身份验证测试会话的屏幕截图

> 肯尼斯·雷利( [8_bit_hacker](https://twitter.com/8_bit_hacker) )是 [LevelUP](https://lvl-up.tech/) 的 CTO
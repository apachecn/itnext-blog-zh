# 建立社会网络:第三部分

> 原文：<https://itnext.io/building-a-social-network-part-iii-9f9d215a624?source=collection_archive---------5----------------------->

## 用 Dart 和 PostgreSQL 创建 REST API 框架

![](img/03ddd402c5f9c8fc9e8a3ff47cbaaafd.png)

软件设计框图的动画插图

## 介绍

T 本文详述了本系列的[第一部分](/building-a-social-network-part-i-25856fc693e1)和[第二部分](/building-a-social-network-part-ii-1e6883ba27f6)(涵盖了模式、安全性和类型定义)，创建了一个 [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) [API](https://en.wikipedia.org/wiki/Application_programming_interface) 框架，这将有助于 REST 端点的开发。

该框架旨在减少[样板代码](https://en.wikipedia.org/wiki/Boilerplate_code)，便于维护和特性开发，并与在 [PL/pgSQL](https://en.wikipedia.org/wiki/PL/pgSQL) 中开发的底层特性集成。诸如 [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) URL 解析、[序列化](https://en.wikipedia.org/wiki/Serialization)和[认证](https://en.wikipedia.org/wiki/Authentication)等常见需求被构建到类中，这些类既可以被扩展以利用功能，也可以应用于其他类的方法，以使用[注释](https://dart.dev/guides/language/language-tour#metadata)来改变输入、输出或其他行为。

这种方法利用 [OOP](https://en.wikipedia.org/wiki/Object-oriented_programming) 、 [@OP](https://en.wikipedia.org/wiki/Attribute-oriented_programming) 、[元编程](https://en.wikipedia.org/wiki/Metaprogramming)和[反射](https://en.wikipedia.org/wiki/Reflective_programming)中的概念来创建易于描述和维护的软件，并利用[测试驱动开发](https://en.wikipedia.org/wiki/Test-driven_development)来确保高质量的结果。

对于这个项目的源代码副本，克隆这个 repo 。

## 包装定义

服务器端 SDK 包文件为 **api_sdk/pubspec.yaml:**

这个 SDK 使用了上一篇文章中的`**core**` ，还使用了 [dotenv](https://pub.dev/packages/dotenv) 、 [jaguar_jwt](https://pub.dev/packages/jaguar_jwt) 以及其他一些标准包。

SDK 的组件从 **api_sdk/api-sdk.dart** 中导出:

## API 服务器

服务 API 的基类在`**lib/api-server.dart**`中:

这实现了一个使用 API 框架的基本服务器，我们将在整篇文章中研究这个框架。环境变量用于配置和初始化身份验证和数据库提供程序。请求被转发到 APIService 类，每十分钟调用一次刷新操作来清除超过 24 小时的内容。

## API 方法注释

接下来让我们看看`**lib/framework/api-method.dart**`:

当实现一个 API 时，这些类被用作注释:

*   使用 RoutePath 在服务上设置根 URL
*   指定`REST`和`WebSocket`端点
*   在`REST`端点上使用`JSON`序列化

注意，`RoutePath`上提供了`+`操作符的实现，以简化 URL 路径的连接。

## API 路线

内部路由处理的类在`**lib/framework/api-route.dart**`中:

`RoutePoint`和`RouteParameter`类扩展了`RouteComponent`来从每个方法上的 REST 路径构建 URL 解析逻辑(即`/users/:id`，其中 URL 中`:id`的值变成了`id`变量的值)。

服务创建`APIRoute`的实例来包装用上面列出的 API 方法之一注释的方法——例如`GET`或`POST`——并且用每个路由的`check`函数测试传入的请求，直到找到正确的路由。一旦服务为请求找到了匹配的`APIRoute`，就使用`MethodMirror`以及方法的位置和命名参数(使用`_args`和`_parse`函数从传入的请求中提取)调用服务上相应的类方法。

## API 服务

服务基类在`**lib/framework/api-service.dart**`中:

当使用这个框架实现一个 API 时，会为每个服务创建一个扩展`APIService`的类，包括对`RoutePath`基本 URL 和`APIRoute` REST 谓词的注释，URL 将被附加到基本 URL。

当一个扩展了`APIService`的服务实例被创建时，反射被用来从`RoutePath`和`APIRoute` REST 动词和路径中检索基本 URL 等信息。对于类中每个被指定为 REST 或 WS 操作的方法，都会创建一个 APIRoute 实例并存储在`_routes`属性中。对于传入的请求，根据 URL 模式检查每个路由，直到找到并调用正确的路由。

## 反射镜

位于 `**lib/types/reflector.dart**`中的抽象反射器类有助于 SDK 对数据模型的自动序列化:

`Reflector`包含一组实用方法，用于处理对序列化对象的检查，以及将`JSON`解码的`Map<String, dynamic>`对象转换为`T`类型的实例，这允许 API 从`JSON POST`数据创建任何`Serializable`对象的实例。

## 认证提供者

认证在`**lib/services/auth-provider.dart**`内处理:

任何用`Authenticate` decorator 标注的端点都将通过标准的`JWT`认证方案得到保护。任何需要登录和认证的 API 都可以使用`AuthProvider`来标记一个`AuthenticatedUser`，并使用这个标记来授权后续的 API 调用。

## **数据提供器**

数据库连接在`**lib/framework/data-provider.dart**`处理:

`DataProvider`类包装了`DB`的一个实例，该实例反过来又包装了数据库连接本身，该数据库连接是根据从控制服务器传入的环境变量配置的。提供便利的方法`findOne`是为了从预期的结果集中返回单个项目。

## 测试服务

在`**lib/services/test-service.dart**`中，包含了一个示例服务来演示如何使用 SDK 定义 REST 服务:

`@RoutePath`注释将该服务的根 URI 路径定义为`'/'`，而`@GET`为每个方法定义了 URI，并将用于自动连接这些端点以响应 HTTP `GET`动词。`@JSON`注释指示 SDK 使用 JSON 序列化，并在响应头上设置适当的内容类型。

## SDK 测试基地

SDK 的测试框架内置于 SDK 本身，并与库的其余部分一起导出，以支持代码在后续 API 测试中重用。

其基类可在`**lib/test/sdk-test-base.dart**`中找到:

`SDKTestBase`类包括一些创建和启动服务器实例的方法，除此之外，测试用例将使用这些方法向服务器发出请求并验证正确的输出。

## 测试跑步者

在`**test/test-runner.dart**`中可以找到一个基本的测试实现:

测试运行人员建立一个`SDKTest`来测试配置和服务。URL 解析和对象序列化等基本操作已经过验证，可以正常工作，未来使用该库构建的 API 将实现基于该框架构建的类似测试，从而确保连续性和可靠性。

## 结论

这个 API 框架将作为实现演示社交网络所需的各种 API 的基础，例如移动和/或 web 客户端、管理工具和其他服务。反射和注释的使用极大地减少了每个服务所需的样板文件，如上面的`TestService`类所示。

请继续关注第四部分，在这一部分中，我们将构建一个 API 来创建用户并与系统中的其他用户进行交互。

感谢阅读！

~ [8_bit_hacker](https://twitter.com/8_bit_hacker)
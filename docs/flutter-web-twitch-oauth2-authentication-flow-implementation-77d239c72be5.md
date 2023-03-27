# Flutter Web: Twitch OAuth2 认证流程实现

> 原文：<https://itnext.io/flutter-web-twitch-oauth2-authentication-flow-implementation-77d239c72be5?source=collection_archive---------0----------------------->

目前在 Flutter Web 上很难实现 OAuth2 这样的认证流程。遇到的困难之一是从 URL 重定向获取身份验证令牌，因为您无法访问 URL 侦听器，也无法跟踪外部导航选项卡中的更改。

我很难理解如何实现这种认证流程，而解决方案实际上非常简单。今天我将以 Twitch 的 API 为例进行解释。

![](img/d89bcb365691f37f8951c3d33229bd2a.png)

# 先决条件

*   创建您的初始 Flutter Web 应用程序
*   拥有有效的 Twitch 和 Twitch 开发者账户

# 建立

首先，你需要在 [Twitch 开发者控制台](https://dev.twitch.tv/console)中注册一个应用程序，你可以随意命名它，并定义一个重定向 URL:[http://localhost:8080](http://localhost:8080)(或者任何你想要的其他端口)。我使用的是本地主机 URL，因为该示例不会在线托管，但您也可以随意添加将部署您的应用程序的 URL。

既然您已经在控制台上创建了应用程序，那么您应该已经获得了一个客户机 id。您已经可以将它复制粘贴到您的`main.dart`中。

```
const String clientId = "YOUR_CLIENT_ID";
```

# 管理认证流程

现在您已经有了客户端 id，下一步是使用 [OAuth 隐式代码流](https://dev.twitch.tv/docs/authentication/getting-tokens-oauth#oauth-implicit-code-flow)从 Twitch 的 API 获取认证令牌。

通过参考文档，在识别和授权之后，代码被返回到重定向 URL 中。

如果你没有改变由 Flutter 生成的基础项目的结构，你应该有一个`MyHomePage`类，它是一个`StatefulWidget`。在类`_MyHomePageState`中，您将添加一个变量:

```
/// To save the OAuth2 token.
String _token;
```

现在，您将在小部件的状态中实现`initState`方法:

让我解释一下这个方法内部发生了什么。

首先，使用`Uri.base`获取 Flutter 应用程序的当前 URL。然后，通过参考 Twitch 的文档，你知道令牌在 URL 中返回如下:`https://<your registered redirect URI>#access_token=<an access token>`这意味着如果你的 URL 包含`access_token=`，你已经被 Twitch 重定向。

现在你有两种情况:

1.  您的 URL 中没有令牌，因此您需要进行身份验证。为了做到这一点，我们使用了`dart:html`方法`window.location(url)`,它将把我们当前标签的 URL 改为 Twitch 授权。
2.  你已经被 Twitch 重定向，并且在你的 URL 中有令牌，所以你需要解析它来获得你的令牌，并把它保存在你的`_token`变量中。

# 启动应用程序

此时，您应该准备好了，您可以添加一个`print(token)`来确保它已被正确获取，您只需运行命令:

```
flutter run -d chrome --web-port=8080
```

该命令将在 Google Chrome 浏览器中运行您的应用程序，添加的参数将强制主机使用本地主机端口 8080。请注意，我使用的是 chrome，但它也适用于您可能传递的任何其他兼容参数。当然，如果你在 Twitch 开发者控制台中注册了一个不同于 8080 的端口，那么就使用这个。

![](img/e3304cc7ec43a4d7047f67299ab6f645.png)

结果

# 结论

这就是你要做的！正如您所看到的，这实际上很容易，重定向为您做了一切，Flutter 足够聪明，不会受到 URL 中包含授权响应这一事实的干扰。通过使用这种实现，您不需要任何 WebView 或监听 URL 更改。虽然它可能不是对每个用例都有效，但我认为它仍然很有帮助。

您可以在这里找到我在本文中使用的完整源代码:

[](https://github.com/TesteurManiak/flutter_web_twitch_auth) [## TesteurManiak/flutter _ web _ twitch _ auth

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/TesteurManiak/flutter_web_twitch_auth)
# 关于如何使用 React 和实现 Azure AD 身份验证的备忘录。网络核心

> 原文：<https://itnext.io/a-memo-on-how-to-implement-azure-ad-authentication-using-react-and-net-core-2-0-3fe9bfdf9f36?source=collection_archive---------1----------------------->

![](img/60e303f5e5614981982a78b5327021bf.png)

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fa-memo-on-how-to-implement-azure-ad-authentication-using-react-and-net-core-2–0–3fe9bfdf9f36)

我发现了许多使用 React 和. NET Core 2.x 后端实现 Azure AD 认证的方法。在本文中，我将演示如何实现这种类型的身份验证。

# 注册您的应用程序

第一步是注册你的 Azure 广告。一旦你做到了这一点，你就可以使用 Azure 生成的密钥在你的应用中实现身份验证。

现在，我们将配置前端以获取 Azure AD 访问令牌，然后在后端使用该令牌。
如果你想查看详细的代码，请查看下面的库:[https://github.com/Odonno/azuread-react-dotnet-core-example](https://github.com/Odonno/azuread-react-dotnet-core-example)

 [## odono/azuread-react-dot net-core-example

### 关于如何使用 React 和实现 Azure AD 身份验证的备忘录。NET Core-odon no/azuread-react-dot NET-Core-example

github.com](https://github.com/Odonno/azuread-react-dotnet-core-example) 

# 前端

前端有很多实现，使用最多的当然是`angular-adal`库，在我们的例子中并不是最好的选择。我们可以使用简单的`adal.js`，但是为什么每次都要重新发明轮子呢？

幸运的是，我在 GitHub 上找到了一个名为`react-adal`的[库，它似乎做得相当不错。所以，我们先从它开始，看看有多简单。](https://github.com/salvoravida/react-adal)

```
npm install react-adal
```

一旦你到了那里，你需要写两件事:

*   一个包含您在上一步中从 Azure 获得的密钥的配置文件
*   调用身份验证方法

完成配置文件后，在`index.jsx`文件中执行该功能:

你已经准备好了。现在，每次用户进入这个功能，将显示一个新的页面，以便他可以设置他的凭证和连接。作为奖励，我给你一个`getToken()`方法，让你轻松获得我们需要的认证令牌，以确保用户能够访问我们的后端。

请记住，使用 Azure AD 身份验证令牌按原样设置您的头，以进行 HTTP 调用。

需要注意的一点是，从回调 url 生成的第一个令牌有 1 小时的生存期。因此，当该令牌接近到期时，库将检索刷新令牌。默认情况下，`react-adal`库将在当前令牌到期日期前至少 5 分钟尝试刷新令牌。

# 后端

现在，每次用户向您的后端发送请求时，您都需要确保令牌是有效的。这个令牌也将帮助您检测谁是用户。

## 配置文件

您必须根据您的按键在`appsettings.json`中设置此信息。

## 配置服务

开始在`ConfigureServices()`方法中添加几行代码。

并且确保在你的`Configure()`方法中有这行代码。

哦，如果您使用 Swagger 生成器，下面是在 Swagger 中添加授权令牌表单的代码。

## 检测用户配置文件

现在，你的后台是安全的，但是我们怎么知道谁登录了呢？
以下是您可以在应用中使用的一些基本方法:

# 结论

瞧，你可以走了。
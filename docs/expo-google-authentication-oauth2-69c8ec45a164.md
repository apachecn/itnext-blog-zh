# 博览会—谷歌认证 Oauth2

> 原文：<https://itnext.io/expo-google-authentication-oauth2-69c8ec45a164?source=collection_archive---------4----------------------->

![](img/02715353751dd13619e4a420fd4fbb3c.png)

[迈卡·威廉姆斯](https://unsplash.com/@mr_williams_photography?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

**世博谷歌认证**

Expo 被许多开发者使用，用户数量增长速度惊人。也许您想知道如何在您的应用程序中使用 google sign in 来为您的用户签名？

今天我将解释你如何为你的项目制作谷歌认证应用程序的例子。

```
**Expo.Google.logInAsync**
```

使用的版本

**“Expo”:“32 . 0 . 0”
“react”:“16 . 5 . 0”**

世博会应用程序只使用基于网络的认证。这意味着它允许您从浏览器使用您的登录用户。

要使用 Google 登录，您需要在 Google 开发人员控制台上创建一个项目，并创建一个 OAuth 2.0 客户端。

要在谷歌开发者控制台创建新项目，请访问下面的链接:
[谷歌开发者控制台](https://console.developers.google.com/apis/credentials)

*   如果你还没有为你的项目创建一个应用程序。
*   然后单击创建凭据

**创建 iOS OAuth 客户端 ID**

*   选择“iOS 应用程序”作为应用程序类型。
*   使用`host.exp.expotest`作为包标识符。
*   点击“创建”
*   现在，您将看到一个带有客户端 ID 的模态。保存客户端 id 供以后使用。这是我们的 iOS 客户端 ID。

**创建一个 Android OAuth 客户端 ID**

*   选择“Android 应用程序”作为应用程序类型。
*   在你的终端上运行`openssl rand -base64 32 | openssl sha1 -c`，它将输出一个字符串拷贝并粘贴为“签名证书指纹”。
*   使用`host.exp.expotest`作为“包装名称”。
*   点击“创建”
*   你创建了 Android ClientID。

# 使用

更换你的 AndroidClientId 和 IosClientId，开始使用 expo 的谷歌认证。

这就是全部的过程。:)

*如果你觉得这篇文章很有帮助，你可以通过使用我的推荐链接注册一个* [***中级会员来访问类似的文章***](https://melihyumak.medium.com/membership) *。*

***跟我上*** [**推特**](https://twitter.com/hadnazzar)

![](img/e09adde9fd734db2f987c8df72839da8.png)

在 Youtube 上订阅更多内容

# 编码快乐！

梅利赫
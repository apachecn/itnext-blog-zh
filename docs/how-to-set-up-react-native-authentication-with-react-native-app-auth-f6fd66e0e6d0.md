# 如何使用 react-native-app-auth 设置 React 本机身份验证

> 原文：<https://itnext.io/how-to-set-up-react-native-authentication-with-react-native-app-auth-f6fd66e0e6d0?source=collection_archive---------1----------------------->

![](img/744187286873d8e5d3ab219d34aad181.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [CMDR Shane](https://unsplash.com/@cmdrshane?utm_source=medium&utm_medium=referral) 拍摄的照片

## 版本

开始之前，请确保您安装了以下版本:

" react": "16.8.3 "，
"react-native": "0.59.1 "，
" react-native-contacts ":" 3 . 1 . 5 "，

**如果你想了解一下的话，这里有 Github 回购的链接**:[https://github.com/FormidableLabs/react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)

**React-native-app-auth** 用于在 React-native 应用程序中提供身份验证。在我的情况下，我试图使用它与谷歌，所以这里有一个解释，你可以安装和使用它的上述版本。

在他们的文档中，它还被解释为用于与 [OAuth 2.0](https://tools.ietf.org/html/rfc6749) 和 [OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) 提供者通信的 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 和[app auth-Android](https://github.com/openid/AppAuth-Android)SDK 的 React 本地桥。

# 经过测试的 OpenID 提供商:

这些提供商是 OpenID 兼容的，这意味着您可以使用[自动发现](https://openid.net/specs/openid-connect-discovery-1_0.html):

*   [身份服务器 4](https://demo.identityserver.io/) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/identity-server-4.md))
*   [身份服务器 3](https://github.com/IdentityServer/IdentityServer3.md) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/identity-server-3.md))
*   [谷歌](https://developers.google.com/identity/protocols/OAuth2) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/google.md))
*   [Okta](https://developer.okta.com/) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/okta.md))
*   [键盘锁](http://www.keycloak.org/) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/keycloak.md))
*   [Azure 活动目录](https://docs.microsoft.com/en-us/azure/active-directory) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/azure-active-directory.md))
*   [AWS 认知](https://eu-west-1.console.aws.amazon.com/cognito) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/aws-cognito.md))

# 经过测试的 OAuth2 提供程序:

这些提供者实现 OAuth2 规范，但不是 OpenID 提供者，这意味着您必须自己配置授权和令牌端点。

*   [优步](https://developer.uber.com/docs/deliveries/guides/three-legged-oauth.md) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/uber))
*   [Fitbit](https://dev.fitbit.com/build/reference/web-api/oauth2/) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/fitbit.md))
*   [Dropbox](https://www.dropbox.com/developers/reference/oauth-guide) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/dropbox.md))
*   [Reddit](https://github.com/reddit-archive/reddit/wiki/oauth2) ( [示例配置](https://github.com/FormidableLabs/react-native-app-auth/blob/master/docs/config-examples/reddit.md))

## **安装**

```
npm install react-native-app-auth --save
react-native link react-native-app-auth
```

**IOS** 在文档中，有三种方法可以实现这种状态，但我更喜欢 **CocoaPods。**

如果您是第一次使用 CocoaPods，请完成以下步骤:

```
sudo gem install cocoapods
```

从您的根文件夹打开

```
cd iospod init
```

pod init 命令将初始化 iOS 目录中的 pod 文件。

然后在 Podfile 的**target‘your _ app’do**之后添加下面这一行

```
pod 'AppAuth', '0.95.0'
```

## **注册重定向 URL 方案**

如果您打算支持 iOS 10 和更早版本，您需要在您的`Info.plist`中定义支持的重定向 URL 方案，如下所示:

注意:您将从 **oauth 提供者**处获得这些值。
谷歌:[https://console.developers.google.com/](https://console.developers.google.com/)

```
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLName</key>
    <string>com.your.app.identifier</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>io.identityserver.demo</string>
    </array>
  </dict>
</array>
```

*   `CFBundleURLName`是任何全局唯一的字符串。一个常见的做法是使用你的应用标识符。
*   `CFBundleURLSchemes`是你的应用程序需要处理的一系列 URL 方案。方案是 OAuth 重定向 URL 的开始，直到方案分隔符(`:`)字符。

## **在 AppDelegate 中定义 openURL 回调**

您需要保留 auth 会话，以便从重定向继续授权流。请遵循以下步骤:

使`AppDelegate`符合`RNAppAuthAuthorizationFlowManager`，对`AppDelegate.h`做如下修改:

```
+ #import "RNAppAuthAuthorizationFlowManager.h"- @interface AppDelegate : UIResponder <UIApplicationDelegate>
+ @interface AppDelegate : UIResponder <UIApplicationDelegate, RNAppAuthAuthorizationFlowManager>+ @property(nonatomic, weak)id<RNAppAuthAuthorizationFlowManagerDelegate>authorizationFlowManagerDelegate;
```

从`AppDelegate.m`中的`UIApplicationDelegate`改变以下方法:

```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *, id> *)options {
 return [self.authorizationFlowManagerDelegate resumeExternalUserAgentFlowWithURL:url];
}
```

**安卓**

成功链接后，您应该添加**Android/app/build . grandle**文件 defaultConfig 值作为您的标识符重定向 url。

```
manifestPlaceholders = [appAuthRedirectScheme: “io.identityserver.demo”]
```

## 使用

```
import { authorize } from 'react-native-app-auth';// base config
const config = {
  issuer: '<YOUR_ISSUER_URL>',
  clientId: '<YOUR_CLIENT_ID>',
  redirectUrl: '<YOUR_REDIRECT_URL>',
  scopes: ['<YOUR_SCOPES_ARRAY>'],
};// use the client to make the auth request and receive the authState
try {
  const result = await authorize(config);
  // result includes accessToken, accessTokenExpirationDate and refreshToken
} catch (error) {
  console.log(error);
}
```

*如果你觉得这篇文章很有帮助，你可以通过使用我的推荐链接注册一个* [***中级会员来访问类似的文章***](https://melihyumak.medium.com/membership) *。*T38

***关注我*** [**推特**](https://twitter.com/hadnazzar)

![](img/e09adde9fd734db2f987c8df72839da8.png)

在 [Youtube](https://www.youtube.com/c/TechnologyandSoftware?sub_confirmation=1) 上订阅更多内容

# 编码快乐！

梅利赫
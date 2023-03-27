# OAuth 2 与 React 本地保护博览会

> 原文：<https://itnext.io/oauth-2-with-react-native-keeping-expo-18e8bfbb71c1?source=collection_archive---------3----------------------->

![](img/9e12bcbc3b2fce1e6c12cf2ffdbcb52a.png)

我是一个无摩擦的人，开发人员，主要是因为**我喜欢创建新应用**的过程，我喜欢使用它们并享受这个过程，试图**尽可能避免花费在手动部署配置过程上的时间**。基本上，**比起解决配置问题，我更喜欢花时间去做这些事情**。

[React Native](https://facebook.github.io/react-native/) (RN)是移动开发领域的一项改变游戏规则的技术，为了将 React 代码透明地转换为本机代码，在后台发生了许多神奇的事情。

但是在没有 Expo 的情况下使用 RN 会让你不得不处理 Xcode、Android Studio 来生成包并将你的应用部署到 Play 和 App Store，这意味着不得不处理更多依赖于两个完全不同的平台的配置 [cocoapods](https://cocoapods.org/) 和 [swift](https://developer.apple.com/swift/) 用于 iOS，以及 [gradle](https://gradle.org/) 和 [Java](https://java.com/en/) 用于 Android。

这就是为什么 [Expo](https://expo.io/) 的存在是有意义的。

# 为什么与 RN 合作时如此需要 Expo

我不得不说,`[create-react-native-app](https://github.com/react-community/create-react-native-app)`的默认输出产生了一个 React Native + Expo 项目，所以我们不需要做任何特殊的事情来启动和运行 Expo。

对我来说，在 expo 上设置项目是有意义的，原因如下:

*   解决测试脚手架问题。
*   允许您在真实设备上执行和使用应用程序，而无需将您的设备设置为开发设备
*   生成`[.ipa](https://en.wikipedia.org/wiki/.ipa)` (iOS 应用商店包)和`[.apk](https://en.wikipedia.org/wiki/Android_application_package)` (Android 包)，通过他们的服务器部署到应用商店，而不必处理 XCode(这使得必须拥有 Mac 电脑)和 JDK。
*   附加功能，如隧道部署(允许网络外的设备在开发期间尝试您的应用)。
*   一个工具，只需点击一下，就可以通过 CLI 或浏览器打开仿真器。
*   并提供他们自己的 [SDK](https://docs.expo.io/versions/latest/sdk) 以透明的方式处理特定设备的可能性。

正如我之前所说的，最好的一点是，**配置是通过** `**create-react-native-app**`解决的

# 我们的第一个身份认证解决方案

在[前期](https://www.precursive.com/)，当我们开始在 React 原生应用程序中使用 [Apex Rest API](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_rest_intro.htm) 时，我们还需要以不同于我们在 Salesforce 开发中使用的任何其他应用程序的方式来解决身份验证问题。

我试着遵循官方文档，在 Salesforce 的案例中，使用了 [Salesforce Mobile SDK](https://developer.salesforce.com/developer-centers/mobile) 和`forcereact create`，正如他们的`[trailhead](https://trailhead.salesforce.com/en/modules/mobile_sdk_react_native/units/mobilesdk_reactnative_create_app)`中所描述的

这个`react-native-force`模块提供了 4 个带有有用的 Salesforce 实用程序`'import {oauth, network, smartstore, smartsync} from 'react-native-force';`的包，我们可以使用`oauth.authenticate`登录到 Salesforce，但是这个项目没有使用 expo。

> *总之；* ***好而无摩擦的*** *方案但是因为它生成的项目用的是*`*react-native-force*`****它没有用 expo*** *。它使用* `*forceios*` *和* `*forceandroid*` *原生模块生成依赖的 iOS 和 Android 代码。**

# *带 RN 的其他 O Auth2*

*Salesforce 为[应用提供 OAuth](https://help.salesforce.com/articleView?id=remoteaccess_authenticate.htm) 认证，因此 OAuth 看起来像用于认证的标准。*

*所以我检查了不同的 RN 库来解决这个问题。*

*   *[OAuth 登录 React Native](https://github.com/adamjmcgrath/react-native-simple-auth) 。这个工具为社交网络(谷歌、脸书、推特和 Tumblr)提供了简单的设置。*
*   *[react-native-oauth](https://github.com/fullstackreact/react-native-oauth) 为 OAuth 1.0 和 OAuth 2.0 提供接口，支持 react 原生应用的以下提供者(Twitter、脸书、Google、Github 和 Slack)*
*   *[React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth)React Native bridge for App Auth——一个用于与 OAuth2 提供者通信的 SDK，**这是我认为看起来更标准的一个，**。*

***但是它们都使用本地模块，expo 目前不支持它们**，这意味着我们需要[弹出](https://docs.expo.io/versions/latest/expokit/eject)应用程序来安装这些库，这意味着项目中没有 Expo😢。*

# *我们的“不要离开世博会”解决方案*

*因此，在接受 OAuth 库与 Expo 不兼容的事实之前，我检查了 Expo SDK，发现了一些可以用来解决我们的问题的东西。*

*这个包已经包含在 Expo 中，所以基本上，我们只需要用它来登录。*

*这里有一个简单快捷的`SignInScreen`组件，显示如何使用 Expo Auth-session 登录到 Salesforce(当然，我们还需要按照 [SF 说明](https://developer.salesforce.com/page/Connected_Apps)正确设置连接的应用程序)。*

```
*import React, { Component } from 'react';
import { SF_OAUTH_URL, REMOTE_ACCESS_CONSUMER_KEY } from 'react-native-dotenv';
import { AsyncStorage, Button, View, Text } from 'react-native';
import { AuthSession } from 'expo';
import { globalStyles } from '../constants/Styles';
import ScreenKeys from '../constants/ScreenKeys';export default class SignInScreen extends Component {
  static navigationOptions = {
    title: 'Please sign in',
  };state = {
    errorCode: null,
  };_signInAsync = async () => {
    const { navigation } = this.props;
    const redirectUrl = AuthSession.getRedirectUrl();
    const result = await AuthSession.startAsync({
      authUrl:
        `${SF_OAUTH_URL}` +
        `?response_type=token` +
        `&client_id=${REMOTE_ACCESS_CONSUMER_KEY}` +
        `&prompt=login` +
        `&redirect_uri=${encodeURIComponent(redirectUrl)}`,
    });
    console.log(result);
    const { type, errorCode = 'You cancel or dismissed the login' } = result;
    if (type === 'success') {
      // Just simple way to store the token in this examples
      await AsyncStorage.setItem('userToken', JSON.stringify(result));
      navigation.navigate(ScreenKeys.main);
    } else {
      /**
       * Result types can be: cancel, dismissed or error (with errorCode)
       */
      this.setState({ errorCode });
    }
  };render() {
    const { errorCode } = this.state;
    return (
      <View style={globalStyles.container}>
        <Button title="Sign in!" onPress={this._signInAsync} />
        {errorCode ? <Text>{errorCode}</Text> : null}
      </View>
    );
  }
}*
```

> **通过这些步骤，* ***我们可以通过 RN 应用程序登录到 Salesforce 组织，然后使用*** `***Apex Rest API***` *并像任何其他标准 Rest 服务器一样使用 Salesforce 服务器，使我们的开发工作成为* ***标准和无摩擦的*** *尽可能保持 Expo🚀。**

**原载于*[*robertovg.com*](https://robertovg.com/blog/o-auth-2-with-react-native-keeping-expo)*。**
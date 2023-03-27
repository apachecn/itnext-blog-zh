# 通过 Expo 的本地化模块在 React Native 中支持多种语言

> 原文：<https://itnext.io/supporting-multiple-languages-in-react-native-with-expos-localization-module-9611f9e3b6a0?source=collection_archive---------1----------------------->

![](img/cc5975056854329fa6211ec6237bf7aa.png)

[乔恩·泰森](https://unsplash.com/@jontyson?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的“你好 LED 标牌”

**这是关于使用 React Native 为**[**uncovercity**](https://uncovercity.com/)**构建 iOS 和 Android 应用程序的 4 篇系列文章中的第三篇。你可以在这里找到其他的:**

1.  [使用 Expo 加速 React Native 中惊喜晚餐应用的构建](/speeding-up-the-build-of-a-surprise-dinner-app-in-react-native-using-expo-2c18beabaaa7)
2.  [在 Expo 中对战测试拼车 API 和 React Native 的 MapView](/battle-testing-a-ride-sharing-api-and-react-natives-mapview-in-expo-c0ce5d50cbf0?source=your_stories_page---------------------------)
3.  在 React Native 和 Expo 本地化中支持多语言
4.  [使用 react-navigation 在 React Native 中嵌套导航器的复杂性](https://medium.com/@marcelkalveram/the-intricacies-of-nesting-navigators-in-react-native-using-react-navigation-fef52ca72964)

嗨！这是另一篇关于我为惊喜晚餐初创公司 [uncovercity](https://uncovercity.com/) 开发 iOS 和 Android 应用的经历的帖子。今天，我将带领大家完成将应用程序的所有内容(文本、视频、图片)从西班牙语翻译成英语的挑战。

由于该应用是基于 [React Native](https://facebook.github.io/react-native/) 和 [Expo](https://expo.io/) 的，最明显的事情就是在这两个工具的生态系统中寻找一些现有的解决方案。

瞧，Expo 有一个[本地化](https://docs.expo.io/versions/v30.0.0/sdk/localization)模块。虽然它仍然在`DangerZone`(Expo 用来对尚未完全批准的功能进行分组的名称空间)中，但我很高兴我的挑战有了一个坚实的起点。

> **重要提示:本文基于世博 30.0.0 及以下版本。世博网近日发布了**[**31 . 0 . 0**](https://blog.expo.io/expo-sdk-v31-0-0-is-now-available-cad6d0463f49)**版本，其中包含了对** [**汉化**](https://docs.expo.io/versions/v31.0.0/sdk/localization) **模块的彻底检修。最显著的变化是，您现在可以同步地从您的设备中检索本地化信息，而不是我将在这篇博文**中介绍的异步方式

如果您已经熟悉了基础知识，您可以直接跳到[常见陷阱](#ddd6)，在那里我将讨论一些更高级的本地化主题。

> 如果你在没有 Expo 的情况下使用 React Native，你可以用 [react-native-languages](https://github.com/react-community/react-native-languages) 得到类似于本地化模块的东西。然而，对于字符串翻译的实际任务，你仍然需要类似于 [react-i18next](https://github.com/i18next/react-i18next) 的东西。

# 本地化的基础

Expo 为这个模块提供的[文档非常简单和复杂(这也是我写这篇文章的动机)。但是实现它实际上非常简单。你需要做的就是:](https://docs.expo.io/versions/v30.0.0/sdk/localization)

1.  为您想要提供的每种语言创建一个 JSON 文件
2.  使用 Expo 的`Localizaton`模块初始化语言存储
3.  设置用户在商店中的当前语言
4.  将应用中的所有内容替换为应用商店的相应密钥

> **要查看下面代码的完整示例，请查看我创建的** [**点心**](https://snack.expo.io/@marcelkalveram/localizaton-example) **，在这里我从一个单独的文件初始化一个字符串存储，并添加一个小切换按钮在美国英语和西班牙语之间切换。**

现在，让我更详细地向您介绍这些步骤:

## 1.为每种语言创建一个 JSON 文件

您需要做的第一件事是将所有可翻译的字符串放入 JSON 存储中。无论您是在同一个文件中这样做，还是使用`import`从一个单独的文件中导入它们，都没有关系。我会在第二步解释为什么我更喜欢后者。

```
const **localizedStrings** = {
  en_GB: { title: "Hello", subtitle: "Welcome" },
  es_ES: { title: "Hola", subtitle: "Bienvenido" }
}
```

这个对象的关键字是语言/地区标识符(即代表英语/英国)和一个键的值是所有可翻译的字符串，在商店被初始化后你就可以使用了，例如。如果您的语言设置为西班牙语(`es_ES`)，则`localeStore.subtitle`将输出“Bienvenido”。

## 2.使用 Expo 的本地化模块初始化商店

其次，我们初始化存储并在其上运行初始化逻辑。像这样从 Expo 导入`Localization`模块:

```
import { DangerZone } from 'expo'
const { **Localization** } = DangerZone// localizedStrings is the object we created in step 1
const localeStore = new Localization.LocaleStore(**localizedStrings**)
```

如果将这行代码放在本地化字符串所在的位置，会更容易，这样您就可以在任何需要的地方访问它们。因此，不是直接导出 JSON 对象，而是像这样导出实例化的存储:

```
const localeStore = new Localization.LocaleStore(**localizedStrings**)
**export default** localeStore;
```

这个`localeStore`现在给我们提供了两个重要的功能:

*   设置我们希望向用户显示的当前区域设置(参见下一步)
*   像这样在我们的视图中使用本地化的字符串:`{ **localeStore**.title }`

## 3.设置用户在商店中的当前语言

接下来，我们需要弄清楚用户的首选语言是什么。我们可以使用来自`Localization`模块的`getCurrentLocaleAsync`函数，它返回一个语言/地区标识符。然后，我们使用`setLocale`将这个值传递给我们之前初始化的存储器。

```
import { localeStore } from "./localeStore"
...
constructor() {
  super();
  **Localization.getCurrentLocaleAsync()**.then(currentLocale => 
    **localeStore.setLocale(locale)**
  );
}
```

`getCurrentLocaleAsync` 函数以字符串的形式返回用户设备的语言/地区标识符(您现在应该很熟悉了)，即:`en_GB`这是一个异步调用，因此我们需要确保无论我们将它放在哪里，它都需要以这种方式执行，要么使用`async` / `await`要么使用`Promise`。

> 请记住，除了使用`*getCurrentLocaleAsync*`，您还可以通过使用各自的功能`*getCurrentDeviceCountryAsync*`或`*getPreferredLocalesAsync*`来访问设备的**国家代码**或**首选地区**。

除此之外，您可能希望使用某种状态管理解决方案或 React Context API 来使您的当前区域设置在其他地方可用，而不仅仅是在根组件的状态中。

## 4.替换所有可翻译的字符串

最后但同样重要的是，我们需要确保在我们的 UI 视图中使用来自我们商店的本地化字符串，而不是硬编码的字符串。

```
// we want to replace this...
<Text>**Hello**</Text>// ...with that:
<Text>**{ localeStore.title }**</Text>
```

就国际化而言，这再简单不过了。对于更高级的翻译功能，比如复数和格式，你可能想使用更完整的库，比如`react-i18next`(顺便说一下，[与 Expo 的`Localization`模块结合使用效果非常好](https://github.com/i18next/react-i18next/tree/master/example/reactnative-expo)，但是也更复杂)。

# 常见陷阱及其修复

这听起来很简单。但是有一些不太明显的问题，只有当你构建一个完全本地化的应用程序，有多个屏幕和一点互动时，你才会遇到。

## 设置和应用默认语言

您可能想知道如果用户的语言不在您的任何翻译文件中，而您在试图设置的**区域**不存在的地方做了类似`setLocale(**locale**)`的事情，会发生什么。

或者，如果您想为`en_GB`和`en_US`提供相同的翻译，但不想复制整个字符串数组，该怎么办？

对我来说，最简单的解决方案是根据某些条件显式设置区域设置，而不是盲目地将区域设置传递给商店的`setLocale`函数。

```
**// instead of blindly relying on this...** Localization.getCurrentLocaleAsync().then(currentLocale => 
  localeStore.setLocale(currentLocale)
);**// ...make sure to explicitly set one of your existing locales** Localization.getCurrentLocaleAsync().then(currentLocale => {
  const **locale** = currentLocale.includes('es') ? '**es_ES**' : '**en_GB**'; 
  localeStore.setLocale(**locale**)
);
```

除此之外，在我们没有首先指定用户的默认区域设置之前，您可能想要保护您的视图不呈现任何内容:

```
if (!**this.state.currentLocale**) {
  return;
}return <App />
```

## 在语言之间切换

现在我们知道了如何设置默认语言，如果用户真的想切换到另一种语言呢？

在`localeStore`上调用`setLocale`并将我们的状态更新为新的语言应该可以做到:

```
localeStore.setLocale(**newlocale**)
```

遗憾的是，我们的视图不会因此而更新。多么令人失望！

那是因为`localeStore`更新发生在 React 状态的领域之外，所以 React 不会重新呈现任何东西。另外，`setLocale`是一个异步调用，所以我们希望在更新发生后应用的任何更改都需要在回调函数中进行处理，回调函数将它作为第二个参数。

```
localeStore.setLocale(locale, 
  **() => this.setState(() => ({ currentLocale: locale })))**
});
```

现在，我们的组件将对状态变化做出反应并相应地更新。

## 带反应导航的导航标题

如果您以前使用过 react-navigation，您可能对静态的`navigationOptions`对象很熟悉，它允许您修改标题栏的样式和内容。

原来，因为它是一个静态属性，所以你在它里面设置的任何属性都将在我们设置初始语言之前被评估，而不是在实际的屏幕被安装时。

```
static navigationOptions = {
  title: localeStore.title **// this won't re-render**
}
```

这里的解决方案是返回一个返回对象的函数。就像这样，每次我们的应用程序发生状态变化时(就像由`setLocale`回调触发的那样)，这些道具也会更新。

```
static navigationOptions = () => {
  **return {**
    title: localeStore.title **// this uses the updated locale**
  **}**
}
```

# 高级本地化解决方案

Expo 的本地化模块相当有限。它允许您访问用户设备的语言，并允许您基于该语言翻译字符串对象。

对于许多较小的项目来说，这可能就是您所需要的全部。但是，如果你的应用变得比这更复杂，你需要国际化选项，如占位符、多元化和货币/时区格式，你可能需要考虑以下更复杂的解决方案之一:

[](https://github.com/i18next/react-i18next) [## i18next/react-i18next

### react 的国际化做对了。使用 i18next i18n 生态系统。— i18next/react-i18next

github.com](https://github.com/i18next/react-i18next) [](https://github.com/fnando/i18n-js) [## fnando/i18n-js

### 这是一个小库，提供 Javascript 上的 I18n 翻译。它带有 Rails 支持。— fnando/i18n-js

github.com](https://github.com/fnando/i18n-js) [](https://github.com/yahoo/react-intl) [## 雅虎/react-国际

### 国际化 React 应用。这个库提供了 React 组件和一个 API 来格式化日期、数字和字符串…

github.com](https://github.com/yahoo/react-intl) 

有没有把一个 React 原生 app 翻译成多国语言的？你的经历是怎样的？您使用了哪些工具，遇到了什么样的问题？

希望你喜欢这篇文章，并发现它很有帮助。如果您有任何反馈，请随时添加评论。谢谢！
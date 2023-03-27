# 如何在 React-Native & Expo 中使用 i18next 本地化

> 原文：<https://itnext.io/how-to-use-i18next-in-react-native-expo-476fa80bd92c?source=collection_archive---------2----------------------->

![](img/f1577dbd4fc1128e34c901bc346f9489.png)

Jelleke Vanooteghem 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# 本土化

目前，本地化对于许多应用程序和开发项目来说非常重要。几乎每个项目都可以选择多种语言。此外，在移动应用程序中，开发人员试图通过多种语言选项为用户提供最佳的用户体验。

您可以定义您自己的 **json** ，并创建尽可能多的您想在应用程序中使用的翻译。那么如何在 react-native 或 react native 应用程序中开始使用它们呢？

# 设置

将**i18 下一个**安装到您的应用程序中

```
# npm**npm install i18next — save****npm install react-i18next --save**# yarn**yarn add i18next****yarn add react-i18next**
```

创建应用程序后，创建 **i18n.js** 文件。

下面是您的 i18n.js 文件的样子。

完成所有这些安装后，您可以开始在您的组件和文件中使用翻译。

在你的 **app.js** 中导入如下的 i18n.js 文件

```
**import ‘./i18n’;**
```

然后，当您导出组件或类时，您应该使用

```
**withNameSpaces()(ComponentName)**
```

# 使用

在所有的设置过程之后，你的内在化就可以使用了！

```
const {t} = this.props;{t(‘foo’)}
```

当您的应用程序语言是英语时，将为您提供**“en”**版本的 **foo** 值。

```
// or: using a function to return namespaces based on propsexport default withNamespaces((props) => props.namespaces)(TranslatableView);
```

*如果你觉得这篇文章很有帮助，你可以通过使用我的推荐链接注册一个* [***中级会员来访问类似的***](https://melihyumak.medium.com/membership) *。*

***关注我的*** [**推特**](https://twitter.com/hadnazzar)

![](img/e09adde9fd734db2f987c8df72839da8.png)

在 [Youtube](https://www.youtube.com/c/TechnologyandSoftware?sub_confirmation=1) 上订阅更多内容

# 编码快乐！

梅利赫
# React 本机导航——老派的实现

> 原文：<https://itnext.io/react-native-navigation-the-old-school-implementation-917e6a0517f8?source=collection_archive---------2----------------------->

![](img/a781e9b942509ff30c26f983bac71be5.png)

继续“老派”系列实现([第一篇文章在此](https://medium.com/@BrodaNoel/state-manager-alternative-a-simple-old-school-library-b06893f4974c))，在`duix`的帮助下，今天我将向您展示如何在 React Native 中实现一个简单的导航模式。因为，记住，`duix`也可以用在 React Native，或者任何另一个 JavaScript 项目中。

## 方法

方法很简单。我们将在`duix`商店中定义一个变量，称为`currentPage`。我们将定义一个默认值指向“加载”页面，然后我们将在每次想要导航时更改它。

## 这个例子

我实现了 App.js 文件的代码，我还实现了“登录”和“主页”,其中我们将有一些模拟按钮的文本，当按下按钮时，它们将登录/注销用户并导航到正确的下一页。

登录页面有一个导航到主页的文本按钮。主页有一个文本按钮，可以再次导航到登录页面。

`App.js`主要组成文件:

App 主要组件

这里，`Login.js`组件:

登录组件

这里，`Home.js`组件文件:

家用部件

那都是男生！我希望你喜欢这种简单。

`npm`和`GitHub`:

[](https://www.npmjs.com/package/duix) [## duix

### 只是一个简单的状态管理器

www.npmjs.com](https://www.npmjs.com/package/duix) [](https://github.com/BrodaNoel/duix) [## 布罗达尼埃尔/杜伊克斯

### 一个专注于 KISS 和 Pareto 原则的州经理——BrodaNoel/duix

github.com](https://github.com/BrodaNoel/duix)
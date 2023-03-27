# 如何在 VueJS 中创建类似苹果的上滑面板

> 原文：<https://itnext.io/how-to-create-apple-like-slideup-panels-in-vuejs-d0d74438139d?source=collection_archive---------1----------------------->

![](img/22a5d7c4c1c29b695dd9487eefbd2ee7.png)

众所周知，苹果的设计和 UI/UX 非常好，非常漂亮，所以如果你想使用 iOS 的一些功能，即上滑面板，你不必重新发明轮子。

当我在开发我的应用程序时，这只是一个普通的 web 应用程序，但对移动设备有反应，我经常有这样的想法，当用户必须选择一些东西时，就拉起“模态”或某种弹出菜单，比如选择用户、网球场(对我来说)或其他东西。所以我有了更多的选择，我不想在一个“视图”中拥有所有的选择。第一个想法是从左边开始的滑动菜单，就像所有移动应用程序使用的下拉菜单一样。它工作得很好，但我仍然觉得这不是我想要的，所以我寻找更有吸引力的东西。当我和 VueJS 一起工作时，我经常访问[http://vuejsexamples.com](http://vuejsexamples.com)，这是 Vue 新插件的一个很好的来源，我偶然发现了“[一个 Cupertino 库](https://vuejsexamples.com/a-vue-3-wrapper-for-cupertino-library/)的 Vue 3 包装器——这正是我所需要的，但它只适用于 Vue 3，我需要这个插件用于 Vue2。所以我做了一点研究，发现 [cupertino-pane](https://github.com/roman-rr/cupertino-pane) 有很棒的 api，非常适合我。我花了一点时间来让它工作，但是最后，它很简单，所以我想和你分享这个。

这是一个起点，你可以复制粘贴这些代码来让你的面板进入工作模式。

```
<template> <div class="panel"> <h1>Header</h1> <p hide-on-bottom>Content</p> </div></template><script>import { *CupertinoPane* } from 'cupertino-pane';export default { data() { return { settings: {}, drawer: {}, }; }, mounted() { *this*.*drawer* = new CupertinoPane('.panel', *this*.*settings*); setTimeout(() => *this*.*drawer*.present({animate: *true*})); }, methods: { presentDrawer() { *this*.*drawer*.present({animate: *true*}); }, destroyDrawer() { *this*.*drawer*.destroy({animate: *true*}); }, hideDrawer() { *this*.*drawer*.hide(); }, isHiddenDrawer() { *console*.log(*this*.*drawer*.isHidden()); } },};</script><style></style>
```

我希望这对你有一点帮助。如果你阅读文件，你会发现有很多选择，所以把手弄脏。
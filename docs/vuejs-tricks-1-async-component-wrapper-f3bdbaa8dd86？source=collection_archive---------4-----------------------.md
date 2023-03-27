# # vuejs 诀窍#1:异步组件包装器

> 原文：<https://itnext.io/vuejs-tricks-1-async-component-wrapper-f3bdbaa8dd86?source=collection_archive---------4----------------------->

![](img/4102ebea6c828473d7c306a50c062c73.png)

我要讲的是如何在 Vue.js 中异步加载组件。

这其实很简单。你可以在下面的链接中看到一个例子。

[](https://vuejs.org/v2/guide/components-dynamic-async.html#Handling-Loading-State) [## 动态和异步组件- Vue.js

### 本页假设您已经阅读了组件基础知识。如果您不熟悉组件，请先阅读该文档。早些时候，我们…

vuejs.org](https://vuejs.org/v2/guide/components-dynamic-async.html#Handling-Loading-State) 

现在，我们来谈谈如何编写一个包装器组件。几天前，我想写一个包装器组件，并在任何地方使用它。同时，我想使用所有关于组件的能力，如道具，事件。

所以我需要将所有的“道具”转移到真正的子组件，将所有的“监听器”转移到父组件。

此代码块中有 2 行导入代码。

```
v-bind.sync="$attrs"
```

在上面的代码中，你说把所有的道具转移给孩子。

```
v-on="$listeners"
```

在上面的代码中，您说将所有侦听器转移到父节点。

由于这些行，您可以像直接使用一样使用任何组件。

示例用法:

你可以在 github 中看到完整的例子。

[](https://github.com/mazlum/vue-async-component-example) [## mazlum/vue 异步组件示例

### Vuejs 示例如何编写异步组件包装器。-maz lum/vue-异步组件-示例

github.com](https://github.com/mazlum/vue-async-component-example)
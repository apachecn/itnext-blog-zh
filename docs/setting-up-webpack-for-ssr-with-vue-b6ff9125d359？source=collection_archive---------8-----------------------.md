# 为带 Vue 的 SSR 设置 webpack—带 Vue 的 SSR 和 web pack 从头开始(2/3)

> 原文：<https://itnext.io/setting-up-webpack-for-ssr-with-vue-b6ff9125d359?source=collection_archive---------8----------------------->

![](img/9d1168ed9abe7d1c13f77e4cfc8cce1d.png)

Vue + Webpack |图片来自 npmjs

在本文中，我将继续我上一篇文章中的[工作，在那里我为 Vue 从头开始设置了一个 webpack 配置。我现在将添加对服务器端渲染的支持，用`vue-server-renderer`。](https://medium.com/@lachlanmiller_52885/webpack-config-from-scratch-for-vue-a422672fc04c)

服务器端呈现是服务器使用 Node.js 动态构建应用程序的 HTML，然后在响应中发回新呈现的 HTML。这与客户端呈现相反，在客户端呈现中，webpack 捆绑的 JavaScript 对于客户端是原样的，在客户端由浏览器 JavaScript 引擎处理。这两种方法都有好处，本文不会讨论。

源代码可以在这里找到[。](https://github.com/lmiller1990/webpack-simple-vue/tree/add_server_rendering)

[![](img/8860e8ef1f39967845929ca6a9e3821a.png)](http://vuejs-course.com/)

来看看我的 [Vue.js 3 课程](https://vuejs-course.com/)！我们涵盖了组合 API、类型脚本、单元测试、Vuex 和 Vue 路由器。

## 拆分`webpack.config.js`

根据应用程序是呈现在服务器上还是客户端上，需要不同的 webpack 配置。我们希望两者都支持——对于开发来说，`webpack-dev-server`是一个强大的工具，它将处理和渲染委托给客户端。在生产中，我们将在服务器上渲染。许多 webpack 配置可以共享，比如`module`，我们在这里声明了`loaders`。为非唯一 webpack 设置创建一个文件夹和两个新文件:

当前`webpack.config.js`包含四个属性:

1.  进入
2.  组件
3.  插件
4.  开发服务器

`module`包含`.vue`文件的加载规则，这是服务器和客户端渲染都需要的。其余的属性在客户端渲染中将是唯一的，因此将它们移动到`config/client.js`:

在`config/server.js`中添加一个最小设置:

同样，将`template.html`移入`config`:

并更新`webpack.config.js`:

```
const server = require("./config/server")
const client = require("./config/client")module.exports = {
  // ...
  plugins: client.plugins.concat(VueLoaderPlugin()) // ... 
}
```

应该还在工作。

# 添加`webpack-merge`

现在`webpack.config.js`有些重复。我们必须通过键入以下内容来加入基本配置和客户端

当我们添加一些服务器配置时，它将看起来像这样:

这很快就变得令人困惑。有一个更好的方法:`webpack-merge`，它将为我们处理合并。

现在使用`webpack-merge`来清理`webpack.config.js`:

现在`module.exports`返回一个函数。Webpack 检查从`webpack,config.js`导出的函数是否存在，如果存在，用`mode`参数调用它。奇怪的是，`mode`对应于`--env`参数，而不是`--mode`，所以更新`package.json`中的`scripts`部分:

`npm run dev`应该还能正常工作。如果您访问`localhost:8080`，检查页面的源代码(不使用 devtools，实际的页面源代码)，您应该会看到:

注意`msg`，`Hello`没有出现在这里——那是因为它是在*客户端*上渲染的。在服务器上呈现时，我们将看到不同的页面源。

# 服务器端`entry`

服务器呈现包将使用不同的入口点。为它创建一个文件:

这个函数将创建我们想要渲染的 Vue 应用程序。添加以下代码:

眼熟吗？类似于`src/index.js`中的代码。看一看:

区别是`document.addEventListener...`。Node.js 中没有其他 Web APIs，这就是为什么我们需要两个不同的呈现器。重构`src/index.js`:

最后，更新`config/server.js`以使用`create-app`:

```
// ...
module.exports = {
  entry: "../src/create-app"
}
```

让我们用`npm run build`尝试新的生产配置。

看起来不错。让我们看看是否可以在 Node.js 环境中使用该模块:

我们有麻烦了。

# `output.library`和`output.libraryTarget`

我们需要在`config/server.js`中设置一些`output`选项。`output`的文档在这里是。

我们对`library`和`libraryTarget`感兴趣。默认值为:

`var`意味着 webpack 会将我们的导出分配给`var`，这些导出被附加到`window`对象上，以便在浏览器(UMD)中使用。然而，因为`library`没有定义，所以 webpack 什么也不做。虽然变量没有被赋值，但是在`src/index.js`中我们会这样做:

所以 Vue 应用还是挂载的。要了解这些选项及其作用，请运行

并尝试添加以下内容:

并运行`npm run build`。让我们比较一下这两者:

现在我们的包被分配给一个名为`Bundle`的变量。如果需要，点`cd dist && python -m SimpleHTTPServer`，然后在浏览器控制台中查看:

我们希望在 Node.js 环境中执行。我们我们需要瞄准`commonjs2`。更新`config/server.js`:

并运行`npm run build`。让我们再次用`diff dist/main.js dist/main_2.js`比较输出

看起来不错！现在我们有了`module.exports`。我们可以使用节点:

还有其他东西可以传递给`library`和`libraryTarget`——在[文档](https://webpack.js.org/configuration/output/)中找到更多信息。

# 添加服务器(快速)

既然我们有了在服务器上创建 Vue 应用程序的方法，我们需要以某种方式服务它。要了解这一点，最简单的方法是使用 express 服务器。我个人更喜欢 Rails，并将在以后的文章中深入探讨。

反正加上`express`，和`vue-server-renderer`

用`touch src/server.js`为服务器代码创建一个文件。

这里有两个有趣的部分:

它实例化服务器呈现器实例；

它获取我们的应用程序，对其进行渲染，并以字符串形式返回标记。剩下的工作就是向用户提供标记。更多关于 Vue 服务器渲染器的信息在[这里](https://ssr.vuejs.org/guide/#rendering-a-vue-instance)。

我们可以通过跑步来尝试:

如果一切正常，你现在可以访问`localhost:8000`了，你应该会看到同样的问候信息。但是，使用 devtools 检查元素会显示:

指示它是在服务器上呈现的。让我们比较一下页面源代码和客户端呈现的源代码，如前面所示:

# 客户端渲染:

# 服务器渲染:

通知`<div app>`哪里都没显示？当我们使用服务器渲染器生成标记时，它被替换，因此所提供的只是实际的标记。我们将在未来添加一个正确的`html`模板供服务器端渲染器使用。

# 结论

在本文中，我们介绍了:

*   webpack 可以针对的不同环境
*   如何使用`webpack-merge`
*   如何使用`library`和`libraryTarget`
*   在服务器和客户端之间重构和共享代码

# 改进

还有许多改进，我们将在未来进行介绍，例如:

*   在服务器上水合应用程序(使用一些动态数据如用户 id 的服务器渲染)
*   路由(客户端和服务器)
*   使用 ES6 语法，像 Node.js 中的`import`和`export`用 babel

源代码可以在[这里](https://github.com/lmiller1990/webpack-simple-vue/tree/add_server_rendering)找到。

*原载于* [*拉克伦·米勒的个人博客*](https://lmiller1990.github.io/electic/posts/server_side_rendering_with_vue.html) *。*
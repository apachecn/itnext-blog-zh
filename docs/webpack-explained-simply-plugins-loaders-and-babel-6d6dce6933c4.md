# Webpack 解释得很简单——插件、加载器和巴别塔

> 原文：<https://itnext.io/webpack-explained-simply-plugins-loaders-and-babel-6d6dce6933c4?source=collection_archive---------0----------------------->

![](img/bb357494be2dd795bc419a76bdf07330.png)

Webpack 席卷了 web 开发世界。自发布以来，它被 Airbnb 和脸书等大公司以及 Angular 和 React 等 web 开发框架所采用。虽然很明显 Webpack 在当今的 web 开发中起着重要的作用，但是直到最近我才弄清楚 Webpack 的确切特性和功能。所以在这篇文章中，我尽可能用最简单的方式来解释 Webpack。

# 什么是 webpack？

简单地说，Webpack 是一个工具，用于将您的代码放入处理管道，并将其打包成一个 JavaScript 文件。

那么我们在谈论什么样的处理呢？为了解释这一点，我们需要回到长期存在的 JavaScript 模块冲突问题。在 Node.js 之前，开发人员必须使用不同的技巧来避免 JavaScript 中的变量名冲突。Node.js 推出了一个很棒的解决方案——CommonJS 模块化系统。不幸的是，浏览器本身并不支持 CommonJS。这就是 Webpack 的用武之地。我们可以使用 Node.js 模块化系统和类似`require()`和`module.exports`的东西来编写我们的应用程序，并使用 Webpack 将其捆绑到一个与所有浏览器兼容的 JavaScript 文件中。让我们看一个简单的例子，以便更好地理解:

想象一下，您有一个包含两个文件`greet.js`和`main.js`的项目。下面是`greet.js`的内容:

该模块使用`module.exports`公开一个函数。该函数简单地返回带有问候字符串的`h2`标签。现在让我们来看看`main.js`:

我们的`main.js`脚本使用`require`导入`greet.js`，这也是特定于 Node.js 模块化系统的。然后，这个脚本调用我们的 greet 函数，并将结果存储在当前 html 页面的`body`标签中。

如果您试图在浏览器中运行这段代码，它肯定会失败。这就是为什么我们需要使用 Webpack。在你的项目文件夹中运行这个命令:`webpack main.js -o bundle.js`，它将捆绑`main.js`及其所有依赖项。

最后一步，让我们创建一个简单的 html 页面，并将我们的包插入其中:

如果你打开我们的 html 页面，你应该会看到问候。使用 Node.js 系统导出和导入模块不是很容易吗？我们不必担心管理依赖项或配置路径，我们所要做的就是导出我们的`greet`模块并使用`require`导入它。

# 加载程序和插件

因为 CommonsJS 可以说是 JavaScript 世界目前提供的最好的模块化系统，仅此一点就足以开始使用 Webpack。然而，Webpack 是一个非常灵活和强大的工具，可以提供更多的功能。Webpack 最突出的功能之一是插件和加载器。

Webpack 中的加载器和插件允许在捆绑过程中添加更多的规则或处理管道。内置和第三方加载器是 Webpack 如此强大和流行的主要原因之一。

虽然乍一看，它们似乎是用来完成相同的事情，但在 Webpack 中，加载器和插件是不同的:

**装载机**:

*   在包生成之前或开始时工作
*   加载器在单个文件级别工作

**插件**:

*   在包生成结束时或之后工作
*   插件在包级别工作
*   插件对捆绑包有更多的控制

## 巴贝尔装载机

Babel loader 用于将以现代风格和 JavaScript 超集编写的代码转换为受旧浏览器支持的普通旧 JavaScript 代码。由于 Babel loader，我们可以享受新的 JavaScript 语法，并使用 EcmaScript 2015 甚至 JSX (React)编写我们的代码。让我们来看一个示例`webpack.config.js`文件，它是一个配置文件，用于声明 Webpack 在您的项目中使用的所有插件和加载器:

正如您所看到的，这个模块所做的只是导出一个配置文件，当您运行命令行来传输您的代码时，Webpack 将使用这个文件。这个文件有`entry`和`output`字段，它们决定了传输器的输入和输出文件的位置。很简单。

`loaders`子字段是我们定义 Webpack 使用的加载器的地方。这里我们只定义`babel-loader`。`test`字段接受正则表达式模式或目录位置，它决定哪些文件将受到加载程序的影响。在这种情况下，位于`src`文件夹中的所有文件都会受到我们的加载程序的影响。

Babel loader 使用预设来处理不同的 JavaScript 风格和超集。Babel 有很多不同的预设。在这种情况下，我们使用`es2015`预设将任何 EcmaScript 2015 代码转换成普通 JavaScript。`presets`数组可以包含不止一个预置，例如我们可以添加`react`预置来在我们的项目中编写 JSX 代码。

## **丑陋插件**

如前所述，插件通常在 transpiling 过程的末尾执行，并在包级别工作。

让我们来看看[丑陋的](https://www.npmjs.com/package/uglifyjs-webpack-plugin/v/1.3.0)插件。这个插件用来混淆你的代码和缩小生成的包。它通过将变量的名称改为较短的名称并从代码中删除空格来实现这一点。下面是你如何在你的`webpack.config.js`中定义一个插件:

在上面的代码中，我们声明了用于定义插件设置的`uglifyJSPlugin`对象。我们设置了`beautify: false`,这样我们生成的包的代码就不会被格式化，这使得恶意用户更难阅读。我们还指定了`dead_code: true`，它将从捆绑包中删除所有的死代码，这在设置动态代码分支时非常有用，也有助于减少捆绑包的大小。

**注意:**为了让上面的代码工作，你需要使用 npm 在你的项目中单独安装巴别塔加载器、预置和丑陋插件。

# Webpack 的其他优势

我们已经看到了足够多的东西来说服我们 Webpack 是多么有用。然而，以下是 Webpack 成为如此伟大的工具的几个原因:

*   Webpack 为许多核心 Node.js 模块提供了浏览器兼容版本。这意味着我们可以使用以前只有 Node.js 才有的模块，比如`http`或者`events`。
*   我们可以从构建中排除不兼容的模块，或者用兼容的模块替换它们。Webpack 可以配置为在构建期间交换模块。
*   您可以从任务管理工具(如 Gulp 和 Grunt)调用 web pack。
*   Webpack 可以用来捆绑不同种类的文件，而不仅仅是 JavaScript。

在这篇短文中，我们讨论了 Webpack 的主要职责和特性。我们学习了 Webpack 中加载器和插件的基本知识以及它们的区别。我们还讨论了为什么 Webpack 对于 web 开发来说是如此重要的工具。

虽然还有其他的 transpiling 工具，如 [Browserfy](http://browserify.org/) 和 [RollupJS](https://rollupjs.org/guide/en/) ，但由于其强大的功能，Webpack 是当今最广泛采用的 transpilers 之一。

感谢您的阅读！

*原载于 2019 年 6 月 22 日*[*https://isamatov.com*](https://isamatov.com/webpack-explained-simply-plugins-loaders-and-babel/)*。*
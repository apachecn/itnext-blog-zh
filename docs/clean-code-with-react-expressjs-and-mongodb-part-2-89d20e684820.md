# 使用 React、ExpressJS 和 MongoDB 清理代码—第 2 部分

> 原文：<https://itnext.io/clean-code-with-react-expressjs-and-mongodb-part-2-89d20e684820?source=collection_archive---------5----------------------->

![](img/188317d984598b0ab1a480cd7db92057.png)

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fclean-code-with-react-expressjs-and-mongodb-part-2–89d20e684820%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

这是一个简短系列文章的第二部分，重点是使用 ES6+ Javascript、React、Redux、ExpressJS 和 MongoDB 创建和部署一个准系统的全栈博客站点。

在这些文章中，我真的试图把重点放在我发现的使用 ES6+ JavaScript 和我称之为“Redux 风格”的项目结构来设置和组织我的代码的一些方法上。目标是编写可读、可扩展且易于测试的代码。

这个系列将分为几个部分:

1.  [设置 ExpressJS 使用 ES6+语法](/clean-code-with-react-expressjs-and-mongodb-part-1-aa6b1a4aef82)
2.  Redux 风格的文件结构👈🏽*你在这里*
3.  [创建简单快速服务器](/clean-code-with-es6-expressjs-reactjs-part-3-2306b1f62c26)
4.  用我们的博客设置 Mongoose(即将推出！)
5.  用 CMS 创建一个 React 博客(即将推出！)
6.  将 Redux 添加到您的前端(即将推出！)
7.  包装您的应用:部署(即将推出！)

# 第 2 部分:Redux 风格的文件结构

现在我们已经设置好了 ES6+，让我们来讨论一下如何使用一种我称之为“Redux-Style”的技术来组织我们的服务器，以确保代码准确且有意义。该方法*不是*专用于 Redux，而是*与* Redux 共同使用。

使用 ES6+语法，如果一个目录被引用(并且*不是*一个特定的文件)，节点在那个目录中寻找一个`index.js`文件来告诉它做什么。目标是从同一个目录*到*到`index.js`中的其他文件导出函数和常量。我们可以使用这个特性来收集我们所有的函数和常量，并通过`index.js`导出它们。

让我们来看看实际情况:我们将在我们的`📂configs/`目录中创建一个名为`server.js`的文件和另一个名为 `index.js`的文件。我们的文件结构现在应该是这样的:

```
📁/
 📂configs/
  📄index.js
  📄server.js
 📂database/
 📂routes/
 📄index.js
 📄.babelrc
 📄package.json
 📄yarn.lock
```

将以下代码添加到各自的文件中:

*。/configs/server.js*

```
export const configureServer = () => { console.log('Server Configs!') }
```

*。/configs/index.js*

现在，将下面的代码添加到**根目录**中的`index.js`中。注`import`语句带空`{}`:

```
import {} from './configs'
```

在大多数像 VS Code 和 Atom 这样的 ide 中，如果你开始在 import 语句`configureServer()`的`{}`中输入我们的函数名，你会看到它会自动建议这个函数。哦哦！但这不仅仅是神奇的🧙🏽‍♀️.这是一个很好的方法来验证我们的常量和函数有正确的名字！

目前，我们只是在命名我们的函数之前使用`export`从`server.js`中导出`configureServer()`函数，但是我们也将创建其他具有常量的文件，并将它们收集在同一个`index.js`中，以便在其他文件中导出和使用。

这是 Redux 中组织代码的一种流行方式，我觉得它很棒，原因有二:

1.  它使我们的代码模块化。这使得测试更加容易。
2.  它允许验证函数和常数的名称是否正确。大多数文本编辑器，如 VS Code 和 Atom，会自动填充目录中的可用导出。

如果你使用过 React，你可能很熟悉将`import`和`{}`一起使用。语法类似于 ES6 中的*析构*，工作方式也类似，尽管它们是不同的东西。我喜欢这样想:

*   **如果没有** `{}`，我们将收集一个模块的所有导出，并在一个名称下导入一个。当然，我们仍然可以使用点符号来分别调用它们:

```
import React from 'react' class App extends React.Component{...
```

*   **有了**`{}`，我们只抢我们需要的出口。

```
import React, {Component} from 'react' class App extends Component{...
```

请记住，您不能对这个方法使用`export default`,因为您需要明确您要导出的内容。例如，对于 React，我们应该写:

`export class Login extends React.Component{...`

而*不是:*

`export default class Login extends React.Component{...`

有关“Redux-style”导入/导出技术的更多信息，请阅读这里:[这里](https://alligator.io/react/index-js-public-interfaces/)和[这里](https://medium.com/lexical-labs-engineering/redux-best-practices-64d59775802e)。

# 下一步是什么？

在本文中，我们学习了“Redux 风格”的导出/导入技术，这种技术可以确保更准确的代码和更少的麻烦。在下一篇文章中，我们将使用我们在这两篇文章中学到的知识建立一个准系统 Express 服务器！敬请期待！
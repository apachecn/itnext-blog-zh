# 从头开始创建 React 应用程序

> 原文：<https://itnext.io/creating-react-app-from-scratch-f99c255f958e?source=collection_archive---------4----------------------->

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fcreating-react-app-from-scratch-f99c255f958e)

**在我们开始这个项目之前，我假设你对 ff 没有多少背景知识:**

*   **NodeJS**
*   **NPM/纱**
*   **Javascript**
*   **反应**
*   **HTML**

***注意:确保你的机器上安装了 NodeJS 和 NPM/Yarn。**

[](http://nodesource.com/blog/installing-node-js-tutorial-using-nvm-on-mac-os-x-and-ubuntu/) [## 安装 Node.js 教程:使用 nvm

### 与任何编程语言、平台或工具一样，使用它的第一步是安装它。他们中的许多人…

nodesource.co](http://nodesource.com/blog/installing-node-js-tutorial-using-nvm-on-mac-os-x-and-ubuntu/) 

**设置:**

1.  **创建项目目录:**

```
mkdir projectName && cd projectName
```

**2。转到项目目录，然后添加 README.md 和 git。**

```
$ touch README.md$ touch .gitignore
$ git init
$ git add .
$ git commit -m "initial commit"
```

**3。现在我们将通过运行:**向我们的项目添加一个空的 package.json

```
npm init -y
```

package.json 的示例内容:

```
{
  "name": "projectName-web",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

既然我们能够将 package.json 添加到我们的项目中，我们现在可以使用其他 javascript 库了。

***安装 ff: babel-core，babel-preset-env，babel-preset-react。***

```
$ yarn add babel-core babel-preset-env babel-preset-react --dev
```

**什么是巴别塔**？

Babel 是一个 JavaScript transpiler，它将 edge JavaScript 转换成普通的旧版本 ES5 JavaScript，可以在任何浏览器上运行(甚至是旧版本)。它提供了用新的 ES6 规范添加到 JavaScript 中的所有语法糖，包括类、粗箭头和多行字符串。

babel-core —包含节点 API 和 require hook。

preset-preset-env —“浏览器的自动修复程序”。它将所有内容从当前规范转换成指定环境可读的格式。这意味着它尽可能少地编译，从而产生更有效的输出。例如，当所有主流浏览器都支持箭头函数时，它们就不需要再被编译了。

babel-preset-react——babel 理解 react ts 的插件集合。

创建一个**。babelrc** 向项目投放 ff:

```
{
  "presets": [
    [ "env", { "modules": false }],
    "react"
  ]
}
```

现在我们要添加 webpack 和 babel-loader。

```
$ yarn add webpack webpack-dev-server babel-loader
```

**什么是 webpack？**

Webpack 将所有的 JavaScript 打包成一个文件。这包括您编写的每个 JavaScript 文件以及您的 npm 包。

**为什么捆绑？**

单人房。js 文件更容易部署，下载速度通常比多个小文件快。

当我们工作时，Webpack 将与 Babel 一起将您的代码从 ES6/ES7 转换到 ES5。网络包也可以缩小你的。用于生产的 js 文件

babel-loader——babel 的 webpack 插件。告诉 webpack 在捆绑之前使用 babel。

webpack-dev-server——一个小 Node.js Express 服务器，它使用 webpack-dev-middleware 来服务 web pack 包。

Webpack 配置部件:

**入口** —这是您的应用程序的入口点和起点。

**输出** —这是您所有 javascript 文件的捆绑文件。

**devServer** —您的 webpack-dev-server 的配置

**模块**——在将每个文件组合成包之前，您可以在其中放置如何处理每个文件的规则。

**resolve**—web pack 应该在哪里查找由 import 或 require()语句引用的文件。这使得您可以在代码中导入 npm 包。

创建一个文件并将其命名为 **webpack.config.js**

```
const path = require('path')const webpack = require('webpack')

module.exports = {
  entry: [ './src/index.js' ],
  output: {
    path: path.join(__dirname, 'public'),
    publicPath: '/',
    filename: 'index.bundle.js'
  },devServer: {
    contentBase: './public',
    historyApiFallback: true,
    hot: true
  },
  module: {    
      rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: [
          'babel-loader'
        ]
      }
    ]
  },
  resolve: {
    modules: [
      path.join(__dirname, 'node_modules')
    ]
  },plugins: [
new webpack.HotModuleReplacementPlugin()]
}
```

在**公共**文件夹下制作一个**index.html**，然后把这个:

```
<html>
  <body>
    <div id="root"></div>
    <script src="/index.bundle.js" ></script>
  </body>
</html>
```

然后在 **src** 文件夹下做一个 **index.js** 文件:

```
console.log('Initial Index file!');
```

将 **public/bundle.js** 添加到你的 ***中。git ignore***

```
$ echo "public/index.bundle.js" >> .gitignore
```

为了运行我们的项目，我们需要更新我们的 **package.json.**

在脚本对象下添加以下内容。

```
"start": "webpack-dev-server --open"
```

您的文件结构必须如下所示:

```
- node_modules- public- index.html- src
  - index.js- .babelrc- .gitignore- package.json- README.md- webpack-config.js
```

现在，您可以使用添加到您的 **package.json.** 中的脚本来运行我们的项目

```
yarn start
```

当浏览器打开时，你会看到一个空白页，但当你去控制台时，你会看到我们的日志。之后，提交它。

现在我们能够运行我们的项目了，我们将添加 react 并更改我们的 **index.js**

```
yarn add react react-dom
```

**index.js** 必须包含

```
'use strict'

import React from 'react'
import { render } from 'react-dom'

render(
  <div>
    Initial Index Page2
  </div>,
  document.getElementById('root')
)
```

*未完待续……*
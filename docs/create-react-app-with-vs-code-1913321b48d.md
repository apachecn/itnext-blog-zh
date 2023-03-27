# 使用 VS 代码创建-反应-应用程序

> 原文：<https://itnext.io/create-react-app-with-vs-code-1913321b48d?source=collection_archive---------2----------------------->

![](img/52952b9191159e14fe9a3c680d17ad81.png)

我做**前端开发者**已经有一段时间了，我看到我们的世界从“如果你想让它工作，你必须配置**千个**东西”到“点击这里，一切都将**只是**工作”。

最近，我开始摆弄 **React** ，因为我的工作涉及 React **单页应用**。我惊讶地发现，使用脸书开发的工具 [create-react-app](https://github.com/facebook/create-react-app) 样板来设置一个新的应用程序是多么容易。

基本上它所做的是隐藏所有无聊的配置文件，并给你一个自以为是的配置，这在大多数情况下应该是有效的。

> 你不需要安装或配置像 Webpack 或 Babel 这样的工具。
> 它们被预先配置并隐藏，以便您可以专注于代码。
> 
> 只要创建一个项目，你就可以开始了。

他们甚至提供了**热重装**，以及 **ESLint** 的默认配置！但是编码真的足够了吗？

我使用 [VS 代码](https://code.visualstudio.com/)作为**文本编辑器。嗯，文本编辑器，我可能应该停止使用这些词来描述它，因为现在它比简单的文本编辑器更接近于一个完整的 IDE。也许一个 ***智能代码编辑器*** ？**

无论如何，你对智能代码编辑器有什么期望？没错，出错时会有红色波浪线，自动完成，自动格式化。

让我们看看如何用 VS 代码实现这一点。

# 安装正确的依赖项

首先，你要做的是安装正确的 npm 依赖项。

如果你使用[纱线](https://yarnpkg.com)，你可以做到:

```
yarn add -D \
  @types/react \
  @types/react-dom \
  eslint-plugin-prettier \
  husky \
  prettier \
  pretty-quick
```

否则，使用 [npm](https://www.npmjs.com) :

```
npm install --save-dev
  @types/react \
  @types/react-dom \
  eslint-plugin-prettier \
  husky \
  prettier \
  pretty-quick
```

*   ***types/react*** 和***types/react-DOM***将为 React 安装 Typescript 定义。这将允许 VS 代码为内置的 React 函数计算出**自动完成**。输入“反应”，立即尝试并点击“CTRL+SPACE”来触发自动完成。
*   ***eslint-plugin-appearlier***和***appearlier***([自以为是的代码格式化程序](https://github.com/prettier/prettier) ) 将协同工作，正确地解析您的文件并自动格式化**您的代码。**
*   ***husky*** 通过将自身链接到一个 git 挂钩，比如 **precommit** ，来防止错误的提交和推送。结合 ***漂亮快速*** ，它会在这样做的同时跟随你更漂亮的配置。

当我们触及依赖项和 **package.json** 文件时，让我们添加一个**脚本**条目:

```
"scripts": {
    "precommit": "pretty-quick staged",
```

这将确保您的文件在提交之前总是按照您的漂亮配置进行格式化。

虽然 create-react-app 为我们抽象了配置文件，但我们仍然需要编写一个小的配置文件。

# 。eslintrc

这个文件对于让 ESLint 理解它对 Lint 的意义是必不可少的。

创建一个名为 ***的文件。应用程序的根目录:***

```
{
  "extends": "react-app",
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

# VS 代码扩展

现在，我们只需要为 VS 代码安装正确的扩展。搜索 **ESLint** 和**beautiful**，并安装两者。

你可能需要重启，因为“重新加载”按钮似乎并不总是足够的。

现在，如果一切都按预期运行，您可以开始在编辑器中键入一些垃圾，并立即看到它随着红线变得疯狂！

# 设置. json

最后一件事:推荐你创建一个***settings . JSON***文件。这是 VS 代码中的标准工作区设置文件。输入“CTRL-SHIFT-P”+“工作区设置”即可访问。

复制-粘贴此示例:

```
{
    "files.associations": {
        "*.js": "javascriptreact"
    },
    "editor.formatOnSave": true
}
```

第一行是告诉 VS 代码，当你在一个 ****的时候。js*** 文件，你要启用“javascriptreact”语法高亮显示。

第二行告诉编辑器，每次保存文件时，它应该运行得更漂亮。永远忘记括号缩进！

# 额外收获:绝对路径导入

如果你能做到呢

```
import MyComponent from "components/MyComponent"
```

从任何地方，而不是必须计数的“../”并做类似的事情

```
import MyComponent from "../../../components/MyComponent"
```

这是远远不需要的，但我觉得它可能是有用的，因为我花了一些时间来使它与 VS 代码和 create-react-app 一起工作，我想我可能会分享它。

这种可能性有两个方面。首先，你需要告诉 [webpack](https://webpack.js.org) 你希望你的路径包含你的“ ***src*** ”文件夹。然后，您希望 VS 代码知道这个选择。

Create-react-app 有一种通过 ***定义环境变量的方法。env*** 文件。创建一个名为**的文件*。env*** "文件，并定义 **NODE_PATH** 变量:

```
NODE_PATH=src/
```

然后，创建一个名为"***jsconfig . JSON***"的文件，同样位于应用程序的根目录下:

```
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "allowSyntheticDefaultImports": true,
    "baseUrl": "./src/",
    "paths": {
      "*": [
        "*"
      ]
    }
  },
  "exclude": [
    "node_modules",
    "**/node_modules/*"
  ]
}
```

本文档中有更多关于其工作原理的信息。

希望你觉得有用。用 React 和 VS 代码快乐编码！
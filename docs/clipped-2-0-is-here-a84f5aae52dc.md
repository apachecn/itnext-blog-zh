# 剪辑的 2.0 在这里！

> 原文：<https://itnext.io/clipped-2-0-is-here-a84f5aae52dc?source=collection_archive---------1----------------------->

## 用“插件”而不是“样板”创建项目

# 什么又被夹住了？

> **夹式**与`create-react-app`相似，但可以定制和扩展，无需`eject`或`vue-cli`，但不限于网络包。

[](https://github.com/clippedjs/clipped) [## clipped js/已剪辑

### 回形针:应该如何进行配置- clippedjs/clipped

github.com](https://github.com/clippedjs/clipped) ![](img/7b82724d52986c86523129cd9136ff29.png)

相关的

# 2.0 中的新功能

## 插件

以前剪辑使用的“预置”，因此所有配置都在一个功能中完成:

它确实使配置变得动态，但是当您有 200 多行代码时，事情会变得很笨拙，很难解释。

现在有了“插件”:

你会注意到钩子和配置可以分开，并作为插件层应用在一个数组中。

这很重要，因为 config 的不同部分现在被分成了可读性更强的部分。

注意数组中的元素和返回数组的函数都是插件，真的可以做一些疯狂灵活的插件:)

## 冗长的

使用插件来编辑配置是非常灵活的，但是有时候你不能确定配置的结果是什么。因此我们添加了`config:watch`命令:

```
$ clipped config:watch
```

这将生成显示最终配置的`clipped-result.json`。

## 发电机

以前生成器只安装预置然后停止，所以你不能制作插件专用生成器。例如，您可能希望准备好一个`src/index.jsx`文件。

我们改变了脚手架的工作方式，所以现在您可以使用命令`create`:

```
$ clipped create
```

会让你选择要安装的插件，然后它会调用每个插件的`init`钩子。

# 生态系统

随着这次升级，我们添加了几个插件和模板，使创建项目更加容易，并将在以后添加更多:

## 模板

*   [@ clipped/template-react](https://npm.im/@clipped/template-react)
*   [@clipped/template-vue](https://npm.im/@clipped/template-vue)

## 插件

*   [@clipped/plugin-docker](https://npm.im/@clipped/plugin-docker)
*   [@clipped/plugin-babel](https://npm.im/@clipped/plugin-babel)
*   [@ clipped/plugin-web pack](https://npm.im/@clipped/plugin-webpack)
*   [@clipped/plugin-rollup](https://npm.im/@clipped/plugin-rollup)

[](https://github.com/clippedjs/clipped) [## clipped js/已剪辑

### 回形针:应该如何进行配置- clippedjs/clipped

github.com](https://github.com/clippedjs/clipped)
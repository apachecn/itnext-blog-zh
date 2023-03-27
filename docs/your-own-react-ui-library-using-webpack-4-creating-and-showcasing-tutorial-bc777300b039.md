# 使用 Webpack 4 创建和展示您自己的 React UI 库

> 原文：<https://itnext.io/your-own-react-ui-library-using-webpack-4-creating-and-showcasing-tutorial-bc777300b039?source=collection_archive---------1----------------------->

![](img/2364a1ac0ec47789cc56721bcd68cf8a.png)

泰勒·弗兰塔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

当在一个有多个项目的公司工作时，您可能希望有设计指南来确保一致性。如果您可以有一个展示您的风格化组件的橱窗，并且仍然能够在其他项目中使用这些组件，就像安装一个 npm 包一样简单，那会怎么样？

本教程展示了如何使用 [Webpack](https://webpack.js.org/) 4 和[package bundler](https://github.com/parcel-bundler/parcel)构建一个可重用组件库，展示它们，并能够在其他项目中轻松导入。Webpack 将负责处理组件库的捆绑。package bundler 将处理组件展示柜的捆绑。

对于本例，您将构建一个简单的文档展示和一个按钮作为共享组件。所有代码都在[https://github.com/pmadruga/myUI](https://github.com/pmadruga/myUI)上

## 你将要建造的东西

*   可以在其他项目中使用的 UI 组件库
*   展示该库所有组件的橱窗

让我们开始吧。🎉

# 文件夹结构

让我们从创建一个文件夹结构开始。首先，让我们创建一个标准的文件夹结构。`lib`文件夹将包含可重用的 React 组件，而`docs`将包含这些组件的展示以及任何其他需要的文档(例如组件的 API)。

此时，文件夹结构如下所示:

```
src/
|- docs
 |-index.html
 |-index.js
|- lib
 |- index.js
 |- Button/
 |- index.js
dist/
webpack.config.js
package.json
```

# 属国

这是你需要的一切:

```
// For showcasing our docs
yarn add react react-dom// For bundling our library and docs
yarn add webpack parcel-bundler webpack-cli webpack-node-externals "babel-loader@^8.0.0-beta" @babel/core @babel/preset-env webpack-node-externals -D
```

# lib/: react 组件库

我们的 React 可重用组件库。当捆绑到`/dist/lib`中时，你可以在另一个项目中使用它们，通过快速安装成一个 npm 包，它可以是公共的(可以使用 npm 注册表)或者私有的(例如，可以使用 gemfury)。在`lib`文件夹下的`index.js`将负责导出库的所有组件。对于这种情况，您将只导出一个简单的按钮。

所以我们的`lib/index.js`包含了库导出的所有组件，(在这个例子中是一个简单的按钮)，所以它将包含

```
export { Button };
```

在`lib/Button/index.js`中，您将创建并导出一个简单的按钮。你也可以有样式，它仍然会被捆绑到一个文件中。

```
import React from 'react';const Button = ({ text }) => <button>{text}</button>;export { Button };
```

# docs/:记录和展示组件

`docs`文件夹将展示我们库的组件以及任何其他需要的文档(比如组件的 API)。它只是我们主应用程序中的一个小的 React 应用程序。当捆绑到/dist/docs 中时，它可以很容易地上传到任何地方。使用 github 页面将是一个很好的例子。

现在，您将创建和`index.js`和`index.html`，其中第一个包括来自`lib`文件夹的组件。

所以，`index.js`将包含

```
import React from 'react';
import ReactDOM from 'react-dom';
import { Button } from '../lib/Button';const App = () => (
  <div>
    <h1>My UI</h1>
    <h2>Button</h2>
    <p>Here's an example of button.</p>
    <Button text="Click me!" />
  </div>
);ReactDOM.render(<App />, document.getElementById('root'));
```

没有什么太花哨的，主要的要点是，可以看到有一个按钮是从组件库中导入的。

# 网络包.配置. js

如前所述，webpack 负责捆绑组件库(而不是文档)。我们来看看它的配置。这里最重要的值是`libraryTarget: ‘commonJS'`，它允许单独导出组件。

```
module.exports = {
  entry: path.resolve(__dirname, 'src/lib/index.js'),
  output: {
    path: path.resolve(__dirname, './build/lib'),
    filename: 'index.js',
    library: '',
    libraryTarget: 'commonjs'
  },
  externals: [nodeExternals()],
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/react']
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
};
```

# Package.json

package.json 最重要的是“main”。在将库上传到注册表(无论是 npm 还是 gemfury 或其他)后，它会指示哪个文件将作为入口点。`./dist/lib/index.js`包含了从`lib`捆绑的所有代码。

```
{ "name": "myui", "version": "0.0.1", "description": "Bootstrapping a UI component library", "main": "./dist/lib/index.js", "scripts": { "start": "./node_modules/.bin/parcel src/docs/index.html", "build": "./node_modules/.bin/webpack --mode=production", "build:docs": "./node_modules/.bin/parcel build src/docs/index.js -d dist/docs/"}, "author": "Pedro Madruga", "license": "MIT", "dependencies": { "react": "^16.2.0", "react-dom": "^16.2.0"}, "peerDependencies": { "react": "^16.2.0", "react-dom": "^16.2.0"}, "devDependencies": { "@babel/core": "^7.0.0-beta.42", "@babel/preset-env": "^7.0.0-beta.42", "@babel/preset-react": "^7.0.0-beta.42", "babel-core": "6", "babel-loader": "^8.0.0-beta", "parcel-bundler": "^1.6.2", "webpack": "^4.1.1", "webpack-cli": "^2.0.12", "webpack-node-externals": "^1.6.0" }}
```

# 从另一个项目导入

**假设您的项目在某处发布**(无论是 npm 还是 gemfury)，这将很容易:

`yarn add myUI`

然后在你的文件里:

`import { Button } from ‘myUI’`

就是这样！记得在[https://github.com/pmadruga/myUI](https://github.com/pmadruga/myUI)中查看这个例子的存储库

— — — —

我是一名软件工程师，也是一家打造 MVP 的公司[*occam . ooo*](https://occam.ooo)*的创始人🚀。我是* [*Scope 爬虫播客*](http://scopecreeperspodcast.com) *的播客联合主持人🎤，开发者和项目经理就不同的软件开发主题发表见解。* [*哥本哈根 React Native Meetup*](https://www.meetup.com/React-Native-CPH/)*的协办单位👥一名志愿者帮助外国人融入丹麦。一个绝对的通过代码和同理心创造东西的粉丝。喜欢练拳击🥊。*

*在推特上关注我*[*@ Pedro foraday*](http://twitter.com/pedroforaday)*。*
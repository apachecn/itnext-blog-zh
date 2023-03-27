# 开始将 Hyperapp 与 Parcel 一起用作捆绑器

> 原文：<https://itnext.io/getting-started-using-hyperapp-with-parcel-as-a-bundler-df1541a3f659?source=collection_archive---------4----------------------->

[Hyperapp](https://hyperapp.js.org/) 是一个 1KB 的 JavaScript 库，用于构建前端 web 应用。它有一个功能设计，将状态管理与 VDOM 引擎结合起来，没有依赖性。[package](https://parceljs.org/)是一个快速、零配置的 web 应用捆绑器。

让我们一起开始使用它们。

# 秘诀🥗🍴

安装包裹。

```
npm i -g parcel-bundler
```

创建一个工作目录和一个`package.json`文件。

```
mkdir my-hyperapp-test
cd my-hyperapp-test
npm init
```

安装 Hyperapp，巴别塔和 JSX 软件包。

```
npm i hyperapp jsx-transform babel-preset-env 
npm i babel-plugin-transform-react-jsx
```

创建一个`.babelrc`文件。

```
{
  "presets": ["env"],
  "plugins": [["transform-react-jsx", { "pragma": "h" }]]                       }
```

创建一个`app.jsx`文件。

```
import { h, app } from "hyperapp";

const state = { name: "world" };

const actions = {
  refresh: () => state => ({
    name: ["Jane", "John"][+(Math.random() < .5)]
  })
};

const view = (state, actions) => (
  <main>
    <h1>Hello {state.name}</h1>
    <buttononclick**=**{actions.refresh}>Refresh</button>
  </main>
);

const main = app(state, actions, view, document.body);
```

然后创建一个`index.html`文件。

```
<html><body><script src="./app.jsx"></script></body></html>
```

然后，运行包裹。

```
parcel index.html
```

打开你的浏览器，进入 [http://localhost:1234，](http://localhost:1234)它工作了！

现在，你可以建立一些真实的东西。

![](img/25941875df4124ca58ede30b355c5d22.png)
# 向 react-boilerplate 项目添加类型脚本

> 原文：<https://itnext.io/add-typescript-to-react-boilerplate-project-332862b53fed?source=collection_archive---------5----------------------->

在 Angular 中使用 typescript 多年来，我一直是 Typescript 的忠实支持者。它真的帮助我写出更可靠的代码，更快地捕捉错误。最近我参与了 React 项目，这个项目是用 react-boilerplate 创建的。由于 react 样板项目不支持类型脚本，我决定使用类型脚本文档在[https://www.typescriptlang.org/docs/handbook/react-&-web pack . html](https://www.typescriptlang.org/docs/handbook/react-&-webpack.html)添加它

以下是步骤:

1.  为 react、node、jest 和 styled 组件添加类型脚本、类型脚本加载器和类型

```
yarn add source-map-loader ts-loader typescript [@types/jest](http://twitter.com/types/jest) [@types/react](http://twitter.com/types/react) [@types/react-dom](http://twitter.com/types/react-dom) [@types/styled-components](http://twitter.com/types/styled-components) -D
```

2.用 internals/web pack/web pack . base . babel . js 中的 typescript 和 js 文件的源映射支持替换 babel。

```
module: {
     rules: [
       {
         test: /\.jsx?$/, 
         exclude: /node_modules/,
         use: {
          loader: 'ts-loader'
         },
       },
      // addition - add source-map support
      { 
        enforce: "pre", 
        test: /\.js$/, 
        exclude: /node_modules/, 
        loader: "source-map-loader" },
...
```

3.在 internals/web pack/web pack . base . babel . js 的 resolve 块中添加 ts 和 tsx 文件扩展名

```
resolve: {
  extensions: ['.js', '.jsx', '.react.js', '.ts', '.tsx']
}
```

4.在项目的根目录下创建 tsconfig.json。

```
{
  "compilerOptions": {
    "baseUrl": "app",
    "outDir": "build/dist",
    "module": "esnext",
    "target": "es5",
    "lib": ["es6", "dom"],
    "sourceMap": true,
    "allowJs": true,
    "jsx": "react",
    "moduleResolution": "node",
    "rootDir": "./",
    "forceConsistentCasingInFileNames": true,
    "allowSyntheticDefaultImports": true
  },
  "exclude": [
    "node_modules",
    "build",
    "scripts",
    "acceptance-tests",
    "webpack",
    "jest",
    "internals"
  ]
}
```

以下是关于配置的一些要点:

a.你需要混合 ts 和 js(尤其是在转换过程的乞讨中。所以使用 **allowJs**

b.为了支持这样的导入，使用 **baseUrl**

```
import HomePage from 'containers/HomePage/Loadable';
```

c.为了能够导入 react，请使用**allowSyntheticDefaultImports**

```
**import** React **from 'react'**;
```

5.将 index.js 重命名为 index.tsx 并重新编译。

就是这样。有用！
# TypeScript +无服务器+ Webpack + Babel

> 原文：<https://itnext.io/typescript-serverless-webpack-babel-15778188c7?source=collection_archive---------2----------------------->

随着 TypeScript 的兴起，我们决定将我们的代码从 JS + Flow 转移到 TypeScript。使用 Flow，我们发现了严格类型化的许多好处，它帮助我们在错误出现之前就将其消除，另外，围绕 ts 的工具也在快速发展(VSCode + ts 是一种惊人的体验)，所以现在似乎是拥抱 TS 的正确时机。

基本上，你可以将这个旅程分为两个部分，第一部分你重写你的代码到 TypeScript，这并不痛苦，因为我们已经在 Flow 上有一段时间了，Flow 非常像 TypeScript，所以这只是花费时间修复语法错误和类型错误`tsc`不喜欢的问题。第二部分是用 TypeScript 配置其他 cogs 播放流畅，这是能让公子哥砸键盘的部分。

![](img/fb337d52c921dd76009b9139ac145f8f.png)

我们使用无服务器框架来管理我们的 lambda 函数，并使用[server less-web pack](https://www.npmjs.com/package/serverless-webpack)+Babel 将 TS 代码转换为 Lambda runtime (NodeJS 8.10)满意的 JS 代码。

```
#Part of serverless.ymlcustom:
  webpack:
    webpackConfig: webpack.config.js
    packager: 'yarn'
```

网络包.配置. js

你可以看到我们用`babel-loader`而不是`ts-loader`，没有特别的原因，但是你需要在巴别塔 7 上有`@babel/preset-env`和`@babel/typescript`。如果你决定使用`ts-loader`，确保你有 **transpileOnly: true，**这将让`ts-loader`做 transpile 工作，而不是类型检查，通常类型检查过程应该是你的 CI 测试任务的一部分，这里它应该只处理 transpile to JS 和 push to Lambda。

对于`@babel/preset-env`，targets 被设置为 **node: true，**因为 transpiled JS 代码是针对 Lambda NodeJS 运行时的，不需要担心所有不同的浏览器，这有多好！

还有一点需要注意的是， **output.filename** 需要是 **[name】。js** ，这将把传输的 js 代码命名为与源 TS 代码相同的名称，这是因为在您的 serverless.yml 中，您可能有类似

```
functions:
  your-function:
    handler: my/lambda/functions/in-ts/hello.world
....
```

这告诉离线模拟器这个函数位于**my/lambda/functions/in-ts**，文件名是 hello.js，如果 webpack 配置成输出文件到【name】-bundle . js，那么你应该把它改成**my/lambda/functions/in-ts/hello-bundle . world .**

如上所述，Babel 不会验证你的类型，它只是去掉所有的类型脚本，把它变成你想要的任何 JS，因此它比`tsc`更快。但是你确实需要在 Babel 开始之前验证输入，所以用`tsconfig.js`和运行 **tsc** 你得到检查。在您的`tsconfig.json`中，确保您有**“no emit”:true**，这将告诉 tsc 不要传输，因为我们有`babel-node`做这项工作。

最后但同样重要的是，不要被迷惑。 **babelrc** 和 **tsconfig.json** ，Babel 看着。babelrc 和不关心 tsconfig.json， **tsc** 只看 tsconfig.json 和不关心. babelrc。

正如我在开始时所说的，围绕 TypeScript 的工具进展很快，今天工作的东西明天可能会改变，所以如果事情不工作，不要砸你的键盘，只是要有耐心，不断尝试。
# 如何使用 Node.js 创建自己的 TypeScript CLI

> 原文：<https://itnext.io/how-to-create-your-own-typescript-cli-with-node-js-1faf7095ef89?source=collection_archive---------0----------------------->

## 编程；编排

## 根据这个指南，准备好学习制作一个简单的“披萨”CLI 吧

![](img/e9a2ad794a8c3b35fa2b80898190e7a1.png)

通过 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的[诺德伍德主题](https://unsplash.com/@nordwood?utm_source=medium&utm_medium=referral)获得照片

在本指南中，我们将使用 Node.js 在 TypeScript 中制作一个小型的 pizza CLI。在完成所有步骤后，您将拥有一个完整的工作 CLI，了解如何设置它，并可能为自己创建一个自定义 CLI。

# 首先创建 package.json 和 tsconfig.json

首先，我们将使用`npm init`初始化 package.json。您可以为自己选择名称、作者、版本、描述、关键字和许可证。

## 属国

*   清除—清除我们的终端屏幕
*   figlet —从字符串中获得一个漂亮的 ASCII 艺术作品
*   粉笔——端子串样式设计正确
*   commander —使 node.js 命令行界面变得简单
*   路径—节点。JS 路径模块

我们需要安装所有的依赖项:

```
npm i clear figlet chalk@4.1.2 commander path --save
```

## 开发依赖性

*   types/node-node . js 的类型脚本定义
*   node mon——node . js 应用程序开发期间的简单监视器脚本
*   ts-node-type script 执行环境和 node.js 的 REPL
*   typescript——一种应用规模的 JavaScript 开发语言

然后安装我们的 devDependencies:

```
npm i @types/node nodemon ts-node typescript --save-dev
```

[](https://jeroenouw.medium.com/concentration-for-software-engineers-8ecafa72be3a) [## 软件工程师专注力

### 如何在更短的时间内更有效

jeroenouw.medium.com](https://jeroenouw.medium.com/concentration-for-software-engineers-8ecafa72be3a) 

## Bin 和 main

在我们的`package.json`中，我们需要设置应用程序的入口点(main 和 bin)。这将是我们在`lib`文件夹:`./lib/index.js`中编译的`index.js`文件。

单词`pizza`是您最终用来调用 CLI 的命令。

```
"main": "./lib/index.js",
"bin": {
  "pizza": "./lib/index.js"
}
```

## 剧本

现在我们需要一些脚本来简化我们的工作。我们有五个脚本:

*   `npm start` —您可以立即查看您的 CLI
*   `npm run create` —一起运行我们的`build`和`test`脚本。
*   `npm run build`—将我们的类型脚本`index.ts`文件编译成`index.js`和`index.d.ts`
*   `npm run local`—使用`sudo npm i -g`全局安装我们的 CLI，然后启动我们的`pizza` CLI 命令。
*   `npm run refresh`—删除节点模块 package-lock.json 并运行`npm install`。

将以下内容粘贴到`package.json`中:

```
"scripts": {
  "start": "nodemon --watch 'src/**/*.ts' --exec 'ts-node' src/index.ts",
  "start:windows": "nodemon --watch 'src/**/*.ts' --exec \"npx ts-node\" src/index.ts",
  "create": "npm run build && npm run test",
  "build": "tsc -p .",
  "local": "sudo npm i -g && pizza",
  "refresh": "rm -rf ./node_modules ./package-lock.json && npm install"
},
```

## TSconfig

对于我们的 CLI，我们在名为`tsconfig.json`的文件中设置了一些 TypesSript 配置，在根目录下创建该文件，并将以下配置复制到其中:

```
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "lib": ["es6", "es2015", "dom"],
    "declaration": true,
    "outDir": "lib",
    "rootDir": "src",
    "strict": true,
    "types": ["node"],
    "esModuleInterop": true,
    "resolveJsonModule": true
  }
}
```

# 现在，让我们开始创建 CLI

## 环境

在`src`文件夹中创建一个名为`index.ts`的文件。在我们的`index.ts`文件的顶部，我们有:

```
#!/usr/bin/env node
```

"这是**一个** [**shebang 行**](https://en.wikipedia.org/wiki/Shebang_(Unix)) 的实例:在类似于 ***Unix 的平台*** *上的一个可执行纯文本文件中的*第一行，它告诉系统通过跟在神奇的`#!`前缀后面的命令行(称为 *shebang* )将该文件传递给哪个解释器来执行*。"— [堆栈溢出](https://stackoverflow.com/a/33510581)*

## 进口

然后我们需要一些导入来利用我们的依赖关系:

```
const chalk = require('chalk');
const clear = require('clear');
const figlet = require('figlet');
const path = require('path');
const program = require('commander');
```

## 漂亮的横幅

接下来，我们调用`clear`(每次调用`pizza`命令时清除我们的命令行)。然后我们想控制台日志一个大横幅:一个红色的文本(pizza-cli ')，使用图和粉笔。

```
clear();
console.log(
  chalk.red(
    figlet.textSync('pizza-cli', { horizontalLayout: 'full' })
  )
);
```

这将看起来像这样:

```
 _ __   (_)  ____  ____   __ _            ___  | | (_)
 | '_ \  | | |_  / |_  /  / _` |  _____   / __| | | | |
 | |_) | | |  / /   / /  | (_| | |_____| | (__  | | | |
 | .__/  |_| /___| /___|  \__,_|          \___| |_| |_|
 |_|
```

## 我们带选项的 CLI

现在，我们到了可以让我们的 CLI 交互的部分。我们利用`program`。我们可以在这里设置我们的 CLI 版本、描述和各种选项，并解析结果。选项包含一个短变量和一个长变量，例如:要添加辣椒，我们可以使用`pizza -p`或`pizza --peppers`。

```
program
  .version('0.0.1')
  .description("An example CLI for ordering pizza's")
  .option('-p, --peppers', 'Add peppers')
  .option('-P, --pineapple', 'Add pineapple')
  .option('-b, --bbq', 'Add bbq sauce')
  .option('-c, --cheese <type>', 'Add the specified type of cheese [marble]')
  .option('-C, --no-cheese', 'You do not want any cheese')
  .parse(process.argv);
```

要查看我们当前拥有的内容，运行`npm run build`，然后运行`npm start`，您将看到:

```
 _ __   (_)  ____  ____   __ _            ___  | | (_)
 | '_ \  | | |_  / |_  /  / _` |  _____   / __| | | | |
 | |_) | | |  / /   / /  | (_| | |_____| | (__  | | | |
 | .__/  |_| /___| /___|  \__,_|          \___| |_| |_|
 |_|
Usage: pizza [options]An example CLI for ordering pizza'sOptions:
  -V, --version        output the version number
  -p, --peppers        Add peppers
  -P, --pineapple      Add pineapple
  -b, --bbq            Add bbq sauce
  -c, --cheese <type>  Add the specified type of cheese [marble]
  -C, --no-cheese      You do not want any cheese
```

## 最后一部分

我们希望用户看到他们点了什么，并看到他们的选项在他们做出不同的选择后得到更新。

```
console.log('you ordered a pizza with:');if (program.peppers) console.log('  - peppers');
if (program.pineapple) console.log('  - pineapple');
if (program.bbq) console.log('  - bbq');const cheese: string = true === program.cheese ? 'marble' : program.cheese || 'no';console.log('  - %s cheese', cheese);
```

有了这个代码，用户可以使用`pizza -h`来获取关于我们选项的信息。

```
if (!process.argv.slice(2).length) {
  program.outputHelp();
}
```

在`index.ts`中获得所有代码后，我们可以在命令行中运行`npm run create`来测试我们的 CLI。你会看到我们的最终结果:

```
 _                                      _   _
  _ __   (_)  ____  ____   __ _            ___  | | (_)
 | '_ \  | | |_  / |_  /  / _` |  _____   / __| | | | |
 | |_) | | |  / /   / /  | (_| | |_____| | (__  | | | |
 | .__/  |_| /___| /___|  \__,_|          \___| |_| |_|
 |_|
you ordered a pizza with:
  - marble cheese
Usage: pizza [options]An example CLI for ordering pizza'sOptions:
  -V, --version        output the version number
  -p, --peppers        Add peppers
  -P, --pineapple      Add pineapple
  -b, --bbq            Add bbq sauce
  -c, --cheese <type>  Add the specified type of cheese [marble]
  -C, --no-cheese      You do not want any cheese
  -h, --help           output usage information
```

## 发布到 NPM？

您可以选择将您的 CLI 发布到 npm，我已经选择在`package.json`中将项目的名称命名为‘pizza-CLI’。如果我运行`npm publish`，它将被发布到 npm 注册表(但是我没有，但是你可以)。其他人可以通过运行`npm i pizza-cli -g`在全球范围内安装我的项目，然后使用`pizza`命令启动并运行 CLI！

自己添加的一个很酷的特性是使用: [Inquirer](https://github.com/SBoudrias/Inquirer.js) 。有了这个，你就可以向用户提问来填写信息了，方式和命令一样:`npm init`为例。

感谢您的阅读！如果你喜欢这篇文章，请点击 50 次👏按钮，跟我来。我的 [Github](https://github.com/jeroenouw/) 。这里是这个 CLI 的全部源代码和我的一些其他文章:

[](https://jeroenouw.medium.com/concentration-for-software-engineers-8ecafa72be3a) [## 软件工程师专注力

### 如何在更短的时间内更有效

jeroenouw.medium.com](https://jeroenouw.medium.com/concentration-for-software-engineers-8ecafa72be3a) [](/create-your-own-advanced-cli-with-typescript-5868ae3df397) [## 使用类型脚本创建高级 CLI

### 已经知道如何做一个简单的 CLI？通过这篇文章增加你的知识

itnext.io](/create-your-own-advanced-cli-with-typescript-5868ae3df397) [](/must-have-tool-for-every-developer-71508e73e9aa) [## 每个开发人员的必备工具

### 通过生成文档节省大量时间

itnext.io](/must-have-tool-for-every-developer-71508e73e9aa)
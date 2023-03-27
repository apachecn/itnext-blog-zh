# Projen: NodeJS 项目样板

> 原文：<https://itnext.io/projen-nodejs-project-boilerplating-15c20665c6b5?source=collection_archive---------2----------------------->

[](https://github.com/projen/projen) [## 项目/项目

### 新一代项目生成器。通过在 GitHub 上创建帐户，为 projen/projen 开发做出贡献。

github.com](https://github.com/projen/projen) 

最近，基于 JavaScript 的项目在质量和工具方面的门槛越来越高。开发人员利用越来越多的工具、服务和标准。编译，测试，捆绑，林挺。跟上这些可能是乏味的。

有些人以前曾多次面临过这个问题，他们选择在创建新项目时维护一个要克隆的样板项目。虽然这肯定能把事情做好，但仍有改进空间。

一个非技术性的例子是，当你走进一家餐馆，问别人要点什么。这种选择很快，但并不完全是你的选择，而且是一次性的，既不可靠，也不可重复。

![](img/031d434313b759a9abeea0c355835941.png)

《当哈利遇见莎莉》(1989)——我要和她一样的东西

当你说“老样子”时，一个更有效的互动(假设你经常在这里吃饭)就发生了。这就是服务员给你带来你想要的所有信息。

这甚至可以把你的选择放在菜单上，这样其他人也可以点同样的组合。

![](img/ab0af8fbb0129f1f8f134007e40070cd.png)

[赛博朋克 2077:“银手特辑](https://www.youtube.com/watch?v=CorClefy-48&t=40s)”

这就是[项目想要帮助的。这就像是连续项目样板化的平台。您指定所需的状态，它会为您生成项目文件。](https://github.com/projen/projen)

让我们从头开始整个设置过程:首先，让我们创建一个新项目 *n* 项目 *ct* 。

```
$ mkdir projen
$ cd projen
$ npx projen new typescript
```

这里我们创建一个新的 typescript 项目。从一个简单的通用应用程序到 k8s typescript 项目，还有许多选项(包括电池)可供选择。

完成后，您会发现之前完全空的项目文件夹中填充了许多熟悉的文件和文件夹。

```
$ ls -lsgna
total 420
  4 drwxr-xr-x   9 1000   4096 May 24 17:08 .
  4 drwxr-xr-x   9 1000   4096 May 24 16:55 ..
  8 -r--------   1 1000   4118 May 24 16:56 .eslintrc.json
  4 drwxr-xr-x   7 1000   4096 May 24 16:57 .git
  4 -r--------   1 1000   1148 May 24 16:56 .gitattributes
  4 drwxr-xr-x   3 1000   4096 May 24 16:56 .github
  4 -r--------   1 1000    816 May 24 16:56 .gitignore
  4 drwxr-xr-x   2 1000   4096 May 24 17:12 .idea
  4 -r--------   1 1000    708 May 24 16:56 .mergify.yml
  4 -r--------   1 1000    320 May 24 16:56 .npmignore
  4 drwxr-xr-x   2 1000   4096 May 24 16:56 .projen
  4 -rw-r--r--   1 1000    753 May 24 16:56 .projenrc.js
  4 -r--------   1 1000    359 May 24 16:56 .versionrc.json
 12 -r--------   1 1000  11330 May 24 16:56 LICENSE
  4 -rw-r--r--   1 1000     14 May 24 16:56 README.md
 28 drwxr-xr-x 702 1000  24576 May 24 16:56 node_modules
  4 -rw-r--r--   1 1000   2175 May 24 16:56 package.json
  4 -rw-r--r--   1 1000    343 May 24 16:56 projen.iml
  4 drwxr-xr-x   2 1000   4096 May 24 16:56 src
  4 drwxr-xr-x   2 1000   4096 May 24 16:56 test
  4 -r--------   1 1000    827 May 24 16:56 tsconfig.eslint.json
  4 -r--------   1 1000    827 May 24 16:56 tsconfig.jest.json
  4 -r--------   1 1000    841 May 24 16:56 tsconfig.json
292 -rw-r--r--   1 1000 295787 May 24 16:56 yarn.lock
```

那么我们刚刚默认得到了什么？

*   基本项目结构
*   Eslint 设置
*   Github 操作和附加功能
*   NPM/纱线配置
*   Typescript 设置
*   测试设置

这是我们大多数人不止一次做过的事情。在这一点上，你可能会想:这很酷，我可以很快地完成一个新项目，但它有什么特别的吗？

首先，您可能已经注意到大多数生成的(“合成的”)文件都是只读的。这是对基于项目配置生成结构这一事实的认可。Projen 不希望我们直接修改生成的文件，因为这些是我们新项目的**实现细节**。通过这种方式，Projen 帮助我们在维护项目样板时获得可靠且可重复的结果。

我们该如何着手解决这个问题？我们先来看看 **.projenrc.js** 文件内部:

这是你的项目脚手架的总部。这是项目应该具有的“想要的”状态。通过修改 **TypeScriptProject** 的构造函数参数，我们可以指示 projen 生成一组我们实际需要的文件。例如，我们有一些 GitHub 动作，我们可能没有必要使用。我们可以通过如下方式更新代码来删除它们:

记住重新生成项目结构:

```
$ npx projen
```

如果您查看结果文件，您可能会注意到 projen 删除了一些 GitHub 工作流。这是这种方法如何工作的一般例子。

现在让我们更新与 package.json 相关的设置:

在这里，我们向`deps`和`devDeps`添加了包，并添加了一些描述。

这是可能感觉不自然的项目之一——通过另一个 javascript 文件编辑 **package.json** 。但是请记住，否则对 **package.json** (此时是实现细节)的更新可能会随着下一次`npx projen`调用而丢失。

当然，还有许多其他参数可以传递到 **TypeScriptProject** 中。如需完整列表，请查看官方文档。

Projen 在 **package.json** 的脚本部分提供了许多非常有用的控件。测试、构建、编译、升级 DEM，林挺。还有另一颗隐藏的宝石——`npm start`，下面是它的输出:

```
$ npm start> projen-demo@0.0.0 start /home/umkus/projects/projen
> npx projen start? Scripts: (Use arrow keys)BUILD
❯ compile                Only compile
  build                  Full release build (test+compile)
  watch                  Watch & compile in the backgroundTEST
  test:compile           compiles the test code
  test                   Run tests
  test:watch             Run jest in watch mode
  test:update            Update jest snapshots
  eslint                 Runs eslint against the codebaseRELEASE
  bump                   Bumps version based on latest git tag and 
  unbump                 Restores version to 0.0.0
  package                Create an npm tarballMAINTAIN
  clobber                hard resets to HEAD of origin and cleansMISC
  upgrade-dependencies
  upgrade-projen
  defaultEXIT
```

管理 cli 工具。令人惊奇的是，它不是静态的，可以用您自己的自定义操作来扩展它。让我们给它添加一个 webpack 操作。

我们首先需要创建一个包含以下基本内容的 **webpack.config.js** 文件:

要做到这一点，我们需要做两件事:

*   更新我们的 **.projenrc.js** 以添加缺失的依赖项
*   改变`build`和`watch`动作的处理方式

默认情况下，projen 的 typescript 项目为`build`命令做 4 件事:

1.  综合项目结构(npx 项目)
2.  测试(npx 项目测试)
3.  编译 TS (npx 项目编译)
4.  打包产生的工件(npx projen 包)

让我们假设一个用例，当我们希望 webpack 处理所有的编译并生成最少的包时。让我们也假设我们不需要打包工件。让我们额外假设我们有一些额外的维护任务(这可能是同步数据库，清理，等等。).下面是`.projenrc.js`文件需要改变的地方:

现在我们需要记住根据更新的配置(`npx projen`)合成项目文件，并运行`npm run start`来调出更新的 cli 工具。

```
$ npx projen start
? Scripts: (Use arrow keys)BUILD
  compile                Only compile
  build                  Full release build (test+compile)
  watch                  Watch & compile in the backgroundTEST
  test:compile           compiles the test code
  test                   Run tests
  test:watch             Run jest in watch mode
  test:update            Update jest snapshots
  eslint                 Runs eslint against the codebaseRELEASE
  bump                   Bumps version based on latest git tag and
  unbump                 Restores version to 0.0.0
  package                Create an npm tarballMAINTAIN
  clobber                hard resets to HEAD of origin and cleans
❯ What time is it?       Prints current system timeMISC
  upgrade-dependencies
  upgrade-projen
  defaultEXIT
```

注意**维护**部分的新任务。这是它的输出:

```
? Scripts: Prints current system time
What time is it? | date
Wed May 25 20:09:45 CEST 2021
```

如果我们选择“ **build** ”，我们现在将获得以下构建输出:

```
? Scripts: Full release build (test+compile)
build | npx projen
default | node .projenrc.js
Synthesizing project...
yarn install --check-files
yarn install v1.22.10
[1/4] Resolving packages...
success Already up-to-date.
Done in 2.19s.
Synthesis complete
build | npx projen test
test | rm -fr lib/
test » test:compile | tsc --noEmit --project tsconfig.jest.json
test | jest --passWithNoTests --all
 PASS  test/hello.test.ts
  ✓ hello (1 ms)----------|---------|----------|---------|---------|----------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line 
----------|---------|----------|---------|---------|----------------
All files |     100 |      100 |     100 |     100 |                  
 index.ts |     100 |      100 |     100 |     100 |                  
----------|---------|----------|---------|---------|----------------
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.545 s, estimated 2 s
Ran all test suites.
test » eslint | eslint --ext .ts,.tsx --fix --no-error-on-unmatched-pattern src test build-tools .projenrc.js
build | npx webpack
asset index.js 81 bytes [emitted] [minimized] (name: main)
asset ../lib/index.d.ts 55 bytes [emitted]
./src/index.ts 826 bytes [built] [code generated]
webpack 5.37.1 compiled successfully in 1005 ms
```

如您所见，所有请求的操作都按照我们指示的方式执行了:

*   构建:projen 合成了我们的项目文件以及更新的 dep
*   测试:清除 lib 目录，编译并运行 jest 脚本，linted
*   构建:webpack 捆绑了应用程序

简而言之，这是一个项目。这不仅仅是另一个新项目启动人员/样板师/架子工。这是一种“代码作为文件结构”的方式，它基于带有明智的自以为是的缺省值的期望状态，持续地进行项目设置。

![](img/32af8ff0b4e2c73c104542b673819df3.png)

用 Projen 启动新的 typescript 项目

为了加深印象，这里有一个来自 Elad Ben-Israel 本人的项目演示，最初是在 2020 年 CDK 日期间进行的:
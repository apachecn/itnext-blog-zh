# 如何自动化 npm 包的版本控制和发布

> 原文：<https://itnext.io/how-to-automate-versioning-and-publication-of-an-npm-package-233e8757a526?source=collection_archive---------0----------------------->

## 使用 npm 脚本和外部包来自动化版本管理和包发布过程的指南、提示和技巧。

![](img/a6e9353444b4e987100fc7d7ebc33389.png)

**TL；你可以在文章的最后部分看到我们自动化发布过程的方法。**

在 [Xing Spain](https://twitter.com/xing_esp) 我们几个月前开始了一个新项目，一个 React 组件库，几个产品将使用它，所以共享这个库的自然方式是通过一个 npm 包。

为了共享一个 npm 包，以一致的方式发布版本是很重要的，我们从一开始就知道我们想尽可能地自动化这个过程。

**这篇文章描述了我们如何自动化版本控制和发布的过程**。如果你想要最简单的方法，我会推荐[语义发布](https://github.com/semantic-release/semantic-release)，它是一个完全自动化的版本管理和软件包发布工具。

在我们的例子中，不使用语义发布的主要原因是因为我们不想为 master 中的每一次推送发布一个新版本，我们希望完全控制每一步。不使用语义发布的另一个原因可能是您需要对 CI 设置进行太多的调整。在任何情况下，这两种方法都基于结构化提交消息的相同基础。

在这篇文章中，我假设你已经知道如何发布一个 npm 包，但是你想改进所有涉及的手工过程。如果您不熟悉发布流程，[您可以开始阅读文档](https://docs.npmjs.com/cli/publish)。

# 工作流程亮点

**能够自动化一个包的版本和发布的唯一要求是每次提交都应该遵循一个消息约定**，我们遵循[常规提交规范](https://www.conventionalcommits.org)。

所以我们的提交看起来像这样:

```
6f88099 - (tag: v1.8.0, master) chore(release): 1.8.0
333dce9 - feat: add composable Avatar component
ffb2263 - fix: add correct styles to Label component
7d34334 - chore: update Jest to 24.7.1
```

您可以在他们的文档站点中查看[不同的常规提交示例。](https://www.conventionalcommits.org/en/v1.0.0-beta.4/#examples)

## 实施常规提交

很容易忘记提交约定，所以为了保持一致，我们使用 [commitzen](https://github.com/commitizen/cz-cli) 来生成提交，使用 [husky](https://github.com/typicode/husky) 来管理 Git *commit-msg* 钩子来验证提交消息。

您可以轻松安装 husky 和 commitzen:

```
npm i -D husky @commitlint/{config-conventional,cli}
```

此设置需要添加到您的`package.json`:

```
"config": {
  "commitizen": {
    "path": "cz-conventional-changelog"
  }
},
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

要测试它，只需尝试用不遵循提交约定的消息提交更改，它将抛出一个错误。

在我们的例子中，我们还使用了一个预提交钩子触发 [lint-staged](https://github.com/okonet/lint-staged) 来 lint，并在提交之前格式化更改，在最后一节中，您可以从我们的`package.json`中看到一个例子。

## 基于提交更新包版本

我们使用[标准版本](https://github.com/conventional-changelog/standard-version)根据提交历史自动更改版本。

要安装标准版，只需运行:

```
npm i -D standard-version
```

然后您可以在您的`package.json`中创建发布脚本:

```
{
  "scripts": {
    "release": "standard-version"
  }
}
```

现在您可以运行`npm run release`来触发版本更新。

请注意，标准版将按照以下指南更改您的版本号:

*   一个`git commit -m “fix: …”`提交将触发一个**补丁更新** (1.0.0 → 1.0.1)
*   一个`git commit -m “feat: …”`提交将触发一个**小更新** (1.0.0 → 1.1.0)
*   提交主体中的`BREAKING CHANGE: …`和任何类型的提交都会触发**重大更新** (1.0.0 → 2.0.0)

总的来说，标准版将完成以下任务:

*   `package.json`撞版
*   更新`CHANGELOG.md`
*   提交两个文件
*   标记新版本

**标准版本没有做的是将提交推送到您的远程存储库**，所以在运行`npm run release`之后，您需要执行
`git push --follow-tags origin master`并发布包。

## 你可以在发布过程中做一些很酷的额外工作

在发布过程中，有几个额外的任务可以自动完成，例如:

*   使用[传统-github-releaser](https://github.com/conventional-changelog/releaser-tools/tree/master/packages/conventional-github-releaser) 的 github 版本(在 Github 中创建一个更好的版本，列出新的特性、修复或突破性变化)
*   使用 [git-authors-cli](https://github.com/Kikobeats/git-authors-cli) 自动更新`package.json`中的贡献者
*   生成 styleguide 的静态版本(可能是 Storybook、Styleguidist、Docz…)
*   使用[尺寸限制](https://github.com/ai/size-limit)检查你的包裹没有达到预先设定的最大尺寸

您可以在本文的最后一节看到如何使用这些额外功能的示例。

## 触发释放

一旦提交到达主机，执行释放只需要一个命令:`npm run release`

接下来会发生什么？幕后可能会发生很多事情，**所有的魔法都是使用一些 npm 包和 npm 脚本**完成的。

这可能是使用标准版本的正常工作流程，但可能性是无限的:

1.  验证更改(运行测试，检查更改是否满足 ESLint 或更漂亮，或者检查包的最大大小)
2.  如果前面的检查都成功了，那么就可以构建甚至部署任何 Styleguide 的静态构建了
3.  此后，更新`CHANGELOG.md`和包版本，并生成新的提交
4.  下一步是将标签和发布版本推送到 Github(这一步需要一个 Github auth 令牌)
5.  最后一步是将新版本发布到 npm，这通常在 CI 服务器中完成，但也可以在本地完成

如果您想看我们用来执行类似工作流程的包和设置，请继续阅读。

# 自动版本化和发布的真实例子

如果您想应用类似的方法，首先需要安装一些依赖项:

```
npm i -D standard-version husky lint-staged @commitlint/{config-conventional,cli} commitizen conventional-github-releaser git-authors-cli conventional-github-releaser npm-run-all eslint prettier
```

然后您可以用一些脚本和配置来更新您的`package.json`:

```
"scripts": {
  "commit": "git-cz",
  "test": "...",
  "build": "...",
  "lint": "eslint src/**",
  "styleguide:build": "...",
  "prettier:check": "prettier --check 'src/**/*.{js,mdx}'",
  "validate": "run-s test lint prettier:check",
  "prerelease": "git checkout master && git pull origin master && npm i && run-s validate styleguide:build && git-authors-cli && git add .",
  "release": "standard-version -a",
  "postrelease": "run-s release:*",
  "release:tags": "git push --follow-tags origin master",
  "release:github": "conventional-github-releaser -p angular",
  "ci:validate": "rm -rf node_modules && npm ci && npm run validate",
  "prepublishOnly": "npm run ci:validate && npm run build",
},
"config": {
  "commitizen": {
    "path": "cz-conventional-changelog"
  }
},
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
    "pre-commit": "lint-staged"
  }
},
"lint-staged": {
  "src/**/*.mdx": [
    "prettier --write",
    "git add"
  ],
  "src/**/*.js": [
    "prettier --write",
    "eslint --fix",
    "git add"
  ]
}
```

这直接取自我们的`package.json`，但是我删除了一些与自动化过程无关的脚本，还清空了一些将根据您的需求设置的其他脚本。

为了正确理解这个例子，你需要知道一些 npm 脚本的特性和技巧:

*   有几个 npm 脚本会被自动触发，例如在安装软件包之前或之后，或者在发布软件包之前，[查看 npm 文档](https://docs.npmjs.com/misc/scripts)，它们非常有用
*   此外，任何自定义脚本都可以通过运行名称匹配的`npm run <script_name>`和 *pre* 和 *post* 命令来执行(例如，prevalidate、validate、postvalidate)
*   `prePublishOnly`是在包准备和打包之前触发的，只针对一个`npm publish`(所以这是一个验证您的包并执行构建的完美地方)
*   使用 [npm-run-all](https://github.com/mysticatea/npm-run-all) 包，您可以并行或顺序运行几个脚本(因此`npm run-s release:*`与`npm run release:tags && npm run release:github`相同)
*   `npm ci`与`npm install`相似，但被调整为在自动化环境中使用，[你可以在 npm 文档中阅读更多关于 npm-ci 的内容](https://docs.npmjs.com/cli/ci)

在简单解释了 npm 脚本之后，让我来解释一下**我们为任何开发者**准备的*真实*工作流程:

*   `npm run commit`而不是`git commit`能够轻松地创建一个提交你的更改的消息
*   `npm run release`触发自动版本变更，得益于 npm 脚本在运行发布之前会触发`prerelease`脚本(验证、styleguide 静态构建和自动 github 贡献者)，而在发布之后会触发`postrelease`脚本(它会将`CHANGELOG.md`和`package.json`变更推送到远程存储库并在 Github 中执行新的发布)
*   在我们的例子中，最后一步是在我们的 CI 环境(Jenkins)中完成的，我们解析每一个推送到 master 的提交，如果是发布提交(例如`chore(release): 1.8.0`，则执行`npm publish`(在此之前`prePublishOnly`将被自动触发以验证和构建代码)

老实说，在一个持续集成的环境中，我们不应该在本地触发发布。我们希望手动触发每个版本，但更好的地方可能是从詹金斯乔布斯的例子。

仅此而已，我希望我们的一些工作流程对您的项目有所帮助，或者至少理解您可以通过一些 npm 魔法实现的所有自动化可能性。这个过程甚至可以更复杂，向 Slack 发送发布通知或发布新的 tweet。

在任何情况下，正如我在本文开头向您推荐的，对于大多数项目来说，使用[语义发布](https://github.com/semantic-release/semantic-release)并轻松地自动化整个过程可能更有意义。
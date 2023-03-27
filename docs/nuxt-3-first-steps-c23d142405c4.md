# Nuxt 3 第一步。

> 原文：<https://itnext.io/nuxt-3-first-steps-c23d142405c4?source=collection_archive---------0----------------------->

![](img/7b5571c956b89fa9205a563316f339f2.png)

[布鲁诺·纳西门托](https://unsplash.com/@bruno_nascimento?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

所以我是一个超级 [**Nuxt**](https://nuxtjs.org) 粉丝，很明显，当新的框架版本【3】最终发布时，我非常兴奋。就在我有机会安装它并玩了一会儿之后。但是，我们都知道这是一个非常早期的测试阶段，在最初的几天里， [GitHub](https://github.com/nuxt/framework) 上的问题标签很快就完成了。很自然地，我的第二个想法是——让我们旋转一下，暖暖身子。

几天前，我决定再次检查这个东西。在此之前，我已经对已经涵盖/翻译的第三方工具，如模块、插件和库做了一些一般性的研究。有趣的是，有一个已经创建的[网站](https://isnuxt3ready.owln.ai)正在收集所有这些信息，并告知我们它们处于哪个阶段。总的来说，如果我相信这一点，它看起来不是很好，但工作正在进行中，我们每天都在接近。

**更新**:*Nuxt 3 准备好了吗*网站已经关闭，现在我们可以关注官方[模块网站](https://modules.nuxtjs.org/?version=3.x)来了解 Nuxt 3 外设的最新状态。

现在，在我的研究过程中，我发现了很多关于如何将 Nuxt 3 与 i18n、Algolia 或 Axios 等新模块连接的教程。还有现成的模板和所有必要的依赖项。这很好，但是这项技术是新的，新鲜的，还不能用于生产。如果至少能更好地了解它，了解它为我们的新项目带来的所有细微差别，那就太好了。我有这种感觉，每个人都只是冲向脂肪的东西，甚至没有检查解决方案稳定性的当前状态。

所以让我们这样做，容忍我，让我们尝试从头安装 Nuxt，并经历第一步和最重要的步骤。这里我有一个小小的免责声明——我遇到了许多最终解决的问题，但是在我看来，这证明这个框架离生产就绪和稳定还很远。但这只是我的主观看法。

还有一个**免责声明**——我正在使用 [WebStorm](https://www.jetbrains.com/webstorm) IDE，所以有些东西可能与使用 [VSC](https://code.visualstudio.com) (Visual Studio 代码)不同，但我猜是超级小的东西。

## 装置

跳转到 Nuxt 3 的[安装](https://v3.nuxtjs.org/getting-started/installation)

```
npx nuxi init nuxt3-app
```

好吧，首先。什么是`nuxi`？新的 Nuxt CLI(命令行界面)将帮助您安装和管理所有的 Nuxt 组件。

作为输出，你会得到这个。

```
🎉  Another dazzling Nuxt project just made! Next steps:📁  `cd nuxt3-app`💿  Install dependencies with `npm install` or `yarn install`🚀  Start development server with `npm run dev` or `yarn dev`
```

太好了，让我们用`nuxt3-app`移动到这个新文件夹。里面是什么？事实证明，没有那么多…但这很好，为什么你会问。这是新的项目结构。

![](img/f1a19d443de3cd39d622640fa2d1b052.png)

哦，我的天，这和我们从 Nuxt 安装过程中了解到的有很大不同。这是新的 Nuxt minimal，starter 设置，允许你开始创建你需要的项目，不删除文件夹和文件，而是添加它们，TBH 对我来说，它非常令人耳目一新，非常有趣，也许很聪明？这里有一个简单的 Vue 组件和 Nuxt，TypeScript 配置的`app.vue`文件。更耐人寻味的是配置完全是空的…嗯， *nuxt.config.ts* 是， *tsconfig.json* 是从已经放在 nuxt 包中的默认配置扩展而来的。

现在我明白了为什么有那么多的模板以及所有的文件夹和依赖项。这很舒服，是的，我知道，不知道你的样板文件是怎么回事是否是致命的？🤔

让我们继续处理初始化后得到的启动器信息。让我们安装依赖项。

```
npm install or yarn 
```

## **邀请**

在安装过程中最重要的是，默认情况下你会得到邀请。这很好，但是，Webpack 也会被安装，所以你可以从一个切换到另一个。怎么会？通过在你的 *nuxt.config.ts* 文件中设置`vite`为`false`。像这样。

**更新**:默认情况下不再安装 Webpack，所以你需要使用一个特殊的模块来使用它。现在，Vite 是默认的，这太好了！

请不要这样做。如果你还不知道 **Vite** 你必须赶上，但是用几个简短的词来说，这是一个超快速的构建/捆绑“引擎”,它将加速你的开发过程。

好了，这里我想停一会儿。所以在我的一台机器(MBP)上，我遇到了一些关于 Vite 的问题——它报告了一些关于 build 和 Nuxt 源文件的问题。

```
[vite:import-analysis] Cannot read property 'uid' of undefined
```

至于现在，我找不到任何解决方案或相关问题，无论是在 Nuxt repo 还是 Vite 上。也许它会被新版本覆盖，也许是我的新 M1 Mac，也许是有依赖关系的东西。不过这很有趣，因为在另一台 Mac (Mini)上，它运行得非常好。尝试了不同版本的库，甚至节点。无济于事。

为了解决这个问题，我使用了 Webpack，但在这里它也不是没有问题。HMR(热模块替换)出了问题，在应用程序运行后它并没有停止，它一直在刷新，而不是等待更改，最终阻塞了浏览器。还是那句话，这可能是与“我”有关的事情，因为我找不到任何围绕 Nuxt 生态系统的类似问题。你可以做的一件事是禁用 HMR，这不是很有效，但就目前而言，这是解决问题的办法。这是你可以做到的。

小心，新的 Nuxt 配置与旧的有点不同，检查 TS 接口，它将帮助你完成所有的设置。

## 埃斯林特和更漂亮

好吧，让我们来点更有趣的。在我所有的项目中，我都在使用 ESLint 和 Prettier，在这里我也想拥有它们。同样，你的新的 Nuxt 项目将会是完全裸露的，所以你需要添加一些额外的第三方助手，就像上面的那些。经过一些测试和研究，我润色了一套最佳的默认规则和建议，你可以用在你的新 Nuxt/Vue 3 项目中。就是这个，你的 *.eslintrc.js* 文件。

运行此命令来安装所有这些组件。

```
yarn add -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser @vue/eslint-config-standard eslint-config-prettier eslint-plugin-prettier eslint-plugin-nuxt eslint-plugin-vue
```

是的，规则和自定义设置是你可以根据自己的需要来设置的，但是我想在这里补充一点。你可能会开始用新的`<script setup>`符号创建你的组件，在导入一些外部资源如组件时，你会遇到未使用的变量的问题——至少在 WebStorm 上，我猜 VSC 也是这样。所以要处理这个问题，你可以在你的 *.eslintrc.js* 文件中添加一些特殊的规则。

```
'vue/script-setup-no-uses-vars': 'off'
```

现在，如果您定义那种组件，您将不会得到关于未使用、已定义的变量的报告问题，而只是关于 Vue 相关的问题。

更漂亮的配置也是个人的东西，所以我不会粘贴在这里。😏对于 ESLint 忽略文件也是一样，创建它并为您的设置填充。

最后，您可以将这个脚本添加到您的 *package.json* 文件中。

```
"lint": "eslint --ext .js,.ts --ignore-path .eslintignore ."
```

## 页面和布局

好的。Nuxt 的一大特色是自动生成路由。它位于页面文件夹结构上。在 Nuxt 识别这个结构之前，你需要从根文件夹中移除 *app.vue* 文件。然后创建`pages`文件夹。把 *index.vue* 文件放在那里。很好，你有了第一个自动生成的页面，就像以前的 good Nuxt 一样，对吗？关于这个神奇文件夹的更多信息，你可以在这里阅读。

> 有了邀请，所有这些行动和过程将会变得非常快。

更重要的是，你的页面需要一些布局，布局功能在 Nuxt 3 中仍然可用，但是它的结构有点不同。现在你不是使用`<nuxt />`组件来放置页面内容，而是使用`slots`。当然，您需要创建默认布局。像这样。

现在，如果您在索引页面中添加内容，它将显示在`<slot />`区域内的默认布局周围。

## 自动导入

可能这一个会让你大吃一惊。现在 Nuxt 3 能够[自动导入](https://v3.nuxtjs.org/guide/concepts/auto-imports)你的组件，甚至组件。你只需要把它们放进专用的文件夹里就行了。不需要在你的组件中使用进口的了，所以性感和干净的解决方案！您可以通过这个[特别演示](https://stackblitz.com/edit/vue-use-state-effect-demo)来检查它的运行情况。

## 页面元

Nuxt 3 这个东西还在我们身边。但是现在你需要用一个稍微不同的配置来使用它。你将不再使用`head`的财产，现在`meta`是你的协议人。可以这样用。

更好的是，现在你可以直接从你的[组件标记](https://v3.nuxtjs.org/docs/usage/meta-tags#meta-components)中定义元数据。这是疯狂和令人敬畏的同时 TBH。

## 用`<script setup>`分页数据

这个可能不是每个人都有的，但是如果你使用`<script setup>`并且你愿意添加一些额外的页面数据，比如标题和你的`script setup`符号，你需要一个额外的脚本。像这样。

记住对两者保持相同的语言定义。

## 静态资产

另一件完全不同的事情是，现在要展示一些静态资产，比如图像、字体等等，你需要在你的根文件夹中创建`public`文件夹，把你的静态内容放在那里。链接看起来就像这样。

```
<img src="/img/icon/icon.svg" alt="Icon" />
```

干得好。我认为你已经具备了使用 Nuxt 3 开始开发的所有必要条件。当然，这堵墙现在就要开始长了。我们可以在下一篇文章中解决这个问题——获取数据、样式、组件等等。请继续查看我的个人资料。你也可以订阅时事通讯，不要错过这个伟大的内容。干杯，卢卡斯。

# **你还想要更多的**？

现在查看 [Nuxt 3 的下一步](/nuxt-3-the-next-steps-9129f876c32b?sk=8ba06425617478fc1c6e0119813bc2a0)文章。

[](/nuxt-3-the-next-steps-9129f876c32b) [## 接下来的步骤。

### 几个月前，我写了这篇关于 Nuxt 的前三步的文章。它甚至不是一个 RC(发布候选)…

itnext.io](/nuxt-3-the-next-steps-9129f876c32b)
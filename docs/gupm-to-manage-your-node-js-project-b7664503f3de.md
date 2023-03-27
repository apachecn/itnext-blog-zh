# GuPM 来管理您的 Node/JS 项目

> 原文：<https://itnext.io/gupm-to-manage-your-node-js-project-b7664503f3de?source=collection_archive---------2----------------------->

![](img/f0ce71306809cb3ae7af5bf59d25e6f9.png)

# GuPM 简介

GuPM 是一个包管理器，其目标是统一包管理场景。统一并不意味着集中:GuPM 的重点还在于通过与分散的存储库合作，确保社区驱动的开发场景，以及用于定制工具行为的强大脚本/插件系统。

> *本指南假设你熟悉 NPM 或同等产品(宝石、纱线、美芬等……)。*
> 
> 为了阅读这篇文章，你需要在你的机器上安装 [GuPM](https://github.com/azukaar/gupm) 和 [NPM 提供者](https://github.com/azukaar/gupm-official)。

```
# Install GUPM
curl -fsSL https://azukaar.github.io/GuPM/install.sh | bash# Install the NPM Provider
g plugin install https://azukaar.github.io/GuPM-official/repo:provider-npm
```

# 它能做什么

*   依赖项管理(npm make / npm install)使用多个源(例如，不使用 NVM，只需将节点设置为项目的依赖项)
*   脚本(g myscript.gs 将以跨平台的方式执行您的脚本，对于构建/发布非常有用)
*   跨平台环境变量
*   智能 Git 挂钩(只对更改的文件执行命令)
*   并行执行您的测试/林挺

# 入门指南

让我们在您的项目中启动一个 cli，并键入:

```
g b -p npm
# short-hand for g bootstrap --provider npm
```

它将询问您是否想要使用 gupm.json 或 package.json。如果您的项目已经存在，gupm 可以将其导入到 gupm.json 中(参见下一段了解 gupm.json)。如果你打算发布一个 lib 到 NPM，让人们用 NPM 安装它，那么你将别无选择，只能跳过导入，继续使用同一个 package.json(别担心，GuPM 也可以使用 package.json！).如果您选择第二种方法，gupm 仍然会在您的项目中创建一个 gupm.json，让您能够从 GuPM 的特性中获益。

一旦完成，GuPM 将生成一个包含基本说明的自述文件。

```
g make # Install your project's dependenciesg i webpack # shorthand for g install webpack
```

您可能想知道:告诉我，如果 GuPM 是一个通用的包管理器，它怎么知道从 NPM 安装那些依赖项呢？让我们在项目中深入一点，看看脚手架是做什么的！

# 了解 GuPM

那么我们在这里生成了什么呢？最重要的是，我们生成了一个`gupm.json`。

```
{
    "author": "",
    "description": "",
    "licence": "ISC",
    "name": "test",
    "binaries": {},
    "cli": {
        "aliases": {}
    },
    "dependencies": {
        "default": {},
        "defaultProvider": "npm"
    },
    "publish": {
        "destination": "docs/repo",
        "source": []
    }
}
```

如果您生成项目是为了使用 package.json，您应该看到以下内容:

```
{
    "author": "",
    "cli": {
        "defaultProviders": {
            "install": "npm",
            "make": "npm"
        }
    },
    "licence": "ISC",
    "name": "test"
}
```

如果您熟悉 package.json，这里已经有一些东西应该可以学习了。首先，名字/作者/描述/许可证是不言自明的。

`cli`选项允许您定制不同的 cli 命令在项目中的工作方式。例如，我们可以设置别名(将其视为 package.json 中的“脚本”)。

```
"cli": {
  "aliases": {
    "start": "node index.js"
  }
},
```

您还可以在这里通过 defaultProvider 告诉 GuPM 在使用特定命令时使用特定的提供者。这就是我们如何让 GuPM 使用 npm 进行制造和安装。

在第一个示例中，您可以注意到我们没有在 cli 选项中设置 defaultProvider，但是我们仍然可以直接安装 webpack。这是因为在依赖关系中，我们有另一个选项来设置 defaultProvider。与那个不同的是，在`resolution`阶段，它被设置为默认提供者。为了更好地理解，让我们假设我们已经设置了 **no config** :

```
# First, let's try to install webpack 
# this will not use NPM 
# because no defaultProvider is set in gupm.jsong i webpack# We can tell GuPM to use NPM to INSTALL a dependency
# If you do this, NPM will take over the whole install process
# Meaning it will save the dependency to package.json
# This is what happens when you set cli.defaultProvidersg -p npm i webpack# A better way to do this, is to still use GuPM's default INSTALL
# But tell it to resolve the dependency from NPM
# Here, only the RESOLVE stage gets overwritten by NPM
# This is what happens when you set dependencies.defaultProviderg i npm://webpack
```

接下来，我们有 publish，这很容易理解，如果你决定发布到一个分散的回购，它包含源文件和回购的目的地。

二进制文件严格等同于 package.json 中的 bin。

我们需要查看的最后一个文件是`make.gs`。如果你决定保留一个 package.json，它会和你的 gupm.json 一起生成，这个文件是在你执行了`g make`之后由 GuPM **自动执行的(你也可以创建 install.gs 等等……)。它的作用是告诉 GuPM“一旦你安装完 GuPM 的依赖项，就安装 NPM 的”。**

```
exec('g', ['make', '-p', 'npm']);
```

# 结论

GuPM 的设计考虑到了无缝性，为您的项目提供了最大的灵活性和定制性。这篇文章向您简要介绍了如何轻松地将它与 NPM 结合使用，当然，您也可以随时查看我们的 github(和⭐ it！)以获得关于如何让你成为 GuPM 忍者的其他文档！

[](https://github.com/azukaar/gupm) [## azukaar/GuPM

### 🐶📦全球通用项目管理器-包管理器，cli 工具，脚本为您的所有项目和您的系统…

github.com](https://github.com/azukaar/gupm) 

不要犹豫，也可以在 Twitter 上访问我:

[](https://twitter.com/stepienik) [## 扬·斯特皮尼克(@斯特皮尼克)|推特

### Yann Stepienik (@stepienik)的最新推文。员工英雄的工程主管- Javascript 土豆，作者…

twitter.com](https://twitter.com/stepienik) 

感谢阅读！
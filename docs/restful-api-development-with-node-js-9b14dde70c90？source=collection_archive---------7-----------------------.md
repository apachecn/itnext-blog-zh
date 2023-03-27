# 使用 Node.js 和 Express 进行 RESTful API 开发:设置和配置开发环境(第 1 部分)

> 原文：<https://itnext.io/restful-api-development-with-node-js-9b14dde70c90?source=collection_archive---------7----------------------->

![](img/ebc0cd65e567c761d7578339fee77045.png)

或许你已经听过很多了，但你仍然不清楚什么是 RESTful API，它做什么以及如何创建一个！别担心。我在这种情况下的时间比你想象的要长，而且花了相当长的时间才学会！学习 RESTful API 的最好方法是边做边学。相信我！如果你刚开始创建一个，你会学到很多东西，我可以向你保证，除了 NodeJs 和 Express 之外，没有其他栈会帮助你入门！

但是等一下！你对 NodeJS 一无所知，还有 Express 甚至 JS。现在怎么办？

别担心，静观其变。这将是一篇由几部分组成的文章，它将帮助你了解 RESTful API 设计，在本系列结束时，你可能不是一个专业的开发人员，但你仍然会学到很多好东西。

让我们从 RESTful API 的一些描述开始吧！开始了。

# 什么是 RESTful API

RESTful API 是一种基于客户端-服务器的架构，它利用无状态操作通过 HTTP 请求来访问和操作数据。REST 代表代表性状态转移。Restful 架构的主要要求之一是它由两个核心元素组成，客户机和服务器。客户端向服务器请求资源来执行不同的操作。基于 RESTful API 的系统是无状态的，因为从客户端到服务器的每个请求必须包含理解该请求所必需的所有信息，并且不能利用服务器上存储的任何上下文。REST API 旨在使用 HTTP 协议和 CRUD 功能(CRUD 代表创建、读取、更新、删除)为特定操作创建端点。HTTP 定义了一组请求方法，根据 CRUD 功能来指示要对给定资源执行的操作。下面列出了用于设计 REST API 的 HTTP 协议。

*   发布—创建资源或通常提供数据
*   获取—检索资源或单个资源的索引
*   上传—创建或替换资源
*   修补—更新/修改资源
*   删除—删除资源

说够了！让我们从动手开始。

# Node.js 和 Express 的 RESTful API 开发:设置和配置开发环境

要开始 reSTful API 开发，首先，我们需要创建一个开发环境。在跳转到编码之前，需要安装和配置以下内容。我们将只使用开源和免费使用的堆栈。

*   编程堆栈和包管理器-NodeJS & NPM
*   源代码编辑器— Visual Studio 代码/ WebStorm(编辑器)
*   REST API 测试工具—邮递员/失眠
*   版本控制— Github

## Node.js 和 NPM

NodeJS 是 Chrome 基于 V8 JavaScript 引擎的运行时，是一个开源、轻量级、高效的服务器，广泛用于开发基于客户端-服务器的应用程序。NodeJs 的性能是非阻塞和异步的，它是一个基于事件驱动的运行时环境。Node 的官方包经理是 NPM。

## Windows(以及 MAC)

从以下链接下载所需的安装程序，双击下载的文件并按照安装提示进行安装。

```
[https://nodejs.org/en/](https://nodejs.org/en/)
```

## 通过自制软件安装节点(MAC 和 Linux)

家酿简化了软件在 Mac OS X 操作系统上的安装。使用以下命令安装适用于 MAC 或 Linux 的 Homebrew:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

或者，您也可以使用以下命令:

```
/usr/bin/ruby -e “$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew doctor
```

在终端中键入以下命令来安装节点。

```
brew install node
```

如果一切安装成功，那么您可以在终端中键入以下命令来检查节点和 NPM 版本。

```
node -v
npm -v
```

## Ubuntu 18.04(从 Ubuntu 二进制发行库安装)

安装 Node 10.x 最新 LTS 版本的最简单方法是使用软件包管理器从 Ubuntu 二进制发行版存储库中获取它。这可以通过在终端上运行以下两个命令非常简单地完成:

```
curl -sL [https://deb.nodesource.com/setup_10.x](https://deb.nodesource.com/setup_10.x) | sudo -E bash — 
sudo apt-get install -y nodejs
```

## 安装源代码编辑器(Visual Studio 代码)

Visual Studio Code 是微软为 Windows、Linux 和 macOS 开发的源代码编辑器。这是一个简单而强大的快速源代码编辑器，支持数百种语言，具有语法突出显示、括号匹配、自动缩进、框选、代码片段、直观的键盘快捷键、轻松定制、社区贡献的键盘快捷键映射等功能。您可以通过从以下链接下载来开始使用 visual studio 代码。打开自动保存模式很重要。

**Linux:**

```
[https://code.visualstudio.com/docs/setup/linux](https://code.visualstudio.com/docs/setup/linux)
```

**苹果电脑:**

```
[https://code.visualstudio.com/docs/setup/mac](https://code.visualstudio.com/docs/setup/mac)
```

**视窗:**

```
[https://code.visualstudio.com/docs/setup/windows](https://code.visualstudio.com/docs/setup/windows)
```

你需要安装一个插件，让你的编码体验更有趣。这个插件的名字更漂亮，它是一个自动将混乱的代码转换成可读格式的工具。从这里安装:

```
[https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
```

## RESTful API 测试工具(邮递员和失眠)

Postman 是一个 API 开发的协作平台。我们将使用来测试在创建 RESTful API 期间创建的端点。可从以下网址下载并安装适用于 Windows 和 MAC 的 POSTMAN:

```
[https://www.postman.com/downloads/](https://www.postman.com/downloads/)
```

Postman 也可从谷歌 chrome 商店获得，可用作 Chrome 扩展:

```
[https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=fi](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=fi)
```

API 测试的另一个著名工具是失眠。它是一个免费的跨平台桌面，用于与基于 HTTP 的 API 进行交互。从这里下载失眠:

```
[https://support.insomnia.rest/article/11-getting-started](https://support.insomnia.rest/article/11-getting-started)
```

## 设置 Github

GitHub 是一个开源的存储库托管服务，具有跟踪 changelog 中的变化的功能，这样您就可以准确地知道每次代码中发生了什么变化。

## Linux 操作系统

打开终端窗口。将以下内容复制并粘贴到终端窗口，然后按回车键。可能会提示您输入密码。

```
sudo apt update
sudo apt upgrade
sudo apt install git
```

## 苹果个人计算机

将以下内容复制并粘贴到终端窗口，然后按回车键。

```
brew install git
```

现在可以用 Git 了。

## Windows 操作系统

Git for windows 可从以下网址下载:

```
[https://gitforwindows.org/](https://gitforwindows.org/)
```

如果您是 Git 新手，我建议您阅读:

```
[https://kbroman.org/github_tutorial/pages/first_time.html](https://kbroman.org/github_tutorial/pages/first_time.html)
```

你最终会对首次配置 GitHub 有更多的了解。

关于建立环境的讨论已经说得够多了。现在是开始开发的时候了。本文的第二部分将指导您如何开始使用 node js 和 express 编写您的第一个 API。这里可以看[。](https://medium.com/@humayun.ashid/restful-api-development-with-node-js-and-express-getting-started-with-coding-part-2-7c79e27fafbb?sk=4b1d9cd0be60190d65c80cd37c6ca507)

编码快乐！
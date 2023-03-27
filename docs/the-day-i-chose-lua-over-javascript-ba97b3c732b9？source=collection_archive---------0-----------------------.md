# 那天我选择了 Lua 而不是 Javascript

> 原文：<https://itnext.io/the-day-i-chose-lua-over-javascript-ba97b3c732b9?source=collection_archive---------0----------------------->

![](img/c4015e783d6aa2e5927604e0994e29e1.png)

过去两年我一直在 Tabnine 工作。

Tabnine 是一个很棒的 AI 代码生成工具，可以直接在你的 IDE 中运行，提供内嵌的、整行的、全功能的代码完成。

虽然 Tabnine 可以在几个 IDE 上工作，但它不支持 Vim(准确地说是 Neovim)中的全功能完成，这是我选择的 IDE。

我对开发一个不能日常使用的产品感到有些不舒服——所以我决定在 Vim 中提供全功能的完成。

# 入门指南

在完成了 VS 代码和 Intellij 插件之后，我知道首先我需要确保虚拟线 API 存在于 Neovim 中。

我谷歌了一下[发现了这个](https://twitter.com/neovim/status/1442152074523734020?lang=en)。

太好了！

虽然我已经使用 Vim 很多年了——并且有了相当不错的配置集——但是我从来没有真正为它开发过东西——我主要是将其他插件的配置复制粘贴到我的`init.vim`文件中。

那么，我应该从哪里开始呢？

我从过去的经验中(主要是从使用其他插件中)知道 Neovim 插件是用 Lua 或 Viml 编写的。

Viml 看起来什么都不像，所以它马上就被搁置了，而且我从来没有用 Lua 编码过..语法看起来有点…嗯。

希望我可以避免使用 Lua，我在网上搜索其他解决方案。

这是第一个引起我注意的东西。

Neovim 的节点客户端！嘣，我不能指望更多了。我最初的想法是派生出`tabnine-vscode`项目(用 typescript 编写),并迅速为 Neovim 做出调整。我估计最多要花两天时间！

1 小时后，我已经很沮丧了。即使是最简单的事情也几乎没有文档。我勉强运行了最基本的例子。

我试图搜索用这个 SDK 编写的 Neovim 插件，以了解如何开始。实际上没有。我看了看`node-client`回购——它勉强维持着。但是我还没准备好放弃。

在花了几个小时试图调整`tabnine-vscode`插件后，我设法得到了一些基本的配置来运行。但是感觉还是没有烤熟。然而，由于没有文档或示例，我陷入了困境。我意识到我走错了路。

我搜索了 Neovim 的 GitHub 页面，在那里我找到了 Neovim 的 Python 客户端`pynvim`。太好了！大家都爱 Python！我以为会有大量的 Python 插件。不对。令我惊讶的是，几乎没有。

好吧，我知道了，尼奥维姆。我去学 Lua！

# 接受我的命运

根据 Lua.org 的说法，Lua 是一种通用嵌入式编程语言，旨在用数据描述工具支持过程化编程。

Lua 的 Neovim 运行时非常简单，所以我理解为什么开发者选择它。

# 全局 Vim 对象

Nevim 公开了一个全局 vim 对象，以便与其 API 进行交互，例如:

`vim.fn.col(“.”)`获取光标下的当前列。

或`vim.api.nvim_buf_set_text`将文本添加到缓冲区。

这个 API 的文档很棒，可以用`:help`命令很容易地搜索到。

我还了解到，虽然 Lua 本身缺少简单的操作，如`array.map`或`string.split`(或者 Lua 行话中的表格)，但 Neovim 会公开它们，例如`vim.tbl_map`或`vim.fn.split`。

# 简单

Lua 怀念卷曲的作用域，这是我在编程语言中不喜欢的。但是我已经学会喜欢它的简单。语法类似于其他高级语言——与 Viml 相反。

您不需要习惯任何花哨/复杂的语法，从而缩短了学习曲线。

基本上，Lua 完美地完成了它的工作，即嵌入到其他平台中。

# Lua 模块

*   Lua 模块类似于 Javascript commonjs 模块
*   Vim 有一个运行时路径的概念，即它在运行时查找模块的目录列表
*   Vimplug(和其他插件管理器)为您将插件加载到运行时路径中
*   当 Lua 模块位于运行时路径中时，可以通过在`init.vim` / `init.lua`中使用`require(‘module-name’)`来轻松加载它

# 证明文件

Vimdocs 很好地覆盖了 Vim 全局对象下的所有 Lua 函数。

Lua 文档[也相当不错](https://www.lua.org/docs.html)。该网站看起来像 90 年代的东西，但内容还不错。

# 摘要

我绝对能理解为什么大家写 Neovim 插件都选择 Lua。

感觉 node & Python 客户端只是 Neovim 团队为了让插件开发更容易而进行的一次不成功的实验，而 Lua 的工作方式正是它的本意。

此外，当你克服了学习一门新语言的学习曲线，这甚至是简单的，我敢说——快乐的！

**底线:** [**当你开发 Neovim 插件**](https://github.com/nanotee/nvim-lua-guide) **的时候，选择 Lua。你甚至可能会喜欢它🙂**

neovim 的 Tabnine 试试吧！https://github.com/codota/tabnine-nvimT2
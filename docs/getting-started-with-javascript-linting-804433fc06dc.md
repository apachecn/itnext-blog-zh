# JavaScript 林挺入门

> 原文：<https://itnext.io/getting-started-with-javascript-linting-804433fc06dc?source=collection_archive---------5----------------------->

ESLint 初学者指南

![](img/f15460af01cf5bb3dc742c90b97aa927.png)

照片由 [Nhu Nguyen](https://unsplash.com/photos/kr-jmsASg8M?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/search/photos/mechanical-keyboard?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fgetting-started-with-javascript-linting-804433fc06dc)

使用 JavaScript 可能是一次疯狂的旅行。如今，我们已经有了一些工具来帮助我们让这种骑行不像过去那么狂野。进入现代 JavaScript 堆栈可能会令人望而生畏，所以这篇文章旨在帮助您尝试一下林挺，以期帮助您更好地理解关于该主题的大量更复杂的文章。

林挺游戏的目的是帮助你在野外部署更好的代码。与测试一起使用，它可以在防止损害用户体验的愚蠢错误方面发挥巨大作用。最初很难理解，所以让我们做一些真正高层次的东西，只关注最基本的元素，这样你就可以看到它是如何工作的。之后，你应该有希望能够继续学习你今天学到的基础知识。

让我们开始吧。

# ESLint 简介

[ESLint](https://eslint.org/) 将自己描述为*“用于 JavaScript 和 JSX 的可插拔林挺工具”。尽管如此，我认为这篇文章还需要进一步完善。*

打个比方，ESLint 就像一个停车场管理员。它无情而迂腐，但尽管你想恨它，你也不能，因为它有助于维持秩序。

## 但是它有什么用呢？

你制定了一些家规，如果违反了这些家规，ESLint 会告诉你。您可以通过多种方式实施这些规则，例如:

*   你可以在你的终端上运行`eslint your-filename.js`
*   您可以将它附加到您的构建设置中，如 Webpack 或 Gulp。如果遇到问题，ESLint 可以立即停止构建
*   你可以在你的代码编辑器中设置它，这样它就可以一直监视你正在处理的文件
*   您可以在预提交挂钩中设置它，这样无效的代码就不会添加到您的存储库中

其中一些场景非常激烈，您是来学习基础知识的，所以让我们就这样做，解决列表中的第三点。我们将学习如何用你的代码编辑器运行 ESLint。

# 入门指南

我们需要做的第一件事是用 NPM 在你的机器上全局安装 ESLint。如果您还没有安装 Node 或 NPM，[继续做第一个](https://nodejs.org/en/download/)。

打开您的终端应用程序并运行以下内容:

```
npm install -g eslint
```

这个命令基本上是说*“从 NPM 获取 ESLint，并在我的机器上全局访问* `*eslint*` *命令”*。

现在我们已经安装了它，我们需要设置我们的规则，我们有几个选项可供我们使用。

第一个选项是通过运行以下命令在终端中导航到您的项目:

```
cd /the/path/to/your/project
```

一旦你在那里，运行:

```
eslint --init
```

这将启动一个安装向导，它会询问你关于你的项目的问题，扫描你的项目或者让你选择一个流行的模板。这很好，但我认为我们会从第二个选项中学到更多。

这个选项是将一个`.eslintrc`文件添加到您的项目根目录并填充它。

一个`.eslintrc`或`.eslintrc.js`文件是你的林挺规则的定义文件。这非常有用，因为它保持了`eslint`终端命令的整洁，并且为您和您的团队创建了一个完全透明的设置，这在您执行编码风格和规则时非常重要。

我们已经从上面的命令进入了项目目录，所以让我们创建一个`.eslintrc`文件:

```
touch .eslintrc
```

如果您运行了 init 向导，您将希望通过运行以下命令来清理生成的文件:

```
rm .eslintrc.js
```

一旦创建了`.eslintrc`文件，打开它，我们将向它添加一些基本规则。让我们从添加我最喜欢的一个开始。将以下内容添加到文件中:

```
{
    "rules": {
        "no-unused-vars": "warn"
    }
}
```

所以，我们添加了一个基本规则:*“如果有任何未使用的变量，给出一个警告”。这是一个很好的入门规则，非常值得添加到您的代码库中。通过删除代码中对未使用变量的引用，您可以在性能和文件大小方面节省*吨*。*

接下来，在继续我们的编辑器集成之前，让我们添加另一个规则。将以下内容添加到我们的`.eslintrc`文件中的“rules”部分:

```
"quotes": [2, "single", { "allowTemplateLiterals": true }]
```

这条规则在 JavaScript 上强制执行单引号策略，除非您在模板文本中。很漂亮吧？

作为参考，您的`.eslintrc`文件现在应该是这样的:

```
{
    "rules": {
        "no-unused-vars": "warn",
        "quotes": [2, "single", { "allowTemplateLiterals": true }]    
    }
}
```

## 整合我们的编辑器

这一步确实取决于你选择的编辑器，所以这里有一些最受欢迎的编辑器的链接列表。给你看完相关的，然后回来继续！

*   [设置 Vim](https://medium.com/@hpux/vim-and-eslint-16fa08cc580f)
*   [设置 VS 代码包](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
*   [设置 Atom 包](https://atom.io/packages/eslint)
*   [设置崇高文本](https://github.com/SublimeLinter/SublimeLinter-eslint)

现在您的编辑器已经配置好了，让我们来测试一下。在项目根目录下创建一个名为`hello.js`的文件。

将以下代码添加到`hello.js`:

```
module.exports = function() {

    var hello = "Hello, world";
    var goodBye = "Goodbye, pal";

    return hello; 
};
```

现在，当你保存的时候，你的编辑器应该给你两个错误和一个警告。这是因为我们设定的两个规则都被打破了:

*   我们的字符串用双引号括起来
*   变量`goodBye`从未在函数中使用过

很直接，对吧？让我们修改代码来结束这个小教程。

用以下内容替换`hello.js`的内容:

```
module.exports = function() {

    var hello = 'Hello, world';

    return hello; 
};
```

当您保存时，错误和警告应该会消失，因为您的代码现在已经完全链接了！🎉

# 包扎

你已经从“WTF 是 ESLint？”到“我现在对什么是什么有了一个很好的想法”，这很棒！

你也许应该更深入地研究并扩展你的`.eslintrc`文件，以便对你的项目更有用。我们添加的只是一些规则，所以你应该[查看文档](https://eslint.org/docs/user-guide/configuring)并从那里开始。如果你想看一个基本的例子，可以看看我添加到[boile form GitHub 库](https://github.com/hankchizljaw/boilerform/blob/master/.eslintrc)的例子，这个库是我维护的。

您也可以返回，删除您的`.eslintrc`文件并再次运行
`eslint --init`命令，现在您已经对正在发生的事情有了更好的理解。

林挺世界现在是你的了！

> 如果你喜欢那个帖子，给它一些掌声，这样它就可以到达其他人那里，帮助他们掌握 ESLint 的基础知识。如果你有评论，请添加回复或[在 Twitter 上联系我](https://twitter.com/hankchizljaw/)。
> 
> 快乐的林挺！
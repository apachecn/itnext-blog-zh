# 学习添加 git 钩子任务:PHP-CS-Fixer

> 原文：<https://itnext.io/learning-to-add-git-hook-tasks-php-cs-fixer-41f34d99aa8a?source=collection_archive---------2----------------------->

![](img/77f6fe084fdea2f68e7321f98504649f.png)

(我想知道我是否能把这个标题变得更古怪些？)

让我们首先描述一下这项任务的各个部分，以及我们今天要做的事情。总体目标是这样的:

我们希望我们所有的应用程序文件都符合特定的代码风格，因此为了减少强制执行的麻烦，我们希望使用 PHP-CS-Fixer(【https://github.com/FriendsOfPHP/PHP-CS-Fixer】T4)。

然而，试图让整个团队总是记得在推送代码之前运行这一点就像放牧猫一样，所以我们也想让这成为提交过程的自动化部分。

不要害怕！下面我将向您展示的是一种利用 git 预提交钩子来设置这一切的方法。我们将这样做来设置 PHP-CS-Fixer，但是很明显如何设置它来自动完成几乎任何事情。

让我们首先使用一个干净的文件或框架(我将使用一个 Laravel 设置来处理这类事情)并用`git init`创建一个空的 git 存储库。(你可以随意使用现有的任何东西，只要你知道你在做什么。)不需要为此设置远程存储库——我们不会进行任何推送，只会提交。

我讨厌那些“不在本文讨论范围之内”的教程，但是我不得不要求你先为你自己的作曲者([https://getcomposer.org](https://getcomposer.org))设置一些东西，然后引入 PHP-CS-Fixer([https://github.com/FriendsOfPHP/PHP-CS-Fixer](http://PHP-CS-Fixer))包。

PHP-CS-Fixer 有很多选项和运行方式；我们将保持简单，只建立一个`.php_cs`文件并从我们的`vendor/bin`本地运行。所以把这个文件放在你的根目录下:

```
<?php // file ".php_cs"

use PhpCsFixer\Config;
use Symfony\Component\Finder\Finder;

$finder = Finder::create()
    ->in(__DIR__)
    ->name('*.php')
    ->ignoreDotFiles(true)
    ->ignoreVCS(true);

return Config::create()
    ->setRules([
        '@PSR2' => true,
        'array_indentation' => true,
        'array_syntax' => ['syntax' => 'short'],
        'binary_operator_spaces' => true,
        'blank_line_after_namespace' => true,
        'blank_line_before_return' => true,
        'braces' => true,
        'class_definition' => true,
        'method_chaining_indentation' => true,
        'no_extra_consecutive_blank_lines' => true,
        'no_multiline_whitespace_before_semicolons' => true,
        'no_short_echo_tag' => true,
        'no_spaces_around_offset' => true,
        'no_unused_imports' => true,
        'no_whitespace_before_comma_in_array' => true,
        'not_operator_with_successor_space' => true,
        'ordered_imports' => ['sortAlgorithm' => 'length'],
        'trailing_comma_in_multiline_array' => true,
        'trim_array_spaces' => true,
        'single_quote' => true,
    ])
    ->setFinder($finder);
```

然后从命令行运行`php vendor/bin/php-cs-fixer fix`。我相信，这是最简单的方式，可以让事情变得……呃，简单。如果有任何运行不正常的地方，请立即停止并确保修复它。这里有一篇来自[Billie Thompson](https://medium.com/u/3ede9289a22b?source=post_page-----41f34d99aa8a--------------------------------)([https://dev . to/purple boot/coding-standards-in-PHP-using-PHP-cs-fixer](https://dev.to/purplebooth/coding-standards-in-php-using-php-cs-fixer))的优秀文章，它解释了关于这个工具做什么的更多信息，或者询问互联网上的好心人。

因此，现在设置强制您的文件符合 PSR-1 和 PSR-2(默认)。进入一个文件，做一些违背你对世界上一切美好事物的感觉的改变——比如说，在关键字`function`的同一行放一个左括号——然后运行这个程序，看看它是否已经被修复。神奇！再次清洗。

害怕自动乱码脚本化的东西的我会很乐意就此打住，在我的 git commits 上设置一个别名或其他东西，以确保我不会经常忘记运行它，但不管怎样，大多数时候都会忘记，并经常被抱怨。新的我将自动完成这项工作。

![](img/905dc7f6ceca7d5a7439ff964eac67fa.png)

旧的我，面临着写一个自动混乱的剧本的前景

正如我在开始时所说的，有无数种方法来做这种事情，但是我们要做的是使用一个叫做“Husky”([https://libraries.io/npm/husky](https://libraries.io/npm/husky))的`npm`包，它被设计成接入现有的 Git 挂钩([https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks))。在你安装好哈士奇之后，我要你打开你的`.git/hooks`目录，然后……天哪，看看那些钩子文件！

不要担心——我们不会直接与他们合作。其中大部分是在安装 Husky 时创建的，node.js 会知道需要做什么(如果你想打开预提交文件，你会看到 Husky 确实创建了它。)相反，我们将进入我们的`package.json`文件，给 Husky 我们想要的命令:

```
"husky": {
  "hooks": {
    "pre-commit": "vendor/bin/php-cs-fixer fix && git add ."
  }
},
```

*注意命令中没有* `*php*` *！*

让我们来测试一下。回去把你之前用过的那个文件弄得难看一点，然后用`git status`快速检查一下，看看它是不是在等着上演。当你准备好了，不要用`git add .`；只需承诺:

```
git commit -m "Committing prettified version of file" 
```

并再次检查状态。已经上演；该文件已经过清理，符合 PSR 标准。你做到了！

请注意，我们通过管道传输了`git add .`——因为这是*预提交，*我们所做的是首先修复文件样式并暂存任何已更改的内容，*然后*提交所有内容。

这个例子遵循了我所能找到的最窄的工作路径，但是当你浏览 Husky 和 Git 钩子的文档时，你会发现几乎每一步都有定制的方法，直到你找到满足你需要的东西。

希望有所帮助！
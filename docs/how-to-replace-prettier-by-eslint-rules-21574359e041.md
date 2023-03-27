# 如何用 ESLint 规则替换 beautiful？

> 原文：<https://itnext.io/how-to-replace-prettier-by-eslint-rules-21574359e041?source=collection_archive---------3----------------------->

![](img/6fbab526858410664ca8bc6340830129.png)

马库斯·斯皮斯克在 [Unsplash](https://unsplash.com/search/photos/code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

# 为什么不漂亮点？

更漂亮的是一个众所周知的解决方案，可以在项目中实施一致的编码风格。它可以解决诸如重写超过 80 个字符的代码、处理尾部逗号或分号等问题

ESLint 作为一个 linter，实现确保代码质量的规则，帮助开发人员避免在代码编译或执行过程中经常会产生影响的错误。但是作为一个重写/格式化系统，它的功能要强大得多，并且有更漂亮的代码格式化规则。

作为一个非常固执己见的工具，你可以配置的选项非常少。因此，如果你也对如何格式化你的代码有很大的意见，这就有很大的冲突。

在这篇文章中，我将列出并解释如何用等价的 ESLint 规则快速替换更漂亮的规则，然后让您真正适应自己的需要。

我知道比漂亮是设计出来的:

> 到目前为止，采用 Prettier 的最大原因是为了停止所有正在进行的关于风格的争论。

但是来自 Python，并被“格式化”为遵循我们所谓的 PEP8，它描述了常见的 Python 代码样式，对我来说更重要的规则如下:

> 愚蠢的一致性是心胸狭窄的大地精

因此，当 Prettier 重写代码以确保 80 个字符的行长度替换一些局部一致性时，明智地选择特定的代码片段，这是一个大恶魔。

例如，这段代码…

```
{ message &&
  <Text style={styles.message}>{message}</Text>
}
{ confirmationMessage &&
  <Text style={styles.message}>{confirmationMessage}</Text>
}
```

变成了…

```
{message && <Text style={styles.message}>{message}</Text>}
{confirmationMessage && (
  <Text style={styles.message}>{confirmationMessage}</Text>
)}
```

真的吗？！…我不想要一个只是这样重写代码的工具，因为它“太长了”。我不想要更聪明的工具。因为我认为对于开发者的体验和可维护性来说，一致性比很多其他事情都更重要。
和 ESLint 规则允许这种“更智能”的配置。

# 从 beauty 到 ESLint

## 线条长度、间距样式和大小

[【更漂亮】打印宽度](https://prettier.io/docs/en/options.html#print-width)可替换为[【ESLint】Max-Len](https://eslint.org/docs/rules/max-len)规则:

`max-len: ["error", { "code": 80 }]`

ESLint 规则可以通过多种方式进行调整，例如允许不同长度的注释。

## 空格/制表符一致性

[【更漂亮】制表符](https://prettier.io/docs/en/options.html#tabs) & [【更漂亮】制表符宽度](https://prettier.io/docs/en/options.html#tab-width)可以替换为[【ESLint】缩进](https://eslint.org/docs/rules/indent):

`indent: ["error", 2]`

可以使用[【ESLint】无混合空格和制表符](https://eslint.org/docs/rules/no-mixed-spaces-and-tabs)来加强文件的一致性。

这些 ESLint 规则有选项来指定特定情况下的缩进，比如处理`switch`时的`case`，比如在某些情况下使用`smartTabs`来垂直对齐项目的能力，…

## 分号

[【更漂亮】分号](https://prettier.io/docs/en/options.html#semicolons)可以换成[【ESLint】分号](https://eslint.org/docs/rules/semi):

`semi: ["error", "always"]`

## 引用

[【更漂亮】语录](https://prettier.io/docs/en/options.html#quotes)[【更漂亮】JSX 语录](https://prettier.io/docs/en/options.html#jsx-quotes)可以换成[【ESLint】语录](https://eslint.org/docs/rules/quotes):

`quotes: ["error", "double"]`

ESLint 选项还允许选择反引号，或者在可能的情况下使用单引号，并使用双引号来避免转义，…

## 尾随逗号

[【更漂亮】尾随逗号](https://prettier.io/docs/en/options.html#trailing-commas)可以替换为[【ESLint】逗号-](https://eslint.org/docs/rules/comma-dangle):

`comma-dangle: ["error", "never"]`

在这里，规则可以被调整为增强或减弱尾随逗号，并选择哪些元素应该有尾随逗号，以实现与`es5`或`all`漂亮器的参数相同的效果。

## 间距一致性

[【更漂亮】括号间距](https://prettier.io/docs/en/options.html#bracket-spacing)可以替换为[【ESLint】对象卷曲间距](https://eslint.org/docs/rules/object-curly-spacing):

`object-curly-spacing: ["error", "always"]`

但是也有[【ESLint】数组括号间距](https://eslint.org/docs/rules/array-bracket-spacing)和[【ESLint】空格间距](https://eslint.org/docs/rules/space-in-parens)来配置数组和括号中的空格，或者[【ESLint】逗号间距](https://eslint.org/docs/rules/comma-spacing)来…逗号！

这些选项允许专门调整嵌套括号、括号中的嵌套数组等的空间

## JSX 支持这一立场

[【更漂亮】JSX 括号](https://prettier.io/docs/en/options.html#jsx-brackets)可以替换为[【ESLint/React】Jsx-closing-bracket-location](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)，它是 [eslint-plugin-react](https://github.com/yannickcr/eslint-plugin-react) 的一部分:

`react/jsx-closing-bracket-location: [1, "ligne-aligned"]`

它存在于其他位置，比如总是在道具之后，或者与开放标签对齐。

当然，插件也有很多其他规则可用。

## 箭头函数括号

[【更漂亮】箭头函数括号](https://prettier.io/docs/en/options.html#arrow-function-parentheses)可替换为[【ESLint】箭头括号](https://eslint.org/docs/rules/arrow-parens):

`arrow-parens: ["error", "as-needed"]`

有一个选项始终只对带有“块”体(带花括号)的箭头函数使用括号。

[【ESLint】箭头体样式](https://eslint.org/docs/rules/arrow-body-style)用于设置箭头函数的体样式。

## 行尾

[【更漂亮】行尾](https://prettier.io/docs/en/options.html#end-of-line)几乎等同于[【ESLint】换行符样式](https://eslint.org/docs/rules/linebreak-style)但是最后一个没有像更漂亮那样的“自动”模式。因此，“最接近”的行为是禁用该规则:

`linebreak-style: 0`

## 特定解析器

[【更漂亮】解析器](https://prettier.io/docs/en/options.html#parser)并不是真正的规则，但是可以在 [ESLint 解析器选项](https://eslint.org/docs/user-guide/configuring#specifying-parser-options)中配置。

## 要格式化的文件

[【更漂亮】文件路径](https://prettier.io/docs/en/options.html#file-path)可以用 ESLint [配置](https://eslint.org/docs/user-guide/configuring)或 [CLI](https://eslint.org/docs/user-guide/command-line-interface) 参数实现，老实说，ESLint 文档比更漂亮的文档更明显、更清晰，所以它允许更多。

# ESLint 中没有(或不容易)提供的选项

*   完全重写:作为一个“过客”，ESLint 更专注于在出错时“警告”你，而不是重写。所以有些规则在使用`eslint --fix`时没有自动解析。例如
*   [【更漂亮】范围](https://prettier.io/docs/en/options.html#range)只允许格式化一段文件。据我所知，ESLint 没有“开箱即用”的配置，但也许一个插件可以完成这项工作
*   [【更漂亮】要求编译指令](https://prettier.io/docs/en/options.html#require-pragma) & [【更漂亮】插入编译指令](https://prettier.io/docs/en/options.html#insert-pragma)可能是 ESLint 中最缺少的漂亮特性，因为它允许“软”迁移代码库。但是，由于 ESLint 更具可配置性，您可以通过[覆盖](https://eslint.org/docs/user-guide/configuring#disabling-rules-only-for-a-group-of-files)并将错误转化为警告来实现同样的事情
*   [【更漂亮】散文换行](https://prettier.io/docs/en/options.html#prose-wrap)是一个“降价”的特定规则，我不知道在 ESLint 中是否有类似的规则。也许用覆盖和[【ESLint】无尾随空格](https://eslint.org/docs/rules/no-trailing-spaces)可以实现一些东西
*   据我所知，ESLint 没有解决 HTML 空白敏感问题。当发生这种情况时，我使用下面的技巧(在 IDE 中这并不难看，配置为用柔和的颜色给 HTML 注释着色) :

```
<div><!--
-->Ne se dici quod cognatione inter vinculum <!--
-->dici disciplina dici disciplina nos dici <!--
-->nos quod in alia disciplina disciplina <!--
-->cognatione nobis dicendi nobis  disciplina <!--
-->uni habent artes ne artes et uni aut quae <!--
-->huic continentur  ratio et ita dicendi studio.
</div>
```

*   [【更漂亮】行尾](https://prettier.io/docs/en/options.html#end-of-line):注意没有“auto”模式，event 如果我觉得现在大部分项目强制执行`lf`应该不成问题。

# TLDR &免责声明

由于我不太喜欢 prettle，我不习惯用它来格式化我的代码，所以我不能保证给定的 ESLint 规则与 prettle 的格式完全相同。但我认为这给了你一个打破束缚的好开始。

```
rules: {
  "max-len": ["error", { code: 80 }],
  "indent": ["error", 2],
  "semi": ["error", "always"],
  "quotes": ["error", "double"],
  "comma-dangle": ["error", "never"],
  "object-curly-spacing": ["error", "always"],
  "react/jsx-closing-bracket-location": [1, "line-aligned"],
  "arrow-parens": ["error", "as-needed"],
  "linebreak-style": 0,
}
```

如果你有任何建议，改进，插件，请评论！
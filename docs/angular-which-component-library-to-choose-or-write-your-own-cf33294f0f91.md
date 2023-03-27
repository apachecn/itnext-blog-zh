# Angular —选择组件库还是自己编写？！

> 原文：<https://itnext.io/angular-which-component-library-to-choose-or-write-your-own-cf33294f0f91?source=collection_archive---------0----------------------->

![](img/0c3c9ace5d4a82e230def29868b66b5b.png)

由[奥斯汀·迪斯特尔](https://unsplash.com/@austindistel?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/developer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fangular-which-component-library-to-choose-or-write-your-own-cf33294f0f91)

# 为什么我更喜欢用组件库？

当然，为了加快我的发展。我遇到的每一个 bug，可能都是别人遇到并修复的。我需要的任何功能都一样。

那么，如果我可以轻松地嵌入一个现有的库，为什么要编写自己的库呢？！

# 我需要考虑什么？

## 1.支持

*   是开源的吗？
*   谁负责问题和拉动式请求？
*   有没有强大的团队来维护这个？

## 2.稳定性

*   组件的运行是否顺畅？
*   有没有 bug？
*   性能和内存都可以吗？
*   所有组件的行为是否一致？
*   覆盖 css 有多难？
*   让它符合我的要求有多难？

## 3.可用组件的数量

如果他们没有/很少支持我的大多数公共组件，那么我需要将它与另一个库相结合。这可能会造成伤害，尤其是在升级依赖项时…而且，我可能没有一致的输入/输出命名约定，这可能会让开发人员感到困惑。

# 2017 年 12 月—哪个库符合以上所有条件？

没有人…(棱角分明，耶？)

# 怎么办？

选择 2 到 3 个回答以上大部分问题的库，上面做一个**抽象**层。

**表示**，用自己的组件包装任何外部库组件。

# 为什么？

1.  **创建一致性**:相同的输入/输出命名约定
2.  不可知的库:当另一个库变得更加成熟时，你可以改变你的观点。不需要改变你所有的代码基础，只需要抽象层。
3.  **简化输入/输出**:

*   可以使用自己的数据类型进行输入/输出，而不是外部库类型:像`primeNg` tree 组件，它处理类型`TreeNode.`中的树数据，你可以通过处理代码库中的每个类来简化它。(稍后我将展示示例)
*   合并分离对你没有意义的事件:像 combox 有很多值变化的输出:`lostFocus`，`keydown.enter`等等…你可以合并成一个事件:`onValueChange`
*   将行为相同的独立组件合并成一个组件:像`Primeng` 有两个不同的组件用于多选——一个作为下拉列表，一个作为列表。您可以使它们成为一个有输入`isDropDown`的组件(参见下面的例子)
*   隐藏不相关的输入/输出

# 简单包装的例子

这里我将多选和列表框包装在一起，但是只有列表框代表多选(通过设置 `[multiple]=”true” [checkbox]=”true”`)

`Primeng` 需要为`options`输入使用它们的`SelectItem`类，为`ngModel`输入使用一个字符串数组(选中的选项)。在我的包装类中，我用两个输入`data**:** T[]`和`value **:** T[],` 替换了它。现在我可以实现两个目标:使命名符合我的约定，以及处理任何数据类型。输入`displayField:keyof T` 帮助我知道呈现哪个值。

# 结论:

使用外部库，但是因为 JS 世界太快，库可能不够成熟，所以在中间使用抽象层。
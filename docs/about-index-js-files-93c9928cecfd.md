# 关于 index.js 文件..

> 原文：<https://itnext.io/about-index-js-files-93c9928cecfd?source=collection_archive---------3----------------------->

Index.js 文件是 Ryan Dahl 在设计 Node.js 时想出的一个“可爱”功能。虽然他[正式对它们表示遗憾](https://www.youtube.com/watch?v=M3BM9TB-8yA)，但我认为它们已经发展成为结构化代码的有用工具。在这篇文章中，我将尝试阐明我为自己制定的规则。

![](img/52107d99ed500a9453b0b171df2d5578.png)

Maksym Kaharlytskyi 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

基本上，*、*索引文件服务于两个目的之一:作为*封装*或作为*名称空间*。其必然结果是索引文件充当了导入的护栏——我避免从包含索引文件的文件夹的子文件夹中导入内容。

## 包装

如果索引文件用于封装，它应该包含主要实体(类、组件、函数等)的默认导出。)在他的文件夹里。它还可能包含许多相关的“次要”项目的名称输出。当您想要隐藏模块的实现细节并仅公开单个接口时，这很有用。例如，考虑以下文件夹结构:

```
src
└── TopBar
    ├── index.js
    ├── TopBar.js
    ├── DropDown.js
    └── SearchBox.js
```

在这种情况下，`TopBar`文件夹包含一个顶栏组件的实现，包括几个子组件。索引文件重新导出 TopBar.js 中的组件:

```
// index.js
export { default } from './TopBar';
```

要使用该组件，您可以从`TopBar`文件夹导入默认导出:

```
import TopBar from './TopBar';
```

## 命名空间

如果一个索引文件被用作名称空间，那么它应该包含文件夹中所有内容的一组命名导出。我以前经常这样做，但是现在我已经停止这样做了，只需要在导入时输入文件夹名。然而，如果相关的实体感觉特别紧密相关，将它们分组在单个标识符下仍然是有用的。

例如，考虑以下文件夹结构:

```
src
└── shared-components
    ├── LoadingButton.jsx
    ├── Spinner.jsx
    └── ModalDialog
        ├── ModalDialog.jsx
        ├── index.js
        └── Overlay.jsx
    ├── Chevron.jsx
    └── index.js
```

在这种情况下，`shared-components`文件夹包含四个组件。中的 index.js 文件

```
export { default as LoadingButton } from './LoadingButton';
export { default as Spinner } from './Spinner';
export { default as ModalDialog } from './ModalDialog';
export { default as Chevron } from './Chevron';
```

这使得`shared-components`像一种包一样工作，您可以从中导入组件。从各自的子目录中导入组件的优势很小，但可能是您的偏好。

## 从子文件夹导入

如果一个文件夹有一个索引文件，我认为这是一个护栏，可以防止我从子文件夹中导入任何东西。现在，文件夹或子文件夹*中的文件本身*被允许导入内容，可以说你不应该越界。这是我想创建一个 linter 规则的原因，因为它可以自动执行，但现在我手动跟踪它。

## 紧急设计

这两种用例都非常适合紧急设计。

在封装的情况下，您可以从在单个文件中编写一些东西开始。当该文件变得太大或太复杂时，您用一个同名的文件夹(sans 扩展名)替换它，从其余代码的角度来看，一切都保持不变。

当用于名称空间时，这是在将相关代码移动到一个单独的包之前对其进行分组的很好的第一步，无论是在使用工作区的 mono-repo 中，还是一直到 NPM，这取决于它的通用程度。

## ..默认导出？？

一些人认为，违约出口的概念是一个错误，我对此有些同情。在这一点上，我使用它们太多了，基本上是肌肉内存，我发现 VSCode 有足够的重构能力来处理它们，缺点可以忽略不计。
# 如何让你缓慢的 Jest 测试进行得更快

> 原文：<https://itnext.io/how-to-make-your-sluggish-jest-v23-tests-go-faster-1d4f3388bcdd?source=collection_archive---------1----------------------->

![](img/620ac12c0eef5069ddb1162d99b6e3a5.png)

由[米歇尔·约克](https://www.pexels.com/@michelle-yorke-260837)拍摄的照片(略有修改)

# 让单元测试再次变得伟大

Jest v23 于 2018 年 5 月发布，此次发布的标题是“超快的令人愉快的测试”。

然而，在 2019 年初，感觉座右铭应该是“[完全迟缓的排斥测试](https://jestjs.io/blog/2018/05/29/jest-23-blazing-fast-delightful-testing.html)”。原因如下。

我最近决定在一个新加入的项目中提高单元测试的性能。感觉测试整个套件(仅 32 个测试)以及在观察模式下测试一个单独的文件花费了太多时间。这样做的直接后果是开发人员不喜欢这个过程，并试图避免接触单元测试。从长远来看，产品质量会受到影响。

接受挑战！我开始调查可能出错的地方。

这是我们最初的项目设置:

*   React v16.5
*   Typescript v3.1
*   Jest v23.6 使用`jsdom`作为测试环境
*   ts-jest
*   Win10

`npx --envinfo --preset jest`

```
System:
OS: Windows 10
CPU: (4) x64 Intel(R) Xeon(R) CPU E5–2698 v3 @ 2.30GHzBinaries:
Node: 8.11.1
npm: 5.6.0
```

长话短说，请在下面找到一系列步骤来加速你的测试*至少是* `2x`。

# Jest v23 存在性能问题

当在`watch`模式下运行时，Jest 需要花费大量的时间来更新，重新开始测试更容易。

结果是在版本`22.4.4`之后引入了[回归，该版本尚未修复，并导致*显著*减速。](https://github.com/facebook/jest/issues/6783)

**第一步** 降级笑话`npm install jest@22.4.4 --save-dev`

# JSDOM 环境比 Node 慢

JSDOM 是 WHATWG DOM 和 HTML 标准的 JavaScript 实现。换句话说，`jsdom`模拟了一个浏览器的环境，除了普通的 JS 之外没有运行任何东西。它运行测试的速度很快，但是没有 pure Node 快。差别可能是双重的。

Create react 应用程序的用户看到了 CRA v2 升级带来的性能的[回归，这反过来又带来了 jest v23。](https://github.com/facebook/create-react-app/issues/5304)

**步骤 2** `”testEnvironment”: “node”`可以设置为你测试的默认设置，它甚至允许浅层渲染和测试 React 组件。

```
// package.json"jest": {
  "testEnvironment": "node"
}
```

在那些需要使用 DOM 的情况下(例如挂载一个组件)，您可以通过在`*.spec.js`文件的顶部添加一个注释来选择使用另一个测试环境:

```
/**
 ** @jest-environment jsdom* */import React from 'react';
```

# 并行测试并不总是好的

虽然 Jest 在装有快速 IOs 的现代多核计算机上可能很快，但在某些设置上可能会很慢。

**步骤 3** 按照 Jest 规定的[，缓解这个问题并将速度提高 50%的一个方法是顺序运行测试。](https://jestjs.io/docs/en/troubleshooting)

`npm test --runInBand`

另一种方法是将最大工作池设置为~4。具体来说，在 Travis-CI 上(自由计划机器只有 2 个 CPU 核心)，这可以将测试执行时间减少一半。

`npm test --maxWorkers=4`

尝试这些标志，看看在本地和 CI 环境中什么最适合您。如果需要，为各种环境设置单独的任务。

我希望上面的步骤将有助于恢复你的单元测试，并使它更令人愉快。请在评论中让我知道在数字上有什么改进，哪些对你有用。

UPD: Jest v24 最近发布了，希望能看到上面一些问题的改进。

*这篇文章最初发表在我公司的博客上。我们在 Appfocused 的使命是通过利用我们丰富的经验、对现代 UI 趋势的了解、最佳实践和代码工艺，帮助公司在 web 上实现* [*卓越的用户体验。*](https://www.appfocused.com)
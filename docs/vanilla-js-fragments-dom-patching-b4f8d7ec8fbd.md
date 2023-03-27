# 普通 JS 片段和 DOM 补丁

> 原文：<https://itnext.io/vanilla-js-fragments-dom-patching-b4f8d7ec8fbd?source=collection_archive---------8----------------------->

我很好奇如何在不依赖`innerHTML`或者坦率地说，不依赖任何东西的情况下，自己做类似 react 的事情。结果很简单。

首先，通过[document . createdocumentfragment()](https://developer.mozilla.org/en/docs/Web/API/Document/createDocumentFragment)获得的片段非常神奇。它们存在于内存中，您可以像往常一样构造它们，基本上是创建一个微型虚拟 dom。

它们的目的是让您在实际渲染之前对未来的 DOM 进行大量的修改。这有助于控制重画计数，例如:

那就是 15k 重画减少到 1。

有了这个，昂贵的`innerHTML`永远不会有任何用例。然而，有时候我们*必须*将字符串解析成 HTML。我们仍然可以这样做。

参见[create contextual fragment](https://developer.mozilla.org/en-US/docs/Web/API/Range/createContextualFragment)——只要字符串有效，它就会(试图)将它解析成 HTML。示例:

为了好玩，我还通过片段更新文本节点，而不是修改 innerText/textContent。
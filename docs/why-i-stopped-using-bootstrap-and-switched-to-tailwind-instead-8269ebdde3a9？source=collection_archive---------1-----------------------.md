# 为什么我停止使用 Bootstrap 而改用 Tailwind

> 原文：<https://itnext.io/why-i-stopped-using-bootstrap-and-switched-to-tailwind-instead-8269ebdde3a9?source=collection_archive---------1----------------------->

![](img/7f993c3ba30525f002e45ed009f1c6ed.png)

李·坎贝尔在 [Unsplash](https://unsplash.com/s/photos/web-design?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

几年前，我爱上了 Bootstrap。当时，当我对 HTML、CSS 或任何其他编程语言几乎没有任何经验时，Bootstrap 似乎是我将终生使用的完美框架。它很容易使用，文档非常清晰，它被大量的网站使用，互联网上有很多关于它的教程。在过去的几年里，我听说过其他的 CSS 框架，但是它们对我来说从来都不感兴趣，所以我只是坚持使用 Bootstrap。

然而，几周前，当我在学习 VueJS 课程更新知识时，导师向我介绍了 Tailwind CSS。虽然这个框架对我来说并不陌生(我以前听说过)，但实际上我从未深入研究过它。当使用 Bootstrap 时，我使用了许多定制的 CSS 类，这经常导致非常不一致和混乱的代码(这主要是我自己的错，但仍然是)。随着课程的进行，我决定尝试一下 Tailwind，看看它是否真的更容易或者改进我的工作流程。剧透一下，确实如此！

对于不了解 Tailwind 的人来说:这是一个高度可定制的实用优先的 CSS 框架(这是他们网站上说的)。将它与 Bootstrap 进行比较实际上是不公平的，因为 Bootstrap 更像是一个 UI 工具包。Tailwind 并没有真正的默认主题，也不包含任何现成的模板或组件。顺风的理念是任何事情都可以被轻易改变。如果你以前和我一样使用过 Bootstrap，你可能会发现自己创建的类接近 Bootstrap 的类，但只是需要经常调整一下。顺风时(几乎)不会发生这种情况。例如，当您在 Tailwind 中指定一种颜色时，它会自动为该颜色生成一个完整的类列表。假设我们想要添加`#C7EA46` (Lime)作为一种颜色，我们简单地将它添加到 Tailwind 配置文件中的 colors 对象。像`bg-lime`和`text-lime`这样的类将会自动生成。这实际上不仅仅是颜色，因为你还可以指定变量、断点和几乎任何其他 CSS 属性。如果你真的想了解顺风的力量，我建议你去看看。

让我们回到我为什么要换。我想使用 Tailwind 做一些简单的东西，所以我开始用 Adobe XD 起草一个基本的仪表板设计，只是为了对我想要创建的东西有一个想法。我过去经常使用 Bootstrap 来创建这样的网站，所以比较一下 Tailwind 和 Bootstrap 的能力是个不错的主意。当最初的设计完成后，我建立了一个新的 Vue 项目并安装了 Tailwind。我不会对如何做这些做太多的细节，因为最初的 [Vue](https://vuejs.org/v2/guide/installation.html) 和 [Tailwind](https://tailwindcss.com/docs/installation/) 文档非常简单。我想指出的一点是，我运行了`npx tailwindcss init --full`命令来获得一个完整的配置文件。我更喜欢这个选项，因为作为一个没有任何使用 Tailwind 经验的人，这让我对这个框架的许多特性有了很好的了解，这使得学习曲线变得更容易一些。

当我开始写一些代码时，我注意到与 Bootstrap 相比，Tailwind 扩展了很多。同样，你不能真的比较他们，但仍然。有这么多新的类可用，您可能会认为很容易迷失在所有不同的名称中，但是因为类的命名非常一致(我使用的是具有自动完成功能的 IDE)，所以找到正确的类真的很容易。当然，有时我不得不在顺风文档中查找一些东西或通过谷歌进行搜索，但总体来说，我觉得我的工作流程比我使用 Bootstrap 时快得多。当我完成最初的设计时，我注意到我只写了 3 个(！)定制类来创建一个完整的仪表板主题，该主题也是可响应的，并且包括诸如过渡、动画等细节。使用 Bootstrap 通常会比使用 Tailwind 花费我更多的时间。

除了有一个更好的工作流程，它还能保持你的代码整洁易读。是的，有时你确实需要添加许多类到一个元素中，但是你可以立即知道每个类是干什么的，并且当你需要的时候你可以很容易地改变元素，而不是必须通过 5 个不同的文件来最终找到一个属性的位置。也许这有点夸张，但是请相信我，当我开始编码的时候，我有大量的 CSS 文件和内联样式标签，这使得我的代码无法阅读。

我不想让这篇文章写得太长，它并不是要教你关于顺风的所有细节，如果你为此而来，我很抱歉，但我只想分享我为什么转换以及它如何帮助我改进我的工作。希望对于那些还没有使用过 Tailwind 的人来说，你可以相信它，并尝试一下。就一次。这就是我从 Bootstrap 切换到 Bootstrap 所花的全部时间，我可能再也不会回去了。

[![](img/4fa6d9716c64c08c3c73e8e7e1b9146d.png)](https://skr.pt/morningscore-medium)

[想找一款一体化的 SEO 工具？点击此处查看晨星公司！](https://skr.pt/morningscore-medium)
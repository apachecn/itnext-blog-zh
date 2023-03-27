# compare-pdf:独立的 pdf 文件比较节点模块

> 原文：<https://itnext.io/compare-pdf-standalone-pdf-file-comparison-node-module-33222cace70f?source=collection_archive---------3----------------------->

![](img/f126dd5aca580f0091b71a1c710af43d.png)

# 让我们从“为什么”开始？

在我的上一篇[帖子](/chai-pdf-awesome-chai-plugin-for-testing-pdf-files-a0752f42e489)中，我展示了一个名为 [chai-pdf](https://www.npmjs.com/package/chai-pdf) 的 [chai](https://www.chaijs.com/) 插件，它可以让你进行 pdf 比较。

[](/chai-pdf-awesome-chai-plugin-for-testing-pdf-files-a0752f42e489) [## chai-pdf:测试 pdf 文件的强大 chai 插件

### 您曾经遇到过项目要求您验证 pdf 文件的情况吗？如果是这样，那么请继续阅读。

itnext.io](/chai-pdf-awesome-chai-plugin-for-testing-pdf-files-a0752f42e489) [](https://www.npmjs.com/package/chai-pdf) [## 柴-pdf

### 牛逼的 chai 插件测试 PDF 文件安装以下系统依赖项安装 npm 模块 npm 安装…

www.npmjs.com](https://www.npmjs.com/package/chai-pdf) 

虽然它确实工作得很好，但因为它是作为插件创建的，所以在添加新特性或功能方面有点限制。此外，它在整合到一个 [Cypress](https://www.cypress.io/) 项目中也有一些问题。当时我想，为什么我不创建一个独立的 pdf 比较模块来解决这些问题，所以我们在这里。介绍 [compare-pdf](https://www.npmjs.com/package/compare-pdf) 。

[](https://www.npmjs.com/package/compare-pdf) [## 比较-pdf

### 比较 pdf 的独立节点模块安装以下系统依赖项安装 npm 模块 npm 安装…

www.npmjs.com](https://www.npmjs.com/package/compare-pdf) 

与 [chai-pdf](https://www.npmjs.com/package/chai-pdf) 类似，默认情况下，该模块将通过将基线和实际 pdf 文件的每一页转换为图像(png)来比较 pdf，然后进行比较。

# 设置它

安装以下系统依赖项:

*   [代写本](https://www.ghostscript.com/)
*   [图形图像](http://www.graphicsmagick.org/README.html)
*   [ImageMagick](https://imagemagick.org/index.php)

一旦安装了这些依赖项，您现在可以将 compare-pdf 模块安装到您的项目中

```
npm install compare-pdf
```

以下是显示路径和图像比较设置的默认配置:

比较-pdf 默认配置

# 告诉我怎么做

在下面的要点中，你可以看到如何将 compare-pdf 集成到你的测试自动化框架中，在这个例子中使用了 [mocha](https://mochajs.org/) 。

使用 mocha 比较 pdf

# 更多功能，更多乐趣

以下是 [compare-pdf](https://www.npmjs.com/package/compare-pdf) 提供的特性和功能列表:

**屏蔽(addMask，addMasks)**

当您测试包含动态数据(如收据号码)的 pdf 文件时。Id 或日期，您可以通过在测试期间屏蔽这些值来避免假阳性。此功能将在页面索引(从 0 开始)上绘制一个黑色矩形，并在比较之前为基线和实际图像提供坐标。

比较-pdf 蒙版要点

使用*添加掩码*添加单个页面索引和坐标对象，使用*添加掩码*添加多个页面索引和坐标条目。

**裁剪(cropPage，cropPages)**

可能会出现这样的情况:您只在测试了 pdf 文件中的某个区域之后。或者也许你的策略是模块化你的 pdf 验证，并希望分离你的测试，并将其分解到 pdf 页面的不同区域。此功能允许您指定页面索引和坐标，以便在比较之前裁剪基线和实际图像。

比较-pdf 裁剪要点

您可以使用 *cropPage* 方法来裁剪单个页面，或者使用 *cropPages* 方法来添加指定页面索引和坐标的对象数组。

**比较特定页面(onlyPageIndexes)**

您在验证 pdf 文件时可能遇到的另一种情况是只需要测试特定的页面。为此，您可以使用方法 *onlyPageIndexes* 来添加特定的页面索引(从 0 开始)进行比较。

比较-pdf 特定页面要点

**跳过页面(skipPageIndexes)**

或者，您可能需要跳过验证 pdf 文件中的特定页面。在这种情况下，您可以使用 skipPageIndexes 方法。只需添加您想要跳过的特定页面索引。

比较-pdf 跳过页面要点

**按 Base64 比较**

正如我前面提到的，默认情况下，这个模块会将 pdf 文件转换成图像文件进行比较。这个过程确实需要时间，但好处是，最后，您将有一个差异文件，它将显示 pdf 页面的哪个特定区域失败了。

另一方面，通过 base64 编码比较 pdf 文件的速度非常快。然而，如果一个比较失败了，你就不会知道它失败的原因。

那么，为什么不两全其美呢？该模块提供了另一种比较模式:ByBase64。如以下要点所示，当 pdf 文件失败时，您可以结合 base64 比较的速度和图像比较的 diff 输出。

比较-pdf 比较模式要点

# 结论

总之，这个用于 pdf 比较的独立节点模块提供了大量的功能，甚至是一个利用 base64 比较模式加速测试执行的解决方案。

我希望这对你的项目有用。如果你有意见或建议，请写在下面的评论里。

一如既往，保持牛逼，下集再见！

**马尔克达茨**
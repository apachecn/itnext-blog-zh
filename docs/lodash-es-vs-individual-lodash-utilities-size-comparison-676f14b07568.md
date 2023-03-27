# Lodash-es 与单个 Lodash 公用事业:规模比较

> 原文：<https://itnext.io/lodash-es-vs-individual-lodash-utilities-size-comparison-676f14b07568?source=collection_archive---------2----------------------->

![](img/41a17c3c72b9e42b2f20aaad0cf77d2f.png)

在工作中，我们在前端应用程序中使用`lodash`。我们还在应用程序使用的共享模块中使用了`lodash`。有时我们的应用程序使用`lodash-es`，而一些模块使用单独的工具(`lodash.utilityName`，反之亦然。显然，代码复制并不理想，所以我们需要选择其中之一。哪一个会导致更小的束尺寸？

**结果**

尽管比较是在`lodash-es`和单个 lodash 工具之间进行的，我也测试了最初的 common-js `lodash`。这些包是使用下面的代码生成的。

```
import drop from 'lodash/drop';
import drop from 'lodash-es/drop';
import drop from 'lodash.drop';
```

该表显示了单个`lodash.utility`封装变小，直到封装数量增加。一旦我们达到 10 个实用程序的标准，`lodash-es`就会以最小的包大小领先。我将此归因于`lodash-es`能够在函数间共享代码，而单个`lodash.utility`函数是孤立的，不能共享代码。

**公用设施是如何选择的？**

这些包是以大致随机的顺序选择的，以避免从一个特定的类型中选择包，例如，只是数组或日期函数。相同类型的函数可能共享许多相同的实用程序，我不希望这成为测试中的一个因素。然而，有些实用程序比其他实用程序大得多，随着实用程序数量的增加，这可以从大小增加的不一致性中看出。

**捆绑包是如何生成的？**

我有一个非常简单的 webpack 配置来运行`webpack-bundle-analyzer`，如下所示:

```
// webpack.config.jsconst path = require('path');const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [
    new BundleAnalyzerPlugin(),
  ]
};
```

然后我的`src/index.js`简单地导入函数并引用它们，这样它们就不会发生树抖动。

```
// src/index.js
import drop from 'lodash-es/drop';
import omit from 'lodash-es/omit';
import fill from 'lodash-es/fill';
import flatten from 'lodash-es/flatten';
import head from 'lodash-es/head';drop
omit
fill
flatten
head
```

**收尾**

感谢您花时间阅读这篇文章，我希望您能从中获得一些价值。接下来，我将看看`Lodash`本身是否可以被`Just`([https://github.com/angus-c/just](https://github.com/angus-c/just))取代，后者是`Lodash`的轻量级替代品。
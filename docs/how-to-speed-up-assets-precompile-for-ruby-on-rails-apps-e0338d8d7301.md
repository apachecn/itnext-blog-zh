# 如何加速 Ruby on Rails 应用程序的资产预编译

> 原文：<https://itnext.io/how-to-speed-up-assets-precompile-for-ruby-on-rails-apps-e0338d8d7301?source=collection_archive---------1----------------------->

![](img/2fe9d85c26213b4e77174e6e75cb51f4.png)

您花费了太多时间来部署您的 rails 项目，尤其是在`assets:precompile`步骤，或者有时您可能会在资产预编译期间看到以下错误:

```
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
```

所以可以肯定的是，这篇短文将帮助你只做一些小的改变就有 50%的提高。

如果你在项目中使用`webpacker`，那么`assets:precompile`由两个阶段组成；我们将这两个阶段称为`sprockets`和`webpacker`。现在我们要分别执行这两个阶段，找出哪一个是你的问题。

运行`sprockets`阶段:

```
WEBPACKER_PRECOMPILE=false rake assets:precompile
```

然后运行`webpacker`构建阶段:

```
rake webpacker:compile
```

> **注意:**如果您想在生产模式下分析上述命令的性能，请将`RAILS_ENV=production`作为环境变量传递给它们。

如果`webpacker`是您的主要问题，请继续阅读，我们将调查并解决问题。有一种方法可以查看`webpacker`编译命令的细节，为了这个我们应该使用`webpack`本身而不是`webpacker`:

```
./bin/webpack --progress
```

> 注意:如果你想在生产模式下运行`webpack`命令，你需要将`NODE_ENV=production`作为一个环境变量传递给这个命令。例如`NODE_ENV=production ./bin/webpack`

应该有一个耗费时间/内存的步骤来生成源映射文件。

```
**% after chunk asset optimization SourceMapDevToolPlugin js/**-**.js generate SourceMap
```

源映射文件有助于将合并/缩小的文件映射回未构建状态。如果您不需要生产服务器上的源映射文件，您可以通过为`webpack/webpacker`设置`devtool`来跳过生成这些文件。我没有发现已经存在，实现了一个环境变量来设置此功能，但不要担心，我们将手动完成。

打开`environment.js`文件，在最后一行(`module.exports = environment;`)前增加以下两行:

```
...const devtool = process.env.DEVTOOL;
if (devtool) environment.config.merge({ devtool });...module.exports = environment; // this line already exists
```

然后测试实现的代码:

```
DEVTOOL=none ./bin/webpack --progress
```

您应该不会再看到源映射生成步骤。

现在您将能够将`DEVTOOL` ENV 传递给`assets:precompile`命令:

```
DEVTOOL=none rake assets:precompile
```

就是这样。你应该会有巨大的进步！
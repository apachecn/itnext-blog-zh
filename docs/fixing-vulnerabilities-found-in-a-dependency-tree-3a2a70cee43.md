# 修复在依赖关系树中发现的漏洞

> 原文：<https://itnext.io/fixing-vulnerabilities-found-in-a-dependency-tree-3a2a70cee43?source=collection_archive---------1----------------------->

![](img/fb33678ca99b1ab01cd8e22ec484e966.png)

我在一家生产金融云解决方案的公司工作。我们正在制作一个新的应用程序——一个在 *React Native* 开发的移动应用程序。该应用程序尚未公开发布，在准备发布期间，我们对 ***进行了全面的代码扫描，以找到任何漏洞*** 。我们发现我们的代码库中使用的一些依赖项依赖于旧的和潜在有害的包。

这就像当我们需要`packageA`时，但是那个库(`packageA`)依赖于易受攻击的库`packageB`。

我们需要采取适当的行动。我被指派去🪂执行任务

# 纱线溶液

我发现的事实是，在 *yarn package* manager 中强制一些深度嵌套的依赖关系是相对容易的:

Yarn 支持[选择性版本解析](https://classic.yarnpkg.com/lang/en/docs/selective-version-resolutions/)，它允许您通过 package.json 文件中的 resolutions 字段在依赖项中定义定制的包版本或范围。通常，这需要在 yarn.lock 文件中进行手动编辑。

## 怎么用？

我们需要将`resolutions`添加到我们的`package.json`中，指导 Yarn 如何处理依赖性:

```
{
  "name": "project",
  "version": "1.0.0",
  "dependencies": {
    "left-pad": "1.0.0",
    "c": "file:../c-1",
    "d2": "file:../d2-1"
  },
  "resolutions": {
    "d2/left-pad": "1.1.1",
    "c/**/left-pad": "^1.1.2"
  }
}
```

这将在本地存储的`c`和`d2`旁边安装`left-pad`的版本 *1.0.0* 。但是它会为`d2` 安装`left-pad`的 *1.1.1* 版本，为`c`下任何需要`left-pad`的依赖项安装 *^1.1.2* 版本。

# NPM 实施

如果您使用 npm 而不是 yarn，您可以使用`package.json`中的[覆盖](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#overrides)设置来实现类似的效果。尽管有一些不同。

我们不能使用全局匹配(双星号— `**`)来代替，我们必须嵌套依赖关系:

```
{
  "overrides": {
    "foo": {
      ".": "1.0.0",
      "bar": "1.0.0"
    }
  }
}
```

在那个例子中，我们覆盖了版本为 *1.0.0* 的`foo`包，并使`bar`在`foo`之外的任何深度也是 *1.0.0* 。

# 最后的话

有了这些不太为人所知的选项，我们可以强制任何深度嵌套包的更新版本(或简单的其他版本)。这是一种当我们的依赖包之一依赖于例如不再维护任何包含例如漏洞的包时修复问题的方法。

克劳迪奥·施瓦兹在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片
# 网络包，因果报应，测试和嘲讽。

> 原文：<https://itnext.io/adult-mocking-for-webpack-9b32eb0ca0d8?source=collection_archive---------0----------------------->

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fadult-mocking-for-webpack-9b32eb0ca0d8)

你如何嘲笑你的依赖？对于 nodejs，你知道，你拥有一切，但是如果你的测试运行在 Karma 中，你使用 Webpack 捆绑它们…那么会怎样？

> 所以，对于 nodejs 环境，您有大量的工具。难以想象的广阔: *Proxyquire，嘲弄，重布线，……*

但是有一天，你会意识到，所有这些牛逼的工具在浏览器里，在 Webpack 里，在你的测试里都不起作用。不是为你工作。

![](img/60e4312286d9304840b7176c21d74037.png)

永远不要迟到

# 怎么 moсk？实际上。

所有 nodejs 模拟工具实际上只使用两种方式来模拟 dep:

1.  重载低级 API(需要扩展处理程序)— Proxyquire
2.  重载高级 API(模块。_load) —嘲弄

所有的 webpack 模仿工具都只使用一种方法:

1.  猴子补丁源—注入加载程序

> 只是 cos“节点 js 环境”在浏览器中不存在。

结果——你会得到*稍微*不同的体验。换句话说——所有的模仿都会有点简单、直白和愚蠢。(仍可能满足您的需求)

因此，你将无法使用**隔离**嘲弄或者**召唤思维**代理询问。或者模仿嵌套依赖、自动生成依赖或者用一个模块替换另一个模块的能力。我知道世界上没有什么比全球性的嘲笑更无助、更不负责任、更堕落的了，我知道我们很快就会陷入那种糟糕的境地。

![](img/ed18710373ddf8560f379c5da2b6f40a.png)

Webpack 内部的 Proxyquire？！

# 如果你嘲笑，为什么不去嘲笑？

两个月前有人问我:“为什么[proxy quire-webpack-alias](https://github.com/theKashey/proxyquire-webpack-alias)在 web pack 中不起作用？”

我知道为什么。

> Webpack 你——这就是为什么！

问题是——“如何让它发挥作用”。

两个月后，问题解决了。 [Rewiremock](https://github.com/theKashey/rewiremock) v2.1 附带了 webpack 支持。老实说，很糟糕，但是

> 99.9%的测试都通过了。

并且所有外部 API 仍然相同。

# 例子

Rewiremock 可以直接替换 Proxyquire:

```
**const** mockedBaz = rewiremock.proxy('../baz'), {
  './foo'   : { func: () => 'aa' },
  '../b/baz': { value: 42 }
});
```

或者嘲弄:

```
rewiremock('./lib/a/foo').with(() => 'aa');
rewiremock('./lib/a/../b/bar').with(() => 'bb');rewiremock.enable();

**const** mockedBaz = require('../baz.js');

rewiremock.disable();
```

Rewiremock 有 4 种方法来创建一个 mock，它可以通过名字加载文件，使用 require，import，sync 或 async，甚至可以提供类型检查。

# **设置**

如何使用 rewiremock 没有区别，所有的事情都是一样的。但是对于 webpack，你必须先添加一些插件..

```
**1\. webpack.NamedModulesPlugin**, to enlight the real names of modules. Encipher "numbers" to names.
**2\. webpack.HotModuleReplacementPlugin**, to provide some information about connections between modules.
**3\. rewiremock.webpackPlugin**, to add some magic and make gears rolling.
```

只有三个插件，其中两个你可能已经用过了(React-Hot-Loader)，仅此而已。

**NamedModules** 将使您能够通过模块的真实名称来调用模块， **HotModulePlugin** 实际上只是将`parents`字段添加到`module`全局变量中，而 **rewiremock** 插件只是将钩子安装到源代码中。

不幸的是，webpack 模块系统…就是不存在。HotModuleReplacementPlugin 提供了`some` `replacement`，这就是为什么它是必需的。我希望有一天，一个(好的)人会在 webpack 中发布正常的模块系统，与 nodejs 变体进行比较。美好的一天…

# Webpack 插件

最难的部分是编写 5 行 webpack 插件。我花了几个小时阅读文档和源代码，但仍然

> 我不知道如何写一个插件。

我所需要的——只是在每个文件的顶部添加一行代码。使用加载器很容易做到，但目标是——拥有一个插件。

> Webpack 正在做它的工作，但是代码没有遵循最佳指南。没有逻辑，没有文档，严格的类型，没有例子。向所有支持者致敬，但是谁来记录生命周期渲染和内部逻辑呢？流量也一样。巴别也一样。

# **结论**

总之——rewiremock 来了。它可以在 nodejs 和 webpack 环境中工作，它可以模仿所有的东西(比你期望的更多)，提供隔离模式，并且足够可爱。它只是*所有其他解决方案中的好东西*。

[](https://github.com/theKashey/rewiremock) [## theKashey/rewiremock

### rewiremock——在 Node.js 或 webpack 环境中模拟依赖关系的正确方法。

github.com](https://github.com/theKashey/rewiremock) 

背后也有一些“逻辑”。关于“为什么应该使用 rewiremock 这样复杂的东西，而不是 inject loader 或者 proxyquire”。我的意思是——为什么 rewiremock 是最好的嘲讽工具，你应该使用它，而不是竞争对手。

[](https://medium.com/tag/rewiremock/latest) [## 关于 Rewiremock - Medium 的最新故事和新闻

### 阅读关于 Rewiremock 的最新文章。每天，成千上万的声音在阅读、写作和分享重要的故事…

medium.com](https://medium.com/tag/rewiremock/latest) 

# 所有的事情！

仅仅几年后，你就可以在 webpack 环境中使用一个`adult`模仿工具。

但是，相信我——你不应该。使用 mocha 或 just 在虚拟浏览器(不是 headless)中运行**单元**测试要好得多，更安全，更快。

保持真实的浏览器，和真实的运行环境(webpack)到另一个测试层。或者—直接用 DI。

不管怎样，这是可能的。现在。

![](img/1d3182c95023bd986af2518c3dbcfec0.png)
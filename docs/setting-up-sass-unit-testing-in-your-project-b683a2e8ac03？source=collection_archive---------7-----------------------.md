# 在项目中设置 Sass 单元测试

> 原文：<https://itnext.io/setting-up-sass-unit-testing-in-your-project-b683a2e8ac03?source=collection_archive---------7----------------------->

这个演练将使您准备好对您的项目进行 Sass 测试。

![](img/47d7efaf5fc32cb5544b46da5bb74433.png)

照片由 [Maik Jonietz](https://unsplash.com/@der_maik_?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

> **https://github.com/diego-codes/sass-testing-template:**结账演示项目:

# 为什么要单元测试 Sass？

Sass 是一个强大的 CSS 预处理器，它使您能够编写逻辑并在代码中重用样式来生成 CSS。如果您正在利用 Sass 函数和 mixin 特性，那么编写一些单元测试来确保您的代码按预期工作可能是一个很好的例子。尽管您的 Sass 编译没有错误，但是您的函数和 mixins 可能会产生意想不到的输出。此外，您可能不会马上注意到，因为 CSS 会无声无息地失败。

为了说明这一点，让我们来看一下将一个`px`值转换成一个`rem`值的函数:

**src/rem.scss**

```
@function px-to-rem ($px) {
  $base: 16px;
  @return #{$base / $px} * 1rem;
}
```

在当前实现中调用该函数不会引发任何 Sass 编译错误，但其结果可能不是您所期望的:

```
px-to-rem(10); // => 1.6px * 1rem (Unexpected outcome)
```

你得到的不是`1.6rem`，而是`1.6px * 1rem`。这就是测试驱动开发(TDD)和一般良好的单元测试派上用场的地方。

# 设置 Sass True

真正的是一个用 Sass 编写测试的开源工具。它依赖于一个 Javascript 测试框架，该框架提供了类似于`describe`和`it`的方法，如 Jest 或 Mocha。在最近的项目中，我选择 Jest 作为测试框架。所以我的例子将使用 Jest，但是你也可以使用你喜欢的框架。

# 安装依赖项

```
npm install --save-dev sass-true node-sass jest glob
```

# 正在为 JS 测试框架创建 shim 文件

如果您有特定的文件夹结构来对 Javascript 测试文件进行分组，或者在整个项目中有特定的文件命名约定，我建议您对 Sass 测试文件遵循相同的约定。这将减少所有参与者在您的项目中开始编写 Sass 测试的障碍。我喜欢使用`*.spec.js`文件扩展名，所以 shim 文件将被命名为`scss.spec.js`，它将搜索所有的`*.spec.scss`文件，并使用:

```
const path = require('path')
const sassTrue = require('sass-true')
const glob = require('glob') describe('Sass', () => {
  const sassTestFiles = glob.sync(
    path.resolve(process.cwd(), 'src/**/*.spec.scss')
  )

  sassTestFiles.forEach(file =>
    sassTrue.runSass({ file }, describe, it)
  )
})
```

# 写作测试

既然您已经创建了 shim 文件来运行 Jest 的所有真实测试，那么是时候编写一些测试了！回到我们之前的例子，你现在可以写一个测试`rem`函数。

**src/rem.spec.scss**

```
@import 'true';
@import 'rem';@include describe('rem()') {
  @include it('should return a rem unit given a unitless value') {
    @include assert-equal(px-to-rem(10), 1.6rem);
  }
}
```

正如你从本文开头的示例代码中所知，上面的测试将会失败。没关系。这意味着现在您可以开始调试该函数以获得预期的结果。

# 运行测试

只要您的 shim 文件包含在要运行测试的文件中，Sass 测试就会与 Javascript 测试一起运行。使用 Jest，您可以从您的终端运行它，并且应该包括 Sass 测试的输出:

```
jest
```

就是这样！不要忘记查看 [True](https://github.com/oddbird/true#usage) 的文档，了解更多测试你的 Sassy 代码的方法。非常感谢 [@mirisuzanne](https://twitter.com/mirisuzanne) 让 Sass 测试成为现实！
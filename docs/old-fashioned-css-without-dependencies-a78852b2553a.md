# 结构良好、干净、没有依赖的 CSS

> 原文：<https://itnext.io/old-fashioned-css-without-dependencies-a78852b2553a?source=collection_archive---------3----------------------->

![](img/e7f9cffbd72af4c9e5577b9d7c1710ed.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [OC Gonzalez](https://unsplash.com/@ocvisual?utm_source=medium&utm_medium=referral) 拍摄的照片

***免责声明:*** *以下文章反映了我的个人观点和经验，绝不是要诋毁某些库、工具或开发风格。*

美被认为是一种主观的东西——每个人都以不同的方式定义和看待它。每当我编写代码时，我不仅努力使我的应用程序在外表上看起来很漂亮，还努力让代码为它们提供动力。对我来说，拥有丑陋、臃肿或糟糕的文档化代码是绝对不行的。

然而，特别是在 HTML 和 CSS 中，我最近注意到了一个转变。自从像 React 这样的市长库流行起来，编写 CSS 和 HTML 的经典方式正在被像 [styled-components](https://www.styled-components.com/) 或 [emotion](https://emotion.sh/docs/introduction) 这样的工具所取代，通常被称为 CSS-in-JS。

# 有什么问题？

但是我越是使用这些技术，就越是怀念 HTML 只是 HTML，CSS 只是 CSS 的“美好旧时光”。在我目前的工作中，我的团队使用 React 和 styled-components，这为我们的整个前端提供了动力。这很有效，但是有些事情我不喜欢:

*   必须安装和交付依赖项，这增加了应用程序的复杂性和捆绑包的大小。还必须学习每个库的 API、语法和怪癖，尽管我只是想设计一些风格。
*   生成的输出经常是混乱的，有不可读的类名和许多不必要的覆盖。
*   市长版本更新可能需要一些工作，因为它们可能会引入突破性的 API 更改。
*   架构不一致。我工作的产品使用了材质 UI、风格化组件等等。换句话说，3 种不同的 CSS 编写方式，我不喜欢同时使用它们而不是一种。我还目睹了它们之间的特异性问题，因为 2 个 CSS-in-JS 引擎将它们的样式注入到头部，但是不知道哪一个覆盖了另一个。
*   用 JavaScript 写 CSS 感觉怪怪的，没有现成的合适的语法突出显示(是的，我知道有可用的[解决方案](https://github.com/styled-components/vscode-styled-components)——感觉还是不对)。
*   对结果的控制更少。例如，styled-components 自动添加供应商前缀，我不需要，因为我不想支持旧的浏览器。为什么我不能禁用它们？它们只是增加了文件大小，降低了 CSS 解析的速度。

我充分意识到，这些并不总是工具本身的问题(这些工具也在日复一日地改进，看看最新发布的 [Material UI](https://material-ui.com/blog/material-ui-v4-is-out/) 就知道了)，而是我们决定如何使用它们。尽管如此，我觉得我不需要这些东西来设计我的应用程序。拥有像有条件渲染 CSS 这样的特性很好，但对我来说没什么大不了的。

这就是为什么——在获得这些经验之后——我不再在我的个人项目中使用这些技术中的任何一种，相反，我更喜欢一种清晰结构化和分离的 CSS 的干净方法。让我们看一看，好吗？

# 命名事物

几年前，当我在寻找组织我的 CSS 的方法时，我遇到了一个叫做[的东西。这不是一个框架，也不是一个库，而是一个想法的集合，它将帮助你保持你的 CSS 整洁有序。](https://rscss.io)

> 一组简单的想法来指导你构建可维护的 CSS。— [https://rscss.io](https://rscss.io/index.html)

如果你想把它比作什么，可以看看 [BEM](http://getbem.com/) 、 [SMACSS](http://smacss.com/) 或者[原子 CSS](https://acss.io/) 。所有这些都是方法，而不是现成的框架，每一个都提供了自己的一套关于 CSS 代码库应该是什么样子的想法。

我以前使用过 BEM 方法，但是不喜欢长类名使标记变得臃肿的速度:

```
<a class="button button--state-danger button--state-disabled">
  <svg class="button__icon button__icon--size-big"></svg>
</a>
```

虽然我欣赏块、元素和修饰符的清晰结构和视觉分离，但我发现上面的代码很难看，对它所实现的来说太多了。

```
<custom-dropdown></custom-dropdown>
```

看看上面这个[定制元素](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)的例子。HTML 规范明确要求用破折号分隔单词；不是下划线，不是双破折号，也不是其他任何东西。理想情况下，我希望我的类和 id 遵循这个方案，尽管从技术上讲这并不重要。

让我们再看一下按钮的例子，现在使用 *rscss* 方法:

```
<a class="button -danger -disabled">
  <svg class="icon -big"></svg>
</a>
```

看到这个看起来有多清晰了吗？我基本上实现了同样的事情，但是现在在我看来干净了很多。我会说，更符合 HTML。

# 带组件的结构

当使用*rscs*时，一个好的做法是将你的应用程序拆分成组件。页面导航就是组件的一个例子:

```
<nav class="main-navigation">
  <a class="logo" href="/">Company XY</a> <a class="link -active" href="about.html">About Us</a>
  <a class="link" href="products.html">Products</a>
  <a class="link" href="contact.html">Contact Us</a>
</nav>
```

组件类应包含至少 2 个单词，用破折号分隔。组件中的元素类，如徽标或链接，应该只包含一个单词。尽可能避免嵌套 HTML，因为这会增加样式和语义的难度。

```
.main-navigation {
  > .logo {...}
  > .link {...}
  > .link.-active {...}
}
```

元素修饰符在名字前用破折号标注，如
`-active`。这种语法针对的是在 UNIX 系统上如何将参数传递到终端，这意味着元素的基本样式将被其他内容覆盖。修改器对于按钮或其他组件的不同变体很有用。

CSS 中另一个常见的东西是助手类。助手类应该只有一个用途，比如对齐项目或重置边距。*rscs*中的助手类以下划线开头，不包含破折号:`_nomargin`或`_aligncenter`。下划线使它们看起来不理想，这就是我们的想法:它们应该在你的代码库中有节制地使用，因为你的布局应该理想地由可重用和可配置的组件组成。

# 组织 CSS

CSS 仍然会变得混乱，尤其是当你有几百行代码的时候。这就是为什么最好不要把所有的 CSS 都放在一个大文件中，而是放在许多小文件中。

如果您以前使用过 SASS 或更少，您很可能熟悉[导入](https://sass-lang.com/documentation/at-rules/import)。它们可以让你轻松地将你的风格分成任意多的文件，在编译时将它们全部载入:

```
@import "buttons";
@import "inputs";
@import "titles";
```

我强烈建议这样做。在我的项目中，我总是试图实施一个描述性的文件夹结构，看起来像这样:

```
css/
  components/
    buttons.scss
    inputs.scss
    headings.scss
  pages/
    about.scss
    index.scss
  base/
    reset.scss
    typography.scss
    variables.scss
```

我的经验是，一个 CSS 或 SCSS 文件应该只负责一件事，例如，一个按钮，版式或页面的一部分。通过这种方式，我的文件保持很小并且易于维护，同时我总是确切地知道在哪里查找，以防我需要做更改。

外面有很多很棒的教程，给出关于如何构建 CSS、SCSS 或更少的建议和最佳实践。我的建议是选择一个能让你开心并且符合你的项目的架构。你可能不总是需要一个`themes`文件夹，所以跳过它。您可能不希望每个页面都有一个样式表，所以不要这样做。没有人强迫你完全复制这些想法，而是去适应和定制它们。

如果您想了解更多关于 SASS 萨斯最佳实践的信息，请点击。

# 不使用预处理器编写 CSS

虽然我非常喜欢 SCSS，但我发现在最近的项目中我不再需要它了，至少当我不需要支持旧的浏览器时。那时我坚持使用普通 CSS，它最近引入了像[变量](https://developer.mozilla.org/en-US/docs/Web/CSS/--*)(或者更好地称为自定义属性)这样的特性。甚至[套料也要来](https://drafts.csswg.org/css-nesting-1/)了！

但是我如何将所有的 CSS 文件编译成一大块，准备好作为主`styles.css`？我不知道。

相反，我把我的`link`标签直接放在我的 HTML 的`body`中的每个文件上，在我需要的地方和时间:

```
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="/css/global.css">
  </head>
  <body>
    <link rel="stylesheet" href="/css/components/header.css">
    <header class="main-header"></header> <link rel="stylesheet" href="/css/components/navigation.css">
    <nav class="main-navigation"></nav> <!-- and so on... -->
  </body>
</html>
```

这种技术在过去是无效的，因为 HTML 规范不允许在`body`中包含`link`元素(尽管这仍然是一个灰色区域)。然而，浏览器现在能够使用它，这给了我们一个机会。杰克·阿奇博尔德有一篇非常好的博客文章，深入探讨了细节并提供了一些见解。

这些是我个人认为的优点:

*   没有必要将单个 CSS 文件“编译”成一个大文件，因为我们可以在 HTML 中直接使用它们。
*   多亏了 HTTP 2，加载许多小文件比加载一个大的单个文件更高效。
*   我们只加载需要的样式。如果我在“关于”页面，我只包括来自`about.css`的 CSS，可能还有一些组件或全局样式。这改进了页面加载和呈现。
*   更清楚 CSS 属于 HTML 的某一部分。将`link`放在你的文件头上方很可能会告诉你它的样式就在这个文件中，而不是必须先浏览一个潜在的大文件夹。
*   感谢 CSS 自定义属性，我通常在`global.css`中定义我的变量，它被加载到`<head>`中的每个页面上。我现在可以从我的站点的任何地方访问它们，不管我当前在哪个文件夹或文件中。我甚至可以在 DevTools 中检查和修改它们，这在 SCSS 或更低版本中是不可能的。

# 分组 CSS 属性

使 CSS 更具可读性的另一个步骤是对属性进行分组，尤其是如果在一个选择器中有许多规则的话。MediaTemple 上有一篇关于分组的文章，展示了你可能喜欢的不同技术。看看这个选择器:

```
.button {
  display: flex;
  margin: 0;
  color: tomato;
  border: 2px solid tomato;
  background-color: white;
  font: inherit;
  transition: all .2s ease-out;
  padding: 1rem 2rem;
  border-radius: 3px;
}
```

上例中的 CSS 属性完全是按类型混在一起的，也就是说布局(padding，flexbox)没有按颜色(color，background)等等分开。

我个人喜欢按照 CSS 属性的类型对它们进行分组，所以上面示例按钮的 CSS 应该是这样的:

```
.button {
  display: flex;
  margin: 0;
  padding: 1rem 2rem; color: tomato;
  background-color: white; font: inherit; border: 2px solid tomato;
  border-radius: 3px; transition: all .2s ease-out;
}
```

您可以定义自己喜欢的分组类型，我通常选择以下 6 种:

*   布局(`flexbox`、`margin`、`padding`、`height`、`width`)
*   位置(`position`、`top`、`right`)
*   颜色(`color`、`background-color`、`blend-mode`)
*   字体(`font-family`、`font-weight`、`text-transform`)
*   边框(`border-width`，`border`)
*   其他(`transition`、`appearance`、`animation`、`opacity`)

# 将这一切结合在一起

让我们看一个真实世界的例子，使用上述技术。假设我们有一个包含页眉、导航、一些内容和页脚的页面:

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <title>My Awesome Site</title>

    <link rel="stylesheet" href="/css/base/global.css">
    <link rel="stylesheet" href="/css/base/typography.css">
    <link rel="stylesheet" href="/css/components/buttons.css">
  </head>

  <body> <link rel="stylesheet" href="/css/layout/main-header.css">
    <header class="main-header">
      <a class="logo -dark">My Awesome Site</a>
    </header> <link rel="stylesheet" href="/css/layout/main-navigation.css">
    <nav class="main-navigation">
      <a class="link -active" href="index.html">Home</a>
      <a class="link" href="about.html">About</a>
      <a class="link" href="contact.html">Contact</a>
    </nav> <link rel="stylesheet" href="/css/layout/main-content.css">
    <main class="main-content container">
      <h1 class="title">Welcome, enjoy this heading</h1>
      <p class="subtitle">What about this awesome subtitle? It seems to be a trend in recent web design, right?</p> <link rel="stylesheet" href="/css/components/gallery.css">
      <figure class="gallery-wrapper">
        <img class="image" src="/images/one.jpg" alt="One">
        <img class="image" src="/images/two.jpg" alt="Two">
        <img class="image" src="/images/three.jpg" alt="Three">
      </figure> <a class="button -small -primary" href="about.html">Learn more</a>
    </main> <link rel="stylesheet" href="/css/layout/main-footer.css">
    <footer class="main-footer">
      <p class="copyright">&copy; 2019 by Some Company Inc.</p>
    </footer> </body>
</html>
```

当然，这是一个相当简单的例子，但是你应该明白了。在`global.css`中，我定义了我的颜色、间距、字体等等:

```
:root {
  --color-red: #f00;
  --color-blue: #00f;
  --spacing-large: 1rem;
  --font-family: 'Open Sans';
}body {
  margin: 0;
}...
```

我的其他 CSS 文件是使用前面描述的 *rscss* 方法编写的，看起来可能是这样的:

```
.main-footer {
  background-color: gray; font-size: 90%;
}.main-footer > .copyright {
  text-align: center;
}
```

如果你确实需要一些嵌套或者花哨的混合，请随意使用 CSS 预处理器。它们仍然可以与这种方法一起使用，甚至可能提供更多的好处，例如[令人惊叹的颜色函数](https://sass-lang.com/documentation/functions/color)。

# 结论

我知道这可能是一篇有争议的文章，因为最终，每个人都是不同的，并且喜欢使用不同的工具、方法和哲学。我的目标不是让你喜欢我的可伸缩和漂亮的 CSS 方法，而是给你灵感。

请记住，底线是**而不是**说 CSS-in-JS 不好，或者 BEM 是一个愚蠢的想法；它们都有存在的理由。非常有才华的人把它们带到了生活中，一个大的社区每天都在用它们来建造令人惊奇的东西。我想说的是，就我个人而言，我并不像喜欢我自己的风格那样喜欢它们。

我使用这种方法的主要原因是，它利用了最新的 web 平台特性，不需要任何额外的库或工具，具有可扩展性和可维护性，并且可以生成漂亮的代码。

然而，最终重要的是我们，开发者，交付给我们的用户的最终产品，并且所述产品是可访问的、高性能的、安全的，并且总的来说，使用起来是愉快的。
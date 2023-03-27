# 编写模块化代码来清理您的 JavaScript 导入！

> 原文：<https://itnext.io/write-modularized-code-to-clean-up-your-javascript-imports-525dd3f7dc08?source=collection_archive---------6----------------------->

![](img/346624539a5b1b8341166664eea7da8f.png)

在这篇文章中，我将教你一个漂亮的小技巧，如果你还没有这样做的话，你绝对应该使用它来清理你的 JavaScript 导入。

我最近在 YouTube 上制作了一个关于 Sub Rendering React Components 的视频，这个视频是基于我在这里写的一篇博文。一些很酷的人评论了这个视频，因为他通过评论(暗示暗示，眨眼，轻推轻推)变得很酷。无论如何，大声喊出来那个自称“Max Equation”的酷哥。他留下了这条鼓舞人心的评论，激励我写下了这篇博文:

![](img/6a0748fc3f1911436e5c511a383f9c4d.png)

我将在其他时间讨论将 TypeScript 与 ReactJS 集成。但是在这篇文章中，我想谈谈模块化 JavaScript 函数和类的概念！

# JavaScript 中模块化是什么意思？

这是一种奇特但简单的技术，它利用了 ES6 附带的`import`和`export`功能。我在我的 React 工作中经常使用它，但是它可以在任何 JavaScript 框架中使用。

为了解释这个概念，我们首先需要浏览一下 ES6 导入和导出是如何工作的。让我们看一个简单的无状态函数式 React 组件的例子，它呈现了一个`Header`、`Content`和`Footer`组件。这些组件被定义在`components`文件夹中的三个独立文件中。`components`文件夹与我们的简单组件在同一个目录级别上，出于演示目的，我们称之为`MyComponent`。下面是我们对`MyComponent`的实现:

```
import React from 'react';
import Header from './components/header;
import Content from './components/content;
import Footer from './components/footer;const MyComponent = () => (
  <>
    <Header />
    <Content />
    <Footer />
  </>
);export default MyComponent;
```

我想把重点放在这个组件的`import`部分，所以我把它保持得非常简单。如果你不熟悉`<>...</>`语法，那么[看看我关于 React 片段的帖子](https://www.barrymichaeldoyle.com/fragment)。

在示例中，`Header`、`Content`和`Footer`组件具有非常相似的实现。让我们把重点放在`Header`组件上。这个`Header`组件或者是在`components`文件夹中名为`header.js`的文件中的默认导出函数。否则，它是默认的导出函数，保存在`components`文件夹中名为`header`的文件夹内的`index.js`文件中。好吧，这是一个口…你还在我身边吗？

基本上对`'./header'`的引用要么是一个`header.js`文件，要么是一个`header`文件夹中的`index.js`文件。如果你现在不明白，我不知道还能说什么。几天后我会制作一个 YouTube 视频来解释这个问题。你可以通过这个链接订阅它。

# 导出默认值与导出命名函数/类

还记得我说过这些导入的组件是导出它们的文件的默认导出函数吗？好吧，让我们看一下`header.js`文件来理解我的意思:

```
const Header = () => <div>Header</div>;export default Header;
```

我们`export default Header;`，所以当我们在我们的主组件中导入它时，我们像这样导入它:

```
import Header from './components/header';
```

您不必总是导出默认值。然而，一个普遍接受的最佳实践是每个文件有一个函数或类，并且默认导出那个函数或类。您也可以将`Header`函数导出为命名导出，如下所示:

```
const Header = () => <div>Header</div>;export Header; // See the 'default' keyword is removed
```

您也可以导出它，命名如下:

```
export const Header = () => <div>Header</div>;
```

这对我们在主组件中导入`Header`的实现意味着什么？因为它现在是一个命名的导出，所以您需要使用像这样的花括号来导入它:

```
import { Header } from './components/header';
```

但是正如我提到的，建议每个文件有一个类或函数，并且这个类或函数应该是默认导出的。

## 那么导出命名的函数和类有什么意义呢？

这是事情变酷的地方。假设我们很懒，我们决定不为我们的`Header`、`Content`和`Footer`组件、#screwbestpractices 创建单独的文件。如果我们像下面这个例子一样，把它们都导出到组件文件夹中的一个`index.js`文件中，会怎么样呢？

```
export const Header = () => <div>Header</div>;
export const Content = () => <div>Content</div>;
export const Footer = () => <div>Footer</div>;
```

这很酷，现在我们可以像这样将它们导入我们的主组件:

```
import { Header, Content, Footer } from './components';
```

出于学习的目的，如果我们做同样的事情，但是像这样导出缺省值`Header`:

```
const Header = () => <div>Header</div>;
export const Content = () => <div>Content</div>;
export const Footer = () => <div>Footer</div>;
export default Header;
```

然后我们将它导入到我们的主组件中，如下所示:

```
import Header, { Content, Footer } from './components';
```

## 更多进出口小技巧:

最后一件事。从技术上讲，您可以将默认导出的类命名为您想要的任何名称。例如，您可以像这样导入相同的内容:

```
import RandomOtherName, { Content, Footer } from './components';
```

或者，您可以将默认导出命名为已命名导入中的其他内容，如下所示:

```
import { default as Header, Content, Footer } from './components';
```

也就是说，从技术上讲，您不必命名默认导出。您可以像这样默认导出您的`Header`组件:

```
export const Content = () => <div>Content</div>;
export const Footer = () => <div>Footer</div>;
export default () => <div>Header</div>;
```

这样做仍然可以让我上面提到的那些导入技术工作。

对，我想这就是你需要知道的关于在 ES6 中导入和导出 JavaScript 模块的所有内容。但是当涉及到编写干净和可维护的代码时，这些实践中的大多数都不是很好。如果你看过我其他的帖子。你会知道我一直在编写干净、可读和可维护的代码。因此，让我们以讨论如何以一种干净有效的方式导入和导出我们的组件作为结束。

# 模块化代码正在运行！

正如我一遍又一遍说过的。为一个函数或类创建一个文件是普遍接受的最佳实践。因此，在我们最初的例子中，我们从不同的文件导入默认的导出函数，我们已经实现了最佳实践。它是这样实现的:

```
import Header from './components/header';
import Content from './components/content';
import Footer from './components/footer';
```

这是正确的，但是你是否注意到了另一个被我们忽略的普遍接受的最佳实践？如果你在想“干”这个词，那你就对了！“干”代表“不要重复自己”。你可以在这里看到一个共同的模式。我们从同一个文件夹导入多个函数，但是它们在不同的文件中。我们能在这做点什么吗？

![](img/629a3c00ebd59152794feabce6bffe75.png)

如果我们把所有的技术和我们在这里所拥有的放在一起，我们可以做一些神奇的事情！记得我们所有的功能都在`components`文件夹里吧？如果一切保持原样，并在我们的`components`文件夹中添加一个`index.js`文件，将我们所有的默认函数作为命名函数导出到一个地方，会怎么样？那是一个绝妙的主意！我们可以像这样轻松地做到:

```
export { default as Header } from './header';
export { default as Content } from './content';
export { default as Footer } from './footer';
```

然后，您可以将它们全部导入主组件，如下所示:

```
import { Header, Content, Footer } from './components';
```

这是一种以有组织的方式导入和导出文件的更干净的方式。使用这种技术引用类和函数要容易得多。

## 另一个可以使用这项技术的地方

我发现这个技巧对于将许多辅助函数分割成多个文件而不是将它们都保存在一个文件中非常方便。这使得在 IDE 中导航时更容易找到您的助手函数，也更容易确定单元测试助手的范围。

# 结论

我们已经讲了很多，我希望你已经学到了很多。如果你已经知道了这个技巧，那么我希望你可以用这篇文章来教导你的同事和朋友。

我写这类帖子的灵感来自于[Code Complete:A Practical Handbook of Software Construction，第二版](https://amzn.to/2LxtXDo)。这本书是包括我自己在内的许多开发者成长的催化剂。如果你没有一本，我强烈建议你帮自己一个忙，弄一本！

反正！如果你喜欢这个帖子，请留下一些赞和掌声。如果你想了解我的更多内容，请务必关注我的[脸书](https://www.facebook.com/barrymichaeldoyle)、[推特](https://twitter.com/barrymdoyle)、[媒体](https://medium.com/@barrymdoyle)、 [YouTube](https://www.youtube.com/barrymichaeldoyle?sub_confirmation=1) 和 [LinkedIn](https://www.linkedin.com/in/barry-michael-doyle-11369683/) 。

编码快乐！

*原载于 2018 年 9 月 10 日*[*www.barrymichaeldoyle.com*](https://www.barrymichaeldoyle.com/modularizing/)*。*
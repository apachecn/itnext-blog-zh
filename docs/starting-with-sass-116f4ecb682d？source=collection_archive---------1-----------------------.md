# 从 Sass 开始

> 原文：<https://itnext.io/starting-with-sass-116f4ecb682d?source=collection_archive---------1----------------------->

![](img/e90cedc1ec3f7e47fb3a6f9b2f926807.png)

你以前可能听说过 CSS 预处理程序，无论是 Sass、LESS 还是 Stylus——它们都是维护 CSS 的绝佳工具，尤其是在处理大型代码库时。对于外行来说:

> 一个 **CSS 预处理器**是一个让你从预处理器自己独特的[语法](https://developer.mozilla.org/en-US/docs/Glossary/syntax)中生成 [CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS) 的程序。有很多 CSS 预处理器可供选择，然而大多数 CSS 预处理器都会增加一些纯 CSS 中没有的特性，比如 mixin、嵌套选择器、继承选择器等等。这些特性使得 CSS 结构可读性更强，更易于维护。— MDN

一旦你掌握了 CSS，下一步自然是利用预处理器。最大的好处是不用重复自己。换句话说，就是让你的 [CSS 干](http://vanseodesign.com/css/dry-principles/)。

此外，它还提供:

*   干净的代码，变量和组件的重用。
*   易于维护和组织
*   实现逻辑和计算的能力，以及
*   节省了很多时间！

在本文中，我将重点讨论 Sass。它无疑是目前使用的最流行的预处理器。我将深入探讨什么是 Sass，如何将 Sass 编译成常规 CSS，并且我们将看看使它如此强大的一些特性。

🤓*想与 web dev 保持同步吗？*
🚀*想要将最新消息直接发送到您的收件箱？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 什么是萨斯？

SASS 是一种脚本语言，它为我们提供了常规 CSS 所没有的特性和工具。使用 Sass，我们可以编写更具可读性、可维护性和可重用性的代码。看待它的一个好方法是把它看作是一个为 CSS 增加力量和优雅的扩展。它为我们提供了各种功能，例如:

*   变量
*   嵌套
*   混合蛋白
*   功能
*   部分和进口
*   继承(扩展功能)
*   控制指令

换句话说，Sass 帮助我们以更易维护的方式组织大型样式表。我们将在本文的后面研究这些特性。

SCSS 还是萨斯？

在 Sass 中有两种书写方式——SCSS**和 Sass**和**Sass**——但是在被编译之后，它们会产生相似的输出。

*   SCSS(又名时髦 CSS)是现代标准。它的语法非常类似于 CSS，因为它使用括号和分号。在这种语法中，即使普通的 CSS 也是有效的。文件扩展名为`.scss`。
*   Sass 是一种较老的语法，它侧重于用缩进来分隔代码块，用换行符来分隔规则。它的文件扩展名是`.sass`。

在本文中，我将使用 SCSS，因为它是更自然的语法。当把普通的 CSS 转换成 SCSS 时，它也非常有用，因为你可以直接粘贴 CSS 并从那里开始工作！

# 安装 Sass

在我们能写 Sass 代码之前，它需要被安装在本地。现在，我们将通过这个过程来设置允许我们编写然后编译 Sass 的环境。

*注意:*当 Sass 被编译时，它被转换成浏览器可以解释和呈现的常规 CSS 代码。

## **环境设置:**

在我们开始之前，您必须在您的计算机上安装 npm，它与 [Node.js](https://nodejs.org/en/) 捆绑在一起，您可以从[这里](https://nodejs.org/en/download/)安装它。如果您还没有安装，请继续安装。如果你不确定你是否安装了 Node.js，从你的终端运行`node -v`。如果你看到一个版本号，它就被安装了！

***端子上的注释:***

如果您不熟悉 Sass，那么您可能也不熟悉从终端运行命令。这并不像看起来那么令人畏惧！一旦你获得了更多的经验，你就能真正节省时间。要在 Windows PC 上打开终端，请右键单击 Windows 图标并选择“Windows Powershell”，如果您在 Mac 上，请转到 Finder >应用程序>实用工具>终端。

## **文件夹结构:**

让我们创建我们的项目文件夹！它们的结构如下:

```
sass-project
   |- sass
   |- css
```

要创建这个结构，打开终端并切换到您希望安装我们的 sass 项目的文件夹(通过`[cd](https://www.git-tower.com/learn/git/ebook/en/command-line/appendix/command-line-101)` [命令](https://www.git-tower.com/learn/git/ebook/en/command-line/appendix/command-line-101))。然后运行以下命令:

```
mkdir sass-project
cd sass-project
mkdir -p sass css
```

## 文件结构:

当然，您需要一个 index.html 和 main.scss。要创建这些文件，请运行:

```
touch index.html
cd sass
touch main.scss
cd..
```

注意:别忘了给你的 index.html 加上`<link rel=”stylesheet” href=”css/style.css”>`。

## 初始化我们的项目目录:

所有使用 npm 的项目都需要初始化。为此，请输入以下命令。这将为我们的项目创建一个 **package.json** 文件。

```
npm init -y
```

## 安装节点-sass

*node-sass* 是允许我们编译的库。运行以下命令将 node-sass 安装为 dev 依赖项。

```
npm install node-sass --save-dev
```

## 将 Sass 代码编译成 CSS

接下来，我们需要创建一个 npm 脚本来运行编译。将这个脚本添加到我们之前创建的 **package.json** 文件的脚本部分。

```
"compile-sass": "node-sass sass/main.scss css/style.css"
```

我们在这里将`main.scss`指定为主 Sass 文件，将`style.css`指定为编译后的 CSS 文件。

在你的脚本中添加一面`--watch`旗帜非常方便。`watch`标志告诉编译器观察源文件的变化，并在每次保存 Sass 文件时自动重新编译成 CSS。将`--watch`添加到脚本并重新保存:

```
"compile-sass": "node-sass sass/main.scss css/style.css **--watch**"
```

现在每次保存时，Sass 都会自动编译成 CSS——太好了！

注意:确保终端窗口在后台运行，如果关闭终端，脚本将停止运行。如果需要退出进程，可以按 CTRL + C。

要将我们的 Sass 代码编译成 CSS，我们需要做的就是运行..

```
npm run compile-sass
```

**直播重装:**

为什么不在我们的项目中添加一个实时重载。为此，请运行以下命令进行全局安装:

```
npm install live-server -g
```

现在确保您仍然在`sass — project`文件夹中，并运行:

```
live-server
```

就像这样，您已经有了一个非常整洁的开发环境，您的项目在 HTTP 上本地运行。你需要让`live-server`和`npm run compile-sass`在两个独立的终端窗口中运行。

我们现在已经设置好了项目环境！我鼓励您在这种环境中进行试验，看看下面将要介绍的特性..

# Sass 功能

除了样式表之外，SASS 有效地为您提供了许多使用代码的好处。让我们直接进去看看吧！

# **变量**

变量是存储希望在整个样式表中重用的信息的一种方式。它们允许我们存储颜色、字体的值，或者任何你想重用的 CSS 值。我们使用`$`符号使某个东西成为变量。例如，在我们的 SCSS 中，我们可以定义一个颜色变量:

```
$color-primary: #ffff00; //yellowbody {
  background-color**:** $color-primary;
}
```

这当然会将我们的`background-color`设置为黄色。注意你可以在 Sass 中用`//`使用单行注释。当我们运行我们的编译时，它将输出下面的 CSS:

```
body {
  **color**: #ffff00;
}
```

这在处理大型项目时变得非常强大。例如，如果您希望更改样式表中使用的颜色。如果在一个位置定义为单个变量，那么修改起来就简单多了。另一种方法是分别查找和改变每个值:-/

# **嵌套**

当你观察一个 HTML 文件的结构时，你会注意到它有一个非常清晰的层次结构。另一方面，CSS 缺乏这种视觉结构。这就是为什么它会很快变得杂乱无章。进入萨斯嵌套！使用嵌套，我们可以在父选择器中嵌套子选择器。这使得代码更加清晰，重复性更低。

例如，以下面的 HTML 为例:

```
<nav class="navbar">
  <ul>
    <li>Home</li>
    <li>Store</li>
    <li>Contact Us</li>
  </ul>
</nav>
```

使用常规 CSS，我们可以这样写:

```
.navbar {
  background-color: orangered;
  padding: 1rem;
}.navbar ul {
  list-style: none;
}.navbar li {
  text-align: center;
  margin: 1rem;
}
```

这里有很多重复。每次我们想要设计一个`navbar`的孩子时，我们必须重复类名。使用 Sass，我们可以编写更简洁的代码。例如:

```
.navbar {
  background-color: orangered;
  padding: 1rem; ul {
    list-style: none;
  } li {
    text-align: center;
    margin: 1rem;
  }
}
```

注意这个凹痕。你会看到`ul`和`li`选择器整齐地嵌套在`navbar`选择器中。

# **Mixins**

Sass 的另一个强大特性是 mixins。使用 mixins，您可以将多个 CSS 声明组合在一起，以便在整个项目中重用。

假设我们想要创建一个 mixin 来保存转换属性的供应商前缀。在 Sass 中，我们这样编码:

```
@mixin transform {
  -webkit-transform: rotate(180deg);
  -ms-transform: rotate(180deg);
  transform: rotate(180deg);
}
```

为了将 mixin 添加到我们的代码中，我们使用了`@include`指令，例如:

```
.navbar {
  background-color: orangered;
  padding: 1rem; ul {
    list-style: none;
  } li {
    text-align: center;
    margin: 1rem;
    **@include transform;**
  }
}
```

transform mixin 中的所有代码现在都将应用于`li`元素。您还可以将值传递到 mixins 中，使之更加灵活。不添加指定的值，而是添加一个名称来表示该值，如下所示:

```
@mixin transform**($property)** {
  -webkit-transform: **$property**;
  -ms-transform: **$property**;
  transform: **$property**;
}
```

现在，无论何时调用 mixin，我们都可以传入我们喜欢的任何值:

```
@include transform **(rotate(20deg))**;
```

# **功能**

与 JavaScript 函数非常相似，Sass 函数可以接收参数并返回值。例如:

```
@function divide($a, $b) {
  @return $a / $b;
}div {
  padding: divide(80, 2) * 3px;
  height: 150px;
  width: 150px;
}
```

有时在 CSS 中做数学运算会很有用。标准的数学运算符`+`、`-`、`*`、`/`和`%`都可以使用。

# **偏旁&进口**

部分是模块化 CSS 的一个很好的方式，有助于保持东西的可维护性。我们将 Sass 分成代表不同组件的独立文件。分部名称总是以下划线`_`开头。

然后*使用`@import` 指令导入*分部。

假设我们的 Sass 文件变得相当大。我们可以制作一个只包含与 header 部分相关的代码的 partial，我们称之为`_header.scss`并将适当的代码移动到这个新文件中。然后我们会像这样将其导入回`main.css` :

```
// in main.scss
@import 'header';
```

*注意:*导入文件时，不需要包含文件扩展名`_`或`.scss`。

# **继承/扩展**

Sass 的另一大特点是继承性。我们可以*将 CSS 属性从一个选择器扩展到另一个选择器。为此，我们使用了`@extend`指令。请参见以下示例:*

```
.button {
  background-color: #0000FF; // Blue
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 1.5rem;
}
```

这是 CSS 按钮的标准代码。如果说，在我们的整个文档中有许多不同的按钮，所有这些按钮都以相似的方式设计，我们将有一个很好的继承案例..

```
.button**-secondary** {
  **@extend .button;**
  background-color: #4CAF50; // Green
} 
```

我们的`.button-secondary`类将接受设置为`.button`类的所有属性和值，除了我们决定设置为绿色的`background-color`。继承的使用确实有助于我们保持代码整洁、干净，并专注于构建可重用的组件。

# **' &'操作员**

嵌套时经常使用这个&符`&`操作符，这是一个非常有用的特性。尤其是在使用 [BEM](https://css-tricks.com/bem-101/) 方法编码时。我以前在这里写过关于 BEM 的文章。

请参见以下 HTML:

```
<button class="btn btn--red">Click me!</button>
```

典型的样式如下所示:

```
.btn {
  display: inline-block;
  padding: 15px 32px;
}.btn--red {
  background-color: #ff0000; // Red
}.btn:hover {
  background-color: #fff; // White
}
```

有了`&`操作符，我们可以更加高效:

```
.btn {
  display: inline-block;
  padding: 5px 8px; &--red {
    background-color: #ff0000; // Red
  } &:hover {
    background-color: #fff; // White
  }
}
```

注意，我们现在已经嵌套了使用相同`.btn`名称的子选择器。编译时,`&`操作符将被其包含的选择器名替换。

# **控制指令**

*控制指令*和表达式用于 Sass 中，仅在定义的条件下包含样式。作为一个特性，它们相当先进，主要在 mixins 中有用。常见的指令有`@if`、`@else`、`@for`和`@while.`

**@if 和@else**

`@if`和`@else`指令类似于 JavaScript 中的 if 和 else 语句。`@if`获取一个表达式并执行其块中包含的样式—如果求值结果不为假(或空)。例如:

```
@mixin heading($size) {
  @if $size == large {
    font-size: 4rem;
  }
  @else if $size == medium{
    font-size: 3rem;
  }
  @else if $size == small {
    font-size: 2rem;
  }
  @else {
    font-size: 1rem;
  }
}.h1 {
  @include heading(large);
}.h6 {
  @include heading(small);
}
```

这里，我们使用一个 mixin `heading`，它接受`$size`作为参数。根据我们传递给 mixin 的值，每个标题可以有不同的大小。

**@for 和@while**

您可以使用`@for`指令将一组语句执行指定的次数。它有两种变化。第一个使用`through`关键字，它执行从`<start>`到`<end>`的语句，例如:

```
@for $i from 1 through 5 {
   .list-#{$i} {
      width: 2px * $i;
   }
}
```

这将产生以下 CSS 输出:

```
.list-1 {
  width: 2px; 
}

.list-2 {
  width: 4px; 
}

.list-3 {
  width: 6px; 
}

.list-4 {
  width: 8px; 
}

.list-5 {
  width: 10px; 
}
```

如果我们用`to`替换`through`关键字，它会使循环具有排他性。当变量等于`<end>` so 时，它不会执行..

```
@for $i from 1 **to** 5 {
   .list-#{$i} {
      width: 2px * $i;
   }
}
```

会产生以下 CSS:

```
.list-1 {
  width: 2px; 
}

.list-2 {
  width: 4px; 
}

.list-3 {
  width: 6px; 
}

.list-4 {
  width: 8px; 
}
```

我们可以使用`@while`指令实现上面的代码。顾名思义，当条件返回 true 时，它将继续输出语句产生的 CSS。语法如下:

```
$i: 1;
@while $i < 6 {
  .list-#{$i} {       
     width: 2px * $i;   
  }
  $i: $i + 1;
}
```

如果您准备好提高您的前端技能并构建更快、更有组织和可维护的代码。查看[变得时髦:时髦实用指南](https://gum.co/getting-sassy)。

我的交互式初学者友好指南将带您了解 SASS 的所有重要信息，包括:

*   从变量、嵌套、混合到 for/each 循环等所有基础知识！
*   深入了解如何构建您的 SASS 项目
*   如何创建生产就绪的构建过程

最后，您将能够迁移现有的 CSS 代码库，并从头开始建立整个项目！

![](img/9a58b525327ed3868b4fcb14ae04647a.png)

*现已上市！👉*[gum.co/getting-sassy](https://gum.co/getting-sassy)

# 结论

这就对了。您已经学习了什么是 Sass，如何在本地服务器上安装和运行它，并且我们已经了解了许多特性，这些特性无疑使它成为您前端技能集的一个非常有用的补充。

使用 Sass，我们可以编写更干净的代码，重用代码并减少重复，更有效地组织项目，甚至将逻辑构建到样式表中。Sass 使 CSS 的维护变得更加容易，并且实际上为我们——作为开发人员——节省了大量时间！

请务必阅读本系列的下一篇文章:[构建您的 Sass](https://medium.com/@timothyrobards/structuring-your-sass-projects-c8d41fa55ed4) 。在这里，我们将研究如何最好地组织我们的目录和文件结构，因为我们要进行更大的项目。

今天就到这里吧！我希望这篇文章对你有用。你可以在媒体上关注我。我也上了我也上了[推特](https://twitter.com/easeoutco)。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用的时候会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉
# 让我们建造一些积木吧！

> 原文：<https://itnext.io/building-bits-and-pieces-375418f9b116?source=collection_archive---------6----------------------->

## web 组件和自定义元素介绍

![](img/27c7fa2101232ded9b376175f9db9a8b.png)

约翰·巴克利普在 Unsplash[拍摄的照片](https://unsplash.com/search/photos/diy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

Web 组件允许创建可重用的定制元素。您可能已经使用了前端库，如 React，它允许轻松地创建组件。使用可重用组件**减少重复**。它让代码更**干**(不要重复自己)，这是编程中常见的最佳做法。组件还**封装了它的样式和功能**，这使得 DOM 更加干净，并增加了关注点的分离。尽管可以通过 React 这样的库获得组件，但创建它们并不需要框架或库。大多数现代浏览器(除了 Internet Explorer 和 Edge)都支持所谓的自定义 web 组件。

在本文中，我们将逐步创建一个 web 组件，它包含信息框的样式、模板和功能。它将包含一个标题，一些文本和一个按钮，表明用户已经看到它。之后，我们的信息框应该可以在不同的情况下重用，比如服务通知和个人信息。

使用 web 组件时，您可以创建自定义的 HTML 元素。这些元素可以是自主的，也可以是定制的内置元素。**自治元素**是独立的，这意味着它们不扩展其他元素。**定制的内置元素**是通过扩展现有的 HTML 元素创建的。在本演练中，我们将使用自主定制元素。

这个例子的起点是一个包含两个文件的文件夹，index.html 和 main.js。然后，通过用浏览器打开文件来运行它。当您进行更改时，请刷新页面以查看它们的显示。

```
<!DOCTYPE html>
<html>
    <head></head>
    <body>
        <h1>Custom Element InfoBox</h1>
        <info-box />
        <script src='./main.js'></script>
    </body>
</html>
```

## web 组件的三个部分

Web 组件由三部分组成。

1.  自定义元素
2.  影子王国
3.  HTML 模板

**定制元素**是一个 JavaScript API，让你定义一个定制元素及其行为。这是通过调用`customElements.define("element-name", Object)`来完成的。以这种方式创建自定义元素后，我们可以使用它的名称作为标签将其添加到我们的 HTML 中。自定义 HTML 元素的要求是其名称必须包含连字符。有效的名称应该是主抽屉、章节按钮或信息容器。不允许使用像 chapterButton 或 info_container 这样的名称。在我们的例子中，定制元素被称为 info-box。

`<info-box />`

web 组件的下一部分是**影子 DOM** 。为了理解影子 DOM 是什么，您应该熟悉 DOM(或文档对象模型)的概念。简单地说，DOM 是一种树状结构的节点，包含了网页中的元素、样式和文本。影子 DOM 是一个 JavaScript API，它允许为定制元素创建一个单独的 DOM。这意味着自定义元素与页面的其余部分分开呈现。影子 DOM 通过**影子根**附加到原始 DOM，在原始 DOM 中有一个**影子宿主**。

阴影 DOM 是用一种模式定义的。

`let shadow = element.attachShadow({mode: 'open'})`

将模式设置为 open 意味着原始 DOM 中的 JavaScript 能够访问影子 DOM。这也会影响样式，因为打开阴影 DOM 模式允许将原始 DOM 的样式设置为其元素。

HTML 模板是一种编写可以在页面上重复的标记的方式。在定义定制元素时，`<template>`和`<slot>`元素非常有用。模板可以被复制和重用，例如现成的表格行。例如，在使用 JavaScript 将其内容附加到 DOM 之前，下面的 markdown 在页面上是不可见的。

```
<template id="large-title">
    <h1>A text within a template</h1>
</template>
```

下面的 JavaScript 将标题追加到`<body>`-元素。

```
let template = document.getElementById('large-title');
let templateContent = template.content;
document.body.appendChild(templateContent);
```

当我们编写 web 组件时，我们可以通过克隆它们并将它们附加到一个影子 DOM 来使用 HTML 模板的内容。如果下面的代码让你感到困惑，不要担心！我们将在下一章中浏览一个实际的例子。

```
class LargeTitle extends HTMLElement {
    constructor() { super(); // get the template element and its contents
      let template = document.getElementById('large-title');
      let templateContent = template.content; // create a shadowRoot with open mode
      const shadowRoot = this.attachShadow({mode: 'open'}) // append the cloned content to the shadow root
      shadowRoot.appendChild(templateContent.cloneNode(true));
  }
}customElements.define('large-title', LargeTitle)
```

## 编写信息框组件

创建自定义元素从创建 ES 2015 (Ecmascript 2015)类开始。我们创建一个扩展了`HTMLElement`的类，这样它就可以附加到 DOM 上。构造函数包含设置元素的逻辑。首先，我们需要创建一个 shadow root，稍后我们可以将元素附加到它上面。影子根是到原始 DOM 的连接点。

```
class InfoBox extends HTMLElement {
    constructor() {
        super() // create shadow root
        const shadowRoot = this.attachShadow({mode: 'open'})

        // our code goes here -->
    }
}
// define info-box as a custom element
customElements.define('info-box', InfoBox)
```

在我们创建了影子根之后，让我们继续定义我们的元素。在我们的信息框中，我们需要一个标题，一个文本和一个按钮。盒子本身是一个`<div>`元素。我们使用`document.createElement()`创建这些元素。

```
// create a div with the class 'container'
const container = document.createElement('div')
container.setAttribute('class', 'container')// create a title
const title = document.createElement('h2')
title.setAttribute('class', 'title')// create the info text 
const info = document.createElement('span')
info.setAttribute('class', 'infoText')// create the button and add the text "Ok!" to it
const button = document.createElement('button')
button.setAttribute('class', 'okButton')
button.textContent = "Ok!"// set the buttons onClick attribute to hide the component
function setHidden() {
    container.setAttribute('hidden', 'true') 
}button.onclick = setHidden;
```

定制组件的要点在于它们是可重用和可修改的。例如，我们希望能够在信息框中显示许多不同种类的文本。让我们将标题文本和信息文本设置为 HTML 中自定义元素的属性。

```
<info-box
    title="Library book order"
    info="You have some books waiting for you at the central
    library. Please pick them up within a week." />
```

现在我们可以使用这些属性来设置 box 的文本内容。我们使用`element.getAttribute()`-函数来检索`<info-box>`元素的属性。

```
// get the title and info attributes and put them inside 
// the info box title and textconst titleText = this.getAttribute('title')
title.textContent = titleText;const infoText = this.getAttribute('info')
info.textContent = infoText;
```

我们还想为自定义元素创建样式。我们可以在 JavaScript 中使用一个模板字符串来做到这一点。

```
// create styles
const style = document.createElement('style')style.textContent =
`.container {
    text-align: center;
    background-color: #fcfc9c;
    border-radius: 10px;
    width: 80%;
}
.title {
    font-size: 1.2em;
    margin: 5px;
}
.infoText { ... }
`
```

最后但同样重要的是，我们现在将已经定义的所有元素附加到影子根。

```
// append styles and container to the shadow root
shadowRoot.appendChild(style)
shadowRoot.appendChild(container)// append title, info and button to the container
container.appendChild(title)
container.appendChild(info)
container.appendChild(button)
```

## 后续步骤

关于 web 组件，我们还没有尝试的一个令人兴奋的事情是它的生命周期回调。您可以利用以下事件来修改组件的功能。

```
connectedCallback() {}
disconnectedCallback() {}
adoptedCallback() {}
attributeChangedCallback() {}
```

我们之前也提到过`<slot>`元素，但是我们没有在信息框组件中使用它们。插槽是在元素内部传递 DOM 元素的一种更方便的方式。也许你可以找到一种方法来重构信息框，这样按钮就可以通过一个槽来传递了。你可以在这里找到一些方便的提示[。](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots)

感谢阅读！

继续使用 CodePen 中的代码。👇

**资源:**

[MDN: Web 组件](https://developer.mozilla.org/en-US/docs/Web/Web_Components)

[谷歌开发者:网络组件](https://developers.google.com/web/fundamentals/web-components/)

[语法播客:影子 DOM](https://syntax.fm/show/143/hasty-treat-the-shadow-dom)
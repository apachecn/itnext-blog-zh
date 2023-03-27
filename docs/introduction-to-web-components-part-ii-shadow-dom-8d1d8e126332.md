# Web 组件介绍—第二部分阴影 Dom

> 原文：<https://itnext.io/introduction-to-web-components-part-ii-shadow-dom-8d1d8e126332?source=collection_archive---------3----------------------->

[Web 组件概述的第一部分](/introduction-to-web-components-part-i-custom-elements-4de6713cef9d)是关于自定义元素标准的。

# 阴影 DOM

规范给 Web 样式的定义带来了范围的概念。这些范围通过嵌入到单个文档中的多个`DOM`树来实现。在某种程度上，`Shadow DOM`的方法可以与文档中的`iframe`元素相比较，因为它有一个自然隔离的`DOM`结构。在文档方面，`Shadow DOM`的定义分布在几个不同的标准之间，如`DOM specification`、`HTML specification`、`CSS Scoping Module`、`UI Events`等。

![](img/fd2b00561dc97c81fc674f5bc48d41ab.png)

Web 组件

`DOM specification`定义了节点树的模型以及节点如何通过事件相互通信。`DOM`节点树是节点的层次结构，其中一些节点可以分支，而其他节点只是叶子。该规范通过与根的连接来区分`Document Tree`和`Shadow Tree`。`document`对象是`Document Tree`的根，而对于`Shadow Tree`，提出了一种特殊类型的根，称为`Shadow Root`。`Shadow Root`必须连接到`host`元素下的任何其他树。可以认为`Shadow Root`是一个`Document Fragment`——规范定义了两个接口之间的归属关系。

```
<script>
 class CustomElement extends HTMLElement {
   constructor() {
     super()
     this.attachShadow({ mode: 'open' })
   }
   connectedCallback() {
     this.shadowRoot.innerHTML = '<a href="#">My link</a>'
   }
 }
 customElements.define('hello-shadow-dom', CustomElement)
</script>
<hello-shadow-dom></hello-shadow-dom>
```

创建`Shadow DOM`的唯一方法是调用`Element`类的`attachShadow()`方法。它为元素创建一个影子根。只能为该元素附加一个影像根。在`attachShadow()` 调用中指定的`mode`属性定义了是否可以从`JavaScript`访问一棵树。“打开”值的替代值是“关闭”值。附加的`shadowRoot`属性是指定节点树的父节点。在下面的例子中，`shadowRoot.host`被设置为`hello-shadow-dom`元素。

```
<hello-shadow-dom></hello-shadow-dom>
 #shadow-root (open)
   <a href="#">My link</a>
```

`Slot`功能在`UI Libraries`中广泛使用。在`Angular,`中，它被称为投影或转换，并通过`ng-content`指令来支持。`Vue`似乎更符合标准——它调用与`slot`相同的功能。最后，在`React`中，这种能力与库更加密切相关，可以通过在元素容器中使用`{props.children}`来实现。`Slot`特征用作占位符。一方面，`Shadow Tree`可能包含用`slot`标签定义的`slots`元素。另一方面，当开发人员想要使用元素时，他或她可以为声明的`slots`指定一个标记，这样内容将在`Shadow Tree`包装器中呈现。

```
<script>
 class OhMySlotElement extends HTMLElement {
   constructor() {
     super()
     this.attachShadow({ mode: 'open' })
   }
   connectedCallback() {
     this.shadowRoot.innerHTML = `<p>
       <slot name="my-text">default</slot>
     </p>`
   }
 }
 customElements.define('oh-my-slot', OhMySlotElement);
</script>
<oh-my-slot>
 <pre slot="my-text">
   anything
 </pre>
</oh-my-slot>
```

最后的`DOM`树包含一个带有`p`子元素的`Shadow Root`和开发者的`pre`元素。后者链接到`slot`节点并替换它的“默认”内容。标签上的 name 属性定义了使用哪个占位符。如果没有可用的替代项，也可以使用未命名的插槽。

规范允许开发者声明独立的样式，并提供一个 API 来使用主文档上下文。外部文档`CSS`选择器无法访问`Shadow Tree`内部的元素。

在下面的例子中，有两个样式的部分——一个在主文档中，另一个在元素的附件`Shadow DOM`中。

```
<style>h3 { color: blue; }</style>
<script>
 class CustomElement extends HTMLElement {
   constructor() {
     super()
     this.attachShadow({ mode: 'open' })
   }
   connectedCallback() {
     this.shadowRoot.innerHTML = `
       <style>h3 { color: red; }</style>
       <h3>Shadow DOM</h3>
     `
   }
 }
 customElements.define('my-element', CustomElement);
</script><h3>Normal DOM</h3>
<my-element></my-element>
```

应用于主文档的`h3`元素的样式为蓝色，而`my-element`内的`h3`被涂成红色。值得注意的是，主文档可以为`host`元素本身定义样式。当`my-element`样式被设置为绿色时，它将被应用到它的子元素上，就像普通的样式继承一样。继承的颜色仍然比任何其他定义的样式优先级低。

`:host`伪类选择器正在查看`Shadow Root`的`host`元素。

```
:host { color: green; }
```

与元素内部的`:host`样式相比，主文档中的样式具有更高的优先级。条件选择器可以使用`:host()`功能

```
:host(.error) { color: red; }
```

当`host`附加了`error`类时，红色将应用于它。

另一个功能选择器是`:host-context()`，它可以检测在`Shadow Tree`之外是否有祖先匹配指定的选择器。

类似地，`::slotted()`函数可以应用于`slots`内部的“外部”元素。

仍然可以在主文档中自定义样式。这就是应用于`Shadow DOM`的`CSS custom properties`或变量特性。它基于几个原则:

*   属性在`Shadow DOM`中定义和使用。所以组件的作者负责提供一个 API(或者一个可以从外部覆盖的属性列表)。
*   变量应用于主文档中的`host`元素。正如规范所说，自定义属性确实继承了。这意味着如果一个元素没有在子元素的级别中定义，它将采用父元素的值。

当自定义元素使用`Shadow DOM`时，其样式定义为:

```
h3 { color: var(--my-color); }
```

它将尝试访问`--my-color`变量。因此，使用该元素的开发人员可以在他或她的样式定义中附加一个变量。

```
custom-element { --my-color: red; }
```

然而，`--my-color`变量没有在任何样式定义的`h3`选择器中定义，它是从`Shadow Root`的`host`中继承的，并被应用于内部元素的`DOM`。`h3`颜色的值是红色。

对那些公共定义的变量使用回退或默认值是合理的。如果客户没有指定变量，它仍然有一个合理的值。

总而言之，`Shadow DOM`提供的功能有:

*   孤立的`DOM`，从文档中分离出来。
*   对树进行私有或公共访问的元素的组合。
*   `slots`以重复使用为目的。
*   `CSS`是有作用域的，所以页面的样式不会与`Shadow Tree`中声明的样式混合在一起。这种方法简化了样式的定义，减少了对层次选择器的需求。

# HTML 模板

`HTML Template`规格通常与`Shadow DOM`和`Custom Elements`组合使用。它解释了一种在不影响文档本身的情况下指定布局的方法。`template`标签中的任何内容都不会在页面加载时呈现，稍后可以被`JavaScript`代码用来实例化元素。

```
<template id="mytemplate">
 <script>
   alert()
 </script>
</template>
```

在给定的例子中，浏览器现在将显示弹出窗口；但是，它是在`script`标签中声明的。

```
const template = document.querySelector('#mytemplate')
const clone = document.importNode(template.content, true)document.body.appendChild(clone)
```

仅当模板被实例化并添加到文档中时，弹出窗口才会出现。`content`是另一个`Document Fragment`，它包含指定的`HTML`结构。文档的方法生成给定节点的副本。它采用的最后一个参数是“deep”标志。默认情况下，它是`false`，唯一的高级元素将被复制，因此应使用`true`来复制整个布局。

创建包含`style`定义的`template`元素是有意义的。

```
<template id="mytemplate">
 <style>
   :host { color: red }
 </style>
</template>
```

该定义符合自然的`HTML`语法，不会在页面加载时呈现，可以在以后开发人员需要追加模板实例时使用。

```
const template = document.querySelector('#mytemplate')
const div = document.createElement('div')
const root = div.attachShadow({ mode: 'open' })
const clone = document.importNode(template.content, true)
root.appendChild(clone)document.body.appendChild(div)
```

# 摘要

`Custom Elements`、`Shadow DOM`和`HTML Template`规范允许开发人员创建具有封装的`DOM`、风格和行为的可重用组件。大多数已声明的 API 已经可以在主流浏览器中使用，并且有多种填充来模仿其余的 API。

有一个不错的编码，并希望看到你的评论！
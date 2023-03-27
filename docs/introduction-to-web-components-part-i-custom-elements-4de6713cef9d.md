# Web 组件简介—第一部分自定义元素

> 原文：<https://itnext.io/introduction-to-web-components-part-i-custom-elements-4de6713cef9d?source=collection_archive---------5----------------------->

`HTML`规范描述了许多不同的标签，这些标签可用于构建网页或应用程序。这些标签定义了具有功能或用户界面意图的元素。`head`和`body`标签都是传统的`HTML`布局。借助于`link`、`script`、`title`和`meta`标签等，在`head`部分中声明了不同的元定义和依赖关系。另一方面，`body`元素用如下元素定义了文档的结构:

*   `div`和`span`有时分别作为`block`或`inline`显示元素的基础
*   `p`描述包含任何文本的段落
*   `h1`到`h6`标签定义了文档中不同的标题级别
*   `button`可作为用户动作，即在`form`内部或用于`navigation`目的
*   标签有助于定位输入用户数据的字段
*   `select`和`input`元素用于`form`字段的描述。为这些标签添加一个`attribute`可以定义验证规则、大小或可能的输入数据类型，如数字、日期或密码。自从`HTML5`标准引入以来，开发人员可以使用更具体的标签，比如:
*   `audio`和`video`元素定位并提供多媒体内容
*   `section`、`article`、`nav`、`header`、`footer`等很多标签勾勒出了结构，为文档增加了更多的语义。这个列表并不完整，它只是显示了`FrontEnd`开发者构建网页的不同可能性。

> 如果一个项目需要一个行为不同的元素呢？

例如，它可以是一个`tab`元素来演示关于所选项目的不同内容，或者是一个`expand`元素来显示文本的额外内容，等等。像`Vue`、`React`和`Angular`这样的现代库和框架围绕`DOM`引入了许多抽象，以简化定制元素的概念和用法。它们都将应用程序分解为组件。它们定义了如何创建组件、如何将组件插入到文档中、如何相互交互以及如何呈现组件的规则。事实上，`Vue`和`React`库内部都使用了一个`Virtual DOM` 算法，它创建了一个与真正的`HTML`结构并行的结构。每当框架要更新视图时，首先`Virtual DOM`实现会比较需要更新的内容，然后再对真正的`DOM`应用任何更改。像`jQuery`这样的老库通过将`JavaScript`逻辑应用于`HTML`结构来解决这种情况。通常，元素被用作`jQuery`插件的入口点，插件包装它，添加额外的元素，绑定事件监听器，并附加一个业务逻辑。同时，文档的结构并不总是代表开发者的意图——神奇的事情发生在`JavaScript`库内部。然而，这种方法适用于较小的网站，页面越大，维护和扩展就越困难。

> 现在有一种本地的方式来实现元素的定制— `Web Components`

`Web Components` 规范是 Web 标准的通用名称，其目的是将基于浏览器的应用程序表示为可重用元素的层次结构。这个想法是让开发人员能够定义他们自己的组件，他们可以封装状态、行为、风格等。组件可以相互通信，对于浏览器环境来说是“自然”的。可以通过`HTML`声明将元素添加到页面中，这样文档的结构可以更一致地表示。

![](img/fd2b00561dc97c81fc674f5bc48d41ab.png)

Web 组件

# 自定义元素

`Custom Elements`规范允许开发者定义他们自己的元素，几乎与内置元素没有区别。事实上，这个过程甚至可以反过来，最终原生元素应该是在浏览器引擎中定义的`Custom Elements`，就像`FrontEnd`开发人员在应用程序中所做的那样。

```
<select>
  <option value="1">1</option>
  <option value="2">2</option>
</select><!-- What if we want multiple select? --><multiple-select>
  <option value="1">1</option>
  <option value="2">2</option>
</multiple-select>
```

浏览器中没有`multiple-select`元素，但是有一个会很有用。所以`Custom Element`可以在页面内的`JavaScript`代码中定义。

```
<script>
class HelloWorldElement extends HTMLElement {
  connectedCallback() {
    this.textContent = "Hello World"
  }
}
customElements.define('hello-world-element', HelloWorldElement);
</script><hello-world-element></hello-world-element>
```

代码中有一个新的`HelloWorldElement`类定义。它继承自基类`HTMLElement`，并在初始化后添加文本。`connectedCallback`是一个生命周期钩子，当一个元素被添加到文档中时执行。它是生命周期系统的一部分，这将在后面描述。在定义了类本身之后，重要的事情发生了- `customElements.define()`被标签名和`HelloWorldElement`类一起调用。`HelloWorldElement`注册在`customElements`中——页面上所有元素的注册表。`customElements`是`CustomElementRegistry`类的实例，为每个`HTML`文档公开。这是每个页面的唯一对象，或者更准确地说，是每个页面的`window`实例的唯一对象。规范还将它与每个`window`名称空间的`HTMLElement`类特异性相关联。定义完成后，可以用任何可能的方式创建`HelloWorldElement` 元素。

要么使用`JavaScript`文档`createElement()`方法，要么调用最近创建的类的构造函数，然后将一个元素附加到`DOM`结构。

```
// use createElement
const helloWorld = document.createElement('hello-world-element')
document.body.appendChild(helloWorld)// use new
const newHelloWorld = new HelloWorldElement()
document.body.appendChild(newHelloWorld)
```

也可以在`HTML` markdown 中声明它。

```
<hello-world-element></hello-world-element>
```

以任何一种选择的方式，元素都会出现在页面上，并根据它的定义来表现。

# 提升

在最新的例子中，`HTML`和`JavaScript`代码是分开的，因此元素的声明可以发生在脚本评估之前、之后甚至期间。非自定义元素可以随时成为自定义元素。这包括在规范中，这个过程被称为`upgrading`程序。当元素被添加到`DOM`时，`upgrading`过程被正常触发。然而，即使元素不在文档结构中，也可以通过`customElements.upgrade()`方法强制完成。

# 属性示例

可以描述符合`HTML`语法的属性`setters / getters`行为。下面的代码演示了这个过程。当执行添加的`attributeChangedCallback`和`connectedCallback`方法时，它们检查当前元素的属性值并同步内部和外部状态——它的属性和特性。

```
class HelloWorldElement extends HTMLElement {
  static get observedAttributes() {
    return ['name']
  }
  attributeChangedCallback(name, oldValue, newValue) {
    this._name = newValue
  }
  connectedCallback() {
    this.name = this.getAttribute('name') || 'World'        
  }
  get name() {
    return this._name
  }
  set name(name) {
    this.setAttribute('name', name)
    this.render()
  }
  render() {
    this.textContent = `Hello ${this.name}`
  }
}
customElements.define('hello-world-element', HelloWorldElement)
```

注意，应该为订阅属性改变事件定义静态方法`observedAttributes()`。它应该返回要监听的名称数组。当元素被声明为

```
<hello-world-element name="Alex"></hello-world-element>
```

文本`Hello Alex`将被打印。以相反的方式

```
<hello-world-element></hello-world-element>
```

将输出`Hello World`作为默认值。

# 扩展现有元素

很难模仿现有的浏览器元素。例如，`HTML` `button`有一组行为，对于新的定制元素是不可用的:

*   当`tab`键盘事件发生时为`focused`。这可以通过向新元素添加一个`tabindex`属性来实现。
*   `button`可由`mouse`和`keyboard`激活或“推动”。在一个表单里面，`button`可以提交数据。需要额外的事件侦听器来添加此功能。
*   它有一个内部`disabled`状态，可以通过`JavaScript`代码实现。
*   它支持与可访问性相关的特性，如`aria-label, role`等。而且只是一个`button`。有了互动元素，比如`input`、`form`或`img`，就更难了。人们很容易忘记这些内置功能，并因为不希望的和意想不到的行为而使用户不高兴。`Custom Elements`规范提出了一种减少内置元素和新元素之间差异的方法。

到目前为止，所有的例子都展示了新类型元素的创建，它们是从基类`HTMLElement`继承的。这些类型的元素被称为“自主定制元素”。

重用不止一个基类的方法是在`customElements`中注册新元素的类型时指定`extends`标志。任何使用`extends`选项的元素都被命名为“定制内置元素”。

任何内置元素都可以通过扩展其`JavaScript`类和使用特殊的定义参数来定制。在下面的例子中，原始的`button`元素被一个新的`MyButton`类继承。

```
class MyButton extends HTMLButtonElement {
  constructor() {
    super()
    // ...
  }
}customElements.define('my-button', MyButton, { extends: 'button' })
document.createElement('button', { is: 'my-button' })
```

当然也可以在`HTML`中声明一个元素。

```
<button is="my-button">Click Me!</button>
```

注意，`is`属性的存在对于定制的内置元素的创建是必要的。

# 生命周期

在`FrontEnd`框架和库的上下文中讨论组件时，`lifecycle`是一个常规术语。它描述了如何创建组件、如何修改组件以及在此过程中会发生什么。

将事件绑定到组件生命周期的某个阶段是很常见的。这些事件被附加到钩子或反应——方法上，每当这样的事件发生时就会被调用。

当事件发生时，如果在自定义元素的类上设置了相应的方法，则会调用该方法。为任何`Custom Element`定义的反应有:

*   `upgrade`反应，包括元素的`constructor`调用。
*   当元素被添加到文档中时，元素的`connectedCallback`被执行，换句话说就是`connected`。
*   `disconnectedCallback`与`connected`事件的反应相反，当元素`disconnects`或从`DOM`中移除时发生。
*   当元素改变它的父文档时，就会发生`adopted`事件。
*   最后，每当元素的属性改变时，包括添加和删除，都会调用`attributeChangedCallback`。当调用它时，回调按以下顺序接收属性名、旧值和新值作为参数。

```
class CustomElement extends HTMLElement {
  constructor() {
    super()
    console.log('constructor')
  }
  connectedCallback() {
    this.textContent = "Hello Custom Element"
  }
  disconnectedCallback() {
    console.log('disconnectedCallback')
  }
  adoptedCallback() {
    console.log('adoptedCallback')
  }
  attributeChangedCallback() {
    console.log('attributeChangedCallback')
  }
  static get observedAttributes() {
    return ['test']
  }
}
customElements.define('hello-custom-element', CustomElement)<hello-custom-element test="test"></hello-custom-element>
```

注意，`constructor()`的主体以`super()`调用开始。它执行一个父类`constructor()`以便初始化可以继续。`constructor()`的主体不应该修改或使用元素或其属性。它的目的是定义初始元素的状态，并附加所有需要的事件侦听器。所有额外的工作应该被推迟到`connectedCallback`。

# 摘要

在这个 Web 组件系列的第一部分中，我们介绍了定制元素规范的基本概念。我们已经概述了如何定义、使用、升级和使用新元素的生命周期挂钩。[下一篇是关于**阴影 DOM** 和 **HTML 模板**用法](https://medium.com/@korzio/introduction-to-web-components-part-ii-shadow-dom-8d1d8e126332)。

有一个不错的编码，并希望看到你的评论！
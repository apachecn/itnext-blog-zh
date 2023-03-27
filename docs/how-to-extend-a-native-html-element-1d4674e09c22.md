# 如何扩展原生 HTML 元素

> 原文：<https://itnext.io/how-to-extend-a-native-html-element-1d4674e09c22?source=collection_archive---------1----------------------->

当标准 HTML 不够用时

![](img/103c7c29d9be5afc11026392553ac72b.png)

瑞安·昆塔尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

从 XML 时代开始，我们就试图用我们自己的标签来扩展 HTML。

HTML 标签的标准库是相当有限的，并且有意由低级构建块组成，这意味着由开发人员组成更高级的功能。

现在所有的现代浏览器都支持 [Web 组件](https://blog.usejournal.com/web-components-will-replace-your-frontend-framework-3b17a580831c)(或者更具体地说[自定义元素](https://caniuse.com/#feat=custom-elementsv1))你可以创建你自己的 HTML 元素，你可以在任何地方使用它们，只需加载一个脚本，然后将标签添加到文档中。

真的就这么简单。

如果您已经创建了自己的图像库，您只需加载脚本并将`<image-gallery></image-gallery>`添加到文档中即可使用它:

```
class ImageGallery extends HTMLElement {
  constructor() {
    super();
  } ...}customElements.define('image-gallery', ImageGallery);<image-gallery></image-gallery>  // presto!
```

这里的`ImageGallery`类包含了`<image-gallery>` HTML 元素的所有功能，我们通过`customElements.define`用`'image-gallery'`标签名注册它。

现在像 React、Angular 和 Vue.js 这样的框架也允许你创建自己的 HTML 标签，但是与框架组件相反，自定义元素是*真正的*一级 HTML 元素。

在这种情况下，`ImageGallery`类扩展了`HTMLElement`，这是所有 HTML 元素的基本接口。这意味着它将继承所有 HTML 元素共有的功能。

例如，您可以通过`addEventListener`将事件侦听器附加到它，使用 CSS 通过它的`style`属性对它进行样式化，或者像任何其他 HTML 元素一样在浏览器 devtools 中与它进行交互。

而且不止于此。

# 站在巨人的肩膀上

除了扩展`HTMLElement`，定制元素还可以扩展其他内置的 HTML 元素，例如`<button>`、`<img>`和`<a>`。

假设我们想要创建一个延迟加载的图像，直到它被滚动到视窗中才会被加载。我们可以通过搜索页面中的所有图像并给每个图像附加一个`IntersectionObserver`来确保图像只有在可见时才会被加载。

但是我们也可以*扩展内置的图像元素本身*并使用增强的图像元素代替常规的`<img>` HTML 元素。

我们可以通过创建一个定制元素来实现这一点，这个定制元素不扩展`HTMLElement`，而是扩展`<img>`元素的接口，也就是`HTMLImageElement`:

```
class LazyImg extends HTMLImageElement {
  constructor() {
    super();
  }...}customElements.define('lazy-img', LazyImg, {extends: 'img'});
```

定制元素是通过对`customElement.define`的调用注册的，但是现在它需要第三个参数`{extends: 'img'}`，指定哪个 HTML 元素将被扩展。

现在，我们不再使用新的 HTML 标签，而是使用带有常规`<img>`标签的增强型图像元素，但是我们通过`is`属性为其添加了新的功能:

```
<img **is="lazy-img"** src="/path/to/image.png">
```

这个图像现在是一个增强的图像，获得了我们在`LazyImg`类中定义的所有功能。

*`*LazyImg*`*的完整实现对于本文来说太大了，但是您可以在我的 Github* *上找到源代码。**

*这种方法的美妙之处在于，任何不支持扩展内置 HTML 元素的浏览器都会简单地忽略`is`属性，只呈现常规图像。*

*渐进增强的最佳表现。*

# *示例:客户端路由*

*这样，我们也可以轻松地增强普通链接，使之成为与客户端路由器一起工作的链接。*

*通常，我们需要遍历所有这些链接，并编写一些代码来防止我们在单击链接时导航到另一个页面，因为我们希望在客户端处理路由。*

*通过扩展原生的`<a>`标签，我们可以简单地添加一个`is`属性来表明它是一个客户端链接，所以它不会使浏览器在被点击时转到其`href`属性中指定的页面。*

*我们通过扩展作为`<a>`标签接口的`HTMLAnchorElement`来做到这一点:*

```
*class RouterLink extends HTMLAnchorElement{
  constructor() {
    super();
  }

  connectedCallback() {
    this.addEventListener('click', e => {
      e.preventDefault();

      this.dispatchEvent(new CustomEvent('route-change', {
        composed: true,
        detail: {link: this}
      }));
    })
  }
}*
```

*在`connectedCallback`中，我们设置了一个事件处理程序来拦截`click`事件。通过调用`e.preventDefault`，我们阻止了浏览器跟随链接，所以当用户点击链接时什么都不会发生。*

*然后我们抛出一个新的`route-change`事件，在`link`属性中使用链接作为有效负载。父元素可以监听该事件并执行客户端路由，例如一个已经扩展的`nav`标记:*

```
*<nav **is="client-side-router**">
  <a href="/path/to/page1" **is="router-link"**>Page 1</a>
  <a href="/path/to/page2" **is="router-link"**>Page 2</a>
  <a href="/path/to/page3" **is="router-link"**>Page 3</a>
</nav>*
```

*通过这种方式，我们可以构建一个导航组件，它可以在不支持浏览器的旧版本中工作得非常好，并且可以在现代浏览器中被增强为客户端路由器(T21)。*

*让我们看看如何通过扩展`<nav>`标签来实现路由器本身。*

*`<nav>`标签没有自己的接口，所以它只是扩展了`HTMLElement`。尽管它是一个内置元素，我们仍然可以向它添加 Shadow DOM，这将使与子元素(链接)的交互变得更加容易和健壮:*

```
*class ClientSideRouter extends HTMLElement{
  constructor() {
    super();

    const shadowRoot = this.attachShadow({mode: 'open'});

    shadowRoot.innerHTML = `
      <slot name="link"></slot>
    `;
  }

  connectedCallback() {
    const slot = this.shadowRoot.querySelector('slot');
    const links = slot.assignedNodes();

    links.forEach(link => {
      link.addEventListener('route-change', e => {
        this.handleRouteChange(e.detail.link);
      });
    });
  }
}*
```

*路由器的影子 DOM 将只包含一个命名的`<slot>`元素，它将作为链接的插入点。然后，我们可以通过`<slot>`的`assignedNodes`方法获取所有链接，并为每个链接添加一个事件监听器，这样我们就可以在单击链接时处理路由更改。*

*我们需要在链接上添加的唯一东西是一个`slot`属性，以确保它们被插入到正确的插槽中:*

```
*<a href="/path/to/page1" is="router-link" **slot="link"**>Page 1</a>*
```

**我们可以省略插槽上的* `*name*` *属性和链接上的* `*slot*` *属性。这也可以，但是* `*nav*` *组件中的任何换行符也会被* `*assignedNodes()*` *作为空文本节点返回，我们需要过滤掉它们。**

*这很好，但我们需要为每个链接添加一个单独的事件处理程序，这有点不幸且效率低下。如果我们能给路由器本身添加一个事件处理器，那就更好了。*

*我们可以通过将`bubbles: true`添加到由`RouterLink`抛出的`route-change`事件的配置对象中来做到这一点:*

```
*this.dispatchEvent(new CustomEvent('route-change', {
  composed: true,
  **bubbles: true,** // <-- add this to make the event bubble up
  detail: {url}
}));*
```

*该事件现在将冒泡，我们可以在路由器上收听它:*

```
*class ClientSideRouter extends HTMLElement{
  constructor() {
    super();

    const shadowRoot = this.attachShadow({mode: 'open'});

    shadowRoot.innerHTML = `
      <slot name="link"></slot>
    `;
  }

  connectedCallback() {
    this.outlet = document.querySelector(this.dataset.outlet);

    **this.addEventListener('route-change', e => {
      this.handleRouteChange(e.detail.link)
    });**
  }

  handleRouteChange(link) {
    // handle route change
  }
}

customElements.define('client-side-router', ClientSideRouter, {extends: 'nav'});*
```

*我们现在还可以制作一个简单的`handleRouteChange`方法实现。我们可以给路由器添加一个`data-`属性，包含一个 CSS 选择器来指定模板应该呈现在哪里，给每个路由器链接添加一个`data-`属性来指定应该呈现哪个模板:*

```
*<nav is="client-side-router" **data-outlet="#main"**>
  <a href="/path/to/page1" is="router-link" slot="link"
     **data-template="./page1.html"**>Page 1</a>

  ...</nav><!-- templates are rendered here -->
<div **id="main"**></div>*
```

*在`handleRouteChange`方法中，我们获取模板，在 outlet 中渲染，并向浏览器的历史记录中添加条目，因此 url 将会改变以反映路由的变化:*

```
*async handleRouteChange(link) {
  const template = link.dataset.template;
  const url = link.getAttribute('href');
  const state = {template, url};

  const html = await (await fetch(template)).text();

  history.pushState(state, null, url);

  this.outlet.innerHTML = html;
}*
```

*这显然是一个幼稚且非常基础的实现，但我希望您已经了解了扩展内置 HTML 元素的可能性。*

*你可以在[我的 Github](https://github.com/DannyMoerkerke/client-side-router) 上找到包含演示页面的源代码。*

*你可以在 Twitter 上关注我，我经常在那里写关于 PWAs、网络组件和现代网络功能的文章。*
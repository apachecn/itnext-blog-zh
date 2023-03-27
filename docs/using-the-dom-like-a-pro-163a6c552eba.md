# 像专业人士一样使用 DOM

> 原文：<https://itnext.io/using-the-dom-like-a-pro-163a6c552eba?source=collection_archive---------0----------------------->

如何停止害怕 DOM，充分利用它的潜力，并真正开始爱上它

![](img/54b8069ddcfde025e5526c06eb73cd2a.png)

照片由 [Pankaj Patel](https://unsplash.com/@pankajpatel?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

当我在 2008 年开始作为一名专业的 web 开发人员工作时，我知道一些 HTML、CSS 和 PHP。与此同时，我也在学习这个叫做 JavaScript 的东西，因为它允许我显示和隐藏元素，做一些很酷的事情，比如下拉菜单。

当时我在一家小公司工作，主要为客户创建 CMS 系统，我们需要一个多文件上传器。这在当时用原生 JavaScript 是不可能的。

经过一番搜索，我找到了一个基于 Flash 和这个叫做 MooTools 的 JavaScript 库的[奇特解决方案。MooTools 有这个很酷的`$`功能来选择 DOM 元素，并带有进度条和 Ajax 请求等模块。几个星期后，我发现了 jQuery，我被震惊了。](http://digitarald.de/project/fancyupload/)

不再有冗长、笨拙的 DOM 操作，而是简单、可链接的选择器，并且附带了一大堆有用的插件。

快进到 2019 年，世界被框架所统治。如果你在过去的十年里开始做一名 web 开发人员，那么你可能很难接触到“原始的”DOM。你甚至不需要。

尽管 Angular 和 React 等框架导致 jQuery 的受欢迎程度大幅下降，[仍有令人震惊的 6600 万个网站在使用它](https://trends.builtwith.com/javascript/jQuery)，估计约占全球所有网站的[74%](https://w3techs.com/technologies/details/js-jquery/all/all)。

jQuery 的遗产令人印象深刻，它如何影响标准的一个很好的例子是模仿 jQuery 的`$`函数的`querySelector`和`querySelectorAll`方法。

具有讽刺意味的是，这两种方法可能是 jQuery 受欢迎程度下降的最大原因，因为它们取代了 jQuery 最常用的功能:轻松选择 DOM 元素。

但是原生 DOM API 是冗长的。

我的意思是，这是`$`对`document.querySelectorAll`。

这也是开发人员不愿使用原生 DOM API 的原因。但是真的没必要这样。

原生 DOM API 非常棒，而且非常有用。是的，它很冗长，但那是因为这些是低级的构建模块，旨在构建抽象概念。如果你真的担心额外的击键:所有现代编辑器和 ide 都提供了优秀的代码补全。您还可以为您最常用的功能起别名，我将在这里展示。

让我们跳进来吧！

# 选择元素

## 单一元素

要使用任何有效的 CSS 选择器选择单个元素，请使用:

```
document.querySelector(/* your selector */)
```

您可以使用这里的任何选择器:

```
document.querySelector('.foo')            // class selector
document.querySelector('#foo')            // id selector
document.querySelector('div')             // tag selector
document.querySelector('[name="foo"]')    // attribute selector
document.querySelector('div + p > span')  // you go girl!
```

当没有匹配的元素时，它将返回`null`。

## 多重元素

要选择多个元素，请使用:

```
document.querySelectorAll('p')  // selects all <p> elements
```

你可以像使用`document.querySelector`一样使用`document.querySelectorAll`。任何有效的 CSS 选择器都可以，唯一的区别是`querySelector`将返回一个元素，而`querySelectorAll`将返回一个包含找到的元素的静态`NodeList`。如果没有找到元素，它将返回一个空的`NodeList`。

一个`NodeList`是一个可迭代的对象，类似于一个数组，但它不是真正的*一个数组，所以它没有相同的方法。您可以在上面运行`forEach`，但不能运行`map`、`reduce`或`find`等。*

如果你确实需要在它上面运行数组方法，那么你可以简单地使用析构或者`Array.from`把它变成一个数组:

```
const arr = [...document.querySelectorAll('p')];*or*const arr = Array.from(document.querySelectorAll('p'));arr.find(element => {...});  // .find() now works
```

`querySelectorAll`方法与`getElementsByTagName`和`getElementsByClassName`等方法的不同之处在于，这些方法返回的`HTMLCollection`是*活动*集合，而`querySelectorAll`返回的`NodeList`是*静态*。

所以如果你做了`getElementsByTagName('p')`并且从文档中删除了一个`<p>`，那么它也将从返回的`HTMLCollection`中删除。

但是如果你做了`querySelectorAll('p')`并且从文档中删除了一个`<p>`，它仍然会出现在返回的`NodeList`中。

另一个重要的区别是，`HTMLCollection`只能包含`HTMLElement` s，而`NodeList`可以包含任何类型的`Node`。

## 相对搜索

你不一定需要在`document`运行`querySelector(All)`。您可以在任何`HTMLElement`上运行它来运行相对搜索:

```
const div = document.querySelector('#container');
div.querySelectorAll('p')  // finds all <p> tags in #container only
```

## 但还是啰嗦！

如果您仍然担心额外的击键，您可以将这两种方法别名化:

```
const $ = document.querySelector.bind(document);
$('#container');const $$ = document.querySelectorAll.bind(document);
$$('p');
```

给你。

## 爬上 DOM 树

使用 CSS 选择器来选择 DOM 元素意味着我们只能沿着 DOM 树向下移动。没有 CSS 选择器来沿着树向上移动来选择父节点。

但是我们可以使用`closest()`方法沿着 DOM 树向上移动，该方法也接受任何有效的 CSS 选择器:

```
document.querySelector('p').closest('div');
```

这将找到由`document.querySelector('p')`选择的段落的最近的父`<div>`元素。您可以将这些调用链接起来，在树中向上延伸:

```
document.querySelector('p').closest('div').closest('.content');
```

# 添加元素

众所周知，向 DOM 树添加一个或多个元素的代码会很快变得冗长。假设您想要将以下链接添加到您的页面:

```
<a href="/home" class="active">Home</a>
```

您需要做:

```
const link = document.createElement('a');
link.setAttribute('href', '/home');
link.className = 'active';
link.textContent = 'Home';document.body.appendChild(link);
```

现在想象一下，必须对 10 个元素进行这样的操作…

至少 jQuery 允许您这样做:

```
$('body').append('<a href="/home" class="active">Home</a>');
```

你猜怎么着？有一个本地对等项:

```
document.body.insertAdjacentHTML('beforeend', 
'<a href="/home" class="active">Home</a>');
```

`insertAdjacentHTML`方法允许您在 DOM 的四个位置插入任意有效的 HTML 字符串，由第一个参数指示:

*   `'beforebegin'`:元素前
*   `'afterbegin'`:在第一个子元素之前的元素内部
*   `'beforeend'`:在最后一个子元素之后的元素内部
*   `'afterend'`:元素后

```
<!-- *beforebegin* -->
**<p>**
  <!-- *afterbegin* -->
  foo
  <!-- *beforeend* -->
**</p>**
<!-- *afterend* -->
```

这也使得指定应该插入新元素的确切位置变得更加容易。假设您想在这个`<p>`之前插入一个`<a>`。如果没有`insertAdjacentHTML`，你将不得不这样做:

```
const link = document.createElement('a');
const p = document.querySelector('p');p.parentNode.insertBefore(link, p);
```

现在你可以做:

```
const p = document.querySelector('p');p.insertAdjacentHTML('beforebegin', '<a></a>');
```

还有一种插入 DOM 元素的等效方法:

```
const link = document.createElement('a');
const p = document.querySelector('p');p.insertAdjacentElement('beforebegin', link);
```

和文本:

```
p.insertAdjacentText('afterbegin', 'foo');
```

# 移动元素

`insertAdjacentElement`方法也可用于移动同一文档中的现有元素。当用`insertAdjacentElement`插入的元素已经是文档的一部分时，它将被简单地移动。

如果您有这个 HTML:

```
<div class="first">
  <h1>Title</h1>
</div><div class="second">
  <h2>Subtitle</h2>
</div>
```

`<h2>`插在`<h1>`之后；

```
const h1 = document.querySelector('h1');
const h2 = document.querySelector('h2');h1.insertAdjacentElement('afterend', h2);
```

这将只是简单地被*移动了*、**而不是复制了**:

```
<div class="first">
  <h1>Title</h1>
  **<h2>Subtitle</h2>**
</div><div class="second">

</div>
```

# 替换元素

一个 DOM 元素可以使用它的`replaceWith`方法被任何其他 DOM 元素替换:

```
someElement.replaceWith(otherElement);
```

它所替换的元素可以是用`document.createElement`创建的新元素，也可以是已经是同一文档的一部分的元素(在这种情况下，它将再次被移动，而不是被复制):

```
<div class="first">
  <h1>Title</h1>
</div><div class="second">
  <h2>Subtitle</h2>
</div>const h1 = document.querySelector('h1');
const h2 = document.querySelector('h2');h1.replaceWith(h2);// result:<div class="first">
  **<h2>Subtitle</h2>**
</div><div class="second">

</div>
```

# 移除元素

就叫它`remove`法:

```
const container = document.querySelector('#container');
container.remove();  // hasta la vista, baby
```

比老办法好多了:

```
const container = document.querySelector('#container');
container.parentNode.removeChild(container);
```

# 从原始 HTML 创建元素

`insertAdjacentHTML`方法允许我们将原始 HTML 插入到文档中，但是如果我们想要*从原始 HTML 创建*和元素并在以后使用它呢？

为此，我们可以使用`DomParser`对象及其方法`parseFromString`。`DomParser`提供将 HTML 或 XML 源代码解析成 DOM 文档的能力。我们使用`parseFromString`方法创建一个只有一个元素的文档，并只返回这一个元素:

```
const createElement = domString => new DOMParser().parseFromString(domString, 'text/html').body.firstChild;const a = createElement('<a href="/home" class="active">Home</a>');
```

# 检查大教堂

标准的 DOM API 也提供了一些方便的方法来检查 DOM。例如，`matches`确定一个元素是否匹配某个选择器:

```
<p class="foo">Hello world</p>const p = document.querySelector('p');p.matches('p');     // true
p.matches('.foo');  // true
p.matches('.bar');  // false, does not have class "bar"
```

您还可以使用`contains`方法检查一个元素是否是另一个元素的子元素:

```
<div class="container">
  <h1 class="title">Foo</h1>
</div><h2 class="subtitle">Bar</h2>const container = document.querySelector('.container');
const h1 = document.querySelector('h1');
const h2 = document.querySelector('h2');container.contains(h1);  // true
container.contains(h2);  // false
```

使用`compareDocumentPosition`方法可以获得关于元素的更详细的信息。此方法允许您确定一个元素是在另一个元素之前还是之后，或者其中一个元素是否包含另一个元素。它返回一个整数，表示被比较元素之间的关系。

下面是一个与上一个示例具有相同元素的示例:

```
<div class="container">
  <h1 class="title">Foo</h1>
</div><h2 class="subtitle">Bar</h2>const container = document.querySelector('.container');
const h1 = document.querySelector('h1');
const h2 = document.querySelector('h2');//  20: h1 is contained by container and follows container
container.compareDocumentPosition(h1); // 10: container contains h1 and precedes it
h1.compareDocumentPosition(container);// 4: h2 follows h1
h1.compareDocumentPosition(h2);// 2: h1 precedes h2
h2.compareDocumentPosition(h1);
```

从`compareDocumentPosition`返回的值是一个整数，它的位代表节点之间的关系，相对于给这个方法的参数。

所以考虑到语法`node.compareDocumentPostion(otherNode)`返回值的含义是:

*   1:节点不是同一文档的一部分
*   2: `otherNode`在`node`之前
*   4: `otherNode`跟随`node`
*   8: `otherNode`包含`node`
*   16: `otherNode`被`node`包含

可以设置多个位，这就是为什么在上面的例子中`container.compareDocumenPosition(h1)`返回 20，而你可能期望 16，因为`h1`被`container`包含。但是`h1`也是*跟随* `container` (4)所以结果值是 16 + 4 = 20。

## 请提供更多细节！

您可以通过`MutationObserver`界面观察任何 DOM 节点的变化。这包括文本更改、向被观察节点添加节点或从被观察节点移除节点，或者对节点属性的更改。

`MutationObserver`是一个非常强大的 API，几乎可以观察到 DOM 元素及其子节点上发生的任何变化。

通过用回调函数调用其构造函数来创建一个新的`MutationObserver`。每当被观察节点上发生变化时，将调用此回调:

```
const observer = new MutationObserver(callback);
```

为了观察一个元素，我们需要调用观察者的`observe`方法，将被观察的节点作为第一个参数，将一个带有选项的对象作为第二个参数。

```
const target = document.querySelector('#container');
const observer = new MutationObserver(callback);observer.observe(target, options);
```

直到调用`observe`后，才开始观察目标。

此选项对象采用以下键:

*   `attributes`:设置为`true`时，将观察节点属性的变化
*   `attributeFilter`:要监视的属性名数组，当`attributes`为`true`且未设置时，会监视对*节点所有*属性的更改
*   `attributeOldValue`:当设置为`true`时，每当发生变化时，将记录属性的先前值
*   `characterData`:当设置为 true 时，这将记录对*文本节点*的文本的更改，因此这仅适用于`Text`节点，而不适用于`HTMLElement` s。要实现这一点，被观察的节点需要是一个`Text`节点，或者，如果观察者监视一个`HTMLElement`，选项`subtree`需要设置为`true`以监视对子节点的更改。
*   `characterDataOldValue`:设置为`true`时，无论何时发生变化，都将记录字符数据的先前值
*   `subtree`:设置为`true`，也观察被观察元素的子节点的变化。
*   `childList`:设置为`true`，监控元素添加和删除子节点。当`subtree`被设置为`true`时，子元素也将被监视以添加和移除子节点。

当通过调用`observe`开始观察一个元素时，传递给`MutationObserver`构造函数的回调被调用，带有一个描述发生的变化的`MutationRecord`对象数组和作为第二个参数调用的观察者。

一个`MutationRecord`包含以下属性:

*   `type`:变更的类型，可以是`attributes`、`characterData`或`childList`。
*   `target`:改变的元素，其属性、字符数据或子元素
*   `addedNodes`:添加节点列表，如果没有添加，则为空`NodeList`
*   `removedNodes`:删除节点列表，如果没有删除，则为空`NodeList`
*   `attributeName`:变更属性的名称，如果没有属性变更，则为`null`
*   `previousSibling`:添加或删除节点的上一个兄弟节点或`null`
*   `nextSibling`:添加或删除节点的下一个兄弟节点或`null`

假设我们想要观察属性和子节点的变化:

```
const target = document.querySelector('#container');
const callback = (mutations, observer) => {
  mutations.forEach(mutation => {
    switch (mutation.type) {
      case 'attributes':
        // the name of the changed attribute is in
        // mutation.attributeName
        // and its old value is in mutation.oldValue
        // the current value can be retrieved with 
        // target.getAttribute(mutation.attributeName)
        break; case 'childList':
        // any added nodes are in mutation.addedNodes
        // any removed nodes are in mutation.removedNodes
        break;
    }
  });
};const observer = new MutationObserver(callback);observer.observe(target, {
  attributes: true,
  attributeFilter: ['foo'], // only observe attribute 'foo'
  attributeOldValue: true,
  childList: true
});
```

当您完成对目标的观察后，您可以断开观察器，如果需要的话，调用它的`takeRecords`方法来获取任何尚未交付给回调的待定突变:

```
const mutations = observer.takeRecords();
callback(mutations);
observer.disconnect();
```

# 不要害怕大教堂

DOM API 是一个非常强大和通用的 API，尽管有些冗长。请记住，它旨在为开发人员提供底层构建模块来构建抽象，因此从这个意义上来说，它需要冗长，以提供明确清晰的 API。

额外的击键不会吓到你，让你不敢充分发挥它的潜力。

DOM 是每个 JavaScript 开发者的基本知识，因为你可能每天都在使用它。不要害怕它，充分发挥它的潜力。

你会成为更好的开发者。
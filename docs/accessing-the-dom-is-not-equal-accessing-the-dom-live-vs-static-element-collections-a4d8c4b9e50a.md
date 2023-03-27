# 访问 DOM 不等于访问 DOM——活动元素集合和静态元素集合

> 原文：<https://itnext.io/accessing-the-dom-is-not-equal-accessing-the-dom-live-vs-static-element-collections-a4d8c4b9e50a?source=collection_archive---------4----------------------->

![](img/31ceef6919f667ebf4efa4648a036fe5.png)

照片:Unsplash.com 伊姆基兰

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Faccessing-the-dom-is-not-equal-accessing-the-dom-live-vs-static-element-collections-a4d8c4b9e50a%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

当浏览器接收到一个 HTML 文档时，它会创建[文档对象模型(DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) ，这是一个文档的树形表示。还有一些 DOM 方法，允许我们作为前端开发人员以编程方式访问部分解析过的文档，并为网站添加功能。到目前为止一切顺利！

您很快会遇到的一种方法是`querySelectorAll`，它用于访问 DOM 中的元素。让我们快速看一下它是如何工作的。

# 使用`querySelectorAll`访问 DOM

```
// <html>
// <head>...</head>
// <body>
//   <ul>
//     <li>foo</li>
//     <li>bar</li>
//     <li>baz</li>
//   </ul>
// </body>
// </html>const listItems = document.querySelectorAll('li'); console.log(listItems);        // NodeList(3) [li, li, li] console.log(listItems.length); // 3for (let i = 0; i < listItems.length; i++) { 
  console.log(listItems[i].innerText);
} // foo
// bar
// baz
```

多亏了现在好的开发工具，当你把一个对象记录到控制台时，浏览器会显示它的类型。正如你在上面看到的，`document.querySelectorAll`的返回值是一个`NodeList`。

在过去，与`NodeLists`打交道对我来说意味着一些惊喜。它们看起来像数组，但实际上不是，特定 MDN 文章中的一个大警告框清楚地描述了这一事实。

> *虽然 NodeList 不是一个数组，但是可以使用 forEach()对其进行迭代。几个较老的浏览器还没有实现这个方法。也可以使用 Array.from.* 将其转换为数组

让我惊讶的是今天`NodeLists`有一个定义好的`forEach`方法，因为当我开始从事 web 开发时，这个方法是缺失的，而这正是几年前我多次遇到的陷阱之一。`NodeLists`提供的其他方法有`item`、`entries`、`keys`和`values`。如果你想了解更多，我推荐你去看看[MDN 的文章](https://developer.mozilla.org/en-US/docs/Web/API/NodeList)。

# 现场收藏的魔力

当我上周阅读`NodeLists`的文档时，我注意到了一些我以前从未见过的东西:

> *在某些情况下，节点列表是一个活动集合[…]*

# 等等，什么？现场收藏？在某些情况下？

结果是`NodeLists`的行为因你如何访问它们而异。让我们看看同一个文档，以不同的方式检索元素。

```
// <html> 
// <head>...</head>
// <body>
//   <ul>
//     <li>foo</li>
//     <li>bar</li>
//     <li>baz</li>
//   </ul>
// </body>
// </html>// retrieve element using querySelectorAll
const listItems_querySelectorAll = document.querySelectorAll('li'); console.log(listItems_querySelectorAll);
// NodeList(3) [li, li, li] // retrieve element using childNodes
const list = document.querySelector('ul');
const listItems_childNodes = list.childNodes; console.log(listItems_childNodes);
// NodeList(7) [text, li, text, li, text, li, text]
```

明显的区别是，当您通过`childNodes`访问元素时，包含了更多的元素。该集合中的文本节点是您在 HTML 中看到的空格和换行符。

```
console.log(listItems_childNodes[0].textContent) // "
 "
```

但我发现的不是这样。两个`NodeLists`的最大区别是**一个是活动的，一个是静态的**，当我向`ul`元素添加另一个列表项时，它就可见了。

```
list.appendChild(document.createElement('li'));// static NodeList via querySelectorAll console.log(listItems_querySelectorAll);
// NodeList(3) [li, li, li]// live NodeList via childNodes
console.log(listItems_childNodes);
// NodeList(8) [text, li, text, li, text, li, text, li]
```

😲如您所见，`listItems_childNodes`(通过`childNodes`访问的`NodeList`)反映了 DOM 的元素，即使添加或删除了元素。由`querySelectorAll`返回的集合保持不变。**那对我来说完全是新消息！**

# 不是每个查询 DOM 的方法都返回一个节点列表

这变得更加令人困惑…你可能知道也有像`getElementsByClassName`和`getElementsByTagName`这样的方法可以让你访问 DOM 元素。原来这些方法返回不同的东西。

```
// <html>
// <head>...</head>
// <body>
//   <ul>
//     <li>foo</li>
//     <li>bar</li>
//     <li>baz</li>
//   </ul>
// </body>
// </html> const listItems_getElementsByTagName = document.getElementsByTagName('li');console.log(listItems_getElementsByTagName);
// HTMLCollection(3) [li, li, li]
```

哦好吧…一个`HTMLCollection`。那么另一种类型是什么呢？它只包括匹配元素，不包括文本节点，它只提供两个方法(`item`和`namedItem`)和**它是活动的**，这意味着它还将包括添加的元素。

```
listItems_getElementsByTagName[0].parentNode.appendChild(document.createElement('li'));// live HTMLCollection via getElementsByTagName console.log(listItems_getElementsByTagName);
// HTMLCollection(4) [li, li, li, li]
```

更复杂的是，当您使用`document.forms`(是的，您可以不查询 DOM 而访问表单)或通过元素的`children`属性访问子元素时，也会返回`HTMLCollections`。

```
// <html>
// <head>...</head>
// <body>
//   <ul>
//     <li>foo</li>
//     <li>bar</li>
//     <li>baz</li>
//   </ul>
// </body>
// </html>const list = document.querySelector('ul');
const listItems = list.children;
console.log(listItems); // HTMLCollection [li, li, li]
```

当你看的说明书`[HTMLCollection](https://dom.spec.whatwg.org/#htmlcollection)`时你会发现下面这句话:

> HTMLCollection 是我们无法摆脱的历史文物。虽然当然欢迎开发人员继续使用它，但是新的 API 标准设计人员不应该使用它[…]

这种说法清楚地表明，在一定时间内`NodeList`和`HTMLCollection`是相互竞争的标准，而现在我们同时被这两种标准所困。

# 进化网络是复杂的

所以，今天我们在`children`(直播`HTMLCollection`)旁边有`childNodes`(直播`NodeList`)，在`getElementsByTagName`(直播`HTMLCollection`)旁边有`querySelectorAll`(静态`NodeList`)以及一些意想不到的边缘情况，这取决于你如何访问元素。

就我个人而言，我很惊讶我以前从来没有听说过活动集合和静态集合，我认为在处理 DOM 时发现这一细节有一天会为我节省很多时间，因为找到由活动集合引起的 bug 肯定是很难找到的。

*如果你想体验一下所描述的行为，你可以看看这个* [*代码。*](https://codepen.io/stefanjudis/pen/NYXGbp)

*原载于 www.stefanjudis.com*[](https://www.stefanjudis.com/blog/accessing-the-dom-is-not-equal-accessing-the-dom/)**。**
# JavaScript 中使用 QuerySelectorAll 遍历 DOM 元素的 5 种方法

> 原文：<https://itnext.io/5-ways-to-loop-over-dom-elements-from-queryselectorall-in-javascript-55bd66ca4128?source=collection_archive---------0----------------------->

## 实用指南

## 用 DOM 中的 JavaScript 遍历元素

![](img/da2c0a22dd527202b6f652c7248400c5.png)

由[萨法尔·萨法罗夫](https://unsplash.com/@codestorm?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

有很多方法可以循环一个由 querySelectorAll 方法返回的 NodeList 对象(DOM 元素列表)!在这篇文章中，我想分享 5 种方法。

让我们从定义一段 HTML 和一个搜索多个元素的常量变量开始。

**HTML**

```
<div class="container" id="myContainer">
		<div class="fake-image">
		  <h2>Fake image</h2>
		</div>
		<div class="fake-image">
		  <h2>Fake image</h2>
		</div>
		<div class="fake-image">
		  <h2>Fake image</h2>
		</div>
		<div class="fake-image">
		  <h2>Fake image</h2>
		</div>
	</div>
```

**JavaScript**

```
const fakeImages = document.querySelectorAll(".fake-image");
```

所以现在我们准备找出我们可以使用哪 5 种方法来循环这个由`querySelectorAll`方法返回的很酷的 NodeList 对象。

如果一个方法给出了一个回调选项，那么我将使用“箭头函数”来实现。

# 1.For 循环

循环遍历所有内容的最著名的函数是老掉牙的 For-loop。这可能不是最漂亮的代码，但绝对是高性能的。

所以如果你需要支持 IE11 或更低版本的浏览器，并且你没有使用像 [Babel](https://babeljs.io/) 这样的编译器，那么这是你最好的武器。

**支持:**每个浏览器！

```
const fakeImages = document.querySelectorAll(".fake-image");
	for (var i = 0; i < fakeImages.length; i++) {
	  console.log('fakeImage: ', fakeImages[i]);
	}
```

# 2.为..关于

我会给[打电话..在普通 For 循环的基础上进行扩展。这是因为这个函数可以循环遍历 iterable 对象(包括 String、Array、类似数组的参数、NodeList 对象、TypedArray、Map 和 Set)。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)

如果你需要支持旧的浏览器，那么你肯定需要一个像 Babel 这样的编译器。但是如果你只需要支持现代浏览器..会是我的首选武器！

**支持:**所有现代浏览器！IE11 或更低版本不支持。

```
const fakeImages = document.querySelectorAll(".fake-image");
	for (const fakeImage of fakeImages) {
	  console.log('fakeImage: ', fakeImage);
	}
```

# 3.为..条目、关键字、值

在前面的方法中，我们只是使用节点列表在 For 中循环..循环的。但是 NodeList 也有更多的方法可以在这个循环中使用。

[条目()](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/entries)、[键()](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/keys)和[值()](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/values)方法返回一个[迭代器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)。迭代器是 ES2015 规范中一种新的可迭代协议。

在 JavaScript 中，某些类型的数据(数组或映射)具有内置的循环功能。

对象没有内置的循环功能。通过迭代器协议，我们可以遍历不默认支持迭代器的数据类型。

## 进入

这个循环中的每一项都是一个数组，首先是键，然后是元素。这看起来可能有点滑稽，但却是意料之中的行为。

```
const fakeImages = document.querySelectorAll(".fake-image");
	for (const fakeImage of fakeImages.entries()) {
	  console.log('fakeImage: ', fakeImage);
	};
```

## 价值观念

其中 entries 方法给出了一个键和值的数组。这个循环中的每一项都是一个元素，换句话说，就是方法名告诉我们的值。

```
const fakeImages = document.querySelectorAll(".fake-image");
	for (const fakeImage of fakeImages.values()) {
	  console.log('fakeImage: ', fakeImage);
	};
```

## 键

就像 values 方法为我们提供 NodeList 中每一项的值一样，keys 方法为我们提供 NodeList 对象中的键。

```
const fakeImages = document.querySelectorAll(".fake-image");
	for (const fakeImage of fakeImages.keys()) {
	  console.log('fakeImage: ', fakeImage);
	};
```

# 4.为每一个

这里有一个我不知道的很酷的方法😁。就像数组方法 forEach 一样， [NodeList 对象也有自己的 forEach](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach) 方法。

最重要的一点是，它只在现代浏览器中受支持。为了支持旧浏览器，你肯定需要一个编译器。

**支持:**所有现代浏览器！IE11 或更低版本不支持。

```
const fakeImages = document.querySelectorAll(".fake-image");
	fakeImages.forEach(fakeImage => {
	  console.log('fakeImage: ', fakeImage);
	});
```

# 5.(ES2015)使用 forEach 扩展语法

在 ES2015 中，我们为阵列提供了[扩展语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)。有了这个语法，你可以做很多很酷的事情！其中之一是，将 NodeList 对象转换为数组，并对其使用 Array forEach 方法。

对于老版本浏览器的支持，你肯定需要一个编译器，因为这种支持并没有在所有的现代浏览器中完全实现。

**支持:** [几乎所有现代浏览器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#Browser_compatibility)！IE11 或更低版本不支持。

```
const fakeImages = document.querySelectorAll(".fake-image");
	[...fakeImages].forEach(fakeImage => {
			console.dir(fakeImage);
	});
```

# 谢谢！

![](img/2bd571bb1ce9f466ae279fa98111b086.png)

读完这个故事后，我希望你学到了一些新的东西，或者受到启发去创造一些新的东西！🤗

如果我给你留下了问题或一些要说的话作为回应，向下滚动并给我键入一条消息。如果你想保密，请在 Twitter @DevByRayRay 上给我发一条 [DM。我的 DM 永远是开放的😁](https://twitter.com/@devbyrayray)

[**通过电子邮件获取我的文章点击这里**](https://byrayray.medium.com/subscribe) **|** [**购买 5 美元中等会员资格**](https://byrayray.medium.com/membership)

# 阅读更多

![RayRay](img/992af170033696163d6cc0269218aedd.png)

[雷雷](https://byrayray.medium.com/?source=post_page-----55bd66ca4128--------------------------------)

## 荒诞的故事

[View list](https://byrayray.medium.com/list/angular-stories-24674407532a?source=post_page-----55bd66ca4128--------------------------------)6 stories![](img/b94f2b7d2929c90566cd2dd6f657a751.png)![](img/02b73423a62d73b113af9fdf9629c79f.png)![Stacked books](img/b02a2f57c0093e04ab1d11d3a55f35ea.png)![RayRay](img/992af170033696163d6cc0269218aedd.png)

[雷雷](https://byrayray.medium.com/?source=post_page-----55bd66ca4128--------------------------------)

## 最新的 JavaScript 和 TypeScript 故事

[View list](https://byrayray.medium.com/list/latest-javascript-typescript-stories-0358ad941491?source=post_page-----55bd66ca4128--------------------------------)14 stories![](img/c93ca03b33796c40dcc47873de2697c2.png)![](img/86f37efa11855f6f0f0f62984c37f696.png)![](img/ddbaa6d0bea676316247e82043d60b63.png)
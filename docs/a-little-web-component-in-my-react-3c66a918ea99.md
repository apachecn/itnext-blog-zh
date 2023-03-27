# React 中的一个小 Web 组件

> 原文：<https://itnext.io/a-little-web-component-in-my-react-3c66a918ea99?source=collection_archive---------4----------------------->

在工作中，我使用 React 和所有与之配套的工具。在家里，我喜欢尽可能地接近金属(浏览器)。我记得当我可以检查页面，下载源代码，然后黑进我自己的东西。不再是了。

考虑到这一点，我已经花了很多时间在 web 组件中使用[定制元素 v1](https://developers.google.com/web/fundamentals/web-components/customelements) 和[影子 DOM v1](https://developers.google.com/web/fundamentals/web-components/shadowdom) 规范。老实说，这两个规范可能会很快改变我们构建 web 应用的方式。根据 CanIUse.com[的](https://caniuse.com/)统计，他们分别为全球用户的 [77.65%](https://caniuse.com/#feat=custom-elementsv1) 和 [78.36%](https://caniuse.com/#feat=shadowdomv1) 启用。当与浏览器中的原生 ES6 模块结合时，我已经能够从我的工具链中删除相当多的工具。

![](img/2db979636985a3ce9e78329070749574.png)

[@ barn images](https://unsplash.com/@barnimages):[https://unsplash.com/photos/t5YUoHW6zRo](https://unsplash.com/photos/t5YUoHW6zRo)

作为一个前端人员，有一件事一直困扰着我，那就是我们为了限定 CSS 的范围而做的所有疯狂的事情。无论是像 BEM 这样的命名约定，通过 CSS 模块散列类，还是用 Styled-Components 这样的工具内联它们，我从未对这个解决方案感到兴奋。事实上，正是这一点让我希望 web 组件能早点出现。不幸的是，我从来没有机会在专业上使用它，坦率地说，支持是不存在的。但是事情发生了多么大的变化。

所以前几天我正准备为前端 PDX 做一个演讲。我正在寻找结合子元素(脸)模式，又名 [renderProps](https://reactjs.org/docs/render-props.html) 的功能，与 es6'ish 组成。没成:(

然而，当我在玩 FACE 模式时，我想我可以用 Shadow Dom 创建一个超级小的 css-in-js 解决方案。会有多难呢？结果并不是这样，只有 48 行代码。

任何被 ShadowDom 组件包装的子元素都将被附加到它的 ShadowRoot。

传递给 lightDom 的任何组件都将被放置在组件 light DOM 中，并且不会出现，除非您已经在 ShadowDom 组件中对其进行了槽处理并声明了一个槽。

给影子世界做造型很老套。只是一个普通的旧样式标签和 BAM，我们就有了一个工作的 react 组件，它具有封装的样式、标记和对生命周期方法的完全访问。

唯一的警告是测试。JSDOM 完全不知道如何处理 attachShadow，在花了几个小时试图让它理解之后，我真的放弃了。我不打算使用这个组件，但我想我会分享它，以防有人想尝试一下。

回购:[https://github.com/EdwardIrby/react-shadow-dom-component](https://github.com/EdwardIrby/react-shadow-dom-component)
现场版:[https://absorbed-pump.glitch.me/](https://absorbed-pump.glitch.me/)
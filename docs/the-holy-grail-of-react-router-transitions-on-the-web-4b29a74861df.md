# React 路由器在网络上过渡的圣杯

> 原文：<https://itnext.io/the-holy-grail-of-react-router-transitions-on-the-web-4b29a74861df?source=collection_archive---------2----------------------->

![](img/617a5f38aa86d601bebf6533f658efb6.png)

[https://variety . com/2018/film/news/monty-python-holy-grail-Michael-Palin-unseen-sketches-1202891971/](https://variety.com/2018/film/news/monty-python-holy-grail-michael-palin-unseen-sketches-1202891971/)

去年我一直在用 react-router-dom 和 react-pose 完善路由转换。本周我破解了密码。

如果你有使用 react-pose 的经验，你可能熟悉 react-router-dom Switch 组件的动画。如果你是，你可能已经看到了很多问题，如动画停留在下一个渲染，或者从页面底部切换到另一个页面顶部时“跳转”。

这是我现在解决的问题，想和大家分享一下。如果您想立即开始，请查看这个 codesandbox 以获得最终解决方案:

要做到这一点，您将需要这些依赖项

```
npm install **react-router-dom react-pose** --save
```

# 动画开关. js

这是我们的神奇组件，取代了 react-router-dom <switch>组件。它处理与路线动画相关的一切，如动画方向，过渡速度和动画类型。</switch>

# Routes.js

我们路线的傀儡主人，拿着一个所有路线的列表，里面有组件。将<route>组件渲染为<animatedswitch>的子组件</animatedswitch></route>

# 要记住的事情

包含所有页面的包装样式非常重要。我们希望将所有的 routes position: fixed，这样当它们出现在页面上时，就不会有其他 DOM 元素干扰动画。如果你向下滚动，它将从下一页的顶部开始。将此类产品的宽度/高度设为 100vw/vh。此外，确保子组件可以向下滚动通过该组件。

应该就是这样了。这是一个简单的包装器组合，您可以将它与整个应用程序分开。再看看 codesandbox 和代码。如果你有问题，请继续问！也想听听大家对此的解决方案。🐱‍👤
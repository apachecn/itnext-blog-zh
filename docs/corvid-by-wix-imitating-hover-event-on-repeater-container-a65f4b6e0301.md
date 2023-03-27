# Velo by Wix:模仿 repeater 容器上的悬停事件

> 原文：<https://itnext.io/corvid-by-wix-imitating-hover-event-on-repeater-container-a65f4b6e0301?source=collection_archive---------1----------------------->

*Velo API 不提供 repeater 容器上的悬停事件。在这篇文章中，我们看看如何模仿悬停事件的一种方法。*

![](img/a02242d823dd42c06930a2aa07aab733.png)

[我博客上的这篇文章](https://shoonia.site/corvid-imitate-hover-event)

# 动机

我们有一个包含用户卡项目的`$w.Repeater`组件。当我们用鼠标光标指向某个项目时，我们希望将该项目的背景色改为浅蓝色`#CCE4F7`,当光标离开该项时，我们希望返回初始的白色。

为此，我们将使用另外两个提供 repeater API 的事件:

*   `[onMouseIn()](https://www.wix.com/velo/reference/$w/element/onmousein)`当鼠标指针移动到元素上时运行。
*   `[onMouseOut()](https://www.wix.com/velo/reference/$w/element#onMouseOut)`当鼠标指针离开元素时运行

此外，重复项没有属性`[style.backgroundColor](https://www.wix.com/velo/reference/$w/style/backgroundcolor)`来改变元素的背景颜色。但是我们可以使用`[background.src](https://www.wix.com/velo/reference/$w/background/background)` 属性来改变背景图像。这样，我们将使用单像素图像。

[这里是单像素图像](https://static.wixstatic.com/media/e3b156_df544ca8daff4e66bc7714ebc7bf95f1~mv2.png)

# **事件处理程序**

首先，为`onMouse{**In**/**Out**}`事件设置处理程序。我们将通过 repeaters 容器为两个事件使用一个函数。我们在上面声明了处理函数，并将函数名作为参数传递给容器方法。

```
**const** items = [ */* here are our users */* ];**function** imitateHover(event) {
  *// our handler for containers*
}

$w.onReady(**function** () {
  $w("#repeater1").data = items; $w("#container1").onMouseIn(imitateHover); *// set handler*
  $w("#container1").onMouseOut(imitateHover);
});
```

请注意，我们不会为了添加处理程序而将任何容器项嵌套到 repeater 中。像这样:

```
*// In this way, each time when onItemReady starts would be set a new handler for containers.*$w("#repeater1").onItemReady( ($item, itemData, index) => {
  $item("#container1").onMouseIn(imitateHover);
  $item("#container1").onMouseOut(imitateHover);
});
```

我们用`$w()`选择器在所有`#container1`上全局设置我们的处理程序。而且效果很好！

# 模仿悬停

我们对两个事件使用一个函数，因此我们需要监听哪种类型的事件正在发生。我们预计有两种事件类型:

*   `event.type === "mouseenter"`当`onMouse**In**()`运行时。
*   `event.type === "mouseleave"`当`onMouse**Out**()`运行时。

让我们看看代码:

```
**function** imitateHover(event) {
  **if** (event.type === "mouseenter") {
    console.log("we have mouseenter if onMouseIn() is running");
  }

  **if** (event.type === "mouseleave") {
    console.log("we have mouseleave if onMouseOut() is running");
  }
}

$w.onReady(**function** () {
  $w("#repeater1").data = items;
  $w("#container1").onMouseIn(imitateHover);
  $w("#container1").onMouseOut(imitateHover);
});
```

对象`event`总是会与当前容器中的项目保持一致，也就是我们所指的鼠标光标。我们可以通过`event.target`改变容器的`background.src` 属性。

```
*// link to one pixel image*
**const** HOVER_PNG = "https://static.wixstatic.com/media/e3b156_df544ca8daff4e66bc7714ebc7bf95f1~mv2.png";**function** imitateHover(event) {
  *// when the cursor over container then set image.*
  **if** (event.type === "mouseenter") {
    event.target.background.src = HOVER_PNG;
  }
  **if** (event.type === "mouseleave") {
    *// when the cursor is gone then remove the pixel image.*
    event.target.background.src = "";
  }
}

$w.onReady(**function** () {
  $w("#repeater1").data = items;
  $w("#container1").onMouseIn(imitateHover);
  $w("#container1").onMouseOut(imitateHover);
});
```

太好了！有效:[演示](https://shoonia.wixsite.com/blog/imitate-hover-event-on-corvid)

# 单像素图像

我们使用了到单像素图像的直接链接。这个图像的大小只有 70 字节。例如，这个图片的链接有 82 个字符长，82 个字节。链接比图像占用更多的内存。\_(ツ)_/

## 数据:URL

[数据 URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)，这是一种协议，允许将嵌入式小文件作为字符串内嵌在文档中。这意味着我们可以将一个像素的 PNG 图像转换成字符串，并将其传递给`background.src.`

我们可以通过 [1x1 PNG 生成器](https://shoonia.github.io/1x1/#cce4f7ff)创建需要的图像。

```
// *one-pixel image encoded to base64* **const** HOVER_PNG = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mM88+R7PQAIUwMo5M6pSAAAAABJRU5ErkJggg==';**function** imitateHover(event) {
  **if** (event.type === "mouseenter") {
    event.target.background.src = HOVER_PNG;
  }
  **if** (event.type === "mouseleave") {
    event.target.background.src = "";
  }
}

$w.onReady(**function** () {
  $w("#repeater1").data = items;
  $w("#container1").onMouseIn(imitateHover);
  $w("#container1").onMouseOut(imitateHover);
});
```

`data:URL`图像比该图像的直接链接稍长。和其他原因使用`data:URL`的小图像，我们不发送 HTTP 请求获取这个图像。

# 资源

*   [Velo API](https://www.wix.com/velo/reference/api-overview/introduction)
*   [数据 URL MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)
*   [1x1 PNG 发生器](https://shoonia.github.io/1x1/)
*   [我博客上的这篇文章](https://shoonia.site/corvid-imitate-hover-event/)
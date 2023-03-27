# 这是如此的交叉

> 原文：<https://itnext.io/thats-so-intersectional-5dcd7e8d0b6b?source=collection_archive---------5----------------------->

使用交叉点观察器 API，加载更加缓慢。

![](img/851b8a8853955541027405a04275356c.png)

laccaria ochropurpurea——一种可爱的小蘑菇，长着粉红色/紫色的鳃。

几周前偶然发现了[路口观察者 API](https://w3c.github.io/IntersectionObserver/) 。它仍处于工作草案阶段，但[已经得到了很好的支持](https://caniuse.com/#feat=intersectionobserver)，只有一个明显的例外:Safari，包括 iOS 版。但是，它是一个整洁的小 API，可以有许多不同的用途，对于它提供增强或者我们可以编写合理的后备的情况，今天没有理由不使用它。

# 使用交叉点观察器 API

为了演示其中一些用途，我将使用[这个简单的滚动照片库。](https://nabrown.github.io/intersection-observer/index-basic.html)

这方面的标记很简单，使用`figure`和`figcaption`，以及带有`srcset`属性的`img`，为不同的视窗尺寸提供大小合适的图像。

## 伟大的想法

与其绑定到`scroll`事件并煞费苦心地检查元素的位置以及它们是否在视口内，我们可以创建一个观察者向我们报告这些信息。

基本的设置是:我们告诉观察者观察什么*目标*元素，测量什么*根*元素的交集(默认为浏览器窗口)，以及我们关心什么交集*阈值*。(相交阈值是与根元素相交的元素的百分比)。

然后，每当一个元素跨越一个阈值(我称之为“交集变化”)时，观察者将执行一个回调函数。

## 增强照片库

其他人，包括 Google，已经写了关于使用这个新 API 的[延迟加载(更多内容在下面)，但是我们可以使用交叉点观察器 API 做什么其他的改进呢？让我们看看](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video/)[为每个路口变化提供的所有数据](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry):

```
var callback = function(changes, observer) { 
  changes.forEach(change => {
    //   change.target
    //   change.isIntersecting
    //   change.time
    //   change.intersectionRatio
    //   change.rootBounds
    //   change.boundingClientRect
    //   change.intersectionRect     
  });
};
```

每个`change`具有关于一个被观察元素的*的交点变化的信息:*

*   `target`是交集发生变化的元素
*   如果目标与根相交，返回真，否则返回假
*   `time`路口发生了变化
*   `intersectionRatio`:元素`target`与所选根相交的比例是多少，用小数表示
*   描述根元素(`rootBounds`)、被观察元素(`boundingClientRect`)和两者交集(`intersectionRect`)的矩形。

为了开始观察十字路口，我们需要做一点设置:

我们有几个配置变量，然后我们获取想要观察的元素，并将它们传递给一个`createObserver`函数。

在`createObserver`中，我们:

1.  为观察者设置`options`。除了提供一组从 0 到 1.0 的阈值之外，我们将坚持使用默认值。每当观察到的元素的交集比率超过该数组中的阈值时，我们的回调函数将被调用。
2.  创建观察者，传入回调函数`handleIntersection`和`options`。
3.  在我们想要观察的每个元素上部署观察者。

## 使用交集比率

现在，我们可以使用新发现的交叉点数据。首先，使用`intersectionRatio`，我们可以根据视图中的百分比淡入和淡出每个`img`:

```
function handleIntersection(changes) {
  changes.forEach((change) => {
    let figure = change.target;
    let img = figure.getElementsByTagName("img")[0];
    let ratio = change.intersectionRatio;
    img.style.opacity = minOpacity + (1 - minOpacity) * ratio;
  });
}
```

这将淡入图像，从我们设置的最小不透明度到完全不透明度。随着`transition: opacity 500ms;`的应用，我们得到了一个很好的平滑过渡。

我还想在图像全图显示时隐藏标题，就像这样:

```
let caption = figure.getElementsByTagName('figcaption')[0];if (ratio < 0.9) {
  caption.style.opacity = 1;
} else {
  caption.style.opacity = 0;
}
```

[画廊现在看起来是这样的](https://nabrown.github.io/intersection-observer/index-enhanced.html)。

## 使用交叉点时间戳

站点分析中一个常用的统计数据是第页上的*时间，但是由于许多简单的站点放弃了多个页面而选择了更长的滚动页面，交叉点观察器 API 可以帮助我们跟踪用户在每个部分花费的时间。在我们的例子中，假设我们想知道用户花多长时间凝视我们的每张照片。*

我们得到了每个路口变化发生的时间。因此，我们可以计算出一个被观察的元素在相交阈值内有多长。在我们的简单例子中——每个`figure`在浏览器视窗中可见多长时间？

除此之外，我们还可以补充:

```
if(ratio > .8 && figure.dataset.startTime === undefined){
  figure.dataset.startTime = entry.time;
}
if(ratio < .8 && figure.dataset.startTime !== undefined){
  console.log(entry.time - figure.dataset.startTime)
  delete figure.dataset.startTime;
}
```

我选择了 0.8，或者说 80%可见，作为阈值，我会把它算作“可见”。我们只是在目标元素上名为`startTime`的数据属性中存储数字第一次越过. 8 阈值的时间。当它第二次越过这个阈值时，我们`console.log`这个差值，然后删除`startTime`数据属性。(您可能不希望在数百个元素上添加和删除数据属性。或者，您可以将数据保存在对象中)。

当然，将日志记录到控制台并不太有趣，因此您可以将计时数据发送到分析工具。例如:

```
if(ratio > .8 && figure.dataset.startTime === undefined){
  figure.dataset.startTime = change.time;
}
if(ratio < .8 && figure.dataset.startTime !== undefined){
  ga('send', {
    hitType: 'timing',
    timingCategory: 'Images',
    timingVar: 'view_time',
    timingValue: change.time - figure.dataset.startTime,
    timingLabel: figure.getElementsByTagName('img')[0].getAttribute('src')
  });
  delete figure.dataset.startTime;
}
```

每当该数字高于和低于 80%阈值时，将记录一个单独的“在视图中”时间，因此我们将在报告端进行一些汇总。

## 使用边界矩形

实际上，我想不出这些的用例。如果你想出了一个，请在下面留下评论！[我做了这个，但是我觉得不算*有用*](https://codepen.io/noraspice/full/roEELv) 。

## 使用 isIntersecting

如果我们只需要知道目标元素是否与根元素相交，我们可以使用布尔值`isIntersecting`。继续懒惰装载！

# 使用交叉点观察器 API 进行延迟加载

我们将使用占位符图像和`data`属性，并添加一个`unveil`类，稍微修改一下我们的标记来实现延迟加载:

```
<img src="img/placeholder-v.png" class="unveil" 
  data-src="img/black-trumpet-700.jpg"
  data-srcset="img/black-trumpet-1400.jpg 1400w, img/black-trumpet-2800.jpg 2800w"
  alt="Top view of a delicate light gray-brown, funnel-shaped mushroom amongst leaf litter on the forest floor."
/>
```

如前所述， [Google 有一个很好的代码示例](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video/)并浏览了一下。我做了一些改变，使它成为一个带有可配置的`lookAhead`和回调的插件。

这种设置与我们目前使用的略有不同。设置了`rootMargin`选项，它扩展了根元素的边界框，因此当图像仍然在浏览器视窗之外时，它们被触发开始加载。此外，在这种情况下，我们只关心一个阈值(默认的`[0]`)，所以没有必要在这里传递我们自己的值。

```
document.addEventListener("DOMContentLoaded", function() {
  let options = {
    // allows us to start loading before the element is visible
    rootMargin: `${lookAhead}px 0px`
  }
  let lazyImageObserver = new IntersectionObserver(unveil, options);images.forEach(function(img) {
    lazyImageObserver.observe(img);
  });
});function unveil(changes, observer) {
  changes.forEach(function(change) {
    if (change.isIntersecting) {
      let img = change.target;
      img.src = img.dataset.src;
      img.srcset = img.dataset.srcset;
      img.classList.remove("unveil");
      // no need to observe this image anymore
      observer.unobserve(img);
      if (typeof callback === "function") callback.call(img);
    }
  });
}
```

最后，一旦图像被加载，我们不再需要观察它，所以我们可以使用`unobserve`方法:

```
observer.unobserve(img);
```

交叉点观察器 API 不可用时的备选方案包括:使用聚合填充、立即加载图像或使用其他延迟加载方法。我认为最后一个选项在保持轻量级和提供延迟加载的性能优势之间取得了很好的平衡，特别是对于 iOS 上的 Safari。对于[这个插件](https://github.com/nabrown/unveil-intersect)，我使用交叉点观察器 API 作为第一道攻击线，并使用一个 jQuery 插件的 [plain-js 重写作为退路。](https://medium.com/@norabrown/breaking-up-with-jquery-is-hard-to-do-27defe486b11)

# 结论

[这是最终的图库](https://nabrown.github.io/intersection-observer/)，有细微的增强和基于交叉点观察者的延迟加载。

Intersection Observer API 易于理解和实现，除了 Safari 之外也得到很好的支持，并且比绑定到`scroll`事件和手动处理元素位置更高效、更容易使用。

如果你已经把它投入使用，我很乐意听到它——请留下评论！

# 进一步阅读

*   [W3C 路口观察器 API 工作草案](https://www.w3.org/TR/intersection-observer/)
*   [MDN 上的交叉点观察器](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
*   [can 使用兼容性信息](https://caniuse.com/#feat=intersectionobserver)
*   [用普通 Javascript 重写 jQuery 延迟加载插件](https://medium.com/@norabrown/breaking-up-with-jquery-is-hard-to-do-27defe486b11)
*   [关于使延迟加载非 js 友好的想法](https://www.robinosborne.co.uk/2016/05/16/lazy-loading-images-dont-rely-on-javascript/)
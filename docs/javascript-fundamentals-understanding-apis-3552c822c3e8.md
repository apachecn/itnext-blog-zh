# JavaScript 基础:理解 API

> 原文：<https://itnext.io/javascript-fundamentals-understanding-apis-3552c822c3e8?source=collection_archive---------1----------------------->

![](img/431348761b23cddd947c31165fc59d1d.png)

应用程序接口(API)可以定义为由操作系统、应用程序或其他服务创建的一组功能和过程。希望在构建自己的应用程序时与这些设定的过程进行交互的开发人员可以使用它们。本质上，创建它们是为了去掉大量更复杂的代码，并提供更简单的语法。

例如，您可能想在您的网站中添加一个实时 twitter feed。通过使用 Twitter API 提供的相关方法(通过开发人员文档)，您可以查询 twitter &显示特定用户或标签的最新推文。

🤓*想了解最新的 web 开发吗？*🚀*想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# JavaScript 中的 API

JavaScript 有许多可用的 API，通常被定义为**浏览器 API**或**第三方 API**。让我们看一看每一个..

## **浏览器 API**

浏览器 API 内置于浏览器中，包含来自浏览器的数据。有了这些数据，我们可以做许多有用的事情，从简单地操纵`window` 或`element`，到使用诸如 WebGL 之类的 API 生成复杂的效果。

您将会遇到的一些更常见的浏览器 API 有:

*   **文档操作的 API:**DOM(文档对象模型)其实就是一个 API！它允许你操作 HTML 和 CSS——创建、删除和改变 HTML，动态地应用新的样式到你的页面，等等。
*   **获取服务器数据的 API:**比如`XMLHttpRequest`和 Fetch API，我们经常用来进行数据交换和部分页面更新。这种技术通常被称为 **Ajax。**
*   **用于操纵图形的 API:**这里我们说的是 Canvas 和 WebGL，它们允许你以编程方式更新 HTML `<canvas>`元素中包含的像素数据，从而创建 2D 和 3D 场景。
*   **音频/视频 API:**如`HTMLMediaElement`和网络音频 API。您可以使用它们来创建用于播放音频和视频的自定控制，在您的视频中显示字幕等文本轨道。对于音频，您可以给音轨添加效果(如增益、失真等)。
*   **设备 API:**用于从设备硬件中操作和检索数据，以便与 web 应用程序一起使用。例如通知用户有更新可用。

## 第三方 API

第三方 API 没有内置在浏览器中，你需要从其他地方获取它们的代码和信息。通过连接第三方 API，您可以访问并使用 API 提供的方法。

一些常见的第三方 API 包括:

*   Openweathermap.org:允许您查询天气数据。例如，您可以捕捉用户的位置并显示他们当前的温度。
*   Twitter API:你可以在你的网站上显示你最新的推文。
*   Google Maps API:允许你完全定制地图并包含在你的网页上。
*   YouTube API:允许你在网站上嵌入 YouTube 视频，搜索 YouTube 并自动生成播放列表。
*   Twilio API:该 API 提供了一个框架，用于将语音和视频通话功能构建到您的应用程序中，从您的应用程序发送 SMS/MMS，等等。

# 那么 API 是如何工作的呢？

尽管不同的 API 以不同的方式工作，但它们都有一些共同的基本因素，我们现在来看看:

## **API 是基于对象的！**

您编写的与 API 交互的代码将使用一个或多个 JavaScript 对象。这些对象充当 API 使用的数据(包含在对象属性中)和 API 提供的功能(包含在对象方法中)的容器。

让我们看一个使用 Web Audio API 的例子——API 由许多对象组成。例如，我们有:

*   这是一个音频图，可以用来操纵浏览器中的音频播放，并且有许多方法和属性可以用来操纵音频。
*   `MediaElementAudioSourceNode`这代表了一个`<audio>`元素，包含您想要在音频上下文中播放和操作的声音。
*   `AudioDestinationNode`这代表音频的目的地，即电脑上实际输出音频的设备，通常是扬声器或耳机。

这些物体是如何相互作用的？让我们看看下面的 HTML:

```
<audio src = "funkybeats.mp3"></audio><button class = "paused">Play</button>
<br>
<input type = "range" min = "0" max = "1" step = "0.10" value = "1" class="volume">
```

这里有一个`<audio>`元素，我们用它来将 MP3 嵌入到页面中。我们不包括任何默认的浏览器控件。然后，我们添加了一个用于播放和停止音乐的`<button>`，以及一个类型为 range 的`<input>`元素，我们将使用它来调整音乐播放时的音量。

让我们看看这个例子的 JavaScript..

我们将首先创建一个`AudioContext`实例，在其中操作我们的轨迹:

```
const AudioContext = window.AudioContext || window.webkitAudioContext;
const audioCtx = new AudioContext();
```

然后我们将创建常量来存储对我们的`<audio>`、`<button>`和`<input>`元素的引用，并使用`AudioContext.createMediaElementSource()`方法来创建一个代表我们的音频源的`MediaElementAudioSourceNode`——它将从`<audio>`元素开始播放:

```
const audioElement = document.querySelector('audio');
const playBtn = document.querySelector('button');
const volumeSlider = document.querySelector('.volume');const audioSource = audioCtx.createMediaElementSource(audioElement);
```

接下来，我们将包括几个事件处理程序，每当按钮被按下时，它们就在播放和暂停之间切换，并在歌曲结束播放时将显示重置回开头:

```
// Play + Pause audio
playBtn.addEventListener('click', function() {
    // check if context is in suspended state (autoplay policy)
    if (audioCtx.state === 'suspended') {
        audioCtx.resume();
    } // If track is stopped, play it
    if (this.getAttribute('class') === 'paused') {
        audioElement.play();
        this.setAttribute('class', 'playing');
        this.textContent = 'Pause'
  // If track is playing, stop it
} else if (this.getAttribute('class') === 'playing') {
        audioElement.pause();
        this.setAttribute('class', 'paused');
        this.textContent = 'Play';
    }
});// If the track ends
audioElement.addEventListener('ended', function() {
    playBtn.setAttribute('class', 'paused');
    this.textContent = 'Play'
});
```

**注意**:正在使用的`play()`和`pause()`方法实际上是`HTMLMediaElement` API 的一部分，而不是 Web Audio API(尽管它们密切相关！).

现在，我们使用`AudioContext.createGain()`方法创建一个`GainNode`对象，该对象可用于调整通过它输入的音频的音量，并创建另一个事件处理程序，该处理程序在滑块值改变时改变音频图形的 gain (volume)值:

```
const gainNode = audioCtx.createGain();volumeSlider.addEventListener('input', function() {
    gainNode.gain.value = this.value;
});
```

最后，我们连接音频图中的不同节点，这是使用每个节点类型上可用的`AudioNode.connect()`方法完成的:

```
audioSource.connect(gainNode).connect(audioCtx.destination);
```

音频从源开始，然后连接到增益节点，以便可以调整音频的音量。增益节点然后连接到目标节点，这样声音可以在您的计算机上播放(`AudioContext.destination`属性表示您的计算机硬件上可用的默认`AudioDestinationNode`，例如您的扬声器)。

## API 有可识别的入口点

无论何时使用 API，都要确保知道入口点在哪里！在网络音频 API 中，它是`AudioContext`对象。这需要用来做任何操作。

使用文档对象模型(DOM) API 时——它的特性往往挂在`Document`对象上，或者挂在您希望以某种方式影响的 HTML 元素的实例上，例如:

```
// Create a new em element
let em = document.createElement('em'); // Reference an existing p element
let p = document.querySelector('p'); // Give em some text content
em.textContent = 'Hello, world!'; // Embed em inside 
p.appendChild(em);
```

Canvas API 也依赖于获取一个上下文对象来操作事物。它的上下文对象是通过获取对您想要绘制的`<canvas>`元素的引用，然后调用它的`HTMLCanvasElement.getContext()`方法来创建的:

```
let canvas = document.querySelector('canvas');
let ctx = canvas.getContext('2d');
```

我们想对画布做的任何事情都是通过调用 context 对象(是`CanvasRenderingContext2D`的一个实例)的属性和方法来实现的，例如:

```
Circle.prototype.draw = function() {
  ctx.beginPath();
  ctx.fillStyle = this.color;
  ctx.arc(this.x, this.y, this.size, 0, 2 * Math.PI);
  ctx.fill();
};
```

## API 通过事件处理状态的变化

当使用`XMLHttpRequest`对象(每个对象代表一个对服务器检索新资源的 HTTP 请求)时，我们有许多可用的事件，例如当包含请求资源的响应成功返回时，触发`load`事件，现在它是可用的。

以下代码提供了一个如何使用它的示例:

```
let requestURL = 'https://a-website.com/json/usernames.json';
let request = new XMLHttpRequest();
request.open('GET', requestURL);
request.responseType = 'json';
request.send();request.onload = function() {
  let usernames = request.response;
  populateHeader(usernames);
  showNames(usernames);
}
```

前五行指定我们想要获取的资源的位置，使用`XMLHttpRequest()`构造函数创建一个请求对象的新实例，打开一个 HTTP `GET`请求来检索指定的资源，指定响应应该以 JSON 格式发送，然后发送请求。

然后,`onload`处理函数指定我们对响应做什么。我们知道响应将在 load 事件被请求后成功返回并可用(除非发生错误)，所以我们将包含返回 JSON 的响应保存在`usernames`变量中，然后将它传递给两个不同的函数进行进一步处理。

*注意:*在下一篇文章中，我们将更详细地了解从服务器获取数据。

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*《JavaScript 掌握完全指南》*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

今天就到这里吧！我们已经了解了 API 是什么，它们是如何工作的，我们比较了浏览器和第三方 API，最后我们了解了许多 API 共有的一些共同特征。我希望这篇介绍能让你更好地理解什么是 API，以及我们如何在我们的项目中使用它们！

我希望这篇文章对你有用！你可以在媒体上关注我。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用的时候会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉
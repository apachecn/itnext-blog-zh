# 五分钟了解异步和延迟

> 原文：<https://itnext.io/five-minutes-to-understand-async-and-defer-67719f289680?source=collection_archive---------5----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

# 脚本标签

当我们想在网页中插入脚本时，标准的方式是使用脚本标签(即

首先，当一个脚本在 html 页面中被引用时，浏览器会为您做两件事:

*   检索/加载脚本内容，这是非阻塞的！
*   运行脚本内容，这是屏蔽！

假设页面上有两个脚本:

```
//script1.js
let t1 = +new Date;
console.log('script1 is running at', t1);console.log('script1 element', document.getElementById('load-experiment'));
while((+new Date) - t1 < 1000) {
    // delay 1 seconds
}
console.log('script1 finishes running at', +new Date);//script2.js
let t2 = +new Date;
console.log('script2 is running at', t2);console.log('script2 element', document.getElementById('load-experiment'));
while((+new Date) - t2 < 2000) {
    // delay 2 seconds
}
console.log('script2 finishes running at', +new Date);
```

# 将脚本标签放在头部

```
<!--all-in-head.html-->
<html>
    <head>
        <title> test js tag async and defer attributes</title>
        <script src='./script1.js'></script>
        <script src='./script2.js'></script>
    </head>
    <body>
        <h1 id='load-experiment'> hello world </h1>
    </body>
</html>
```

控制台输出:

```
script1 is running at 1496747869008
script1.js:4 script1 element null
script1.js:8 script1 finishes running at 1496747870008
script2.js:2 script2 is running at 1496747870009
script2.js:4 script2 element null
script2.js:8 script2 finishes running at 1496747872009
```

结论:

*   当我们在浏览器中打开 html 时，我们可以注意到页面加载的延迟。页面在正确呈现之前会变成空白。这是因为两个脚本的运行阻塞了 DOM 呈现。
*   当脚本运行时，它们无法获取 DOM 元素(即 document . getelementbyid(' load-experiment ')为空)。这是因为脚本是在 DOM 呈现之前运行的。
*   脚本本身是互相屏蔽的。它们按照 html 中指定的顺序运行。Script2 在 script1 完成运行后运行。

# 将脚本标签放在正文的底部

这是《高性能网站》一书第六条规则的建议。

```
<!--all-in-body.html-->
<html>
    <head>
        <title> test js tag async and defer attributes</title>
    </head>
    <body>
        <h1 id='load-experiment'> hello world </h1>
        <script src='./script1.js'></script>
        <script src='./script2.js'></script>
    </body>
</html>
```

控制台输出:

```
script1 is running at 1496751597679
script1.js:4 script1 element <h1 id=​"load-experiment">​ hello world ​</h1>​
script1.js:8 script1 finishes running at 1496751598679
script2.js:2 script2 is running at 1496751598680
script2.js:4 script2 element <h1 id=​"load-experiment">​ hello world ​</h1>​
script2.js:8 script2 finishes running at 1496751600680
```

结论:

*   在浏览器中打开页面时没有页面延迟和空白页面，因为脚本是在 DOM 准备好之后放置/运行的。
*   脚本可以正确地获取 DOM 元素。
*   脚本本身互相阻塞，和第一个例子一样。

然而，将所有脚本放在 body 的底部有时并不适合一些特定的实际情况。例如，如果我们的目标是计算页面的 ATF[[https://en.wikipedia.org/wiki/Above_the_fold](https://en.wikipedia.org/wiki/Above_the_fold)]，我们不能简单地等待 DOM 的渲染。我们必须预先加载并运行一些脚本，然后在 ATF 准备好的时候触发 ATF 标记，这肯定是在 DOM 为具有滚动内容的网页做好准备之前。所以，想出下一个解决方案似乎也在情理之中。

# 根据需求将脚本分别放在头部和身体中。

把需要预运行的脚本放在头部，其他的放在身体的底部。例如

```
<html> 
    <head> 
        <script src="headScripts.js"></scripts> 
    </head> 
    <body>
        <h1 id='load-experiment'> hello world </h1>
        <script src="bodyScripts.js"></script> 
    </body>
</html>
```

# 推迟！

将脚本放在主体底部的主要缺点是，脚本只有在 DOM 呈现后才能被检索/加载。正如我们所说，检索/加载脚本内容是非阻塞的，而运行脚本内容是阻塞的部分。如果我们可以在 DOM 渲染时检索/加载脚本，而不是等待 DOM 完成渲染，那么我们就可以提高 web 性能。这非常有效，尤其是当脚本很大的时候。这就是为什么引入了*延迟*的原因。*延迟*同时加载脚本，但是只在 DOM 准备好之后运行脚本。

```
<!--defer.html-->
<html>
    <head>
        <title> test js tag async and defer attributes</title>
        <script defer src='./script1.js'></script>
        <script defer src='./script2.js'></script>
    </head>
    <body>
        <h1 id='load-experiment'> hello world </h1>
    </body>
</html>
```

控制台输出:

```
script1 is running at 1496760312686
script1.js:4 script1 element <h1 id=​"load-experiment">​ hello world ​</h1>​
script1.js:8 script1 finishes running at 1496760313686
script2.js:2 script2 is running at 1496760313686
script2.js:4 script2 element <h1 id=​"load-experiment">​ hello world ​</h1>​
script2.js:8 script2 finishes running at 1496760315686
```

结论:

*   结果和把脚本放在主体的底部是一样的。我们可以从结果中得出结论，脚本是在 DOM 准备好之后运行的，因为我们确实可以获取 DOM 元素。
*   甚至*延迟的*脚本也遵循 html 中指定的顺序规则，脚本 1 在脚本 2 之后运行。

# 异步！

网页通常包含一些严格独立的附加功能，而不是必须具备的功能，例如某些页面上的评论和聊天功能。由于特征是独立的，它们没有运行顺序的限制。在这种情况下，我们可以使用*异步*

```
<!--async.html-->
<html>
    <head>
        <title> test js tag async and defer attributes</title>
    </head>
    <body>
        <h1 id='load-experiment'> hello world </h1>
        <script async src='./script1.js'></script>
        <script async src='./script2.js'></script>
    </body>
</html>
```

刷新页面时，我们可以观察到不同的控制台输出:

*   脚本 1 和脚本 2 的运行顺序不同
*   提取 DOM 元素的结果不一致

由于异步脚本不能保证运行顺序，这通常是潜在隐藏错误的来源。在使用 async 之前要三思，确保这些脚本是严格独立的。

# 结论

导入脚本的一般规则是:

```
<html> 
    <head> 
        <!--headScripts.js is the script that has to be loaded and run before DOM is ready-->
        <script src="headScripts.js"></scripts> 
        <!--bodyScripts.js loads first and runs after DOM is ready-->
        <script defer src="bodyScripts.js"></script> 
    </head> 
    <body>
        <!--body content-->
        <h1 id='load-experiment'> hello world </h1>
        <!--independent scripts，nice-to-have -->
        <script async src="forumWidget.js"></script>
        <script async src="chatWidget.js"></script>
    </body>
</html>
```

[代码示例](https://github.com/n0ruSh/the-art-of-reading/tree/master/javascript/Async%20Javascript/defer-async)

# 参考

*   [异步 JavaScript](https://www.amazon.com/Async-JavaScript-Responsive-Pragmatic-Express-ebook/dp/B00AKM4RVG)
*   [高性能网站](https://www.amazon.com/High-Performance-Web-Sites-Essential/dp/0596529309/ref=sr_1_1?s=books&ie=UTF8&qid=1521248657&sr=1-1&keywords=high+performance+websites)

# 通知；注意

*   读书笔记系列如果想关注最新的新闻/文章，请[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。
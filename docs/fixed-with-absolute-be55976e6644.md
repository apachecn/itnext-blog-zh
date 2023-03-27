# 绝对固定

> 原文：<https://itnext.io/fixed-with-absolute-be55976e6644?source=collection_archive---------5----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

# 几个概念

*   [offsetWidth](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetWidth) :给出元素的宽度，以像素为单位。
*   [offsetHeight](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetHeight) :给出元素的高度，以像素为单位。
*   [clientWidth](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth) :给出元素内部空间的大小，忽略边框宽度。
*   [clientHeight](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight) :给出元素内部空间的大小，忽略边框高度。
*   [pageXOffset](https://developer.mozilla.org/en-US/docs/Web/API/Window/pageXOffset) :返回沿横轴(左右)滚动的像素数。
*   [page yo offset](https://developer.mozilla.org/en-US/docs/Web/API/Window/pageYOffset):返回文档当前沿纵轴滚动的像素数，值 0.0 表示文档的上边缘当前与窗口内容区域的上边缘对齐。
*   [getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect) :返回一个具有 top、bottom、left 和 right 属性的对象，指示元素各边相对于视口左上角的像素位置。如果希望它们相对于整个文档，则必须添加当前滚动位置，可以在 pageXOffset 和 pageYOffset 绑定中找到该位置。
*   [innerWidth](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerHeight) :浏览器窗口视区的宽度(以像素为单位)，包括垂直滚动条(如果渲染的话)。
*   [innerHeight](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerHeight) :浏览器窗口视区的高度(以像素为单位)，包括水平滚动条(如果渲染的话)。
*   [scrollWidth](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollWidth) :元素内容宽度的度量，包括因溢出而在屏幕上不可见的内容。
*   [scrollHeight](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollHeight) :测量元素内容的高度，包括因溢出而在屏幕上不可见的内容。

# 实践

让我们把上面的概念放在一起，用`absolute`位置实现`fixed`位置效果，以及滚动进度功能。

```
// fixed.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Fixed</title>
    <style>
        body {
            margin: 0;
            position: relative;
            /* extend the body so that it can scroll to see the fixed effect */
            width: 2000px;
            height: 2000px;
        }
        #greet {
            width: 100px;
            height: 100px;
            line-height: 100px;
            padding: 5px;
            text-align: center;
            border: 2px solid green;
            position: absolute;
        }
        .progress {
            text-align: center;
            border: 1px solid black;
            /* fixed for progress as we only want to demonstrate the usage of scrollWidth and page offset for progress functionality */
            position: fixed;
            left: 50%;
            padding: 10px;
            /* center the progress info divs */
            transform: translateX(-50%);
        }
        #xprogress{
            top: 0;
        }
        #yprogress{
            top: 50px;
        }
    </style>
</head>
<body>
    <div id="greet"> Hello World </div>
    <div id="xprogress" class="progress">horizontal: 0</div>
    <div id="yprogress" class="progress">vertical: 0</div>
    <script>
        let node = document.querySelector('#greet');
        console.log(JSON.stringify({
            offsetWidth: node.offsetWidth, // 114 - with padding and border
            offsetHeight: node.offsetHeight, // 114 - with padding and border
            clientWidth: node.clientWidth, // 110 - with padding but without border
            clinetHeight: node.clientHeight, // 110 - with padding but without border
            innerWidth: innerWidth, // 1670, vary based on your setting
            innerHeight: innerHeight // 700, vary based on your setting
        }, null, '\t'));
        node.style.left = (innerWidth / 2 - node.offsetWidth / 2) + 'px';
        node.style.top = (innerHeight / 2 - node.offsetHeight / 2) + 'px'; // for fixed div
        document.addEventListener('scroll', function() { // listen to scroll event, adjust the position based on the scroll infomation
            let rec = node.getBoundingClientRect();
            let {top, left, width, height} = rec; // top and left is based on viewport, need to take scroll offset into account
            console.log(JSON.stringify({
                width,
                height,
                top,
                left,
                pageYOffset,
                pageXOffset
            }, null, '\t'));
            node.style.left = (pageXOffset + innerWidth / 2 - width / 2) + 'px';
            node.style.top = (pageYOffset + innerHeight / 2 - height / 2) + 'px';
        }); let yMax = document.body.scrollHeight - innerHeight;
        let xMax = document.body.scrollWidth - innerWidth;
        // for progress bar
        document.addEventListener('scroll', function () {
            let xProgress = document.querySelector('#xprogress');
            let yProgress = document.querySelector('#yprogress'); xProgress.textContent = `horizontal: ${pageXOffset * 100/ xMax}%`;
            yProgress.textContent = `vertical: ${pageYOffset * 100 / yMax}%`;
        });
    </script>
</body>
</html>
```

[代码示例](https://github.com/n0ruSh/the-art-of-reading/blob/master/javascript/Eloquent%20Javascript/fixed.html)

# 参考

*   [雄辩的 JavaScript](https://www.amazon.com/Eloquent-JavaScript-2nd-Ed-Introduction/dp/1593275846)

# 通知；注意

*   读书笔记系列如果想关注最新的新闻/文章，请[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。
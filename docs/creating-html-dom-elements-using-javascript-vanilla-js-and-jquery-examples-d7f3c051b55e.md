# 使用 JavaScript 创建 HTML DOM 元素(普通 JS 和 jQuery 示例)

> 原文：<https://itnext.io/creating-html-dom-elements-using-javascript-vanilla-js-and-jquery-examples-d7f3c051b55e?source=collection_archive---------2----------------------->

![](img/e53505cb9e7f92067645af98409d4b4f.png)

照片由 [Jackson So](https://unsplash.com/@jacksonsophat?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/html?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

Javascript 有一些有用的方法，允许我们创建 HTML 元素。如果我们不想对标记进行硬编码，而是在浏览器中发生特定事件时动态生成标记，这一点非常重要。这可以使用普通的 JavaScript 或浏览器兼容大师 jQuery 来完成。本文的目的不是评价普通 JS 和 jQuery 哪个更好，而是向您展示使用两者创建 DOM 元素的方法。

## **香草方法**

使用普通 Javascript，只涉及两个步骤:一、创建元素；第二，把它附加到一个已经存在的元素上。以下示例:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="style.css">
    <title>DOM</title>
</head><body> <div id="div">

    </div>

    <script> const div = document.getElementById("div");

    const paragraph = document.createElement("p");

    paragraph.textContent = 'Hello. I was created dynamically';

    div.appendChild(paragraph); </script></body>
</html>
```

**步骤说明:**

```
const paragraph = document.createElement("p");
```

使用 **document.createElement()** 方法，我们创建了没有内容的*<p></p>*元素。创建任何其他元素的语法都是一样的；假设一个 span 元素是 document . createelement(“span”)。

```
paragraph.textContent = 'Hello. I was created dynamically';
```

创建元素后，我们使用 **Node.textContent** 属性将一个字符串传递到段落中。

```
div.appendChild(paragraph);
```

最后，我们使用 **appendChild()** 方法将段落附加到已经存在于标记中的 div。

这就是我们的 HTML 的样子:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="style.css">
    <title>DOM</title>
</head>
<body>
    <div id="div">
      <p> Hello. I was created dynamically </p>
    </div>
```

步骤很简单:使用**document . createelement()**方法创建一个元素，然后使用 **appendChild()** 方法将它附加到一个现有元素。

## jQuery 方法

```
$(document).ready(function(){ const $div = $('#div');

    const $paragraph = $('<p> Hello. I was created dynamically </p>'); /*note that you can create an empty element
    like we did in the previous example by doing this:
    const $paragraph = $('<p>');
    $paragraph.text('Hello. I was created dynamically');
    */ $div.append($paragraph);
})
```

对于 jQuery，步骤是相同的，但是语法有显著的不同。这很容易掌握。如果你有任何问题，请在评论区提出，我会在那里回复，比 V8 JavaScript 引擎更快。

你也可以订阅，把我的文章直接发到你的邮箱里。

编码快乐！
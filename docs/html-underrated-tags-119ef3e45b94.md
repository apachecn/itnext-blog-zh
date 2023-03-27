# HTML:被低估的标签

> 原文：<https://itnext.io/html-underrated-tags-119ef3e45b94?source=collection_archive---------2----------------------->

## 未广泛使用的有用功能

HTML 比你想象的要宽泛一点，或者你可能已经忘记了这个特性就在 HTML 中。然而在这里，我将向你展示一些 HTML 被低估的元素。

![](img/f0d5aae61ef628938a83e0802618fca9.png)

其结构类似于带有 **<来源>** 的视频标签，但这是用于图像的。有人可能会说:“等等，我们不是已经有了 **< img >** 了吗？，为什么我会用这个……没用”。好吧，不是这样的，如果你认为比 **< img >** 标签好一点。

当你有两种或两种以上特质的形象时，你是如何处理的？。也许你可以尝试使用服务器渲染来检测用户代理，这样你就可以为设备提供高质量的服务。。为什么我们不直接使用*图片*元素？。没错，使用 Picture element，您可以使用媒体查询来告诉浏览器应该显示哪个图像，其他来源将被忽略，以便图像可以更快地加载。如果将 Picture 与 CSS 响应式经典属性相结合，就可以实现真正的响应式图像。*查看示例并调整大小以分析变化。*

```
<picture>
  <source media="(max-width: 500px)" srcset="alt/image.img">
  <img src="path/to/image.img">
</picture>
```

使用响应图像的图片示例

[*https://codepen.io/adrian-legaspi/pen/PowpEPY*](https://codepen.io/adrian-legaspi/pen/PowpEPY)

没错，HTML 自带原生的“表单组”。在我看来，有时最好使用这个标签来节省创建布局的时间，实际上对 SEO 更好。我不打算说谎，它是如此丑陋，但这是另一个好消息，因为你不用担心定位元素，而只需要弥补它们。

```
<fieldset>
  <legend>Login:</legend>
  User: <input type="text"><br>
  password: <input type="password"><br>
  <button>Login</button>
</fieldset>
```

我的字段集示例

[*https://codepen.io/adrian-legaspi/pen/RwNpmqg*](https://codepen.io/adrian-legaspi/pen/RwNpmqg)

嗯，我对 CSS 不是很有创造力，但是你用我的笔得到了灵感，希望你比我更擅长 CSS。

外面有这么多带有复杂进度条的库，但是我不明白为什么这个标签没有被充分利用。很容易应用 CSS，如果你不介意设置一个最小值和最大值，这没关系，因为它会显示为未确定的加载栏。

```
<progress max=100 value=13></progress>
```

【https://codepen.io/adrian-legaspi/pen/QWwpRPJ*T21*

正如你所看到的，用 JavaScript 设计和操作这个值是非常容易的。下次你需要进度条的时候，考虑一下这个老朋友。

这个标签是我的最爱之一，它比你想象的更有用。这将有助于更好地处理相对的**<>**链接，因为你只需要在头部用 **< base >** 声明根 URL，所有具有相对路径的链接将不会采用默认的，而是声明的基本 URL，通用目标属性也可以重写。我知道，默认情况下，相对链接被重定向到实际路径，但有时与生成的动态内容，这是不好的。图像或文件的相对路径也将采用默认的**<>**基路径，除非你给出一个完整的 URL，如果你这样做的话 **<基路径>** 将会被忽略而不会有问题。

```
<base href=”[https://www.example.com/](https://www.example.com/)" target=”_blank”>
<a href="a-post.html">A post</a>
```

[*https://codepen.io/adrian-legaspi/pen/qBErzEq*](https://codepen.io/adrian-legaspi/pen/qBErzEq)

# 更多输入标签

你知道吗，输入不仅仅是文本*或密码*类型？不，我也不是在说电子邮件输入。要调用它们的能力，只需更改类型属性，浏览器就会完成这项工作。**

## 日期输入

也许最熟悉这种类型的是**【type = date】**，但是我们有更具体的月、周甚至小时的输入。

```
<input type=”date” />
<input type=”datetime-local” />
<input type=”month” />
<input type=”week” />
<input type=”time” />
```

这些输入将为 Date()类返回有效且可读的格式。

## 颜色选择器

老实说，**【type = color】**输入提示的风格在很大程度上取决于你的浏览器，但它是一个快速、简洁的解决方案，可以让用户选择颜色。

```
<input type=”color” />
```

该输入将返回一个十六进制数值。

## 范围

范围输入类型是当你想给用户一个选择一个范围之间的数字的选项时的解决方案。一旦我用它在我自己的视频播放器中创建了一个音量滑块，它就使用了被接受的最小*和最大*值。**【类型=范围】**

```
<input type=”range” />
```

现实世界中的这些输入显示在这里:

[*https://codepen.io/adrian-legaspi/pen/bGNqXdN*](https://codepen.io/adrian-legaspi/pen/bGNqXdN)

# <details></details>

就这么简单，一个可切换的，原生的和可定制的隐藏部分。只需在 **<中点击>** 标签内的**<>**细节就足以显示所有隐藏的内容。顺便说一下，不需要 JavaScript，是的，你可以用 CSS 让它更漂亮。

[*https://codepen.io/NielsVoogt/pen/YbaNPd*](https://codepen.io/NielsVoogt/pen/YbaNPd)

# 文本格式标签

有一些我喜欢的格式化标签，但是它们不足以占据我的条目的整个空间。这些标签并没有应用太多的风格，但是对于连贯和用更好的实践编写 HTML 来说是很重要的，它们在接下来的地方会有很大的用处。

使用 **< span >** 来包围文本然后赋予背景色实际上是一种不好的做法，因为 **<标记>** 的存在，请避免使用 **< span >** 并在需要突出显示文本时开始使用 **<标记>** 。 **<标记>** 默认带有经典荧光笔的黄色，但它对颜色 CSS 属性的响应没有任何问题。

```
<p>
 Lorem ipsum dolor sit amet consectetur adipisicing elit. <mark>Dolores temporibus vitae praesentium quaerat</mark> quidem eaque dignissimos corporis dolorum explicabo? Alias? 
</p>
```

如果你需要在你的文本中使用一个缩写名称，这个标签是坏男孩将接受这项工作。

```
<abbr title=” Do I Eat Today?”>DIET</abbr>
```

Title 属性是一个很好的补充，可以用来获得首字母缩略词的本机工具提示。

预格式化文本或**<pre>**for friends，用于显示书写的文本，带有所有空格、制表符和支持的字符。文本将完全按照其在块中的格式呈现。

```
<pre>
    /* Code */
    var json = {
       a: 1,
       b: 2,
       c: 3
    }

    function example(){
       console.log(“Hello world”);
       return true;
    }

    example();

 </pre>
```

你可以给一些响应度，因为默认情况下，这个元素不关心你的屏幕宽度，但这可能会带走部分 **< pre >** 魔法。

## 

* * *

这不是一个完全格式化的文本，但是在**<>**元素和元素中效果不错，所以我将把它放在这里。用作分隔线，默认情况下，它具有白色背景的对比色，并使用 100%的父宽度。

```
<!-- Simple as this: -->
<hr />
```

改变*边框颜色* CSS 属性很容易。这个标签真的很好。

所有这些文本标签的用法如下所示:

[*https://codepen.io/adrian-legaspi/pen/wvBJLqN*](https://codepen.io/adrian-legaspi/pen/wvBJLqN)

# [内容可编辑]

如果你想创建一个*所见即所得*(所见即所得)的 HTML 编辑器，真的比你想象的要容易。只有将属性[**【content editable】**](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Editable_content)添加到放置编辑器的 div 中，才会自动将其中的所有内容转换为用户可写的。

```
<div contenteditable>
    <p>A lorem text</p>
</div>
```

这比你想象的更强大，因为当一个元素处于设计模式时， [**execCommand**](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) 函数将是可用的，你可以将它们绑定到按钮上，这样用户就可以高亮、粗体、斜体、下划线等。在您的自定义编辑器中。有些编辑器库像[*【quill js】*](https://quilljs.com/)(推荐)、 *TinyMCE* (不推荐用于[【LGPL】](https://en.wikipedia.org/wiki/GNU_Lesser_General_Public_License))甚至中型编辑器，使用**【content editable】**来创建它们的编辑器库，它们并不像某些人可能认为的那样使用大的**<*textarea>****。当需要将内容存储在数据库中时，可能会遇到一些麻烦，需要避免换行符、引号或 HTML 标签，但最终还是值得的。*

*这是 Quilljs 的一支笔，显示他们的编辑，这不是我的，但真的很好:*

*[https://codepen.io/davelozier/pen/KMgNzeT21](https://codepen.io/davelozier/pen/KMgNze)*

# *结论*

*别忘了给 *&红心；对所有标签来说，即使它们不是很受欢迎，它也可能是比你经常使用的标签更好的选择。我希望这篇文章能让你开始搜索另一个有趣的属性，比如**模式**、**隐藏**或**数据-*** 和其他语义元素。**

> *参考:*
> 
> *W3Schools*→*[https://www.w3schools.com/](https://www.w3schools.com/)*
> 
> *MDN 网络文档*→*[https://developer.mozilla.org/](https://developer.mozilla.org/)*
> 
> *我自己*→https://twitter.com/ImLuyou**
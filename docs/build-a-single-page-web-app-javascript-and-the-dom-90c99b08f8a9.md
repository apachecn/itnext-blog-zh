# 使用 DOM 和 JavaScript 构建单页面 Web 应用程序

> 原文：<https://itnext.io/build-a-single-page-web-app-javascript-and-the-dom-90c99b08f8a9?source=collection_archive---------0----------------------->

在这里，您将使用最少的 HTML 创建一个动态 web 应用程序，而不是使用文档对象模型(DOM)和 JavaScript

![](img/63ef509a36cbed24af4a38c96ff89ba2.png)

单页应用程序没有你想象的那么复杂。恰恰相反。

为什么？为什么我不能只写 HTML，CSS，把它们链接在一起，我就有我的网站了？！是的，是的，看起来我把事情做得比需要的更复杂，但是单页应用程序的效率可以在开发动态 web 应用程序时节省时间，尤其是在进行更改时。

解决这个问题有很多角度，最明显的是从浏览器的角度(例如 chrome、Firefox……)。如果你有一个动态的 web 应用程序，它有很多页面，可能在某个地方有重复的代码(比如导航栏、页脚等等)。当您浏览这些页面时，浏览器每次都会刷新页面，重新加载上一页已经加载的内容。这只是浏览器的额外工作，会导致糟糕的用户体验；页面重新加载所浪费的几秒钟可能会促成或破坏对客户的销售。

这就是单页应用程序派上用场的地方。如果您只需要根据用户的操作用新信息更新站点的一部分，那么您可以不重新加载页面，而只需获取数据并将其添加到页面上。我们开始吧。

为你的主页创建一个模板，将首先加载的模板，我们将只在这一个基础上构建。就叫它 index.html 吧

```
<-- in index.html -->
<html>
  <head>
    <meta charset="utf-8">
    <title>Navigation Example</title>
  </head>
  <body>
    <a href="">Home</a>
    <a href="">About</a>
    <a href="">Contact</a>

  </body>
</html>
```

假设你已经知道 HTML，你会知道' a '标签是链接。如果我们把 URL 放在 href 属性中，这些通常会把你引导到网站的另一个页面。

但是由于我们只处理单页应用程序，我们不希望页面重载。我们可以在 URL 中添加一个片段标识符，它将被添加到查询字符串的末尾，而不需要重新加载页面。让我们执行以下操作:

```
<-- in index.html -->
<html>
  <head>
    <meta charset="utf-8">
    <title>Navigation Example</title>
  </head>
  <body>
    <a href="#home">Home</a>
    <a href="#about">About</a>
    <a href="#contact">Contact</a>

  </body>
</html>
```

片段标识符(#home、#about 等)仅用于标识文档的一部分，如锚。启动 index.html 并点击链接。您的 url 中的 URL 将会改变，但页面不会重新加载。

现在是 JavaScript。我们可以使用事件监听器监听这些变化，当它们发生时，我们可以相应地修改页面。

创建一个名为 main.js 的新文件，并使用脚本标签将 index.html 链接到该文件。

```
// in main.js

console.log(location.hash)
```

console.log 将把括号内的值打印到控制台。要查看控制台，请在您的浏览器页面上单击右键，然后单击“检查元素”，尽管它可能因您的浏览器而异。众所周知，Chrome 开发工具是开发者的最爱。

location.hash 是我们之前讨论过的片段标识符。如果我们在查询字符串(url)的末尾添加一个散列，并按 enter 键，控制台将打印出片段标识符。酷，但这没有帮助。让我们尝试以下方法:

```
// This will listen for the fragment identifier change
window.addEventListener("hashchange", function() {
  console.log(location.hash);
});
```

现在，我们有了一个事件监听器，正如其名称所表明的那样——它是一个等待事件的函数！在事件侦听器中，我们等待一个“hashchange”，这意味着每当 URL 中的片段标识符发生变化时，这个函数就会被激活。

如果单击 home 链接，事件监听器将被激活，因为 url 现在有了一个不同的片段标识符。使用 location.hash 获取该值，并将其输出到控制台。尝试一下，看看每次点击链接时，每个片段标识符是如何打印到控制台上的。

现在 console.log 有点无聊，但对于尝试一些东西来说总是好的——但是让我们实际上把它输出到页面上让每个人都看到。首先，让我们回到 index.html，为未来的文本创建一个占位符。

```
<-- in index.html -->

<html>
  <head>
    <meta charset="utf-8">
    <title>Navigation Example</title>
  </head>
  <body>
    <a href="#home">Home</a>
    <a href="#about">About</a>
    <a href="#contact">Contact</a>

    <div id="app"></div>
    <script src="main.js"></script>
  </body>
</html>
```

我添加了一个 id 为 app 的 div 元素。这是空白的，稍后我们会把东西放进去。把它想象成一个空盒子，它现在是空的，但是你可以随时把它装起来，或者把它拆开，装上你喜欢的任何东西。让我们用 JS 打包一些文本

```
// in main.js

window.addEventListener("hashchange", function (){
  var contentDiv = document.getElementById("app");
  contentDiv.innerHTML = location.hash;
});
```

复制代码并进入浏览器。当您单击一个链接时，页面应该在文档上显示片段标识符，而不需要重新加载页面。简单。

我们现在唯一的问题是，当我们进入像 yourdocument.html#about 这样的页面时，事件监听器将无法工作，因为文档已经加载。发现问题？即使有一个片段标识符，我们也会有一个空白页。我们希望页面基于片段标识符加载内容，即使不使用事件监听器。看看下面。

```
// in main.js
function loadContent(){
  var contentDiv = document.getElementById("app");
  contentDiv.innerHTML = location.hash;
}

window.addEventListener("hashchange", function (){
  loadContent();
});

loadContent();
```

好多了。我们有一个名为 loadContent 的新函数，它完成了我们在事件监听器中所做的工作。这很好，因为现在我们可以在文档加载时通过声明函数名 loadContent()来调用它。我们也可以在事件监听器中调用这个函数，以避免重复代码。

看一下代码，看看是否可以修改得更多，特别是在事件监听器中。看一看:

```
// in main.js
function loadContent(){
  var contentDiv = document.getElementById("app");
  contentDiv.innerHTML = location.hash;
}

window.addEventListener("hashchange", loadContent);

loadContent();
```

我们正在取得进展，但我们仍然有一个小问题，没有片段标识符的第一页是空白的，但它应该是 home，对吗？请执行以下操作:

```
// in main.js
function loadContent(){
  var contentDiv = document.getElementById("app");
  contentDiv.innerHTML = location.hash;
}

if(!location.hash) {
  location.hash = "#home";
}

loadContent();

window.addEventListener("hashchange", loadContent)
```

我们现在有一个 if 语句。让我们孤立地来看:

```
if(!location.hash){
    location.hash = "#home";
}
```

这基本上是说‘如果没有位置散列，就把位置散列设为# home。’的！在 location.hash 的前面对表达式求反。

我们还有另一件事要解决:我们不想打印出带有页面的散列符号。我们用一个叫 substr 的 JS 方法把它拿出来。

```
// in main.js
function loadContent(){
  var contentDiv = document.getElementById("app"),

      // This gets rid of the first character of the string
      fragmentId = location.hash.substr(1);

  contentDiv.innerHTML = fragmentId;
}

if(!location.hash) {
  location.hash = "#home";
}

loadContent();

window.addEventListener("hashchange", loadContent)
```

substr 方法删除字符串的第一个字母，并返回其余的字母。这导致从片段标识符中删除#号。

打印出片段标识符很好，但是有点无聊，没有令人兴奋的内容来真正展示你的网站。

```
function getContent(fragmentId){

// lets do some custom content for each page of your website
  var pages = {
    home: "This is the Home page. Welcome to my site.",
    about: "This page will describe what my site is about",
    contact: "Contact me on this page if you have any questions"
  };

// look up what fragment you are searching for in the object
  return pages[fragmentId];
}

function loadContent(){

  var contentDiv = document.getElementById("app"),
      fragmentId = location.hash.substr(1);

  contentDiv.innerHTML = getContent(fragmentId);
}

if(!location.hash) {
  location.hash = "#home";
}

loadContent();

window.addEventListener("hashchange", loadContent)
```

这里我们有一个名为 getContent 的新函数。它需要一个片段的参数。在这个函数内部，我们有一个 JS 对象，它就像一个字典。它由键值对填充，比如一个单词和一个描述。在我们的例子中，是页面名称和与之相关的内容。我已将对象命名为页面。

在函数的底部，我们通过向 partials 对象传递 fragmentId 的值来搜索它(这是我们所在的页面)。然后浏览器遍历 partials 对象并返回匹配的值。我们离动态网站又近了一步。

对于本教程的最后一步，让我们更改 getContent 函数，使其包含一个回调函数，从而使程序中的函数异步运行。如果你现在不明白，不要担心。简而言之，你想让你的程序尽可能高效。我们更喜欢异步(同时)运行函数，尽量减少浪费的时间。这是一个很难理解的复杂概念——所以如果你不能马上理解，不要担心，如果你想了解更多，做一些研究。

```
function getContent(fragmentId, callback){

  var pages = {
    home: "This is the Home page. Welcome to my site.",
    about: "This page will describe what my site is about",
    contact: "Contact me on this page if you have any questions"
  };

  callback(pages[fragmentId]);
}

function loadContent(){

  var contentDiv = document.getElementById("app"),
      fragmentId = location.hash.substr(1);

  getContent(fragmentId, function (content) {
    contentDiv.innerHTML = content;
  });

}

if(!location.hash) {
  location.hash = "#home";
}

loadContent();

window.addEventListener("hashchange", loadContent)
```

回调在 getContent 方法中。当调用 getContent 时，该函数继续完成它的工作，同时获取内容放入 HTML 页面。围绕这个概念，看看你能构建多大的单页应用程序。
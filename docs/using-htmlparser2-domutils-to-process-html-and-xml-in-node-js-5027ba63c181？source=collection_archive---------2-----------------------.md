# 使用 HTMLParser2，DOMUtils 处理 Node.js 中的 HTML 和 XML

> 原文：<https://itnext.io/using-htmlparser2-domutils-to-process-html-and-xml-in-node-js-5027ba63c181?source=collection_archive---------2----------------------->

## Node.js 简化了服务器端 DOM 处理

![](img/059963615d2bd6379cddb8d4e17b51ae.png)

**HTMLParser2 是 Node.js 包(domhandler、domutils、css-select、dom-serializer)集群的一部分，支持对 HTML 和 XML DOM 对象树的强大操作。这些包不仅可以用于 web 抓取，还可以用于服务器端的 DOM 操作，它们构成了 Cheerio 的大部分基础，Cheerio 是 Node.js 包，用于在 Node.js 上进行类似 jQuery 的 DOM 操作。**

似乎大多数人使用 HTMLParser2 及其相关包进行 Web 抓取。这意味着下载一个页面并从该页面的 HTML 中解析出数据。虽然这可以用来创建非常有用的信息资源，但是这些包可以用于许多其他任务，包括 HTML 和 XML 数据的服务器端 DOM 操作。这些包中的每一个都专注于一个特定的目的，当一起使用时，程序员可以获得强大的能力来接收 XML 或 HTML 文档、提取数据和转换这些文档。

不幸的是，这些软件包的文档并不清楚。我写这篇文章的目的是为理解如何使用它们创建一个有用的资源。

我是作为一个静态网站生成器平台 AkashaCMS 的作者来处理这个问题的。AkashaCMS 的一个核心特性是使用 Cheerio 实现服务器端的 DOM 操作，以创建最终的 HTML 显示给访问者。在某些情况下，它会修复 HTML，重写 URL，或者将自定义标签如`<embed-video>`转换成 YouTube 视频播放器。

在使用 Cheerio 时，我没有注意实现。在诊断最新 Cheerio 版本的问题时，我开始感兴趣。作为回应，我想深入了解 Node.js 中的 XML 和 HTML 处理。第一站是探索 HTMLParser2、DOMHandler、DOMUtils、CSS-Select 和 DOM-Serializer。

我还对 Node.js 中服务器端 DOM 操作的选项范围感兴趣。我知道在前端工程中，许多人正在逐步淘汰 jQuery，因为 DOM API 的改进使得 jQuery 变得不那么必要了。我很好奇是否有 Node.js 包用 jQuery API 的简洁性实现了 DOM 操作。

本文的目标包括用 DOMHandler 和 DOMUtils 评估 DOM 操作。

# 如果没有浏览器，DOM 是什么？

我们需要谈谈大教堂的事。DOM 不仅仅是 web 浏览器基于网页内容生成的东西。在正常情况下，对于 web 浏览器中显示的每个网页，浏览器都将其转换为 DOM，然后我们使用 CSS 来设置 DOM 的样式，并使用 JavaScript 来操作它。通过浏览器端的 DOM 操作，可以在 web 浏览器中实现非常高级的应用程序。

在这种情况下，DOM 意味着文档对象模型，它是一个跨平台和独立于语言的接口，将 XML 或 HTML 文档视为一个树结构，其中每个节点都是表示文档一部分的对象。DOM 用一个逻辑树表示一个文档。

换句话说，XML 和 HTML 看起来像文本，但它们实际上是序列化的数据结构。HTML 或 XML 文本是序列化格式。读取这些文件的软件，如 web 浏览器，将文本表示反序列化为 DOM 结构。因此，有可能将 XML/HTML 反序列化为 DOM，操纵 DOM，然后将其序列化回 XML 或 HTML。

DOM 是自 20 世纪 90 年代以来不断发展的标准。实现存在于多种编程语言中。标准 DOM 模型涉及各种类型的节点对象，这些节点对象具有属性，并且包含零个或多个子节点对象。一种类型是元素对象，代表我们在 XML 或 HTML 中熟悉的`<tag>`。

这就是我们前进的方向，使用实现类似于 Node.js 上 DOM 标准的 API 的包。DOM API 从来不局限于浏览器，因为它存在于多种语言中。

# 为 HTMLParser2、DOMHandler、DOMUtils、CSS-Select 和 DOM-Serializer 设置 Node.js 项目

为了探究这些包，让我们用几个示例脚本建立一个简单的项目。在此之前，您当然必须在您的计算机上安装 Node.js。我目前运行的是 Node.js 16.13.0，但我相信它可以在 14.x 上运行。

```
$ npm init -y
Wrote to /Volumes/Extra/ws/techsparx.com/projects/node.js/htmlparser2/package.json:

{
  "name": "htmlparser2",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "David Herron <david@davidherron.com>",
  "license": "ISC"
}

$ npm install htmlparser2 domhandler domutils css-select dom-serializer --save

added 11 packages, and audited 12 packages in 3s

10 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

我将目录命名为`htmlparser2`，因此`npm init -y`导致`package.json`将项目命名为`htmlparser`。否则，这给了我们一个包含这一组包的空白项目。

另一项任务是获取一个或多个您可以使用的 HTML 文件。我使用的文件来自 AkashaRender 测试套件之一，因此它有一些自定义标记。

```
<!doctype html> 
<!-- paulirish.com/2008/conditional-stylesheets-vs-css-hacks-answer-neither/ --> 
<!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7" lang="en"> <![endif]--> 
<!--[if IE 7]>    <html class="no-js lt-ie9 lt-ie8" lang="en"> <![endif]--> 
<!--[if IE 8]>    <html class="no-js lt-ie9" lang="en"> <![endif]--> 
<!-- Consider adding a manifest.appcache: h5bp.com/d/Offline --> 
<!--[if gt IE 8]><!--> <html class="no-js" lang="en"> <!--<![endif]--> 
<head> 
<meta charset="utf-8" />
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" /> 
<meta name="viewport" 
      content="width=device-width, initial-scale=1.0"/> 
<title>Show Content</title> 
<meta name="foo" content="bar"/> 
<funky-bump></funky-bump> 
<ak-stylesheets></ak-stylesheets> 
<ak-headerJavaScript></ak-headerJavaScript> 
<rss-header-meta href="/rss-for-header.xml"></rss-header-meta> 
<external-stylesheet href="http://external.site/foo.css"></external-stylesheet> 
<dns-prefetch control="we must have control"    
              dnslist="foo1.com,foo2.com,foo3.com"></dns-prefetch> 
<site-verification google="We are good"></site-verification> 
<xml-sitemap></xml-sitemap> 
<xml-sitemap href="/foo-bar-sitemap.xml"
             title="Foo Bar Sitemap"></xml-sitemap> 
</head> 
<body> 
<h1>Show Content</h1> 
<section id="teaser"><ak-teaser></ak-teaser></section> 
<article id="original">
     <div class="article-head"><h2>Article title</h2></div>
     <show-content id="simple" 
                   href="/shown-content.html"></show-content>
     show-content id="dest" dest="http://dest.url" 
                  href="/shown-content.html"></show-content>
     <show-content id="template"
              template="ak_show-content-card.html.ejs"
              href="/shown-content.html"
              content-image="/imgz/shown-content-image.jpg">
     Caption text
     </show-content>
     <p><show-content id="template2"
              template="ak_show-content-card.html.ejs"
              href="/shown-content.html"
             dest="http://dest.url"
             content-image="/imgz/shown-content-image.jpg">
     Caption text
     </show-content></p>
</article>
<ak-footerJavaScript></ak-footerJavaScript> 
</body> 
</html>
```

你可以随意使用这个文件，或者任何你感兴趣的 HTML 文件。我自己当然对将这些定制标签转换成底层 HTML 的能力感兴趣。但这并不是唯一可能的应用，因为服务器端 DOM 操作的可能性是无限的。例如，可以在服务器上生成 SVG 文件，以便在浏览器中显示。

# 解析 HTML 文件，将其序列化为 HTML

让我们从一个简单的例子开始，即将 HTML 读入 DOM 树，然后立即将其序列化为 HTML。换句话说，在我们开始跑步之前，我们需要学习爬行。

```
import { default as htmlparser2, Parser } from "htmlparser2"; 
import { default as render } from "dom-serializer"; 
import { default as fs, promises as fsp } from 'fs';  
const rawHtml = await fsp.readFile(process.argv[2], 'utf8');  
const dom = htmlparser2.parseDocument(rawHtml);  
console.log(dom);  
const serilzd = render(dom);  
console.log(serilzd);
```

这是用 ES6 模块格式写的。您可能会对这里的关键字`await`感到困惑，但是从 Node.js 14.x 开始，在 ES6 模块的顶层使用 await 成为可能。要了解更多信息，请参见: [Node.js 脚本编写者:顶级异步/等待现已推出](https://techsparx.com/nodejs/async/top-level-async.html)

我们将文件读入内存，然后使用`parseDocument`方法将其直接解析成 DOM 结构。

`htmlparser2`包是一个 SAX 风格的解析器，这意味着它会发出事件，记录它在传入文本中找到的语法元素。那些事件不是 DOM 对象树。相反，`domhandler`包使用这些事件来产生一个 DOM 对象树。因此，`parseDocument`方法必须在后台实例化`domhandler`来实现。

接下来是`render`功能。这需要一个由`domhandler`生成的 DOM 树，并将其序列化为 HTML

不幸的是，当我们运行这个脚本时，我们得到了一个错误:`TypeError: render is not a function`。也就是说，虽然这个包是在 TypeScript 中实现的，但它明确针对 CommonJS 环境，在 ES6 上不能很好地工作。

因此我们必须把这个例子改写成这样:

```
const htmlparser2 = require('htmlparser2'); 
const render = require('dom-serializer').default; 
const fs = require('fs'); 
const fsp = require('fs').promises; (async () => {
      const rawHtml = await fsp.readFile(process.argv[2], 'utf8');
      const dom = htmlparser2.parseDocument(rawHtml);
      console.log(dom);
      const serilzd = render(dom);
      console.log(serilzd);
})().catch(err => {
    console.error(err);
});
```

这是相同的脚本，但是使用了 CommonJS 模块语法。因为我们不能使用顶级的`await`，我们必须实现一个`async`函数来运行脚本。

让我们确保您理解这些示例中使用的代码结构。它从一个匿名箭头函数开始:

```
async () => { ... }
```

围绕它的是一个函数调用:

```
(async () => { ... })()
```

匿名函数在括号内被实例化，然后立即被调用。如果你想传入参数，应该是这样的:

```
(async (fileName) => {
    const rawHtml = await fsp.readFile(fileName, 'utf8');
    ...
})(process.argv[2])
```

这个程序本身只是将 HTML 解析成 DOM，然后立即打印出来。您的输出应该等同于输入文件。可能会有细微的差别，但是 HTML 的本质允许以多种方式表示同一个数据结构。重要的是输出和输入在语义上是否相同。

# 从 HTML 输入产生 XHTML 输出

如果您的用例是将 HTML 转换成 XHTML，那该怎么办？通过对脚本的微小调整，我们生成了 XHTML。

```
const serilzd = render(dom, {
     xmlMode: true 
});
```

这将序列化更改为 XML 模式，也称为 XHTML。

在这种情况下，你会发现许多变化。例如，`<meta>`标签现在有一个结束斜杠，使它们成为`<meta/>`，而有结束标记(`<tag></tag>`)的标签现在是一个单独的标签(`<tag/>`)。

# 使用 CSS-Select 查找 DOM 元素

一个典型的任务是使用“选择器”在 DOM 中搜索一个项目，以提取一些数据或操作 DOM。在 HTMLParser2 世界中，`css-select`包实现了一个从 CSS4 和 jQuery 派生的选择器语法。

它不提供任何 DOM 操作，只提供基于选择器选择 DOM 节点的能力。

```
// Demonstrate CSS selectors to extract data from XML or HTML  
const htmlparser2 = require('htmlparser2'); 
const render = require('dom-serializer').default; 
const CSSselect = require("css-select"); 
const fs = require('fs'); 
const fsp = require('fs').promises; 
const util = require('util'); (async () => {
      const rawHtml = await fsp.readFile(process.argv[2], 'utf8');
      const dom = htmlparser2.parseDocument(rawHtml);
      for (let h1 of CSSselect.selectAll('h1', dom)) {
         console.log(`h1 ${render(h1)}`);
      }
      for (let articleHead of 
             CSSselect.selectAll('article .article-head', dom)) {
         console.log(`articleHead ${render(articleHead)}`);
      }
      for (let articleHead of 
             CSSselect.selectAll('article .article-head h1,h2,h3,h4,h5,h6', dom)) {
         console.log(`articleHead Hn ${render(articleHead)}`);
      }
      console.log(CSSselect.selectAll(
                      'article .article-head', dom));  
})().catch(err => {
     console.error(err);
});
```

这个例子显示了使用`CSSselect.selectAll`选择所有匹配选择器的元素，然后打印所选元素的 HTML。最后一个用法打印原始的 DOM 数据结构，这样我们可以熟悉一下`domhandler`生成的 DOM 数据结构。

```
$ node css-selector.js example1.html 
h1 <h1>Show Content</h1>
articleHead <div class="article-head"><h2>Article title</h2></div>
articleHead Hn <h2>Article title</h2>
[
  <ref *1> Element {
    type: 'tag',
    parent: Element {
      type: 'tag',
      parent: [Element],
      prev: [Text],
      next: [Text],
      startIndex: null,
      endIndex: null,
      children: [Array],
      name: 'article',
      attribs: [Object]
    },
    prev: Text {
      type: 'text',
      parent: [Element],
      prev: null,
      next: [Circular *1],
      startIndex: null,
      endIndex: null,
      data: '\n    '
    },
    next: Text {
      type: 'text',
      parent: [Element],
      prev: [Circular *1],
      next: [Element],
      startIndex: null,
      endIndex: null,
      data: '\n    '
    },
    startIndex: null,
    endIndex: null,
    children: [ [Element] ],
    name: 'div',
    attribs: { class: 'article-head' }
  }
]
```

输出中的前三行显示了选择器找到的 HTML。我们通过使用`render`函数运行所选的 DOM 子树，得到该子树的 HTML 代码片段。

对 DOM 中选择的元素使用`render`将序列化所选元素下的 DOM 节点。

最后一个显示了从这个方法返回的实际 DOM 数据。因为它是`selectAll`，所以它返回一个匹配的数组。这个数组有一个对象，一个`Element`实例。它有类型`tag`，标签名为`div`，有一个包含`article-head`的`class`属性的`attribs`数组，所有这些都与文档中的 HTML 相匹配。`children`元素是一个包含该元素的 DOM 对象数组。有`parent`、`prev`和`next`对象，这样任何接收到这个 DOM 对象的人都可以遍历树。

回头参考我们实现的代码。虽然循环结构相当简单，但不如等价的 jQuery 代码简洁。但是，请注意，`selectAll`方法返回一个数组。这意味着该示例的内部部分可以这样实现:

```
CSSselect.selectAll('h1', dom).forEach(h1 => {
     console.log(`h1 ${render(h1)}`); 
});
CSSselect.selectAll('article .article-head', dom) 
                  .forEach(articleHead => {
     console.log(`articleHead ${render(articleHead)}`); 
});
CSSselect.selectAll('article .article-head h1,h2,h3,h4,h5,h6', dom) 
                  .forEach(articleHead => {
     console.log(`articleHead Hn ${render(articleHead)}`); 
});
```

这使用了`Array.forEach`方法，更接近于等效的 jQuery 代码。这意味着我们可以使用其他操作，如`Array.map`或`Array.filter`方法。

# 使用 DOMUtils 操作 DOM

接下来，让我们做一点 DOM 操作。在这个 HTML 文档中，你可以看到几个定制的 HTML 标签。让我们实现代码来将定制标签转换成正确的 HTML。

为此，我们将使用 CSS-Select 来选择要处理的 DOM 元素，然后使用 DOMUtils 包中的函数来处理这些元素。

```
const htmlparser2 = require('htmlparser2'); 
const domhandler = require('domhandler'); 
const domutils = require('domutils'); 
const render = require('dom-serializer').default; 
const CSSselect = require("css-select"); 
const fs = require('fs'); 
const fsp = require('fs').promises; 
const util = require('util'); (async () => {
      const rawHtml = await fsp.readFile(process.argv[2], 'utf8');
      const dom = htmlparser2.parseDocument(rawHtml); for (let fb of CSSselect.selectAll('funky-bump', dom)) {
         domutils.removeElement(fb);
      }
      for (let sm of CSSselect.selectAll('xml-sitemap', dom)) {
         // console.log(sm);
         if (sm.attribs.href) {
             const template = 
               '<link rel="sitemap" type="application/xml" href=""/>';
             const link = htmlparser2.parseDocument(template);
             const links = CSSselect.selectAll('link', link);
             links[0].attribs.href = sm.attribs.href;
             // console.log(`sitemap link ${render(link)}`);
             domutils.replaceElement(sm, link);
         } else {
             domutils.removeElement(sm);
         }
      }
      const serilzd = render(dom);
      console.log(serilzd);
})().catch(err => {
      console.error(err); 
});
```

第一个循环寻找`<funky-bump>`元素，并简单地删除标签。我添加这个标签是为了调试目的，它不需要显示在生产网站中。

问问自己，将 URL 插入 DOM 元素的`href`属性，然后再插入页面的 DOM，最安全的方法是什么？请记住，HTML 不是文本格式，而是表示为文本的数据结构。我的信念是，最好将 HTML 作为一种数据结构来操作，而不是通过文本替换。

如果您使用 JavaScript 模板字符串，难道没有一系列可能的脚本注入攻击吗？我的意思是:

```
const replacement = `<link rel="sitemap" type="application/xml" href="${sm.attribs.href}"/>`;
```

虽然这要简单得多，但它不是容易注入恶意 URL 吗？

在这段代码的另一个分支中，如果没有提供`href`，我们就使用`removeElement`。

```
$ node manipulate.js example1.html 
<!doctype html>
     ... 
<head> 
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"> 
<meta name="viewport" 
      content="width=device-width, initial-scale=1.0"> 
<title>Show Content</title> 
<meta name="foo" content="bar">  
<ak-stylesheets></ak-stylesheets> 
<ak-headerjavascript></ak-headerjavascript> 
<rss-header-meta href="/rss-for-header.xml"></rss-header-meta> 
<external-stylesheet 
          href="http://external.site/foo.css"></external-stylesheet> 
<dns-prefetch control="we must have control"       
          dnslist="foo1.com,foo2.com,foo3.com"></dns-prefetch> 
<site-verification google="We are good"></site-verification>  
<link rel="sitemap" type="application/xml" href="/foo-bar-sitemap.xml"> 
</head> 
<body>
     ... 
</body>
</html>
```

而且，这就是结果。与任何减速带一样，`<funky-bump>`元素消失了。`xml-sitemap`元素已被正确的`<link>`标签替换。

还有更多的标签可以实现 DOM 操作，但是我们会看到如何进行。对于每个元素，我们处理提供的属性，然后使用`replaceElement`用正确的 HTML 元素进行替换。

# 在 Node.js 上使用 Cheerio 进行 DOM 操作

DOMUtils 让我们可以操纵 DOM。但是，让我们看看它是如何与啦啦队相抗衡的。

安装:

```
$ npm install cheerio --save
```

在撰写本文时，安装了`cheerio@1.0.0-rc.10`。顾名思义，Cheerio 团队还是觉得不配叫`1.0`。

```
const cheerio = require('cheerio');
const fs = require('fs');
const fsp = require('fs').promises;
const util = require('util');

(async () => {

    const rawHtml = await fsp.readFile(process.argv[2], 'utf8');

    // const dom = htmlparser2.parseDocument(rawHtml);
    const $ = cheerio.load(rawHtml, {
        _useHtmlParser2: true
    });
    $('funky-bump').remove();
    for (let sm of $('xml-sitemap')) {
        if (!$(sm).attr('href')) $(sm).remove();
        else {
            let $template = cheerio.load(
                    '<link rel="sitemap" type="application/xml" href=""/>',
                    null, false);
            $template('link').attr('href', $(sm).attr('href'));
            $(sm).replaceWith($template.html());
        }
    }

    console.log($.html());
    // console.log($.root().html());
})().catch(err => {
    console.error(err);
});
```

这实现了与前一个例子相同的操作。由于类似 jQuery 的 API，代码更加简洁。

因为我的示例 HTML 使用了非标准的定制 HTML 标记，所以使用 Cheerio 的默认设置存在问题。默认情况下，它使用 HTML 的`parser5`包。该模式的默认设置是将`<head>`中的定制标签移动到`<body>`中。您可以通过注释掉`_useHtmlParser2`选项来看到这一点。顾名思义，未记录的选项强制使用`htmlparser2`，否则将使用`parser5`。

使用`htmlparser2`的方法是注释掉的代码行。但是，该版本会出现不可理解的错误消息。使用这个选项(通过仔细阅读源代码找到的)可以工作，产生与上一节相同的输出。

当创建`$template`变量时，我们再次使用了`cheerio.load`，就像我们在上一节中所做的一样。这个方法解析 HTML 并生成一个适合 Cheerio 使用的 DOM 树。`null`和`false`选项对于确保它被视为 HTML 片段是必要的。否则生成的 DOM 会被`<html><body></body></html>`包装，这会导致不必要的行为。

在 Cheerio 中，`.html()`方法将 DOM 序列化为文本。记录了`root`方法来访问文档的根。因此，使用第二种(注释掉的)方法将 DOM 序列化为文本似乎更正确，但是在这个例子中，`$.html()`和`$.root().html()`产生了相同的结果。

# 处理 RSS 源

HTMLParser2 包包含用于解析 RSS 或 Atom 提要的内置模式。让我们用 RSS 提要来旋转一下。

让我们从获取提要开始。

```
$ wget [https://akashacms.com/news/rss.xml](https://akashacms.com/news/rss.xml)
```

有一整个世界的 RSS 源，你不必选择这个。

```
const htmlparser2 = require('htmlparser2'); 
const fs = require('fs'); 
const fsp = require('fs').promises; (async () => {
      const rawRSS = await fsp.readFile(process.argv[2], 'utf8');
      const feed = htmlparser2.parseFeed(rawRSS, {});
      console.log(feed);
})().catch(err => {
      console.error(err); 
});
```

Node.js 提供了大量的 RSS 和 Atom 解析包，这很容易使用。

```
$ node feed1.js rss.xml
{
  type: 'rss',
  id: '',
  items: [
    {
      media: [],
      id: 'https://akashacms.com/news/2021/06/stacked-dirs.html',
      title: '<![CDATA[Stacked Directories - A directory/file watcher for static website generators]]>'
    },
    {
      media: [],
      id: 'https://akashacms.com/news/2021/05/gridjs.html',
      title: '<![CDATA[Using GridJS for fancy searchable HTML tables on statically generated websites]]>'
    },
   ...
  ]
}
```

这很简单，但是你可能会问`<![CDATA[...]]>`是什么。这是一个叫做 CDATA 的 HTML 结构，它很大程度上是透明的。它恰好在 AkashaCMS 生成的 RSS 提要中:

```
<title><![CDATA[Stacked Directories - A directory/file watcher for static website generators]]></title>
```

如果我们添加一些选项，像这样:

```
const feed = htmlparser2.parseFeed(rawRSS, {
  recognizeCDATA: true,
  decodeEntities: true,
  recognizeSelfClosing: true 
});
```

那么输出会提高:

```
items:
  [
    {
      media: [],
      id: 'https://akashacms.com/news/2021/06/stacked-dirs.html',
      title: '<![CDATA[Stacked Directories - A directory/file watcher for static website generators]]>',
      description: "It's very convenient when a ...." 
    },
    {
      media: [],
      id: 'https://akashacms.com/news/2021/05/gridjs.html',
      title: '<![CDATA[Using GridJS for fancy searchable HTML tables on statically generated websites]]>',
      description: "There's a wide variety of ...." 
    }
    ...
  ]
```

描述现在显示出来了，它没有 CDATA 标记。但是 CDATA 结构仍然存在。这可以通过手动编辑 XML 从 RSS 提要中删除 CDATA 结构来从输出中删除。

HTMLParser2 文档实际上建议使用其他 RSS 提要处理器包。但是，这个例子让我们体验了使用 HTMLParser2 不仅操作 HTML，还操作 XML 文件。

# XML 文件处理

在上一节中，我们使用`htmlparser2`中的专用函数处理了一个 XML 文件，特别是一个 RSS 提要。但是，这个包可以用于一般的 XML 操作。

让我们从一个简单的 XML 文件开始:

```
<data>
     <hello>World</hello>
     <table>
         <row id="1" brand="Tesla" mode="electric"/>
         <row id="2" brand="Dodge" mode="gas guzzler"/>
         <row id="3" brand="Ford"
                     mode="multiple, possibly moving to electric"/>
         <row id="4" brand="GM" 
                     mode="multiple, possibly moving to electric"/>
         <row id="5" brand="VW"
                     mode="forced move to electric after emissions fraud conviction"/>
     </table> 
</data>
```

FWIW，我在世界上的一些工作包括写关于电动汽车的新闻文章。

首先，让我们复制第一个读取文件并立即序列化它的示例:

```
const htmlparser2 = require('htmlparser2'); 
const render = require('dom-serializer').default; 
const fs = require('fs'); 
const fsp = require('fs').promises; 
const util = require('util'); (async () => {
      const rawHtml = await fsp.readFile(process.argv[2], 'utf8');
      const dom = htmlparser2.parseDocument(rawHtml, {
         xml: true,
         recognizeSelfClosing: true
      });
      const serilzd = render(dom);
      console.log(serilzd);
})().catch(err => {
      console.error(err); 
});
```

这个例子是这样运行的:

```
$ node xml-simple.js data.xml
  <data>
     <hello>World</hello>
     <table>
         <row id="1" brand="Tesla" mode="electric"></row>
         <row id="2" brand="Dodge" mode="gas guzzler"></row>
         <row id="3" brand="Ford" mode="multiple, possibly moving to electric"></row>
         <row id="4" brand="GM" mode="multiple, possibly moving to electric"></row>
         <row id="5" brand="VW" mode="forced move to electric after emissions fraud conviction"></row>
     </table> 
</data>
```

如你所见，输出等于输入。例如，自结束标签被转换成`<row></row>`形式。但是，语义上是一样的。

区别在于传递给`parseDocument`的两个选项启用了 XML 模式和识别自结束标签。

接下来，让我们使用带有 XML 的 CSS-Select 包:

```
const htmlparser2 = require('htmlparser2'); 
const render = require('dom-serializer').default; 
const CSSselect = require("css-select"); 
const fs = require('fs'); 
const fsp = require('fs').promises; 
const util = require('util'); (async () => {
      const rawHtml = await fsp.readFile(process.argv[2], 'utf8');
      const dom = htmlparser2.parseDocument(rawHtml, {
         xml: true,
         recognizeSelfClosing: true
      });
      for (let row of CSSselect.selectAll('row', dom)) {
         console.log(`${row.attribs.brand} - ${row.attribs.mode}`);
      }
})().catch(err => {
      console.error(err); 
});
```

运行方式如下:

```
$ node xml-table.js data.xml 
Tesla - electric 
Dodge - gas guzzler 
Ford - multiple, possibly moving to electric 
GM - multiple, possibly moving to electric 
VW - forced move to electric after emissions fraud conviction
```

我们能够轻松地从 XML 文件中提取数据。

最后，让我们尝试一点 DOM 操作。我们将删除`<hello>`标签，然后给每个`<row>`添加一个属性。

这个程序是这样运行的:

```
const htmlparser2 = require('htmlparser2'); 
const render = require('dom-serializer').default; 
const domutils = require('domutils'); 
const CSSselect = require("css-select"); 
const fs = require('fs'); 
const fsp = require('fs').promises; 
const util = require('util'); (async () => {
      const rawHtml = await fsp.readFile(process.argv[2], 'utf8');
      const dom = htmlparser2.parseDocument(rawHtml, {
         xml: true,
         recognizeSelfClosing: true
      });
      for (let row of CSSselect.selectAll('row', dom)) {
         row.attribs.seen = "yes";
      }
      for (let hello of CSSselect.selectAll('hello', dom)) {
         domutils.removeElement(hello);
      }
      const serilzd = render(dom);
      console.log(serilzd);
})().catch(err => {
      console.error(err); 
});
```

这个例子使用了我们前面讨论过的相同的 DOMUtils 函数。我们看到，我们用来修改 HTML DOM 的工具也适用于 XML DOM。

# 摘要

有很多 Node.js 包可以用来处理 HTML、XML 甚至 RSS 提要。我们在这里所做的是探索这些包的一个集群。这为我们使用这些包读取 XML/HTML 文件、提取数据或操纵它们的结构提供了一个良好的起点和基础。

但是关于软件包提供的 API 是否比 jQuery 和 Cheerio 更容易使用的问题呢？

使用 DOMUtils 和 CSS-Select 不如使用 jQuery API 简洁。但是这已经很接近了，尤其是在与普通的 JavaScript 编程特性相结合的情况下。可以说，这样访问和更改属性更容易，因为它使用普通的 JavaScript 对象访问和赋值操作符，而不是像在 jQuery 中那样使用`attr`函数。

我们没有做全面的比较来看 Cheerio/jQuery 或者 DOMUtils 等人是否提供了更多的 API 函数。我们可以看到，这些包提供了或多或少相同的功能，优势是更接近 DOM API 标准，并且能够用普通的 JavaScript 代码直接操作 DOM 对象。

# 链接

**HTMLParser2**

*   [https://www.npmjs.com/package/htmlparser2](https://www.npmjs.com/package/htmlparser2)
*   [https://feedic.com/htmlparser2/](https://feedic.com/htmlparser2/)

**DOMHandler**

*   【https://www.npmjs.com/package/domhandler 
*   [https://github.com/fb55/domhandler](https://github.com/fb55/domhandler)

**DOMUtils**

*   [https://github.com/fb55/domutils](https://github.com/fb55/domutils)
*   [https://www.npmjs.com/package/domutils](https://www.npmjs.com/package/domutils)
*   [https://domutils.js.org/](https://domutils.js.org/)

**CSS 选择**

*   [https://www.npmjs.com/package/css-select](https://www.npmjs.com/package/css-select)
*   [https://github.com/fb55/css-select](https://github.com/fb55/css-select)
*   [https://feedic.com/css-select/](https://feedic.com/css-select/)

**DOM 序列化器**

*   [https://www.npmjs.com/package/dom-serializer](https://www.npmjs.com/package/dom-serializer)
*   [https://github.com/cheeriojs/dom-serializer](https://github.com/cheeriojs/dom-serializer)

# 关于作者

[***大卫·赫伦***](https://davidherron.com) *:大卫·赫伦是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。了解更多请访问*[](https://davidherron.com)

*【https://techsparx.com】最初发表于[](https://techsparx.com/nodejs/web/htmlparser2.html)**。***
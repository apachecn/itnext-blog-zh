# 您的单页应用程序现在是一个多填充

> 原文：<https://itnext.io/your-single-page-app-is-now-a-polyfill-7881fb01694e?source=collection_archive---------0----------------------->

你根本无法击败平台

![](img/38843e5fa67728d349af24a498266509.png)

照片由 [C M](https://unsplash.com/@ubahnverleih?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

我们为什么要构建单页应用程序？两个主要原因。

我们希望我们的网络应用程序感觉“即时”，页面之间没有任何丑陋的空白屏幕，提醒我们我们的应用程序并不真正像一个应用程序。

空白屏幕会导致糟糕的用户体验。当用户点击一个链接或按钮时，他们不希望等待来自服务器的内容。他们希望网站像本地应用一样快。

因此，我们构建了单页面应用程序，其中只有页面中发生变化的内容会被替换，避免了整个页面的重新加载，因此导航到另一个页面的感觉是即时的。

这样做的另一个好处是，现在我们只需要从服务器获取更改的内容，而不是一个全新的页面。这减少了我们需要从网络获取的数据量，使我们的应用程序更快。这是我们构建单页应用的第二个主要原因。

但是单页应用程序带来了额外的复杂性。

我们现在绕过浏览器的路由，而是在客户端自己处理。大多数情况下，还会添加一个前端框架来处理这些页面的呈现，这进一步增加了复杂性。

现在，框架当然可以做更多的事情，但这一切都是从消除页面之间的空白屏幕和减少有效载荷大小的愿望开始的。

如果我告诉你，你也可以拥有一个超快的多页面应用程序，页面之间没有任何空白屏幕，会怎么样？

一个不需要任何客户端路由的多页面应用程序，其中每个新页面都是一个完整的页面重新加载，但只从服务器获取更改的内容。

# 流式 HTML

让多页面应用程序快速运行的诀窍其实很简单:我们利用浏览器的流式 HTML 解析器。

事情是这样的，浏览器在下载的同时呈现 HTML *。它不需要等待整个响应的到达，但它可以在内容可用时立即开始呈现内容。*

由`fetch`返回的`Response`对象在其`body`属性中公开了响应内容的`ReadableStream`,因此我们可以访问它并开始传输响应:

```
fetch('/some/url')
.then(response => response.body)
.then(body => {
  const reader = body.getReader(); // we can now read the stream!}
```

典型的单页面应用程序使用应用程序外壳，它实际上是内容注入的单个页面。它通常由页眉、页脚和放置每页内容的内容区域组成。

问题是，任何在加载后添加到 HTML 页面*的内容都会绕过*流 HTML 解析器，因此呈现速度较慢。

然而，我们可以从浏览器流中受益，让服务人员获取我们需要的所有内容，并让它将所有内容传输到浏览器。

# 客户端上的服务器端呈现

为了实现这一点，我们需要将所有页面分成一个页眉和一个页脚，缓存这些模板，然后从网络上获取正文内容(如果需要的话)。

服务工作者将拦截任何传出的请求，获取页眉和页脚，然后确定需要获取哪个主体内容。这可以是一个简单的 HTML 模板，也可以是一个模板和一些从网络上获取的数据的组合。

然后，服务人员将这些部分组合成一个完整的 HTML 页面，并将其返回给浏览器。这就像服务器端的渲染，但都是在客户端以流的方式完成的，使用的是`ReadableStream`。

这意味着它可以在内容和页脚还在下载的时候就开始渲染页面的页眉，从而带来巨大的性能优势。

让我们看一下代码，特别是每当服务工作者拦截到一个传出请求时就调用的`fetch`事件处理程序:

```
const fetchHandler = async e => {
  const {request} = e;
  const {url, method} = request;
  const {pathname} = new URL(url);
  const routeMatch = routes.find(({url}) => url === pathname);

  if(routeMatch) {
    e.respondWith(***getStreamedHtmlResponse***(url, routeMatch));
  }
  else {
    e.respondWith(
      caches.match(request)
      .then(response => response ? response : fetch(request))
    );
  }
};self.addEventListener('fetch', fetchHandler);
```

`fetchHandler`函数检查传入的请求，并试图通过请求的`url`在`routes`数组中找到匹配的路由:

```
const routes = [
  {
    url: '/',
    template: '/src/templates/home.html',
    script: '/src/templates/home.js.html'
  } ...
];
```

对于 home route(“`/`”)，它将在`home.js.html`中的`script`标签内找到`home.html`模板和附带的 JavaScript。

然后，服务人员将获取模板`header.html`和`footer.html`，将它们与`home.html`和`home.js.html`组合成一个完整的 HTML 页面，并将其流回浏览器。

在前面的例子中，这是在`getStreamedHtmlResponse`函数中处理的。让我们来看看:

```
const getStreamedHtmlResponse = (url, routeMatch) => {
  const stream = new ***ReadableStream***({
    async start(controller) {
      const pushToStream = stream => {
        const reader = stream.getReader();

        return reader.read().then(function process(result) {
          if(result.done) {
            return;
          }
          controller.enqueue(result.value);
          return reader.read().then(process);
        });
      };

      const [header, footer, content, script] = await Promise.all(
        [
          caches.match('/src/templates/header.html'),
          caches.match('/src/templates/footer.html'),
          caches.match(routeMatch.template),
          caches.match(routeMatch.script)
        ]
      );

      await pushToStream(header.body);
      await pushToStream(content.body);
      await pushToStream(footer.body);
      await pushToStream(script.body);

      controller.close();
    }
  }); // here we return the response whose body is the stream
  ***return new Response(stream, {
    headers: {'Content-Type': 'text/html; charset=utf-8'}
  });***
};
```

在`getStreamedHtmlResponse`内部，我们构造了一个新的`ReadableStream`，它被传递了一个`underlyingSource`对象，包含了在流被构造之后立即被调用的`start`方法。

向`start`传递一个`controller`参数，该参数是一个`ReadableStreamDefaultController`，允许控制`ReadableStream`的内部状态和队列。

在`start`方法中，我们获取 HTML 页面的模板，并使用`pushToStream`函数将模板的内容作为单独的流推入主流。

这个函数从模板中逐块读取单个流，并使用`controller.enqueue()`将它们排队。

因为`start`函数是异步的，所以会立即返回一个新的`Response`，并将`ReadableStream`作为响应的主体。

浏览器现在可以传输响应，页面几乎立即出现在屏幕上。

让我们暂时记住这一点:我们现在能够即时提供回复，就像单页应用程序一样，但是没有单页应用程序带来的任何复杂性。

没有客户端路由，没有框架，也没有复杂的服务器端渲染。

所有渲染都由服务人员处理，他们提供极快的流响应。

# 单页应用程序是多种多样的

这基本上将单页应用简化为多页，这是一个相当大胆的声明，但原因如下:

*   这款多页面应用程序的渲染速度将与单页面应用程序一样快，甚至在页面大小增加时会更快，因为我们使用了浏览器的流式 HTML 解析器。单页应用程序*绕过*流解析器，而*未能*利用它。
*   就像在单页应用程序中一样，只从网络中获取发生变化的内容。但是因为服务工作者缓存所有资产并在本地为它们提供服务，所以网络流量被限制在绝对最低限度。
*   应用程序的复杂性大大降低。不再需要客户端路由，也不需要框架来呈现页面。服务工作者负责所有的渲染，它在自己的线程中运行，与主 UI 线程分开。
*   服务器端渲染是免费的，只需将单独的模板拼接在一起，就像其他 HTML 页面一样提供服务。当服务工作者还没有控制页面时，这些将在第一次呈现时被提供。在随后的呈现中，服务工作者将无缝地提供缓存的页面，因为不需要考虑客户端路由。

# 真的管用吗？

现在你可能会想，在速度和性能方面，这样的多页面应用程序是否真的能打败单页面应用程序。

[我制作了一个演示](https://instantmultipageapp.com/),这样你就可以亲眼看到使用流 HTML 的多页面应用程序有多快。你可以在 Github 上的这里找到源代码[。](https://github.com/DannyMoerkerke/instant-multi-page-app)

如果你四处点击，你会注意到页面的标题牢牢地留在原处，即使每个页面都需要重新加载，有些页面相当重。

如果我们使用流式 HTML 解析器，这就是浏览器在呈现 DOM *方面有多好。*

浏览器不慢。DOM 并不慢。我们试图将单页应用程序模型硬塞进一个本质上是多页的介质中，向它扔进框架和大量的库，这种方式使它变得很慢。

# 结论

使用服务人员并正确利用浏览器的流 HTML 解析器可以极大地提高 web 应用程序的性能，并且通常会破坏拥有单页应用程序的目的。

你现在是在*与*平台合作，而不是*对抗*平台。

不要把一个框架和十几个库扔在你的应用上，保持简单，使用平台。

你可能会发现这就是你所需要的。

*你可以* [*在 Twitter 上关注我*](https://twitter.com/dannymoerkerke) *在那里我定期写关于 PWAs、web 组件和现代 Web 功能的文章。*
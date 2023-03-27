# 如何在浏览器中创建 Web 服务器

> 原文：<https://itnext.io/how-to-create-web-server-in-browser-ffaa371d2b53?source=collection_archive---------2----------------------->

你有没有想过能否在浏览器中构建一个 web 服务器？或者更准确地说，创建文件并使用 URL 将它们作为静态资产提供服务。在本文中，我将解释如何做到这一点。

![](img/aa2561c665baa3befbb78d3f4e848399.png)

# 我关于这个问题的故事

当我使用名为 [GIT Web 终端](https://jcubic.github.io/git)的[同构 git](https://isomorphic-git.org/) 创建我的辅助项目时，它也使用了我的另一个开源库 [jQuery 终端](https://terminal.jcubic.pl/)，我想在应用程序内部创建一个 Web 浏览器(使用 iframe ),并允许它在一个新标签页中打开文件。我在想这样的事情是否可能。

# 服务行业人员

然后我意识到，通常用于在 [PWA](https://en.wikipedia.org/wiki/Progressive_web_application) 中缓存文件的[服务器工作器](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)，可以用来动态生成新文件，并使用 fetch 事件将它们发送到浏览器。

# 浏览器

为了使用同构 git，你需要一个库来给你类似文件系统对象的节点。对于我的 GIT Web 终端，我使用了 BrowserFS，这是用于同构 GIT 的主要库，在 [lightning-fs](https://github.com/isomorphic-git/lightning-fs) 之前，创建了专用的同构 git fs 实现。有了浏览器，你可以使用不同的后端，我正在寻找的东西，也将在服务工作者的工作。我找到了最合适的，索引为 DB。

# 解决方案的代码

代码非常简单，我决定使用前缀 **__browserfs__** ，其后的所有内容都是指向我的本地 browserfs 文件的路径，这样真实的文件和 idexedDB 中的文件就不会混淆。

下面是完整的代码:

服务人员允许使用自定义头和自定义 HTTP 响应，这里我使用了 301 重定向和 404 错误，以及一些带有适当内容类型头的基本 HTML。

要初始化服务人员，您可以使用以下代码:

```
**if** ('serviceWorker' **in** navigator) {
  **var** scope **=** location.pathname.replace(/\/[^\/]+$/, '/');
  **if** (**!**scope.match(/__browserfs__/)) {
    navigator.serviceWorker.register('sw.js', {scope})
      .then(**function**(reg) {
         reg.addEventListener('updatefound', function() {
           **var** installingWorker **=** reg.installing;
           console.log('A new service worker is being installed:',
                       installingWorker);
         });
         *// registration worked*
         console.log('Registration succeeded. Scope is ' **+** reg.scope);
      }).**catch**(function(error) {
        *// registration failed*
        console.log('Registration failed with ' **+** error);
      });
  }
}
```

您可以在您的项目中使用这些代码，并且您将在浏览器中拥有自己的 web 服务器。您还可以修改代码并添加新功能。如果你做了一些有趣的东西，请分享结果，并且请不要忘记在文件的顶部包括许可和我的版权(你可以在我的上面或下面包括你自己的版权)。这可能是创建一个好的库的起点，这个库可以做很多很酷的东西。

# 工作演示

要查看工作演示，请访问我的项目 [GIT Web 终端](https://tinyurl.com/git-gaiman)。当你打开那个链接时，它会打开 web 终端，克隆我的另一个项目 repo(我的正在进行的编程语言叫做 Gaiman)并在一个内置的浏览器中查看 HTML 文件。请注意，克隆需要一些时间，所以请耐心等待。

如果您检查 DOM，您将看到有一个 iframe 保存该文件，如果查看源文件，您将:

```
/git/__browserfs__/gaiman/docs/demo/index.html
```

您也可以通过打开编辑器(如 vi 或 emacs)直接在终端中创建文件:

```
vi test.txt
i Hello World :wq
view test.txt
```

如果您键入 vi，编写一些文本并保存文件，您将能够在浏览器中查看它。

**编辑:**这个拥有纯浏览器内 HTTP 请求的技巧被抽象到了[开源库 Wayne](https://github.com/jcubic/wayne) 。

如果你喜欢这个，你可以在 Twitter 上关注我: [@jcubic](https://twitter.com/jcubic) 。
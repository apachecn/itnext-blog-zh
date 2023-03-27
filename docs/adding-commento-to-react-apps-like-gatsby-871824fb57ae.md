# 将 Commento 添加到 React 应用程序(如 Gatsby)

> 原文：<https://itnext.io/adding-commento-to-react-apps-like-gatsby-871824fb57ae?source=collection_archive---------3----------------------->

![](img/fb769f75eca462f0d9cb48019e3eceaf.png)

因为我最近用 Commento 替换了 [Disqus，所以我认为分享如何将一个不同的评论系统嵌入到你的 Gatsby 站点(或一般的 React 应用程序)是一个好主意。](https://nehalist.io/no-more-disqus-hello-commento)

# 零依赖分量

Disqus 有一个[官方包](https://github.com/disqus/disqus-react),用于将它嵌入到 React 应用程序中，我曾经将 Disqus 添加到我的博客中。对于 Commento，我想摆脱额外的依赖；额外的依赖增加了维护，并且总是伴随着一定的风险。这并不是说依赖开源是一个坏主意——只是有时候为小东西添加[一个包只是过度夸张，不值这个价。](https://www.npmjs.com/package/isarray)

所以我自己实现了一个*非常*小的组件，负责用不到 40 行代码嵌入 Commento。

> *由于大多数评论系统都以相同的方式工作(通过在页面中嵌入一个简单的 JavaScript 文件)，这种方法应该适用于其他系统，如*[*Isso*](https://posativ.org/isso/)*、*[*Schnack*](https://schnack.cool/)*[*remark 42*](https://remark42.com/)*等。也是(有小调整)。**

*它是一个功能组件，利用钩子将 Commento 嵌入到所需的页面上。此外，它使用两个助手函数(借用自 [disqus-react](https://github.com/disqus/disqus-react/blob/master/src/utils.js#L3) )来添加和删除页面中的脚本。*

*整个实现相当简单:*

```
*import React, {useEffect} from 'react';// Helper to add scripts to our page
const insertScript = (src, id, parentElement) => {
  const script = window.document.createElement('script');
  script.async = true;
  script.src   = src;
  script.id    = id;
  parentElement.appendChild(script);return script;
};// Helper to remove scripts from our page
const removeScript = (id, parentElement) => {
  const script = window.document.getElementById(id);
  if (script) {
    parentElement.removeChild(script);
  }
};// The actual component
const Commento = ({id}) => {
  useEffect(() => {
    // If there's no window there's nothing to do for us
    if (! window) {
      return;
    }
    const document = window.document;
    // In case our #commento container exists we can add our commento script
    if (document.getElementById('commento')) {
      insertScript(`<your commento url>/js/commento.js`, `commento-script`, document.body);
    }// Cleanup; remove the script from the page
    return () => removeScript(`commento-script`, document.body);
  }, [id]);return <div id={`commento`} />
};export default Commento;*
```

*不要忘记用正确的 URL 替换`<your commento url>`。*

*一旦找到相关联的容器(其 id 等于`commento`)Commento 脚本就会被添加到我们的页面中，并且一旦`id`属性(应该是文章或页面 id)发生变化就会被重新呈现(更多信息，请参见[通过跳过效果优化性能](https://reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects))。*

*我们现在可以通过简单地将`<Commento id={uniquePostId} />`组件添加到我们想要添加评论的地方来为所有页面添加评论。*

**如果你喜欢这篇文章，请留下👏，关注我上* [*推特*](https://twitter.com/nehalist) *并订阅* [*我的快讯*](https://nehalist.io/newsletter/) *。本帖原载于 2019 年 9 月 2 日* [*我的博客*](https://nehalist.io/adding-commento-to-react-apps-like-gatsby) *。**
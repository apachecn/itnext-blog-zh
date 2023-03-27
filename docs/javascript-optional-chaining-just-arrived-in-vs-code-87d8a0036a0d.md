# 在 VS 代码中使用 JavaScript 可选链接的邪恶力量！

> 原文：<https://itnext.io/javascript-optional-chaining-just-arrived-in-vs-code-87d8a0036a0d?source=collection_archive---------4----------------------->

## 一个不那么恐怖的介绍

![](img/b7e648bedae38734ace811fe3005190d.png)

我胳膊和腿上的毛都竖起来了。一滴眼泪顺着我的脸颊滑落。当我低声说出“不可能”时，我的声音颤抖了。我凝视时看到的是某种万圣节的恐惧吗？或许是个食尸鬼？不要！这些都是喜悦的鸡皮疙瘩。胜利的眼泪*。*喜悦的颤抖*。女士们先生们，我说的当然是 VS 代码中的可选链接语法！*

*你为什么要在乎？因为可选的链接撕裂。你值得拥有。*

## *什么是可选链接？*

*你做过这个吗？*

```
*console.log(person && person.location && person.location.city);*
```

*还是这个？*

```
*if (onError) {
  onError(error.message);
}*
```

*像这样对潜在的`undefined`或`null`值进行检查会不必要地弄乱您的代码，并让您的母亲感到紧张。这是你想要的吗？她担心死了。给她打个电话，告诉她你的代码有了可选的链接会变得更加整洁。此功能允许读取位于连接对象链深处的属性值，而不必明确验证链中的每个引用是否有效。*

## *语法是什么？*

*现在，您可以像这样实现上面的示例:*

```
*console.log(person?.location?.city);*
```

*如果属性链中的任何值是`undefined`或`null`，那么`undefined`将被记录到控制台，而不是被抛出`TypeError`！*

```
*onError?.(err.message);*
```

*这是给你的小奖励。*您可以对潜在的未定义函数使用可选链接！*这对于回调特别有用。如果`onError`是`undefined`或者`null`，什么都不会发生！你妈妈会为你骄傲的。*

## *(仅仅算是)坏消息*

*到目前为止，还没有任何本地环境支持这个特性。它在 [TC39 第三阶段](https://github.com/tc39/proposal-optional-chaining)，这意味着它很可能在未来被接受到 JavaScript 中，但是多亏了[这个巴别塔插件](https://babeljs.io/docs/en/babel-plugin-proposal-optional-chaining)，你今天可以使用它。好消息是，如果你安装了 [JavaScript 和 TypeScript Nightly](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next) 扩展，VSCode 现在不会抱怨这个语法了。*

*现在出去写点代码吧，你这个十足的荡妇。❤️*

# *跟我来。*

*如果你喜欢这篇文章，请关注我！或者至少给我一两下掌声。你能省下一点掌声，对吧？*

***网站**:[https://sreisner . github . io](https://sreisner.github.io/) **媒体**:[@ Shawn . web dev](https://medium.com/@shawn.webdev)
**推特** : [@ReisnerShawn](https://twitter.com/ReisnerShawn)*

# *查看我的更多作品！*

*[](/the-most-understated-benefit-of-graphql-95bac3651403) [## GraphQL 最被低估的优势

### 我完全相信 GraphQL 是 API 架构中的下一件大事。看到一项技术是如此令人兴奋…

itnext.io](/the-most-understated-benefit-of-graphql-95bac3651403) [](/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a) [## 这就是为什么映射一个构造的数组在 JavaScript 中不起作用

### 以及如何正确地去做

itnext.io](/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a) [](/understanding-the-react-context-api-through-building-a-shared-snackbar-for-in-app-notifications-6c199446b80c) [## 学习 React Context API，并将其应用到您的应用中

### 为应用内通知构建共享素材 UI Snackbar

itnext.io](/understanding-the-react-context-api-through-building-a-shared-snackbar-for-in-app-notifications-6c199446b80c)*
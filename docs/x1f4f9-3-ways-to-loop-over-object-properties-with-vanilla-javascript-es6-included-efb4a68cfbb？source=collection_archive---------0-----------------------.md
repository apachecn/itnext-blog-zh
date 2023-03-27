# 几秒钟内在 JavaScript 中循环对象属性的 3 种方法

> 原文：<https://itnext.io/x1f4f9-3-ways-to-loop-over-object-properties-with-vanilla-javascript-es6-included-efb4a68cfbb?source=collection_archive---------0----------------------->

## 它永远不会变得容易！

![](img/22074fd801410a83ddc5de78cf9b81d6.png)

罗曼·辛克维奇在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

经常需要用 JavaScript 对象遍历数组！但有时你只是不知道那个物体有什么样的性质。

在这篇文章中，我想向你展示如何用 3 种不同的方法轻松循环对象属性。

# **客体**

首先，我们需要一个示例对象进行循环。所以我把我的一些经历放了进去😉(哈哈哈)！保持其中的乐趣！

```
let experienceObject = {
    name: 'Raymon',
    title: 'Lead Frontend/JavaScript Developer',
    yearsExperience: 8,
}
```

# **1。Object.keys(对象)**

Object.keys()方法将返回一个键数组。如果您将它放入一个变量中，并放入 console.log()中，您将看到一个密钥数组。

```
var objectKeys = Object.keys(experienceObject);
// Result is:  ["name", "title", "yearsExperience"]
```

# **2。Object.entries(object)**

Object.keys()方法将返回一个键数组。如果您将它放入一个变量中，并放入 console.log()中，您将看到一个密钥数组。

```
var objectEntries = Object.entries(experienceObject);
// Result is:  0: [
//   ["name", "Raymon"],
//   ["title", "Lead Frontend/JavaScript Developer"],
//   ["yearsExperience", 8],
// ]
```

# **3。For-in 循环**

最后一个例子是 For-in 循环，它遍历对象的属性。

for-in 循环比普通的 for 循环简单得多。

```
for (property in experienceObject) {
   console.log(`key= ${property} value = ${experienceObject[property]}`)
}
```

# 资源

JavaScript 中循环对象属性的 3 种方法是:

*   Object.keys ( [Mozilla 开发者参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys))
*   Object.entries ( [Mozilla 开发者参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries))
*   For-in 循环( [Mozilla 开发者参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in)

# 结论

感谢您阅读这篇简短但有意义的帖子。我希望你能在日常工作中使用这些方法。

如果你对这些方法有任何疑问，请在评论中告诉我。

> 你想用最简单的方法学习 JavaScript 吗？我正在做一个项目，以一种低门槛的方式教你 JavaScript，这样你就可以构建你自己的交互式 ui 组件👍

# 阅读更多

[](https://medium.com/better-programming/5-reasons-why-you-should-write-technical-blog-posts-as-developer-30cd349ece60) [## 作为开发人员，你应该写技术博客的 5 个理由

### 撰写技术博客文章如何在开发人员职业生涯的早期帮助你

medium.com](https://medium.com/better-programming/5-reasons-why-you-should-write-technical-blog-posts-as-developer-30cd349ece60) [](https://medium.com/better-programming/make-your-javascript-objects-more-predictable-by-creating-maps-20ac1a795442) [## 通过创建地图使您的 JavaScript 对象更加可预测

### 不再有未定义的属性

medium.com](https://medium.com/better-programming/make-your-javascript-objects-more-predictable-by-creating-maps-20ac1a795442) [](https://medium.com/better-programming/7-steps-to-dockerize-your-angular-9-app-with-nginx-915f0f5acac) [## 使用 Nginx 对 Angular 9 应用程序进行分类的 7 个步骤

### 在 Docker 环境中设置 Angular 9 应用程序，并立即进行部署

medium.com](https://medium.com/better-programming/7-steps-to-dockerize-your-angular-9-app-with-nginx-915f0f5acac) [](https://medium.com/better-programming/a-practical-introduction-to-typescript-class-decorators-afb996af0763) [## TypeScript 类装饰器实用介绍

### 使用类装饰器的 TypeScript 中着火的类

medium.com](https://medium.com/better-programming/a-practical-introduction-to-typescript-class-decorators-afb996af0763) [](https://medium.com/better-programming/an-introduction-to-typescript-property-decorators-1c9db23b6ca1) [## TypeScript 属性装饰器简介

### 对 TypeScript 装饰器的深入探究

medium.com](https://medium.com/better-programming/an-introduction-to-typescript-property-decorators-1c9db23b6ca1)
# Vue.js 应用性能优化:第 2 部分——惰性加载路径和供应商捆绑反模式。

> 原文：<https://itnext.io/vue-js-app-performance-optimization-part-2-lazy-loading-routes-and-vendor-bundle-anti-pattern-4a62236e09f9?source=collection_archive---------1----------------------->

**在** [**上一篇文章**](/vue-js-app-performance-optimization-part-1-introduction-to-performance-optimization-and-lazy-29e4ff101019) **中，我们学习了什么是代码拆分，它如何与 Webpack 一起工作，以及如何在 Vue 应用程序中通过延迟加载来使用它。现在，我们将更深入地挖掘代码，学习对代码分割 Vue.js 应用程序最有用的模式。**

该系列基于从 [Vue 店面](https://github.com/DivanteLtd/vue-storefront)性能优化流程中获得的经验。通过使用下面的技术，我们能够将初始包的大小减少 70%,并在一眨眼的时间内完成加载。

[第 1 部分—性能优化和延迟加载简介。](https://vueschool.io/articles/vuejs-tutorials/lazy-loading-and-code-splitting-in-vue-js/)

[第 2 部分—惰性加载路线和供应商捆绑反模式。](https://vueschool.io/articles/vuejs-tutorials/vue-js-router-performance/)

[第 3 部分—惰性加载 Vuex 模块](/vue-js-app-performance-optimization-part-3-lazy-loading-vuex-modules-ed67cf555976)

第 4 部分——提供良好的等待体验和延迟加载单个组件——很快

第 5 部分—延迟加载库并寻找更小的等价库—很快

第 6 部分——UI 库的性能友好使用

第 7 部分—利用服务工作者缓存—很快

第 8 部分—预取

# 文章已移动

嘿，这篇文章已经更新了更多的见解，并转移到 VueSchool 博客[这里](https://vueschool.io/articles/vuejs-tutorials/vue-js-router-performance/)！
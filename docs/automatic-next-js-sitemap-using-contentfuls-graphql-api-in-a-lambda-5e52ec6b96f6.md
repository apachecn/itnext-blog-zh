# 在 lambda 中使用 Contentful 的 GraphQL API 的自动 Next.js 站点地图

> 原文：<https://itnext.io/automatic-next-js-sitemap-using-contentfuls-graphql-api-in-a-lambda-5e52ec6b96f6?source=collection_archive---------4----------------------->

有很多解决方案`next-sitemap`等在构建时这样做，但我认为这种方法更酷，如果不是过分的话。

我们只需要先做两件事:

```
> npm i sitemap
> vercel secrets set HOSTNAME [www.domain.com](http://www.domain.com) production
```

`HOSTNAME` var 是我们的 FQDN，与我们在 Google 网站管理员工具、GA/GTM 等中设置的主域相匹配。对于非生产，为了保持整洁，我们可以更改默认值 w/ `VERCEL_URL`

基本配置要求我们暴露`HOSTNAME`，添加从`/sitemap.xml`到`/api/sitemap.ts`的重写，并在`vercel.json`中设置一些头:

然后是函数本身:

我们还可以组合不同的内容类型，并为每种类型设置优先级，如下所示:
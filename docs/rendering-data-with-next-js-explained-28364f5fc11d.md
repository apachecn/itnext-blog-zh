# 解释了使用 Next.js 呈现数据

> 原文：<https://itnext.io/rendering-data-with-next-js-explained-28364f5fc11d?source=collection_archive---------1----------------------->

## Next.js React

## 简单解释了 getStaticPaths、getStaticProps 和 getServiceSideProps

![](img/23b9803752b8729a13d6f457e9dd1028.png)

马丁·桑切斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

本指南将简要说明在 [Next.js](https://nextjs.org/) 中呈现数据的三个内置函数。没有困难的深入功能，所以每个人都可以理解并开始使用这些功能。

js 是一个框架，用于生成静态和服务呈现的内容，以及其他各种内容。在您的 Next.js 应用程序中，您可能会有不同的页面，其中放入了 JSX 代码来显示内容。比如下面的例子:

```
function Home() {
  return <p>some html</p>
}// More code here with Next.js
```

有三个重要的函数你应该记住，为了使用这些函数，你需要像例子一样写它们。所以它需要默认导出，并与确切的名称异步。让我们开始吧(包括例子)

# 1.getStaticPaths

第一个 Next.js 函数被称为`getStaticPaths`。在这里，您可以定义特定的路径或路线，您知道这些路径或路线将在您的应用程序中存在或使用。路径将在构建时预先呈现，而不是在服务器端呈现。

您可以返回两个属性，一个是`paths`属性中的特定路径，另一个是`fallback` 属性。如果 fallback 为 *false* ，它将在构建时生成每个预先描述的路径，其他路径将为 404。如果 fallback 为 *true* ，那么所有其他路径都将引导您进入一个 fallback 页面，而其他路径则由服务器加载。如果回退被*阻塞*，用户需要等待，直到另一条路径被服务器加载。

```
function Home() {
  return <p>some html</p>
}export default async function **getStaticPaths**() {
  return {
    paths: [], // describe your paths here
    fallback: false // or true or blocking
  }
}
```

# 2.getStaticProps

下一个 it `getStaticProps`，看起来和上一个功能差不多。但是现在我们也有了一个`context`或`ctx`参数。使用这个函数，您可以在构建时获取数据，因此内容将立即显示给用户，而不是等待获取。

`context`包含各种属性，你可以在这里阅读更多关于它的[。更重要的是*三个*可选属性你可以返回。](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)

第一个是`props`，这里大部分人都很熟悉。您可以在这里定义您想要的所有属性，例如，您刚刚获取的一些数据(在构建时)，以通过您的组件。

接下来，你有`revalidate`。您可以在这里输入页面重新生成(运行时)的毫秒数，这样您的用户就可以获得最新的内容。这可以通过将值设置为 *false* 来关闭，或者只保留该属性。

最后，还有`notFound`房产。如果设置为*真*，那么它将返回一个 404 页面，例如，如果没有可用的数据。

```
function Home({ example }) {
  return <p>{example}</p>
}export default async function **getStaticProps**(context) {
  // some fetch call here

  return {
    props: { example: 'something' },
    revalidate: false, // or some milliseconds like; 1000
    notFound: false, // or true
  }
}
```

# 3.getServerSideProps

到了最后一个重要的功能，叫做`getServerSideProps`。这用于在运行时从服务器获取数据。同样，我们有一个`context`参数，它与`getStaticProps`的上下文有不同的属性。

在这里，您还可以返回三个可选属性。`notFound`和`props`属性在前面的章节中已经解释过了。

另一个可选属性叫做`redirect`，在这里你可以引导你的用户到你网站内部或外部的不同页面。例如，如果没有可用的数据。在这个对象中，你可以定义`destination`路径，以及它是否是一个`permanent`重定向。

```
function Home({ example }) {
  return <p>{example}</p>
}export default async function **getServerSideProps**(context) {
  // some fetch call here

  return {
    props: { example: 'something' },
    redirect: { destination: '/some-url', permanent: false },
    notFound: false, // or true
  }
}
```

仅此而已！

我希望你今天学到了一些新东西。如果你觉得这篇文章有用，请点击 50 次👏按钮并关注我了解更多内容。
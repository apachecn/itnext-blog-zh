# 使用 Transloco 在 Angular 中延迟加载翻译文件

> 原文：<https://itnext.io/lazy-load-translation-files-in-angular-using-transloco-2d3afed116ce?source=collection_archive---------2----------------------->

![](img/2b3535b9ead848449f18ac873eea1432.png)

在本文中，我们将学习如何使用 Angular Transloco 的国际化库在我们的 Angular 应用程序中延迟加载翻译文件。这种方法有两个优点。首先，我们将初始包变得更小，这将改善应用程序的加载时间。第二，我们较少暴露于合并冲突。

让我们看看使用 Transloco 有多简单💪

假设我们有一个用户资料页面。我们希望为该页面创建单独的翻译文件，并仅在用户导航到该页面时加载它们。

首先，我们需要创建一个 user-profile 文件夹(或者您选择的任何名称)；其中，我们为想要支持的每种语言创建了一个翻译文件:

```
├─ i18n/
  ├─ en.json
  ├─ es.json
  ├─ profile-page/
    ├─ en.json
    ├─ es.json
```

在每个文件中，将只添加与此页面相关的翻译:

```
{
  "title": "User Profile"
}
```

让我们继续创建用户配置文件模块和路由:

向提供者添加`TRANSLOCO_SCOPE`指示 Transloco 根据活动语言加载相应的作用域，并将其合并到作用域名称空间下的活动语言翻译对象中。

例如，如果活动语言是 en，它将加载 user-profile/en.json 文件，并将翻译设置为以下内容:

```
{
  header: ‘’,
  login: ‘’,
  userProfile: {
    title: ‘User Profile’
  }
}
```

现在，我们可以通过使用 user profile 名称空间来访问每个用户配置文件键:

默认情况下，名称空间将是作用域名称(骆驼大小写)，但我们可以用两种方式覆盖它:

通过使用`config.scopeMapping`配置:

```
{
  provide: TRANSLOCO_CONFIG,
  useValue: {
    defaultLang: 'en',
    scopeMapping: {
      'user-profile': 'profile'
    }
  }
}
```

通过在模块/组件范围提供程序中提供自定义别名:

```
{
  provide: TRANSLOCO_SCOPE,
  useValue: { scope: ‘user-profile’, alias: ‘profile’ }
}
```

现在我们可以通过我们提供的自定义名称来访问它，而不是原来的作用域名称(在我们的例子中是 profile):

如果你喜欢这篇文章，请👏🏻并分享一下！

[](https://netbasal.com/introducing-transloco-angular-internationalization-done-right-54710337630c) [## 🚀Transloco 简介:角度国际化进展顺利

### 有一段时间，我一直在考虑创建一个 Angular i18n 库，它包含了我脑海中的一些概念。

netbasal.com](https://netbasal.com/introducing-transloco-angular-internationalization-done-right-54710337630c) [](https://netbasal.com/good-things-come-to-those-who-wait-whats-new-in-transloco-5dadf886b485) [## 🎉好事降临在耐心等待的人身上:Transloco 有什么新鲜事

### Transloco 的新特性:Angular 的下一代 i18n 库！

netbasal.com](https://netbasal.com/good-things-come-to-those-who-wait-whats-new-in-transloco-5dadf886b485) [](https://medium.com/@shahar.kazaz/creating-search-engine-friendly-internationalized-apps-with-angular-universal-and-transloco-ab9583cfb5ac) [## 使用 Angular Universal 和 Transloco 创建搜索引擎友好的国际化应用程序🌐

### 在本文中，我将向您展示使用下一个……

medium.com](https://medium.com/@shahar.kazaz/creating-search-engine-friendly-internationalized-apps-with-angular-universal-and-transloco-ab9583cfb5ac)
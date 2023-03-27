# 什么是鼻涕虫

> 原文：<https://itnext.io/whats-a-slug-f7e74b6c23e0?source=collection_archive---------1----------------------->

![](img/90eb28ae8159dfe30192238394e25679.png)

图片来源于[Peter Stevens 的原始照片](https://www.flickr.com/photos/nordique/43507387102)并根据其许可条款进行了改编。

## 我为什么要用它？

`slug`是人类可读的唯一标识符，用于标识资源，而不是像`id`这样不太可读的标识符。当你想引用一个项目，同时保留一眼就能看到这个项目是什么的能力时，你可以使用`slug`。

典型地，slugs 是在制作搜索引擎优化的 url 时使用的，所以例如这篇文章的 URL 是[https://medium.com/@davesag/whats-a-slug-f7e74b6c23e0](https://medium.com/@davesag/whats-a-slug-f7e74b6c23e0)。在那个 url 中实际上有两个 slug，我的`username`，`[@davesag](http://twitter.com/davesag)` 和这个特定帖子的 slug， `whats-a-slug-f7e74b6c23e0`。其中的`whats-a-slug`部分来自标题，出于只有 Medium 的人知道的原因，他们在它的结尾加了一个简短的`uuid`。我想到这样做的唯一原因是如果用户能够用同一个标题写两个故事，所以我想这是可能发生的。

# 一个例子

一个更有趣的例子是，当您的系统存储一组产品时，每个产品都有一个类别。

在伪代码中，这里有一组你会在数据库中看到的相当典型的模型。

```
Product {
  id: integer (auto-incremented, unique),
  name: string,
  slug: string (derivedFrom(name), unique),
  category: belongsTo(Category, id)
}Category {
  id: integer (auto-incremented, unique),
  name: string,
  slug: string (derivedFrom(name), unique),
  products: hasMany(Products, id)
}
```

现在，假设您想要提供一个列出给定类别的产品的 API。

您可以按如下方式定义路线:

```
GET /category/:id/products
```

但这会给你一个相当难看的 URL。

所以最好把路线定义为:

```
GET /category/:slug/products
```

所以如果你有一个`category`‘books’那么有人调用你的 API 就会调用:

```
GET /category/books/products
```

# 精益 API 设计

## 总则

原料药应当:

*   不返回数据库内使用的内部`id`，而是返回一个`slug`，
*   返回最少量的有用数据，并且
*   返回足够的数据以减少重复请求的需要

## 基于上述模型的示例数据

`Category`

```
{
  id: 1,
  name: "Books",
  slug: "books"
}
```

`Products`

```
[
  {
    id: 1,
    name: "Old Man's War",
    slug: "old-mans-war",
    category: 1
  },
  {
    id: 2,
    name: "All Systems Red",
    slug: "all-systems-red",
    category: 1
  }
]
```

对 `GET /category/books`的调用将返回:

```
{
  name: "Books",
  slug: "books",
  products: [
    {
      name: "Old Man's War",
      slug: "old-mans-war"
    },
    {
      name: "All Systems Red",
      slug: "all-systems-red"
    }
  ]
}
```

不需要返回每个`product`的`category`信息，因为请求者已经知道了`category`，然而，按照相同的逻辑，也不应该需要返回`category`的`slug`。

为了保持一致性，返回一个条目的`slug`总是一个好的做法，即使这个信息是多余的。如果返回了`slug`，API 的消费者通常可以节省一些复杂性。

## 一个更有趣的例子

本地化通常通过使用以下形式的数据结构来处理:

```
{
  [locale]: string
}
```

在伪代码中，它在数据库中的结构如下:

```
Product {
  id: integer (auto-incremented, unique),
  name: hstore({
    [locale]: string
  }),
  slug: string (derivedFrom(name), unique),
  category: belongsTo(Category, id)
}Category {
  id: integer (auto-incremented, unique),
  name: hstore({
    [locale]: string
  }),
  slug: string (derivedFrom(name), unique),
  products: hasMany(Products, id)
}
```

每个模型都需要一个实例函数，如下所示:

```
function getLocalised(field, locale) {
  return this[field][locale]
}
```

API 需要知道返回哪个`locale`的方法。这通常是通过向`request`添加一个`locale`参数，或者在`request`的`Accept-Language`头中添加一个`locale`参数来实现的，或者如果`request`已经过验证，那么您可以将所需的`locale`与一个`user`记录相关联。

## 基于上述模型的示例数据

`Category`

```
{
  id: 1,
  name: {
    en: "Books",
    vn: "Sách"
  },
  slug: "books"
}
```

`Products`

```
[
  {
    id: 1,
    name: {
      en: "Old Man's War",
      vn: "Chiến tranh của lão già"
    },
    slug: "old-mans-war",
    category: 1
  },
  {
    id: 2,
    name: {
      en: "All Systems Red",
      vn: "Tất cả các hệ thống đều đỏ"
    },
    slug: "all-systems-red",
    category: 1
  }
]
```

对`GET /product/all-systems-red?locale=vn`的请求将返回:

```
{
  name: "Tất cả các hệ thống đều đỏ",
  slug: "all-systems-red",
  category: {
    name: "Sách",
    slug: "books"
  }
}
```

# 播种您的数据库

设置系统时，您希望避免在种子数据中指定对象`ids`。种子数据通常在`yml`文件中定义。使用上面定义的数据库模型，我们可以编写如下种子文件:

`seed-data.yml`

```
- slug: books
  name:
    en: Books
    vn: Sách
  products:
    - slug: old-mans-war
      name:
        en: Old Man's War
        vn: Chiến tranh của lão già
    - slug: all-systems-red
      name:
        en: All Systems Red
        vn: Tất cả các hệ thống đều đỏ
```

像这样的数据很容易导入。

# 记录

当将信息写到日志中时，这些日志被人和机器使用是很正常的。

将`ids`写到日志中通常不是一个好主意。另一方面，写出`slugs`让人们更容易阅读日志，因为`slugs`也有`ids`的功能，它们对机器仍然有用。

# 如何制作鼻涕虫

一个好的 slug 是简短的，描述性的，小写的，没有重音符号，或者含糊不清或者难以一眼读懂的字符，并且是独一无二的。

# 链接

`Slugify`不同语言

*   `Javascript`[https://github.com/simov/slugify](https://github.com/simov/slugify)
*   `Rust`[https://github.com/mattgathu/slugify](https://github.com/mattgathu/slugify)
*   `Java`[https://github.com/slugify/slugify](https://github.com/slugify/slugify)
*   [https://github.com/Slicertje/Slugify](https://github.com/Slicertje/Slugify)
*   `Python`[https://github.com/un33k/python-slugify](https://github.com/un33k/python-slugify)
*   【https://github.com/nodes-vapor/slugify】T14`Swift`
*   `Go`[https://github.com/mozillazg/go-slugify](https://github.com/mozillazg/go-slugify)

## 书

*   [老人的战争](https://www.goodreads.com/book/show/36510196-old-man-s-war)
*   [所有系统红色](https://www.goodreads.com/book/show/32758901-all-systems-red)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。
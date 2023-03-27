# Javascript 按日期排序数组

> 原文：<https://itnext.io/javascript-sorting-arrays-by-date-3c64b2684de3?source=collection_archive---------1----------------------->

两种方式，哦，等等，还有一种奖励方式

![](img/63d1e762cef647ce77fbf0a075144634.png)

照片由 [Max Panamá](https://unsplash.com/@imaxpanama?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

你需要对一个数组排序多少次？如果你是一个前端开发人员，你可能每天都在做这件事。实际上，如果你是 node js 开发人员，你可能也一直在做这件事。现在，更具挑战性的是尝试对一组日期或日期对象进行排序。我将向您展示两种方法，一种是使用 javascript 本地函数，另一种是使用名为 loadash 的库。在这两个例子中，我将使用 [moment.js](https://momentjs.com/) 这是一个非常方便的比较和格式化日期的工具。

# 原生 Javascript

我是一个纯粹主义者，所以我先向你展示这个方法。当谈到使用 javascript 函数时，我喜欢尝试先用 javascript 创建者给我们的原生函数做一些事情。我会告诉你怎么做，然后解释它。

```
import moment from 'moment'const dateArray = [{date:"2018-05-11"},{date:"2018-05-12"},{date:"2018-05-10"}]const sortDates = (a, b) => moment(a.roomtypedate).format('YYYYMMDD') -moment(b.roomtypedate).format('YYYYMMDD')const sortedDates = dateArray.sort(sortDates)
console.log(sortedDates)
```

所以解释如下。首先，我们使用原生函数 sort。如果你想阅读更多关于这个函数如何工作的内容，请查阅我的另一篇关于函数的文章。

除了排序功能之外，这里的神奇之处在于利用了`moments`比较日期的能力。现在，这将只比较日期和日子，但不会比较时间和分钟。

# Lodash orderBy

如果你不喜欢排序，你也可以使用洛达什的功能`orderBy`。这是一个方便的函数，你可以用它来按照`asc`或`desc`的顺序进行排序。感觉比原生排序函数更直观一点。

```
import moment from 'moment'
import _ from 'lodash'const dateArray = [{date:"2018-05-11"},{date:"2018-05-12"},{date:"2018-05-10"}]const sortedArray = _.orderBy(array, (o: any) => {
  return moment(o.date.format('YYYYMMDD');
}, ['asc']);console.log(sortedDates)
```

这是关于这个函数的文档。[摘自 lodash 的文档网站。](https://lodash.com/docs/4.17.15)

`_.orderBy(collection, [iteratees=[_.identity]], [orders])`

这个方法类似于`[_.sortBy](https://lodash.com/docs/4.17.15#sortBy)`，除了它允许指定迭代排序的顺序。如果未指定`orders`，则所有值按升序排序。否则，为相应值的降序指定“desc”顺序，或为升序指定“asc”顺序。

## 争论

1.  `**collection**` ***(数组|对象)*** :要迭代的集合。
2.  `**[iteratees=[_.identity]]**` ***(数组[]|函数[]|对象[]|字符串[])*** :排序所依据的迭代。
3.  `**[orders]**` ***(字符串[])***`**iteratees**`的排序顺序。

## 返回

***(数组)*** :返回新排序后的数组。

如果你想要这个功能的文档，你可以[在这里](https://lodash.com/docs/4.17.15#orderBy)阅读更多。

# 奖金

现在，如果你想更具体地比较日期，那么只需按日、周、月和年。你可以试试这个方法。甚至更短。但是这依赖于数组中的那些值已经是日期对象的事实。

```
const sortedArray = array.sort((a, b) => a.valueOf() - b.valueOf())
```

这是因为日期对象可以通过减法进行比较。

# 结论

对值进行排序的算法有很多。幸运的是，javascript 已经为我们发布了一些排序方法。不要去重新发明轮子。

如果你对日期排序有其他想法，请告诉我。

祝你好运！

编码快乐！！！

***Avery duff in****是* [*【吃午餐的开发者】*](https://www.developerswholunch.com) *的所有者和创始人，这是一个播客和博客，旨在帮助那些学习编程并寻求在软件开发领域建立职业生涯的人。他目前在*[*rai focus*](https://www.rainfocus.com/)*担任软件工程师，居住在美国犹他州。你可以在*[*Twitter*](https://twitter.com/DuffinAvery)*和*[*linkedIn*](https://www.linkedin.com/in/avery-duffin-69317228/)*上和他联系。*
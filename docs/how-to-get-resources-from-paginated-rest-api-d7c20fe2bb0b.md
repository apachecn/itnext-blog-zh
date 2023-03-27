# 如何使用递归和 JavaScript 承诺从分页 REST API 获取资源

> 原文：<https://itnext.io/how-to-get-resources-from-paginated-rest-api-d7c20fe2bb0b?source=collection_archive---------1----------------------->

最近，我用星球大战 API([https://swapi.co/](https://swapi.co/))做了一个兼职项目。该项目的要求之一是在一页上列出所有的行星。然而，你不能在一个 API 调用中得到所有的行星。相反，您需要使用正确的分页信息进行多次 API 调用来获取所有行星。在这篇文章中，我将谈谈我是如何做到这一点的。

我们来看看调用以下 API 端点时返回的 JSON 数据:【https://swapi.co/api/planets】

```
*{ "count": 61, "next": "https://swapi.co/api/planets/?page=2", "previous": null, "results": [ { "name": "Alderaan", "rotation_period": "24", "orbital_period": "364", "diameter": "12500", "climate": "temperate", "gravity": "1 standard", "terrain": "grasslands, mountains", "surface_water": "40", "population": "2000000000", "residents": [ "https://swapi.co/api/people/5/", "https://swapi.co/api/people/68/", "https://swapi.co/api/people/81/" ], "films": [ "https://swapi.co/api/films/6/", "https://swapi.co/api/films/1/" ], "created": "2014-12-10T11:35:48.479000Z", "edited": "2014-12-20T20:58:18.420000Z", "url": "https://swapi.co/api/planets/2/" },
        ... // 9 more planets here 
    ]
}*
```

*如您所见，单个 API 调用只能返回总共 61 个行星中的 10 个。如果您想要获得接下来的 10 颗行星，您需要使用从 JSON 数据中的' next '键获得的端点 URL 进行另一次 API 调用，因此为了获得总共 61 颗行星，我们需要进行 7 次 API 调用。*

*我的第一个想法是，端点 URL 是可预测的，因为我知道行星的总数和 URL 的结构，我可以很容易地找出第 2、3、…、7 页的 URL 是什么。然后，我可以使用 for 循环和循环中的一些条件检查，对所有这些硬编码的端点进行 API 调用，以获得所有的行星。这应该可以，但是，它一点也不灵活。*

*最后，我想出了一个使用递归和 JavaScript 承诺的解决方案:*

*如果你觉得这篇文章有帮助，请访问[我的 YouTube 频道](https://www.youtube.com/channel/UCoQ7C3JajfBOIjp7yWDj8Yg)，在那里你也可以找到一些有用的编程技巧。*
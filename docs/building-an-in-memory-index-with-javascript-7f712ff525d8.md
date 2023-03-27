# 用 JavaScript 构建内存索引

> 原文：<https://itnext.io/building-an-in-memory-index-with-javascript-7f712ff525d8?source=collection_archive---------2----------------------->

![](img/87b8bf8dd8d282f43e441d263a11601c.png)

Maksym Kaharlytskyi 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

作为程序员，数组是我们日常生活的一部分。假设您正在构建一个电子商务应用程序，并且在某个时候您有一个如下所示的数组:

```
const skus = [
  {
    "productId": 1,
    "colorId": 1,
    "sku": "11-M"
  },
  {
    "productId": 1,
    "colorId": 1,
    "sku": "11-L"
  },
  {
    "productId": 1,
    "colorId": 2,
    "sku": "12-M"
  },
  {
    "productId": 1,
    "colorId": 2,
    "sku": "12-L"
  },
  {
    "productId": 2,
    "colorId": 1,
    "sku": "21-M"
  },
  {
    "productId": 2,
    "colorId": 1,
    "sku": "21-L"
  }
];
```

给定这个数组，你的新任务将是获得某个`productId`的所有 [SKU](https://es.wikipedia.org/wiki/Stock-keeping_unit) 。你首先想到的是使用`skus.find(s => productId === s.productId)`,对于某些场景来说，这可能是一个好的解决方案。但是，当您的应用程序处理大量数据时，这可能会导致性能问题。

对于这种情况，更好的方法是将这个数组转换成一个索引，以便直接访问 SKU，而不是遍历数组。

为此，我们可以使用`reduce`:

```
const productIndex = indexByField(skus, “productId”);
```

我们的指数是:

```
{
  "1": [
    {
      "productId": 1,
      "colorId": 1,
      "sku": "11-M"
    },
    {
      "productId": 1,
      "colorId": 1,
      "sku": "11-L"
    },
    {
      "productId": 1,
      "colorId": 2,
      "sku": "12-M"
    },
    {
      "productId": 1,
      "colorId": 2,
      "sku": "12-L"
    }
  ],
  "2": [
    {
      "productId": 2,
      "colorId": 1,
      "sku": "21-M"
    },
    {
      "productId": 2,
      "colorId": 1,
      "sku": "21-L"
    }
  ]
}
```

现在您可以获得特定产品的所有 SKU！

# 等等…我如何获得特定颜色的 SKU？🤔

您可能注意到了，我们只通过`productId`索引了我们的数组。让我们也用`colorId`来索引我们的数组。

考虑到我们当前的索引状态，应该使用我们的`indexByField`函数索引它的值。是的，在这种情况下这是可行的，但是如果我们有许多层次的指数化呢？…

***递归救援！***

下面我们来详细解释一下这个`mapObjectValues`:

*   `value`将是我们第一次调用中的数组和下一次递归调用中的索引。
*   `shouldMap`将决定何时停止进行递归调用。在我们的例子中，它将是`Array.isArray`。
*   `mapFn`将转换我们的索引值，就像`Array.prototype.map`对数组项所做的那样。在我们的例子中，它将是我们的`indexByField`函数。

# 把所有的放在一起🧩

我们准备定义一个函数，通过多个字段索引一个数组。此外，我们扩展了`Array.prototype`,这样我们可以直接从实例中调用它:

```
const productColorIndex = skus.indexBy(“productId”, "colorId");
```

它的值将是:

```
{
  "1": {
    "1": [
      {
        "productId": 1,
        "colorId": 1,
        "sku": "11-M"
      },
      {
        "productId": 1,
        "colorId": 1,
        "sku": "11-L"
      }
    ],
    "2": [
      {
        "productId": 1,
        "colorId": 2,
        "sku": "12-M"
      },
      {
        "productId": 1,
        "colorId": 2,
        "sku": "12-L"
      }
    ]
  },
  "2": {
    "1": [
      {
        "productId": 2,
        "colorId": 1,
        "sku": "21-M"
      },
      {
        "productId": 2,
        "colorId": 1,
        "sku": "21-L"
      }
    ]
  }
}
```

现在，我们只需查阅我们的索引就可以获得 SKU:

`productColorIndex["1"]["2"];`

# 综上📝

在时间复杂度方面，使用索引而不是数组对于优化应用程序中的许多用例非常有用，尤其是在处理大量数据的情况下。

然而，这种收益是有代价的，创建一个索引并将其保存在内存中在某些情况下是非常昂贵的。

# 资源

*   [https://developer . Mozilla . org/en-US/docs/Web/JavaScript/reference ia/Objetos _ global es/Array/reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
*   https://en.wikipedia.org/wiki/Stock-keeping_unit

[](https://medium.com/better-programming/javascript-iteration-v-s-recursion-and-behind-the-scene-e12fe1756343) [## JavaScript 中的迭代与递归

### 幕后的差异，以及如何做出使用哪个的正确决定

medium.com](https://medium.com/better-programming/javascript-iteration-v-s-recursion-and-behind-the-scene-e12fe1756343)
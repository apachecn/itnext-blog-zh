# 用这个简单的递归函数将一个数组分解成一个深度嵌套的对象。

> 原文：<https://itnext.io/explode-an-array-into-a-deeply-nested-object-with-this-simple-recursive-function-4094ac1eeb8b?source=collection_archive---------1----------------------->

![](img/30857ea7485bb4d230e34f676a5c3aa5.png)

堪培拉的秋天是我一年中最喜欢的时候。(图片由作者提供)

## 因为递归规则。

从 API 获取数据时，一个非常常见的模式是在应用程序中索引该数据，这样您就可以在不搜索的情况下回读它。

# 一个例子

假设您有一个 API，它返回一个项目列表，比如。

```
const ITEMS = [
  {
    title: 'The One',
    contract: '1',
    author: 'alice'
  },
  {
    title: 'It Takes Two',
    contract: '2',
    author: 'alice'
  },
  {
    title: '3x a lady',
    contract: '1',
    author: 'beck'
  },
  {
    title: 'Four on the Floor',
    contract: '1',
    author: 'candice'
  },
  {
    title: "In de'Pipe Going Five by Five",
    contract: '3',
    author: 'candice'
  }
]
```

在你的应用程序中，你会发现你经常需要通过`author`和`contract`来查找特定的`item`。一种天真的方法可能是这样的

```
const getItemBy = (author, contract)
  => ITEMS.find(
    item => author === item.author && contract === item.contract
  )
```

如果你只是偶尔查一下`item`，这没什么。但是如果你需要一直这样做，它会变得有点慢，要一直搜索整个 T4。

相反，如果你能写一个函数，比如

```
const getItemBy = (author, contract) => ITEMS_INDEX[author][contract]
```

或者更安全

```
const getItemBy = (author, contract )
  => ITEMS_INDEX[author] ? ITEMS_INDEX[author][contract] : undefined
```

在这种情况下,`ITEMS_INDEX`需要这样的结构

```
const ITEMS_INDEX = {
  alice: {
    '1': {
      title: 'The One',
      contract: '1',
      author: 'alice'
    },
    '2': {
      title: 'It Takes Two',
      contract: '2',
      author: 'alice'
    },
  },
  beck: {
    '1': {
      title: '3x a lady',
      contract: '1',
      author: 'beck'
    },
  },
  candice: {
    '1': {
      title: 'Four on the Floor',
      contract: '1',
      author: 'candice'
    },
    '3': {
      title: "In de'Pipe Going Five by Five",
      contract: '3',
      author: 'candice'
    }
  }
}
```

# 创建索引

为了以一种灵活的方式从`ITEMS`构建`ITEMS_INDEX`。

```
const ITEMS_ARRAY = explode(ITEMS, 'author', 'contract')
```

## 一把钥匙

用一个键分解一组项目很容易。

```
const explodeOne = (array, key)
  => array.reduce((acc, item) => ({
    ...acc,
    [item[key]]: item
  }), {})
```

所以`explodeOne(array, 'author')`会给你一个对象，其中的条目是由它们的作者来定义的，但是在这个例子中，每个作者可以有多个条目，通过条目的`contract`来区分。

## 许多钥匙

如果你需要多个键，就像我们上面的例子，你需要使用*递归*。

```
const *explode* = (array, key, ...rest)
  => array.*reduce*((acc, item) => ({
    ...acc,
    [item[key]]: rest.length ? {
      ...acc[item[key]],
      ...*explode*([item], ...rest)
    } : item
  }), {})
```

## 这是怎么回事？

这里使用了许多方便的 Javascript 特性。

1.  [*Rest 语法*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#rest_syntax_parameters) 处理任意数量的按键`(array, key, ...rest)`、
2.  [*一个数组缩减器*](/map-and-reduce-a-primer-9e18a6b7e587) 循环遍历数组并构造我们的最终对象，
3.  [*Computed properties*](https://codeburst.io/computed-property-names-ftw-61000332fff1)`[item[key]: ...`意思是你可以在事先不知道的情况下，在一个对象中引用或者创建一个属性，
4.  [*对象展开批注*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals) `{ ...acc, ...*explode*([item], ...rest) }`，以及
5.  *递归*，即我们在`*explode*`函数中再次调用`explode`来处理特定`item`的下一个键。

## 逐行解释

`const *explode* = (array, key, ...rest)`将函数`*explode*`定义为接受参数`array`，还有一个`key`，以及可选的一组其他键，我们称之为`rest`。如果没有更多的键，那么`rest = []`。

该功能将把`array`、*还原为`object`。数组 *reducer* 返回一个不断扩展的对象，`acc`(accumulator 的缩写)，在每一遍中，添加`...acc`，即对象中已经存在的内容，加上一个带有一些相关数据的新键`[item[key]]`。在我们的例子中，如果`item`是`{ title: 'The One', contract: '1', author: 'alice', data: 'one' }`并且`key`是“作者”，那么`{ [item[key]]: ... }`将是`{ alice: ... }`。*

因为 Alice 可以是多个项目的作者，有不同的合同，如果有更多的键(即`rest.length !== 0`)，我们需要确保`[item[key]]`处的对象包括之前已经在`acc[item[key]]`中的任何内容，然后我们想要`*explode*`只使用键的`rest`的单个`item`。

如果`rest`中没有更多的钥匙，我们只需插入`item`本身。

## 利益

`*explode*`函数为您提供了一种非常简单的方法来索引以非优化格式提供的数据。许多 API 将返回`key, value`对的数组，将这些数组转换成一个对象，用提供的键进行键控，可以节省您以后的大量工作。

# 增强:读者的一个练习

如果我们已经有了一个有效的`author`和`contract`，那么我们真正需要在索引中存储的就是标题。

```
const TITLES_INDEX = {
  alice: {
    '1': 'The One',
    '2': 'It Takes Two'
  },
  beck: {
    '1': '3x a lady'
  },
  candice: {
    '1': 'Four on the Floor',
    '3': "In de'Pipe Going Five by Five"
  }
}const *getTitle* = (author, contract)
  => TITLES_INDEX[author]
    ? TITLES_INDEX[author][contract]
    : undefined
```

你会对上面的`explode`函数做什么修改来生成`TITLES_INDEX`？

请在回复中张贴您的答案。

# 链接

*   [https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Operators/Spread _ syntax # rest _ syntax _ parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#rest_syntax_parameters)
*   【https://itnext.io/map-and-reduce-a-primer-9e18a6b7e587 
*   [https://code burst . io/computed-property-names-ftw-61000332 fff 1](https://codeburst.io/computed-property-names-ftw-61000332fff1)
*   [https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Operators/Spread _ syntax # Spread _ in _ object _ literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals)
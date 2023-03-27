# 有用的“减少”用例

> 原文：<https://itnext.io/useful-reduce-use-cases-91a86ee10bcd?source=collection_archive---------3----------------------->

![](img/667e12eaeee8be74a955030d3f2b0298.png)

我最近看到一个开发人员的推文，他说他理解 array `reduce`函数，但是想不出很多有用的用例。在这篇文章中，我的目标是给出几个真实世界的`reduce`用例。

帖子会假设你对 JavaScript 承诺感到满意；如果没有，我在之前的帖子里有一个非常简单的入门教程，你可以看看。

# 承诺链

`Promise.all()`并行运行多个承诺非常有用，但是如果你的承诺应该解决的**顺序**很重要呢？在这种情况下，您需要按顺序运行它们。下面的函数将完全做到这一点。

```
const promiseQueue = (promiseFn, list) =>
  list.reduce(
    (queue, item) => queue.then(async result => {
      const itemResult = await promiseFn(item);
      return result.concat([itemResult]);
    }),
    Promise.resolve([])
  );
```

# 数字总和

使用`reduce`最常见、最经典的例子是获取一组数字的总和，所以没有它这个列表就不完整。但是它不一定是一个数字数组——您可以使用更复杂的数据结构。

```
const cities = [
  {
    city: "Chongqing",
    population: 30165500
  },
  {
    city: "Shanghai",
    population: 24183300
  },
  {
    city: "Beijing",
    population: 21707000
  },
  {
    city: "Lagos",
    population: 16060303
  },
  // ...etc
]const totalPopulation = cities.reduce(
  (sum, city) => sum + city.population,
  0
);
```

# 过滤和映射

当你有一个数据集需要通过一些标准过滤，并且你也想操作每一个剩余的项目时，一个好的但是有点幼稚的选择是首先对它运行`filter`，然后再对它运行`map`，本质上比需要做更多的循环项目。更好的选择是使用`reduce`。

比方说，你有所有写了某篇论文的学生的成绩，但你只对所有分数超过 80 分的学生的全名感兴趣。

```
const studentsData = [
  {
    firstName: "Albert",
    lastName: "Einstein",
    score: 53
  },
  {
    firstName: "Charles",
    lastName: "Dickens"
    score: 84
  },
  {
    firstName: "Marilyn",
    lastName: "vos Savant",
    score: 99
  },
  // etc.
];const smartestStudents = studentsData.reduce(
  (result, student) => {
    // do your filtering
    if (student.score <= 80) {
      return result;
    } // do your mapping
    result.push(`${student.firstName} ${student.lastName}`);
    return result;
  },
  []
);
```

# 数组到对象的转换

有时，您需要一个对象作为一个数组的输出，而不是另一个数组的输出。所有其他数组高阶函数将*始终*产生一个数组作为输出。

比方说，您有一个带有有效性约束的表单字段数组，您想要一个约束对象，其中字段的名称是对象中的顶级键。

```
const fields = [
  {
    type: 'text',
    title: 'Title',
    name: 'title',
    constraints: {
      required: {
        message: '^Title is required',
        allowEmpty: false
      }
    },
  },
  {
    type: 'text',
    title: 'Slug',
    name: 'slug',
    constraints: {
      required: {
        message: '^Slug is required',
        allowEmpty: false
      },
      format: {
        pattern: '[a-z0-9_-]+',
        flags: 'i',
        message: '^Can only be a valid slug'
      }
    },
  },
  // etc.
];const validationRules = fields.reduce(
  (rules, field) => Object.assign(rules, { [field.name]: field.constraints }),
  {}
);
```

这会产生类似这样的结果:

```
{
  title: {
    required: {
      message: '^Title is required',
      allowEmpty: false
    }
  },
  slug: {
    required: {
      message: '^Slug is required',
      allowEmpty: false
    },
    format: {
      pattern: '[a-z0-9_-]+',
      flags: 'i',
      message: '^Can only be a valid slug'
    }
  },
  // etc.
}
```

*注意:如果你不关心数据的变化，你可以考虑使用* `*Object.assign({}, rules, { [field.name]: field.constraints })*` *或者使用对象扩展语法，* `*{ ...rules, [field.name]: field.constraints }*` *。我建议不要这样做。就个人而言，我坚信不变数据的价值，但带有实用主义色彩。在这种情况下，将为每次迭代创建一个新对象，这可能会在应用程序中使用大量不必要的内存。*

# 管道或合成功能

有时您有一系列函数，您希望为特定值调用这些函数。有用的函数式编程方法`compose`和`pipe`非常适合这一点，但不幸的是，它们不是 JavaScript 语言的固有部分(目前还不是！).不用担心，我们可以轻松地编写自己的代码:

```
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x)
```

`pipe`是一个接受函数列表的函数，依次调用每个函数，前一个函数的输出作为下一个函数的输入。

`compose`很相似；它只是以相反的顺序调用函数，从最后到第一个，所以它可以这样实现:

```
const compose = (...fns) => x => fns.reverse().reduce((v, f) => f(v), x)
```

一个例子可以是计算一个在线商店的结帐总额:

```
const calculateTotal = pipe(
  applyAnySales,
  minusStoreCredit,
  applyCouponCode,
  addTaxes
)const cardValue = 250;const total = calculateTotal(cardValue)
```

如果你还有其他使用 `*reduce*` *的实例，请留下评论，以便我向你学习。*
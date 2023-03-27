# 您的 Javascript 中可能需要懒惰(懒惰评估)

> 原文：<https://itnext.io/you-may-need-laziness-in-your-javascript-f03e8a2d4629?source=collection_archive---------3----------------------->

![](img/2586ecfa127735cd8824724497fd592c.png)

也许有些时候你过度使用了数组方法 **map，reduce，filter，find 等等。这可以让应用程序使用更多本该使用的内存。**

让我们看一个例子:

```
const myArray = [1,2,3,4,5,6,7,8,9,10]myArray
  .map(*x* *=>* x ***** x)
  .filter(*x* *=>* x **%** 3 **===** 0)
```

当然，你已经做过几次类似的事情了，这里的“问题”是什么，我们每次链接数组方法都要创建新的数组。也就是说**。map** 用每个索引中的 x 创建另一个数组( **[ 1，4，9，16，25，36，49，64，81，100 ]** )，然后创建另一个过滤后的数组 **[ 9，36，81 ]** 在这种情况下，您可能不会有任何内存使用性能的问题，但是如果 myArray 非常大呢？你将再次创造另一个大的，一次又一次，直到你得到你想要的结果。

在这些情况下，带有 iterables 的惰性求值可能会很有用。但是首先，我们如何创建一个迭代器，以及它是如何工作的。

# 什么是迭代器？

迭代器是一个对象，它定义了一个序列，当序列结束时可能会返回值，这些序列可以是有限的，也可以是无限的。

迭代器对象将需要一个名为 **Symbol.iterator** 的属性函数，该函数将使用 **next()** 方法返回另一个对象。那个**下一个**方法将是返回序列的每个值的方法，并且如果序列完成或者没有使用具有属性 **value** 和 **done** as 的对象

```
const JustOneIterator = {
  [Symbol.iterator]: function () {
    return {
      next() {
        return { done: false, value: 1 }
      }
    }
  }
}const iterator = JustOneIterator[Symbol.iterator]()
iterator.next() // { done: false, value: 1 }
iterator.next() // { done: false, value: 1 }
iterator.next() // { done: false, value: 1 }
iterator.next() // { done: false, value: 1 }
// this will never return done: true
```

在这个例子中，你可以看到一个无限序列的 1。

让我们尝试一个有限数列

```
const rangeIterator = (from, to = Infinity, step = 1) => ({
  [Symbol.iterator]: function () {
    let done = false
    let value = 0
    return {
      next () {
        value = from
        done = from > to
        from = !done ? from + step : value
        return { done: done, value: value }
      }
    }
  }
})const iterator = rangeIterator(0, 4)[Symbol.iterator]()
iterator.next() // { done: false, value: 0 }
iterator.next() // { done: false, value: 1 }
iterator.next() // { done: false, value: 2 }
iterator.next() // { done: false, value: 3 }
iterator.next() // { done: false, value: 4 }
iterator.next() // { done: true, value: 5}var range = rangeIterator(0, 4)const arr = [...range] // [0, 1, 2, 3, 4]
```

因为这是一个有限迭代器，我们可以扩展成一个数组或者使用 **Array.from** 方法从迭代器创建数组。

```
Array.from(range) // [0, 1, 2, 3, 4]
```

现在，我们如何创建自己的 iterables 对象，以便映射、简化、过滤等。

有了这个实用程序，我们现在可以做一些事情:

## 对于有限的数组

```
const cubicPar = compose(
  map.bind(null, x => x * x * x),
  filter.bind(null, x => x %2 === 0)
)const fiveElements = take(10, cubicPar([1,2,3,4,5,6,7,8,9,10]))getValue(fiveElements)
// [ 8, 64, 216, 512, 1000 ]
```

## 用一个无限序列

```
const Numbers = generate(function () {
  let n = 0
  return {
    next: () => ({ done: false, value: n++ })
  }
})const cubirParRest = compose(
  map.bind(null, x => x * x * x),
  filter.bind(null, x => x %2 === 0),
  take.bind(null, 10),
  rest
)Array.from(cubirParRest(Numbers))
// [ 8, 64, 216, 512, 1000, 1728, 2744, 4096, 5832 ]
```

既然我们设法从这些实用程序中创建了一些迭代器，我们不想总是组合我们的函数，相反我们想链接**方法**。我们可以在**生成**方法上添加一个依赖项

```
const LazyIterable = {
  value: function () {
    return Array.from(this)
  },
  map: function (fn) {
    return map(fn, this)
  },
  reduce: function (fn, seed) {
    return reduce(fn, seed, this)
  },
  filter: function (fn) {
    return filter(fn, this)
  },
  find: function (fn) {
    return filter(fn, this).first()
  },
  first: function () {
    return first(this)
  },
  rest: function () {
    return rest(this)
  },
  until: function (fn) {
    return until(fn, this)
  },
  take: function (numberToTake) {
    return take(numberToTake, this)
  }
}const Numbers = generate(function () {
  let n = 0
  return {
    next: () => ({ done: false, value: n++ })
  }
}, LazyIterable)Numbers
  .map(x => x * x * x)
  .filter(x => x %2 === 0)
  .take(10)
  .rest()
  .value()// [ 8, 64, 216, 512, 1000, 1728, 2744, 4096, 5832 ]
```

我们可以做一个斐波那契数列

```
const Fibonacci = generate(function () {
  let n1 = 0
  let n2 = 1
  let value
  return {
    next: () => {
      [value, n1, n2] = [n1, n2, n1 + n2]
      return { value }
    }
  }
}, LazyIterable)Fibonacci
  .take(10)
  .value()
// [ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34 ]Fibonacci
  .map(x => x * 2)
  .filter(x => x % 4 === 0)
  .take(10)
  .value()
// returns 0, 4, 16, 68, 288, 1220, 5168, 21892, 92736, 392836 ]
```

我们甚至可以创建一个 LazyArray 对象，从数组中扩展

```
const LazyIterableDescriptor = Object.keys(LazyIterable).reduce((acc, key) => {
  acc[key] = { value: LazyIterable[key] }
  return acc
}, {})function LazyArray () {
  this.push.apply(this, arguments)
}LazyArray.prototype = Object.create(Array.prototype, Object.assign({
  constructor: {
    value: LazyArray
  }
}, LazyIterableDescriptor))LazyArray.from = function (iterable) {
  const lazy = new LazyArray()for (let element of iterable) {
    lazy.push(element)
  }
  return lazy
}const lazy = new LazyArray(1,2,3,4,5) 
// or LazyArray.from([1,2,3,4,5])lazy.value() // [1,2,3,4,5]
lazy.push(6) // 6
lazy.value() // [1,2,3,4,5,6]lazy
  .map(x => x * x * x)
  .value()
// [ 1, 8, 27, 64, 125, 216 ]
```

我们可以创建另一种类型对象，如 **Stash** 、 **LinkedList** 等等，它们都可以使用惰性迭代。

我希望这些信息对你们中的一些人有用，并且你们可以开始在应用程序中使用懒惰。

# 关于性能

如果我们运行这段代码(在我的电脑上)

```
function arrayOfNumbers (n) {
  const array = []
  let i = 0
  while (i < n) {
    array.push(i)
    i++
  }
  return array
}const array = arrayOfNumbers(100000000)let i = 0
while (i < 1000) {
  i++
  array
    .map(x => x * x)
    .filter(x => x % 2 === 0)
    .reduce((acc, x) => acc + x)
}
```

我得到了这个

```
$ time node __tests__/array.js
rss 67.5 MB
heapTotal 54.04 MB
heapUsed 32.11 MB
external 0.01 MBreal 0m57.297s
user 0m54.974s
sys 0m1.851s
```

但是如果我运行这个

```
const { LazyArray } = require('../lib/perezoso')function arrayOfNumbers (n) {
  const array = new LazyArray()
  let i = 0
  while (i < n) {
    array.push(i)
    i++
  }
  return array
}const lazyArray = arrayOfNumbers(100000000)let i = 0
while (i < 1000) {
  i++
  lazyArray
    .map(x => x * x)
    .filter(x => x % 2 === 0)
    .reduce((acc, x) => acc + x)
}
```

完成于

```
$ time node __tests__/lazy2.js
rss 33.38 MB
heapTotal 19.21 MB
heapUsed 9.36 MB
external 0.01 MBreal 0m19.202s
user 0m18.774s
sys 0m0.692s
```

但是改变迭代器的一切

```
const { Lazy } = require('../lib/perezoso')const Numbers = Lazy.generate(function () {
  let n = 0
  return {
    next () {
      return { value: n++ }
    }
  }
})const lazyArray = Numbers.take(100000000)let i = 0
while (i < 1000) {
  i++
  lazyArray
    .map(x => x * x)
    .filter(x => x % 2 === 0)
    .reduce((acc, x) => acc + x)
}
```

我得到了这个

```
$ time node __tests__/lazy.js
rss 32.72 MB
heapTotal 18.23 MB
heapUsed 9.16 MB
external 0.01 MBreal 0m12.556s
user 0m12.218s
sys 0m0.728s
```

另一件重要的事情是，如果我使用一个更大的**arrayOfNumbers(100000000)**乘*100(再加两个 0)**数组**版本和 **LazyArray** 版本会因内存错误而失败，但如果我使用唯一的迭代器示例(3rdone)就会完成

愿原力与你同在

【2019 年 5 月 9 日更新

使用 perezoso.js 的函数方法，我得到了一个与使用完全惰性对象非常相似的内存消耗结果，但是速度更快一些。

```
$ time node scripts/funtional.js
rss 32.92 MB
heapTotal 18.71 MB
heapUsed 10.12 MB
external 0.01 MBreal 0m7.166s
user 0m6.907s
sys 0m0.557s$ time node scripts/funtional2.js
rss 32.45 MB
heapTotal 17.23 MB
heapUsed 7.23 MB
external 0.01 MBreal 0m9.077s
user 0m8.793s
sys 0m0.679s
```

下面是所有的[https://github . com/higher comve/perezo so-js/tree/master/scripts](https://github.com/highercomve/perezoso-js/tree/master/scripts)
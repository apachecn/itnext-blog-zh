# 使用代理的 JavaScript 元编程

> 原文：<https://itnext.io/meta-programming-in-javascript-with-proxies-64fa4898070e?source=collection_archive---------0----------------------->

![](img/12b71e0c010a843609ff927db1f06f02.png)

由[弗朗西斯·麦克唐纳](https://unsplash.com/@photocapebreton?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

*以前，我写过一篇文章* *关于通过手动定义对象的属性/访问器描述符来定制对象。从精神上讲，本文是那篇文章的延续，因为在我将描述符与我们可以从代理中获得的描述符进行比较时，我将假设一些描述符的知识。如果对描述符及其好处有任何疑问，我推荐阅读我以前的文章。*

在我学习 JavaScript 之前，我研究了几年 Python。Python 的一个非常好的地方是它努力做到对开发人员友好，它的一个亮点是使列表易于使用。对于任何类似数组的数据结构，开发人员的一个常见需求是能够根据数组的结尾(而不仅仅是开头)访问元素。幸运的是，在 Python 中，这就像写`some_python_list[-1]`一样简单。

想象一下，当我学习 JavaScript 并试图用同样的方法获取数组中的最后一个元素时，我有多惊讶:

```
javaScriptArray[-1]; // undefined
```

真的吗？我不能用这个方法获取数组中的最后一个元素？我可以用类似 Ruby 的方法来代替吗？

```
javaScriptArray.last; // undefined
```

不。不幸的是，为了得到最后一个元素，我们仍然需要使用笨拙的`javaScriptArray[ javaScriptArray.length - 1 ]`技术。不幸的是，这使得代码更加冗长(因此有点难以阅读)，似乎一直是 ECMAScript 标准中的一个主要疏忽。

然而，虽然负索引不是一个我们可以立即使用的特性，但是 JavaScript 确实有一个非常方便的特性，可以让我们创建这样的行为(等等！)自己:[代理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)。

# 委托书

代理是一个对象，充当被执行的代码和被访问的实际对象之间的替身。代理的主要目的是公开一个接口，该接口拦截我们希望在对象上执行的各种操作，并以某种方式修改该操作的默认行为，以产生我们通常不会得到的结果。

这种修改代码执行的能力被称为元编程，它为 JavaScript 开发人员提供了很大的自由，可以对行为进行微小或剧烈的改变。这些偏差是以满足某些要求(称为“不变条件”)为代价的，目的是不允许代理的代码弯曲偏离那些在典型 JavaScript 对象上执行的操作太远。

使用代理的一些最佳用例是允许执行各种副作用(*例如*、控制台日志记录、API 调用)或扩展默认行为，如获取/设置属性。为了说明其中的一些概念，我们将继续我上一篇关于属性描述符的文章中的数据库示例，尽管我们不是处理数据库中的一行，而是处理数据库本身。

虽然代理可以拦截大量操作(称为“陷阱”)，但我们将举例说明的是:

*   `has`
*   `get`
*   `set`
*   `ownKeys`
*   `deleteProperty`

# 树立榜样

因为我们的例子是我们以前的数据库系统的扩展，所以我们将用一些虚拟数据创建一个简单的数组。

```
const db = [
    {
        name: 'John Doe',
        age: 28,
        lastModified: Date.now(),
        lastAccessed: Date.now()
    },
    {
        name: 'Jane Smith',
        age: 30,
        lastModified: Date.now(),
        lastAccessed: Date.now()
    },
        {
        name: 'Albert Einstein',
        age: 52,
        lastModified: Date.now(),
        lastAccessed: Date.now()
    }
];
```

当然，目前我们只能做典型阵列能做的事情。我们不能使用负索引，获取和设置属性完全基于 JavaScript 的开箱即用的`[[Get]]`和`[[Set]]`规范，检索数组的键实质上会给我们一个数字索引列表。

我们将改变这一切。我们的目标是实现以下行为:

*   允许负指数
*   以不同于我们习惯的方式获取和设置某些属性
*   轻松检索数据库行使用的键集(而不是我们行的数字索引)
*   无需显式调用`Array.prototype.find()`即可确定数据库中是否存在某行
*   使用一个简单的`delete`命令删除数据库行中的属性

# 代理我们的数据库

创建代理很容易:

```
const dbProxy = new Proxy(db, {});
```

我们将两个参数传递到我们的`new Proxy()`调用中:被代理的对象(我们上面的示例数据库)和处理程序对象，指定我们想要捕获的内容。无论我们在这个对象中没有指定什么陷阱，都意味着我们希望 JavaScript 引擎使用默认行为。

因此，让我们建立我们的自定义功能。

## ``get`

由于我们想使用负指数，我们可以创建一个名为`get`的陷阱，将负指数转换为正指数:

```
const handler = {
    get(tgt, prop, rcvr) {
        const _prop = +prop; if (Number.isInteger(_prop) && _prop >= 0)
            return tgt[_prop];
        if (Number.isInteger(_prop) && _prop < 0)
            return tgt[tgt.length + _prop];

        return tgt[prop];
    }
};
```

我们将`prop`转换为一个数字，并检查结果是否是一个整数(而不是一个浮点数，或者，如果`prop`是字母数字，则是一个`NaN`)，并将负的属性重写为正的。如果我们传递一个字母数字属性，我们只是把它当作我们试图正常访问一个属性。如果我们没有在最后指定`return tgt[prop]`，我们将无法访问目标数组的任何其他属性，比如`length`。

关于为我们的`get`陷阱指定的参数的两个快速注释:

1.  `tgt`表示我们正在代理的底层对象，`prop`表示我们正在访问的属性，`rcvr`表示接收代理。因此，如果我们说`dbProxy[-1]`，那么我们的参数最终将是`db`、`'-1'`、`dbProxy`，分别对应于我们的陷阱中列出的顺序。我将使用简写的`tgt`和`rcvr`来帮助保持代码简洁。
2.  在`get`陷阱中访问`rcvr`上的属性时，我们必须小心。这是因为`rcvr`本身就是代理，所以如果我们在陷阱中访问它的一个属性，我们就要重新进入`get`陷阱，我们冒着无限递归进入这个处理程序的风险，直到 JavaScript 引擎抛出一个 callstack 错误。

## 更多的

关于代理，我们还应该展示两件事:我们希望在任何时候访问它的一个属性时更新我们的`lastAccessed`字段，并指定一个属性`last`来检索数据库中的最后一行。

```
const handler = {
    get(tgt, prop, rcvr) {
        const _prop = +prop;

        if (prop in tgt) tgt[prop].lastAccessed = Date.now(); if (Number.isInteger(_prop) && _prop >= 0) 
            return tgt[_prop];

        if (Number.isInteger(_prop) && _prop < 0) {
            const posProp = tgt.length + _prop;
            if (posProp in tgt)
                tgt[posProp].lastAccessed = Date.now();
            return tgt[posProp];
        } if (prop === 'last') return rcvr[-1];

        return tgt[prop];
    }
};
```

现在，每当我们访问一个属性时，我们立即在我们正在访问的行上直接设置`lastAccessed`(假设我们的属性实际上在数组中)，如果我们曾经调用过`dbProxy.last`，我们简单地在我们的代理上访问`-1`属性(然后它再次被捕获，但是这次解析为直接从底层`tgt`数组中获取我们的一行，这样我们避免了无限递归)。

## ``set``

通常一个对象的属性可以很容易地随意改变，只需执行`obj.prop = val`。通过调用像`Object.preventExtensions()`或`Object.freeze()`这样的函数来防止属性被改变是可能的，但是这些更多的是全有或全无的方法，并且没有对什么可以被改变的*进行粒度控制。*

使用我们的代理，我们将锁定添加任何新的属性，除非我们调用`dbProxy.new = ...`，以及添加自定义排序我们的数据库的能力。

```
const handler = {
    set(tgt, prop, val, rcvr) {
        if (prop === 'sort') {
            const { by, dir } = val;
            tgt.sort(
                (a, b) => dir === 'asc'
                    ? (a[by] < b[by] ? -1 : 1)
                    : (b[by] < a[by] ? -1 : 1)
            );
        }

        if (prop === 'new') {
            tgt.push({
                ...val,
                lastModified: Date.now(),
                lastAccessed: Date.now()
            });
        }

        return true;
    }
};
```

现在，我们可以通过运行以下命令向数据库中添加一个新行

```
dbProxy.push({ name: 'Test', age: 100 });
```

我们的数据库没有任何变化，尽管调用`dbProxy.new = ...`会给我们想要的结果。我们也可以简单地通过调用`dbProxy.sort = { by: 'age', dir: 'asc' }`按照数据库的任何字段对数据库进行排序。

## `有'

假设我们想知道数据库中是否有符合特定标准的行。通常，我们会在数组上做`.find()`并传入一个合适的函数。然而，相反，我们想简单地说一些类似于`{ name: 'Albert Einstein' } in dbProxy`的东西，并得到一个`true`或`false`，而不必总是编写一个函数。

通常`in`要求我们检查一个字符串是否作为直接键存在于我们的对象上，但是 trap 允许我们写任何我们想要的存在检查逻辑。不幸的是，我们被限制在操作符的左边使用一个*字符串*，所以我们不能提供上面列出的对象。所以我们先把它转换成 JSON 就够了。

```
const handler = {
    has(tgt, prop) {
        const obj = JSON.parse(prop);
        const criteria = Object.entries(obj);

        return tgt.some(obj =>
            criteria.every( ([key, val]) =>
                obj[key] === val
            )
        );
    }
};
```

我们将字符串(参数`prop`)转换为一个对象，然后将其转换为一个条目数组，并检查我们的底层数组(参数`tgt`)中至少有一个文档符合所有标准。

```
const johnDoe = JSON.stringify({ name: 'John Doe' });
const albertEinstein = JSON.stringify(
    { name: 'Albert Einstein', age: 50 }
);johnDoe in dbProxy; // true
albertEinstein in dbProxy; // false: ages don't match
```

## ` ownKeys '

因为我们的数据库是一个文档风格的数据库，没有严格的模式，如果后面的文档添加了与前面不同的键，我们可能不知道当前使用的所有键(反之亦然)。因此，我们可以捕获像`Object.keys()`或`Reflect.ownKeys()`这样的调用来检索数据库行中使用的列(而不是数组的典型数字索引)。

```
const handler = {
    ownKeys(tgt) {
        const keys = tgt.reduce(
            (keys, doc) => {
                const docKeys = Object.keys(doc);
                docKeys.forEach(key => {
                    keys.add(key);
                });

                return keys;
            },
            new Set()
        );

        return [ ...keys, 'length' ];
    }
};
```

这个陷阱恰好是我们可以使用的最严格的陷阱之一，它限制了我们可以(并且必须)返回什么值，以及我们可能想要返回什么值。

我们必须只返回字符串或符号，任何不可配置的 own 属性必须在列表中——因此在我们的数组中返回了`'length'`,它当然没有在我们的任何行中使用，但是它的省略抛出了一个`TypeError`。

此外，`Reflect.ownKeys()`将返回预期的结果，而`Object.keys()`只返回一个空数组，所以使用这个陷阱需要我们注意代理上使用了哪些按键抓取函数。

## `删除属性'

我们将使用`deleteProperty`陷阱不删除代理本身的属性(实质上是从数据库数组中删除一个条目),而是删除数据库中每一行的属性。因此，如果我们决定不再需要一个`lastModified`字段，我们可以简单地通过调用`delete dbProxy.lastModified`来修剪我们的数据库。

```
const handler = {
    deleteProperty(tgt, prop) {
        const keys = this.ownKeys(tgt).slice(0, -1);

        if (keys.includes(prop)) {
            tgt.forEach(row => {
                delete row[prop];
            });

            return true;
        }

        return false;
    }
};
```

这个陷阱使用了一个有趣的特性。在代理处理程序的上下文中，`this`被绑定到处理程序本身，所以假设我们已经定义了其他陷阱，我们可以通过调用`this.some_trap()`在特定的陷阱中使用它们。

在我们的例子中，我们假设已经定义了前一节中定义的`ownKeys`陷阱，因此我们可以获得组成数据库模式的键，减去`'length'`属性(因此，调用`.slice(0, -1)`)。如果我们试图删除的属性实际上在我们的数据库中使用，我们从每一行中删除该属性，并返回`true`表示成功。否则我们返回`false`来表明这是不可能的。

# 比较代理和描述符

我们可以在代理和属性/访问器描述符之间进行许多比较，部分原因是两者通过以下方式打开了许多相同的大门:

*   允许在我们的行动中出现副作用
*   阻止手术发生
*   自定义某些操作的行为

那么，我们可以用描述符和代理做些什么呢？

从技术上讲，我们可以用访问器描述符实现负索引。每当底层数组发生变化时，这将需要大量的开销来维护负索引到其正对应项的映射，而对于代理，我们基于数学等式进行重新映射。此外，使用描述符将意味着负索引实际上是数组上的键，而不是代理上的纯幻像键。

我们使用`set`陷阱所做的事情大部分可以使用描述符来完成。通过设置`.sort`属性和我们在`.new`属性上插入的方式对数组进行排序，不需要代理就可以很容易地完成。然而，如果不调用像`Object.freeze()`这样的辅助函数，他们不会允许我们取消所有其他的`set`操作。

我们在代理上使用的所有其他陷阱都不能以其他方式完成。当我们考虑到代理的目的是为*元*程序提供一种方法，并从根本上改变典型的代码执行行为时，这是显而易见的，而描述符只是一种特性，允许在*正常*编程期间更好地控制对象。

# 结论

我们已经看到了在使用代理拦截和修改典型的 JavaScript 操作时可以发挥的威力。虽然代理不会拦截每一个命令(*例如*，我们不能重载数学运算符)，但它们确实拦截了足够多的命令，非常有用。

我们可以使用上面的陷阱实现的东西是无穷无尽的(例如，我们没有实现像 Python 中如此常见的`array[1:3]`这样的片段),我们也无法说明由代理暴露的其他一些非常重要的陷阱——主要是因为它们是基于函数的陷阱，我们只处理对象。它们非常重要，无论如何都值得单独讨论。

我将把这个要点[作为我们在本文中放在一起的代理的完整示例。](https://gist.github.com/ryandabler/fbfb05313c4c42168bdc84ea15579b57)
# 如何用 JavaScript 中的 map()、reduce()和 filter()简化代码库

> 原文：<https://itnext.io/15-useful-javascript-examples-of-map-reduce-and-filter-74cbbb5e0a1f?source=collection_archive---------0----------------------->

![](img/bf7740c41c0668af7298e6b4400ab364.png)

安德斯·吉尔登在 Unsplash[拍摄的照片](https://unsplash.com/photos/Sc5RKXLBjGg)

当你读到 **Array.reduce** 以及它有多酷的时候，你发现的第一个，有时是唯一的例子就是数字的和。这不是我们对‘有用’的定义。🤓

此外，我从未在真正的代码库中见过它。但是，我经常看到的是 7-8 行 for-loop 语句来解决常规任务，而 **Array.reduce** 可以在一行中完成。

最近我用这些伟大的函数重写了一些模块。令我惊讶的是，代码库变得如此简单。所以，下面是一份好东西的清单。

如果你有一个使用 **map** 或 **reduce** 方法的好例子，把它贴在评论区。👍

我们开始吧！

## 1.从数字/字符串数组中删除重复项

嗯，这是唯一一个没有关于**贴图** / **缩小** / **滤镜**的，但是它非常紧凑，很难不把它放在列表中。另外，我们还会在一些例子中用到它。

```
const values = [3, 1, 3, 5, 2, 4, 4, 4];
const uniqueValues = [...new Set(values)];// uniqueValues is [3, 1, 5, 2, 4]
```

## 2.简单搜索(区分大小写)

**filter()** 方法创建一个新数组，其中所有元素都通过了由所提供的函数实现的测试。

```
const users = [
  { id: 11, name: 'Adam', age: 23, group: 'editor' },
  { id: 47, name: 'John', age: 28, group: 'admin' },
  { id: 85, name: 'William', age: 34, group: 'editor' },
  { id: 97, name: 'Oliver', age: 28, group: 'admin' }
];let res = users.filter(it => it.name.includes('oli'));// res is []
```

## 3.简单搜索(不区分大小写)

```
let res = users.filter(it => new RegExp('oli', "i").test(it.name));// res is
[
  { id: 97, name: 'Oliver', age: 28, group: 'admin' }
]
```

## 4.检查是否有任何用户拥有管理员权限

方法测试数组中是否至少有一个元素通过了由提供的函数实现的测试。

```
const hasAdmin = users.some(user => user.group === 'admin');// hasAdmin is true
```

## 5.展平数组的数组

第一次迭代的结果等于:[…[]，…[1，2，3]]意味着它转换为[1，2，3]——这个值我们在第二次迭代时作为*‘ACC’*提供，依此类推。

```
const nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];
let flat = nested.reduce((acc, it) => [...acc, ...it], []);// flat is [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

我们可以通过省略一个空数组`[]`作为 **reduce()的第二个参数来稍微改进这段代码。**那么**嵌套**的第一个值将作为初始 **acc** 值。感谢[弗拉基米尔·埃法诺夫](https://medium.com/@vladimir_efanov?source=responses---------2---------------------)。

```
let flat = nested.reduce((acc, it) => [...acc, ...it]);// flat is [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

注意，在**变径**中使用[扩展操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)对性能没有太大影响。这个例子是衡量性能对您的用例有意义的一个例子。☝️

感谢[pawewolak](https://medium.com/@pawewolak?source=responses---------0---------------------)，这里有一条没有**阵法的捷径**

```
let flat = [].concat.apply([], nested);
```

还有 [*Array.flat*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) 也要来了，不过还是实验性的特性。

## 6.创建一个包含指定键的频率的对象

让我们对数组中每一项的“年龄”属性进行分组和计数:

```
const users = [
  { id: 11, name: 'Adam', age: 23, group: 'editor' },
  { id: 47, name: 'John', age: 28, group: 'admin' },
  { id: 85, name: 'William', age: 34, group: 'editor' },
  { id: 97, name: 'Oliver', age: 28, group: 'admin' }
];const groupByAge = users.reduce((acc, it) => {
  acc[it.age] = acc[it.age] + 1 || 1;
  return acc;
}, {});// groupByAge is {23: 1, 28: 2, 34: 1}
```

感谢[赛克里希纳](https://medium.com/@krishnasai952?source=responses---------8-31--------------------)建议这个！

## 7.索引对象数组(查找表)

我们可以构造一个对象，其中用户的 id 代表一个键(搜索时间不变)，而不是处理整个数组来通过 id 查找用户。

```
const uTable = users.reduce((acc, it) => (acc[it.id] = it, acc), {})// uTable equals:
{
  11: { id: 11, name: 'Adam', age: 23, group: 'editor' },
  47: { id: 47, name: 'John', age: 28, group: 'admin' },
  85: { id: 85, name: 'William', age: 34, group: 'editor' },
  97: { id: 97, name: 'Oliver', age: 28, group: 'admin' }
}
```

当你不得不像`uTable[85].name`一样通过 id 访问数据时，这很有用。

## 8.提取数组中每一项的给定键的唯一值

让我们创建一个现有用户组的列表。 **map()** 方法创建一个新数组，其结果是在调用数组中的每个元素上调用一个提供的函数。

```
const listOfUserGroups = [...new Set(users.map(it => it.group))];// listOfUserGroups is ['editor', 'admin'];
```

## 9.对象键值映射反转

```
const cities = {
  Lyon: 'France',
  Berlin: 'Germany',
  Paris: 'France'
};let countries = Object.keys(cities).reduce(
  (acc, k) => (acc[cities[k]] = [...(acc[cities[k]] || []), k], acc) , {});// countries is
{
  France: ["Lyon", "Paris"],
  Germany: ["Berlin"]
}
```

这个一行程序看起来相当棘手。我们在这里使用了[逗号运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comma_Operator)，这意味着我们返回括号中的最后一个值— `acc`。让我们以一种更适合生产和性能的方式重写这个例子:

```
let countries = Object.keys(cities).reduce((acc, k) => {
  let country = cities[k];
  acc[country] = acc[country] || [];
  acc[country].push(k);
  return acc;
}, {});
```

这里我们不使用 [spread 操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)——它在每次 **reduce()** 调用时创建一个新数组，这导致一个很大的性能损失:O(n)。取而代之的是旧好的**推()**法**。**

## 10.从摄氏温度值数组创建华氏温度值数组

可以把它想象成用给定的公式处理每个元素🤓

```
const celsius = [-15, -5, 0, 10, 16, 20, 24, 32]
const fahrenheit = celsius.map(t => t * 1.8 + 32);// fahrenheit is [5, 23, 32, 50, 60.8, 68, 75.2, 89.6]
```

## 11.将对象编码成查询字符串

```
const params = {lat: 45, lng: 6, alt: 1000};const queryString = Object.entries(params).map(p => encodeURIComponent(p[0]) + '=' + encodeURIComponent(p[1])).join('&')// queryString is "lat=45&lng=6&alt=1000"
```

## 12.将用户表打印为仅带有指定键的可读字符串

有时你想将带有选中键的对象数组打印成一个字符串，但是你意识到 **JSON.stringify** 并没有那么好😦

```
const users = [
  { id: 11, name: 'Adam', age: 23, group: 'editor' },
  { id: 47, name: 'John', age: 28, group: 'admin' },
  { id: 85, name: 'William', age: 34, group: 'editor' },
  { id: 97, name: 'Oliver', age: 28, group: 'admin' }
];users.map(({id, age, group}) => `\n${id} ${age} ${group}`).join('')// it returns:
"
11 23 editor
47 28 admin
85 34 editor
97 28 admin"
```

**JSON.stringify** 可以使字符串输出更具可读性，但不能作为表格:

```
JSON.stringify(users, ['id', 'name', 'group'], 2);// it returns:
"[
  {
    "id": 11,
    "name": "Adam",
    "group": "editor"
  },
  {
    "id": 47,
    "name": "John",
    "group": "admin"
  },
  {
    "id": 85,
    "name": "William",
    "group": "editor"
  },
  {
    "id": 97,
    "name": "Oliver",
    "group": "admin"
  }
]"
```

## 13.在对象数组中查找并替换键值对

假设我们想改变约翰的年龄。如果知道索引，可以写这行:`users[1].age = 29`。然而，让我们来看看另一种方法:

```
const updatedUsers = users.map(
  p => p.id !== 47 ? p : {...p, age: p.age + 1}
);// John is turning 29 now
```

这里，我们没有改变数组中的单个元素，而是创建了一个新的元素，只有一个元素不同。现在我们可以像`updatedUsers == users`一样通过引用来比较我们的数组，这非常快！React.js 使用这种方法来加速协调过程。[这里有一个解释。](https://blog.logrocket.com/immutability-in-react-ebe55253a1cc)

## 14.数组的并集(A ∪ B)

比导入和调用 [lodash 方法 union](https://lodash.com/docs/4.17.11#union) 更少的代码。

```
const arrA = [1, 4, 3, 2];
const arrB = [5, 2, 6, 7, 1];[...new Set([...arrA, ...arrB])]; // returns [1, 4, 3, 2, 5, 6, 7]
```

## 15.数组的交集(A ∩ B)

最后一个！

```
const arrA = [1, 4, 3, 2];
const arrB = [5, 2, 6, 7, 1];arrA.filter(it => arrB.includes(it)); // returns [1, 2]
```

作为练习，尝试实现数组的差分(A \ B)。提示:使用感叹号。

感谢 [Asmor](https://www.reddit.com/user/Asmor) 和[化身伟大的](https://www.reddit.com/user/incarnatethegreat)对#9 的评论。

# 就是这样！

如果您有任何问题或反馈，请在下面的评论中告诉我，或者在 [Twitter](https://twitter.com/alex_permiakov) 上 ping 我。

## 如果这有用，请点击拍手👏下面扣几下，以示支持！⬇⬇ 🙏🏼

以下是我写的更多文章:

[](https://medium.freecodecamp.org/how-to-get-started-with-internationalization-in-javascript-c09a0d2cd834) [## 如何开始 JavaScript 的国际化

### 通过调整我们的应用程序以适应不同的语言和国家，我们提供了更好的用户体验。对用户来说更简单…

medium.freecodecamp.org](https://medium.freecodecamp.org/how-to-get-started-with-internationalization-in-javascript-c09a0d2cd834) [](https://medium.com/@alex.permyakov/production-ready-node-js-rest-apis-setup-using-typescript-postgresql-and-redis-a9525871407) [## 使用 TypeScript、PostgreSQL 和 Redis 设置生产就绪 Node.js REST APIs。

### 一个月前，我接到一个任务，要建立一个简单的搜索 API。它所要做的就是从第三方获取一些数据…

medium.com](https://medium.com/@alex.permyakov/production-ready-node-js-rest-apis-setup-using-typescript-postgresql-and-redis-a9525871407) 

感谢你阅读❤️
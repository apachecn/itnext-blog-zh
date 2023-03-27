# JS 和 ES6 针对较短语句的酷提示

> 原文：<https://itnext.io/js-and-es6-cool-hints-for-shorter-statements-be766983a75e?source=collection_archive---------5----------------------->

## java 描述语言

## 已知或未知的快捷方式…

众所周知，Javascript 或 JS 是一种强大的编程语言，它允许创建许多令人印象深刻的应用程序。

![](img/0d43e2103a3d2db92049e22e3b1abe0c.png)

照片由 [parth upadhyay](https://unsplash.com/@parthposts?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

顺便说一下，让我们停止称它为 Javascript，而更喜欢用 **JS** 作为 [Brendan Eich](https://mobile.twitter.com/BrendanEich) 几天前宣称的名称:

无论如何，在这篇文章中，我将分享一些我日常使用的有用技巧，使我的陈述更简短，有时更具可读性。

# 字符串操作

## —字符串串联数组—

也许你知道`join`函数，我肯定你知道。但是如果我告诉你有一个秘密的方法来声明分隔符呢:

我们习惯说:

```
['a', 'b', 'c', 'd'].join(', ') => 'a, b, c, d'
```

我给你介绍一种更简单的方法:

```
['a', 'b', 'c', 'd'].join`, ` => 'a, b, c, d'
```

> 是的，它节省了 2 个角色，但这是展示你技能的一种很酷的方式*😉*

## —字符串拆分—

如前所述，我们可以对`split`函数使用相同的快捷方式:

之前:

```
'abcd'.split('') => ['a', 'b', 'c', 'd']
```

然后:

```
['a', 'b', 'c', 'd'].split`` => ['a', 'b', 'c', 'd']
```

> 再一次，2 个角色被保存，这是一个很酷的方式来证明你不可思议的技能*😉*

## —弦铸造—

要将变量转换成字符串原语，有很多方法:

```
let a = 4; a.toString()   =>  '4'
String(3)                 =>  '3'
'' + true                 =>  'true'
`${ { 'a': 7 } }`         =>  '[object Object]'
```

## —访问一个角色—

访问字符有助于在呈现字符串时对其进行格式化，为此，有两种可能的方法:

```
'IamAString'[3] => 'A'
let b = 'IamAnotherString'; b.charAt(9) => 'r'
```

> 实际上，字符串只是一个字符数组，对吧！？*😊*

# 布尔检查

## —一体化检查—

在代码中的某些地方，您可能需要检查几个布尔值，因此您最终会得到以下代码:

```
isConnected && isAdmin && !isLoading && !dataFetched && …
```

断言可能会很长…相反，我们可以结合使用`every`函数和一个简单的断言:

```
[
isConnected,
isAdmin,
!isLoading,
!dataFetched
].every(x => x === true)
```

这种方式使您的代码更具可读性，并易于修改。

> 当您的检查使用 *||* 运算符时，这也可以与`some`函数一起使用。

## —短路评估—

这是非常常见的检查和调用函数如下:

```
if (isAdmin) {
  deleteUser(userToDelete);
}
```

然而，通过*短路评估*，您可以将 3 行代码转换成 1:

```
isAdmin && deleteUser(userToDelete)
```

相当得心应手，对！？

## —无效运算符—

好吧，我猜你知道:

```
const isSimpleUser = !user.hasAdminRights()
```

但是，关于:

```
const isUserDefined = !!user
```

> 事实上，`!`操作符将一个原语或一个对象转换成它的*布尔值*并返回它的`falsy`值。再次应用`!`会将其转换为`truthy`值。

或者甚至:

```
const user = getLocalUser() ?? new User()
```

当左边的成员为`null`或`undefined`时[无效合并](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator)操作符`??`返回右边的成员。与`||`运算符不同的是，当左边的运算符为 *falsy* 时，这个运算符返回右边的成员。

> 小心，也许你的编译器不会接受这种语法。为了克服这个障碍，可以考虑添加[相关的 Babel 插件](https://babeljs.io/docs/en/babel-plugin-proposal-nullish-coalescing-operator)。

# 设置功能

## —数组函数—

`**pop**`

这个函数用于移除数组的最后一个值，但是你知道它会返回值吗？

而这是移除的值，所以是数组的最后一个值！

`**shift**`

我猜你看到它来了…是的，它和`pop`完全一样，`shift`返回数组的第一个值，当然，这个值已经通过调用这个方法被移除了。

## — ES6 阵列功能—

`**Removing duplicates**`

如果你知道`Set`在 JS 中是如何工作的，也许你知道这个技巧，它使用了扩展操作符`…`:

```
const a = [1, 1, 2, 2, 3, 4, 4, 5]function unique (arr) {
  return [...new Set(arr)]
}const b = unique(a) => [1, 2, 3, 4, 5]
```

它有一个使用`reduce`数组函数的版本，但是没有前面的代码片段那么短。

```
function removeDuplicates(arr) {
  return arr.reduce((acc, cur) => {
    if(acc.indexOf(cur) === -1) acc.push(cur);
    return acc;
  }, []);
}
```

# 对象、析构和函数

*用 ES6 特性增强您的代码……*

## —箭头功能和对象—

我毫不怀疑你知道并使用箭头功能。

```
(name) => {
  return `Hello World ${name}!`
}
```

有一个有用的快捷方式可以避免大括号并简化上面的代码:

```
(name) => `Hello World ${name}!`
```

这也可以用于对象和对象文字:

```
const rights = (user) => user.rights// ORconst customRights = (user) => ({
  canAdd: user.rights.addition,
  canEdit: user.rights.edition,
  canDelete: user.rights.deletion
})
```

> *⚠️* 小心，记住箭头函数的作用域是它们父函数的作用域。

## —覆盖对象属性—

如你所知，我们可以像下面这样析构一个对象:

```
const { name } = { name: 'Adrien' }
```

例如，我们可以使用扩展运算符`…`来合并数组:

```
const a = [1, 2, 3]
const b = [4, 5, 6]const c = [...a, ...b] => [1, 2, 3, 4, 5, 6]
```

该运算符也可用于对象:

```
const objA = { a: 1, b: 2 }
const objB = { ...objA, c: 3 }
```

甚至覆盖属性:

```
const objC = { ...objB, c: 4 } => { a: 1, b: 2, c: 4 }
```

> *⚠️* 注意:被覆盖的属性必须出现在`…`之后，因为最后一个值被考虑在内。

## —参数和解构—

有时，你有一个带参数的函数，你只使用一个属性或一组属性。多亏了对象析构，你可以得到你需要的属性:

```
// Beforefunction parseValue (data) {
  const val = data.value
  // ...
}// Afterfunction parseValue ({ value }) {
  // you can directly use value
  // ...
}
```

# 结尾词

这些快捷方式相对来说比较方便。根据你的观点，它们可能看起来更易读，否则，对初级开发人员来说是不可理解的。反正用不用都是你自己的事。

我强烈建议你在 Codewars 上面对一些挑战:

[](https://www.codewars.com/) [## 代码战争:通过挑战达到精通

### Codewars 是开发人员通过挑战掌握代码的地方。在道场训练形，达到你的最高境界…

www.codewars.com](https://www.codewars.com/) 

这是提高和挑战自己技能的好方法。玩几个不同难度的谜题，我学到了很多。尤其是发现 JS 函数的意外用法时…
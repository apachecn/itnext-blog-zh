# Javascript-ES 建议的可选链接

> 原文：<https://itnext.io/optional-chaining-for-javascript-es-proposal-441eb63e14c8?source=collection_archive---------4----------------------->

![](img/2c9ba86aefe386a8b14f1cf84372442d.png)

昨天我们看到了 [nullish 合并](https://solankiamit.com/nullish-coalescing-for-javascript)，今天我们来看看可选链接另一个令人敬畏的 ESnext 特性。不谈这个太棒了。

就像 nullish 合并一样，可选链接帮助我们处理`undefined`和`null`情况。让我们考虑下面的例子

```
letcompany={
  *name*:'Github',
  *revenue*:2000,
  *users*:[
    { *name*:'John', *handle*:'@john'},
    { *name*:'doe', *handle*:'@doe'},
  ],
  *getUserNames*(){
    *return users.map*(*user* => *user.name*)
  },
}
```

要访问这个对象的值，如果它们或对象可以是`undefined`或`null`，我们通常会做以下事情。

```
const *companyName* =
  company!== *undefined* &&company!== *null* ? *company.name* : *undefined*
```

与可选链接相比，我们可以

```
const *companyName* = *company?.name*
```

语法

```
*obj?.*prop // *optional static property access* obj*?.*[expr] // *optional dynamic property access
func?.*(...args) // *optional function or method call*
```

让我们看看如何处理访问对象属性的多种方式。

# 静态属性

```
letcompany={
  *name*:'Github',
  *revenue*:2000,
  *users*:[
    { *name*:'John', *handle*:'@john'},
    { *name*:'doe', *handle*:'@doe'},
  ],
  *getUserNames*(){
    *return users.map*(*user* => *user.name*)
  },
}// *with optional Chaining* const *companyName* = *company?.name*// *without optional Chaining* const *companyName* =
  company!== *undefined* &&company!== *null* ? *company.name* : *undefined*
```

# 动态性能

```
let company = {
  name: 'Github',
  revenue: 2000,
  users: [
    { name: 'John', handle: '[@john](http://twitter.com/john)' },
    { name: 'doe', handle: '[@doe](http://twitter.com/doe)' },
  ],
  getUserNames() {
    return users.map(user => user.name)
  },
}// with optional Chaining
const companyName = company?.['name']// without optional Chaining
const companyName =
  company !== undefined && company !== null ? company['name'] : undefined
```

# 函数调用

```
let company = {
  name: 'Github',
  revenue: 2000,
  users: [
    { name: 'John', handle: '[@john](http://twitter.com/john)' },
    { name: 'doe', handle: '[@doe](http://twitter.com/doe)' },
  ],
  getUserNames() {
    return users.map(user => user.name)
  },
}// with optional Chaining
company.getUserNames?.()// without optional Chaining
const companyName =
  company !== undefined &&
  company !== null &&
  Object.prototype.hasOwnProperty('getUserNames') &&
  typeof company.getUserNames === 'function'
    ? company['name']
    : undefined
```

# Dom 访问

可选链接支持 DOM 及其方法，所以我们可以这样做

```
const val = document.querySelector('input#name')?.value
```

如果表达式失败，可选链接将总是返回`undefined`。

```
const value = company?.name// 'value' will be undefined if company is undefined.
```

因此，我们可以将可选链接与 nullish 合并结合起来处理这些情况。

```
const value = company?.name ?? 'default name'// 'value' will be 'default name' if either the company or name is undefined or null.
```

用一个完整的例子来结束它

```
let company = {
  name: 'Github',
  revenue: 2000,
  users: [
    { name: 'John', handle: '[@john](http://twitter.com/john)' },
    { name: 'doe', handle: '[@doe](http://twitter.com/doe)' },
  ],
  getUserNames() {
    return users.map(user => user.name)
  },
}// Static access with default value
const value = company?.name ?? 'default name'// Dynamic access with default value
const companyName = company?.['name'] ?? 'default value'// Function call
company.getUserNames?.()
```

可选的链接和无效合并带来了很多好处，降低了复杂性，使代码可读。给他们一个机会，并在下面的评论中告诉我你的观点。

今天可以用下面这些巴别塔插件来使用。

在推特上关注我

*原载于 2019 年 11 月 30 日*[*https://solankiamit.com*](https://solankiamit.com/optional-chaining-for-javascript-es-proposal)*。*
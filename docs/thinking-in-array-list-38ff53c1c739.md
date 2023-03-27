# 在数组/列表中思考

> 原文：<https://itnext.io/thinking-in-array-list-38ff53c1c739?source=collection_archive---------1----------------------->

## TypeScript/JavaScript 中的函数编程

![](img/070823dfede4a9854648843add0849f2.png)

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fthinking-in-array-list-38ff53c1c739)

# 为什么函数式编程喜欢列表？

你可能听说过这个:

> Lisp 中的一切都是列表！

事实是，并不是*一切*在函数式编程的世界里都是列表，但是列表起着非常重要的作用。这是为什么呢？

我认为答案在于**简单性**和**可读性**。

让我们以 TypeScript/JavaScript 为例。

首先让我们看看两个事实:

*   一，在 TypeScript/JavaScript 中，列表就是数组。
*   数组有一系列预定义的方法，如 map，filter，includes 等。那些方法就像常识一样，每个熟悉这门语言的人都知道它们的意思。

编码时，我们希望**用最少的术语表达最多的信息**。从这个意义上说，如果我们可以用预定义的方法在数组中表达我们的意图，为什么我们需要使用定制的函数/方法呢？

# 数组思维的例子

## 例 1“在任何”

假设您有一张信用卡，该卡有几种状态:活动、损坏、取消和关闭等。要求是当卡状态不是损坏/取消/关闭时允许某些操作。

这可以用“或”*或*“| |”(我知道，很难读作‘或或或’:p):

```
public disallowActon(cardStatus: string): boolean {
  return cardStatus === DAMAGED' || cardStatus === 'CANCELED' || cardStatus === 'CLOSED';
}
```

但是你可以看到我们在这里和那里写了几个`cardStatus`。现实中，你在这里想表达的是，如果我的卡状态是状态中的任意一个**，就不允许。原来，**中的**和**中的**都是数组中预定义的方法:**

```
public disallowActon(cardStatus: string): boolean {
  return ['DAMAGED', 'CANCELED', 'CLOSED'].includes(cardStatus);
}
```

## 例 2: **【任意测试】**

你正在编写一个验证器来验证一个用户输入地址，规则是，如果任何一个地址字段为空，它就是无效的。

通常我们也可以使用 OR 语句:

```
public isValid(address: Address): boolean {
  return address.lineOne === ''
        || address.cityName === ''
        || address.state === ''
        || address.postalCode === '';
}
```

话又说回来，在现实中，你这里要表达的是:对于 address 中的每一个字段，如果**测试**中的任何一个为真(我们也称之为**谓词)，**我们将其标记为无效。结果是我们在数组上有一个预定义的方法:

```
public isValid(address: Address): boolean {
  return [address.lineOne, address.cityName,
          address.state, address.postalCode].some(this.isEmpty);
}private isEmpty(field: string): boolean {
  return field === '';
}
```

上述代码利用了**点自由样式**。如果你对此不熟悉，请查看我的另一篇文章:

[](/whats-point-free-style-in-typescript-39337000c8cb) [## TypeScript 中的点自由样式是什么？

### Typescript/JavaScript 中的函数编程

itnext.io](/whats-point-free-style-in-typescript-39337000c8cb) 

## 示例 3:“过滤器”

假设您想打印用空格分隔的用户名。用户可能有中间名，也可能没有。所以你可能会有这样的话:或者郑。

通常这可以使用字符串操作来编写:

```
public fullName(nameObj: Name): string {
    let fullName = nameObj.givenName;
    if (nameObj.middleName) {
        fullName += ` ${nameObj.middleName}`;
    }
    fullName += ` ${nameObj.familyName}`; return fullName;
}
```

实际上，这里要表达的是，打印所有姓名字段，排除中间空着的姓名。原来我们在数组上有一个预定义的方法:

```
public fullName(nameObj: Name): string {
    return [nameObj.givenName, nameObj.middleName, nameObj.familyName]
           .filter(this.notEmpty)
           .join(' ');
}private notEmpty(field: string): boolean {
    return field !== '';
}
```

*本文属于 TypeScript/JavaScript 中* [*函数式编程系列。*](https://medium.com/@hamxiaoz/functional-programming-in-typescript-javascript-d9d79663bc4)
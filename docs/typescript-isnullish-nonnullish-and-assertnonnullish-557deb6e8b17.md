# TypeScript: isNullish、nonNullish 和 assertNonNullish

> 原文：<https://itnext.io/typescript-isnullish-nonnullish-and-assertnonnullish-557deb6e8b17?source=collection_archive---------2----------------------->

![](img/e55d750af4e000a3b783f0b0d65dd5ef.png)

[巴蒂斯特·比松](https://unsplash.com/fr/@shoots_of_bapt_)

我们在 [NNS-dapp](https://nns.ic0.app/) 中实现了一些宝石，让我们的开发人员的日常生活更加轻松。其中，下面三个小的 [TypeScript](https://www.typescriptlang.org/) 函数被证明是非常有用的。

# isNullish

你多久编写一次`if…else`语句来检查一个对象是`undefined`还是`null`？

```
// Pseudo code (assuming optional chaining does not exist 😉)
const test = (obj: MyObject | undefined | null) => {
    if (obj === undefined || obj === null) {
        return;
    }

    obj.fn();
}
```

多亏了[泛型](https://www.typescriptlang.org/docs/handbook/2/generics.html)和[类型谓词](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)，我们开发了一个助手，让我们在保持类型安全的同时避免代码重复。

```
export const isNullish = <T>(argument: T | undefined | null): argument is undefined | null =>
   argument === null || argument === undefined;
```

泛型`T`的使用将把函数的使用范围扩大到我们在项目中声明的类型。

至于类型保护，它缩小了 TypeScript 的范围，以理解变量确实来自特定的预期类型。换句话说，这个函数使 TypeScript 理解这个参数——如果它匹配函数的检查——确实是`undefined`或`null`。

我们可以如下使用助手:

```
const test = (obj: MyObject | undefined | null) => {
    // 1\. Avoid code duplication
    if (isNullish(obj)) {
        return;
    }

    // 2\. TypeScript checks it is defined
    obj.fn();
}
```

# 非努利语

有时候我们需要相反的东西，我们需要知道某个东西是否被定义。虽然我们可以否定前面的速记函数，以了解情况是否如此，但我们也希望保持类型安全。

这可以通过使用实用程序[不可空](https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype)通过排除`undefined`和`null`来构造类型来实现。

```
export const nonNullish = <T>(argument: T | undefined | null): argument is NonNullable<T> =>
   !isNullish(argument);
```

这样，TypeScript 将理解确实定义了一个对象。

```
const test = (obj: MyObject | undefined | null) => {
    //1\. Avoid code duplication
    if (nonNullish(obj)) {
        // 2\. TypeScript checks it is defined
        obj.fn();
    }
}
```

# assertNonNullish

除了检查指定的条件是否正确之外，如果不正确，抛出错误也很方便。特别是开发带有断言模式的防护。

为此，我们可以用 TypeScript 3.7 中引入的[断言签名](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html)概念来增强我们之前开发的功能。

使用`asserts`关键字和一个条件，我们可以让 TypeScript 知道一个助手将执行一个检查，如果条件不满足就抛出一个错误。

```
export class NullishError extends Error {}

export const assertNonNullish: <T>(
   value: T,
   message?: string
) => asserts value is NonNullable<T> = <T>(value: T, message?: string): void => {
   if (isNullish(value)) {
      throw new NullishError(message);
   }
};
```

应用到我们前面的代码片段，我们可以转换函数来执行代码，只有当警卫是匹配的。

```
const test = (obj: MyObject | undefined | null) => {
    // 1\. Avoid code duplication
    // 2\. TypeScript understands it might throw an error
    assertNonNullish(obj);

    // 3\. TypeScript checks it is defined
    obj.fn();
}
```

# 结论

我发现这些助手非常有用，以至于我现在在我最近的作品中使用它们——包括我新的“秘密疯狂”副业——我打赌你也会这样做😁。

更多冒险，请在🖖的推特上关注我
# 设计自以为是的功能 API(类型脚本)🔺 🔻

> 原文：<https://itnext.io/designing-an-opinionated-functional-api-typescript-e7e89e8ab338?source=collection_archive---------1----------------------->

![](img/765fec845e6a7cba3c25572d6bd42ecd.png)

这张图片是一个伟大的“石器时代的女王”专辑的艺术作品

在这篇文章中，我打算与你分享我选择的处理我领导的项目中的错误的方法，以及我选择向我的用户公开的功能 API。

处理错误是设计 API 时的一个关键部分，因为最终你编写的优秀软件会失败，而不是以一种来自你的代码或你使用的第三方的神秘异常的形式，或者以一种更好的分类错误的形式。

一个程序可能因为很多原因而失败，例如，当物理资源被超过时的运行时崩溃，或者由于误用或未处理的异常等等..

将错误视为返回的数据类型迫使您映射所有可能的失败并解决它们。

**回到我的案例，让我们对错误进行分类**

首先，我遇到了程序中会导致错误的部分。
在这种情况下，我们希望指出具体的失败，所以我们将定义一个枚举类型，将这种情况表示为 **ErrorType** 。

![](img/b6512e0903461f4ee59ffcc023577afc.png)

```
export enum ErrorType {
    *UnsupportedFormat* = 'UnsupportedFormat',
    *InvalidParameter* = 'InvalidParameter'
}
```

之后，我们将定义一个引用 errorType 的类型。
此时，我们刚刚创建了一个数据/模型表示，我们的目标是使我们的开发体验更加流畅。

```
export interface ErrorClassification {
  message: string;
  type: ErrorType;
}
```

**旁注—在这里使用 enum 而不是 const enum 并不是禁忌...

我们基本上想要标记一个错误案例，并将其传递给程序的下一部分进行处理。

由于我受到了 Rust lang 的启发，我想创建一个结果类型，以提供一个密切的开发体验。

```
export type Result<E, T> = Error<E> | Just<T>;
type Just<T> = T;
type Error<E> = Just<E>; //LOL :)
```

为了知道在运行时 Result 将返回哪种类型，我创建了 isError() util。util 包含一个 E 类型的约束来扩展我的特定错误分类类型+双重检查我自己的错误分类的“消息”和“类型”属性的存在性..

## 随波逐流

花点时间想想下面的函数返回类型。

```
function koko(input: string): **Result<ErrorClassification, string>** {
  if (...leadToErr) {
    return {
      message: 'cannot use given input',
      type: ErrorType.badInput,
    };
  }
  return doSomethingWitthInput(input);
}
```

作为以下项目的替代方案:

```
function koko(input: string): **string** {
  if (...leadToErr) {
    throw new ***Error***('cannot use given input');
  }
  return doSomethingWitthInput(input);
}
```

那么，使用**结果**类型会立即告诉你这段代码可能会返回一个失败。

第二段代码表明这个函数只是返回一个字符串，但是:
1。我应该有一个注释来记录这个函数可能抛出一个错误。
2。一旦抛出一个错误，程序就会崩溃，如果我没有捕捉到它，我可能会捕捉到它，但在这种情况下的精神状态不是“流动的”。

现在我们在库代码中使用了一个 koko()函数，它显式地返回了 **Error | string** ，所以我必须对它做些什么。
但是我如何从**结果<错误分类，字符串>** 中提取实际结果呢

以下是我不喜欢的部分:

```
const result: Result<ErrorClassification, string> = koko();
if (isError(result)) {
  return result as ErrorClassification;
}
***console***.log(result as string);
```

我会解释，Result(或者)不是运行时存在的东西，由于我们在 Typescript 中没有模式匹配，transpiler (tsc)不会以优雅的方式为我解析类型，如下面的 Rust 片段所示。

```
match foo.run(result) {
    *Ok*(_) => {}
    *Err*(e) => {...},
}
```

> [*模式是 Rust 中的一种特殊语法，用于匹配复杂和简单类型的结构。将模式与*](https://doc.rust-lang.org/book/ch18-00-patterns.html) `[*match*](https://doc.rust-lang.org/book/ch18-00-patterns.html)` [*表达式和其他结构结合使用，可以更好地控制程序的控制流。*](https://doc.rust-lang.org/book/ch18-00-patterns.html)

但是等等，我不希望我的库消费者使用 isError()函数来提取他们的结果！！
我希望他们(和我自己)能写出令人愉快的代码，比如:

```
*result*.ok(transformValFn)
    .map((transformedVal) => whatever..)
    .err((transformedErr: ErrorClassification) => {
      //do something
      return ***result***;
    });
```

## 结果结构:

为了制作这样的 API，我们需要一个结构，输入 ResultT:

ResultT 有两个方法，ok()和 err()用于注册结果值或错误数据类型，还有一个 map()方法用于更常规/惯用的转换。

[ResultT](https://en.wikipedia.org/wiki/Functor) 是函子吗？
让我们按照[FP-行话](https://github.com/hemanth/functional-programming-jargon#functor)基于[幻境](https://github.com/fantasyland/fantasy-land)规范，

`1\. **Identity**: object.map(x => x) ≍ object`

```
const isInstanceofResultT = result.map((a) => a) instanceof ResultT;
***console***.log('isInstanceofResultT', isInstanceofResultT); // true
```

ok()，err()和 map()都返回 ResultT 标识。

`2\. **composition**: object.map(compose(f, g)) ≍ object.map(g).map(f)`

```
function one(s: string) {
  const one = 'one';
  return one + s;
}
function two(s: string) {
  const two = 'two';
  return two + s;
}const fooBox = new ResultT('foo');const isResultT =
  fooBox.map(() => two(one('foo'))).map(***console***.log) ===
  fooBox
    .map(() => one('foo'))
    .map(two)
    .map(***console***.log);
***console***.log('isResultT', isResultT); //true both 'twoonefoo'
```

使用 resultT 很容易(将来我可能会发布一个 ResultT 实用程序)

```
export function myAwesomeAPIFn(*input1: string*, input2: Foo): ResultT<ErrorClassification, string> 
{
  return new ResultT(_myAwesomeAPIFnThatMayFail(*input1*, input2));
}
```

暂时就这些了:)

一个可运行的例子可以在这里找到:

[](https://github.com/LironHazan/advanced-patterns-in-typescript/blob/master/functional/error-handling%20/error-classifier.ts) [## LironHazan/高级模式输入脚本

### 打字稿中的高级模式。为 LironHazan/advanced-patterns-in-typescript 开发做出贡献，创建一个…

github.com](https://github.com/LironHazan/advanced-patterns-in-typescript/blob/master/functional/error-handling%20/error-classifier.ts) 

如果你喜欢这篇文章，你可能会喜欢我上一篇关于使用 [fp-ts](/fp-ts-in-action-d7d5f41c4858) 的文章。

干杯，
立伦！
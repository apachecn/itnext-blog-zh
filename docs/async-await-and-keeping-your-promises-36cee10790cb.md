# 异步，等待，信守承诺

> 原文：<https://itnext.io/async-await-and-keeping-your-promises-36cee10790cb?source=collection_archive---------1----------------------->

![](img/ac7549c8a097e900ddd04ac1b8600635.png)

照片由 [Cytonn 摄影](https://unsplash.com/@cytonn_photography?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

作为 ECMAScript 2017 的一部分引入的`async function`和`await`关键字确实在承诺之上提供了非常有用的语法糖。`Promise`本身在编写异步代码时提供了回调的替代方法。承诺可以被链接，其内置的方法如`all`、`any`和`race`有助于管理多个异步任务。

检查下面的例子，这里的`getData`函数模仿异步行为。在现实世界中，你可以认为它是你的数据层，使用像`fetch`这样的函数或者第三方库，它们仍然使用回调来进行异步编程。

```
const getData = (n: number) => {
    return new Promise<number>((res, rej) => {
        if (n === 3) {
            rej('Can not use 3.');
            return;
        }
        res(n * n);
    });
}
```

如果必须的话，获取 2 的数据，并根据响应获取 3 和 4 的数据，那么代码将如下所示。

```
const check = () => {
    getData(2)
        .then(x2 => {
            console.log(x2);
            return getData(3);
        })
        .then(x3 => {
            console.log(x3);
            return getData(4);
        })
        .then(x4 => {
            console.log(x4);
        }).catch((ex) => { // This is catch handler
            console.log('Error occurred : Check with Promise.');
            console.log(ex);
        });
}
```

如果我们使用 async 和 await，相同的代码将更易读和易于理解。

```
const check = async () => {
    try {
        const x2: number = await getData(2);
        console.log(x2);
        const x3: number = await getData(3);
        console.log(x3);
        const x4: number = await getData(4);
        console.log(x4);
    } catch (ex) { // This is catch block
        console.log('error occurred : check with async and await.');
        console.log(ex);
    }
}
```

错误处理仍然是一个挑战。如果承诺被拒绝，那么要么执行 catch 处理程序，要么抛出一个异常。使用`await`关键字，处理拒绝承诺的唯一方法是`try-catch`阻塞。

这在某些情况下可能行得通，但是如果在加载 3 和 4 的数据时出现错误也没关系呢？catch 块没有给出处理控制流的好方法。您可能最终会为每个`await`使用单独的`try-catch`块，这将使问题恶化。

像`go`这样的语言有不同的哲学来处理错误。`go`将`error`与`exception`分离，用普通值作为返回参数传递错误。
让我们看看当我们尝试这种哲学时会发生什么。

> 永远信守诺言

让我们改变 getData 函数，使它永远不能拒绝承诺。承诺将总是被解决，并且错误将通过返回类型被报告。

```
type PromiseResponse<T> = Promise<[string] | [null, T]>;

const getData = (n: number) : PromiseResponse<number> => {
    return new Promise((res) => {
        if (n === 3) {
            // no reject here 
            res(['Can not use 3.']);
            return;
        }
        res([null, n * n]);
    });
}
```

我在这里声明了一个类型`PromiseResponse`，这是一个承诺返回元组，将帮助 TypeScript 进行更好的语法检查。

*   第一项将是 error : string 或 null。
*   第二项将是 T 类型或未定义的实际结果。

```
const check3 = async () => {
    const [e2, x2] = await getDataV2(2);
    // Here for TypeScript x2 is either number or undefined
    if (x2 === undefined) {
        console.log('Error while fetching data for 2');
        return;
    }
    // As x2 is checked for undefined
    // at this line x2 is number
    console.log(x2);

    // now fetch data for 3 and 4
    const [e3, x3] = await getDataV2(3);
    if (x3 !== undefined) {
        console.log(x3);
    }

    const [e4, x4] = await getDataV2(4);
    if (x4 !== undefined) {
        console.log(x4);
    }
}
```

有了新的方法，代码不需要使用`try-catch`块，我们可以更好地控制流程。

我将这种技术用于位于 UI 和底层数据之间的应用层，它使生活变得更加容易。

根据您的需求，您可以将类型`PromiseResponse`扩展为一个类，并使用像`Success`和`Error`这样的帮助方法来使您的代码更具可读性。

我有一个效用函数，在 propose 上命名为`aKeptPromise`。有了这个函数，getData 可读性更强。

```
function aKeptPromise<T>(
  callback: (
    success: (result: T) => void,
    failure: (error: string) => void
  ) => void
): PromiseResponse<T> {
  return new Promise((res) => {
    callback(
      (r) => res([null, r]),
      (e) => res([e])
    );
  });
}

const getDataV3 = (n: number) : PromiseResponse<number> => {
    return aKeptPromise((success, failure) => {
        if (n === 3) {
            failure('Can not use 3.');
            return;
        }
        success(n * n);
    });
}
```

[打字游戏场](https://bit.ly/3dSOssW)

感谢阅读。如果你有意见，请告诉我。请在这里查看我的另一篇[打字稿](https://bit.ly/3cCWdmu)。
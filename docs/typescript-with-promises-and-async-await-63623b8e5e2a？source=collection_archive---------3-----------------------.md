# 带有承诺、异步/等待和生成器函数的类型脚本

> 原文：<https://itnext.io/typescript-with-promises-and-async-await-63623b8e5e2a?source=collection_archive---------3----------------------->

寻找一个伟大的(仅远程)反应开发？或者只是想聊聊天？访问 LinkedIn 上的[我的个人资料](https://www.linkedin.com/in/bengrunfeld/)并问好！😃

![](img/51780f5445cf7005f4f1bfc4ac151c2e.png)

出于某种奇怪的原因，TypeScript [文档](https://www.typescriptlang.org/docs/handbook)没有解释如何实现承诺、异步/等待或生成器函数的类型检查。

# 承诺

解决方案非常简单。基本上，承诺的返回类型是紧接在关键字`Promise`之后定义的。例如

```
const p = new Promise<boolean | string>((resolve, reject) => {
    const random = Math.random() * 10;if (random > 5) {
      resolve(true);
      return;
    }reject("It was lower than 5");
  });p.catch(err => console.log("ERROR - ", err));
```

注意`reject`是如何返回一个字符串的？嗯，我们只是在承诺的类型声明中添加了一个联合类型，这样,`resolve`和`reject`的返回类型都是可以接受的。

# 异步/等待

用`async/await`实现 TypeScript 非常类似于一个承诺。

```
const randomAboveFive = () =>
    new Promise<boolean | string>((resolve, reject) => {
      const random = Math.floor(Math.random() * 15); if (random > 5) {
      resolve(true);
      return;
    } reject("It was lower than 5");
});const checkRandomNumber = async (): Promise<boolean | string> => {
    let result; try {
      result = await randomAboveFive();
    } catch (err) {
      console.log("ERROR -- ", err);
      result = err;
    } console.log("----->>>", result); return result;
};checkRandomNumber();
```

在上面的代码中，我们声明了一个承诺，它可能以任何一种方式出现—如果生成的随机值大于`5`，我们解析`true`，如果不是，我们用一个解释错误的字符串`reject`。

接下来，我们用 **Async/Await** 设置一个等待**承诺**的函数，并处理它的成功或失败。

我们用`Promise<boolean | string>`设置异步/等待操作的返回类型。

# 发电机功能

下面是我们如何用`function*`实现 TypeScript，也称为生成器函数:

```
function* genCounter(i: number): IterableIterator<number> {
  while (true) {
    yield i--; if (i === 0) return;
  }
}const count = genCounter(4);let keepGoing = true;while (keepGoing) {
  const { value, done } = count.next();
  console.log("=>", value, done); if (done) {
    keepGoing = false;
  }
}
```

主要的要点是，要使**生成器函数**与**类型脚本**配合良好，您必须设置一个返回值`IterableIterator<number>`，其中`number`是它返回的任何基本类型脚本类型。

## 上面的代码是怎么回事:

我们声明一个生成器函数，它接受一个`number`，然后在每次调用`count.next()`时递减它。如果数字达到零，我们返回并停止循环，退出生成器函数。

然后我们声明另一个循环，该循环一直调用`count.next()`并记录输出，直到`count.next().done`为`true`。

现在你知道了！
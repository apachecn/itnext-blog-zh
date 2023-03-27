# 在 JavaScript 中使用 async/await 内部循环

> 原文：<https://itnext.io/using-async-await-inside-loops-in-javascript-5b0ccc9f0ef4?source=collection_archive---------1----------------------->

![](img/01c61b9bf5105e9d07b0851fcb85035c.png)

由[雷顿钻石](https://unsplash.com/@layton?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](/s/photos/marina?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

遍历条目和处理异步逻辑(即 API 调用)可能是我们作为 JavaScript 开发人员必须执行的两个最常见的任务。在本文中，我们将讨论结合 async/await 和迭代逻辑的最佳方法。有时，您会希望在 for 循环(或任何其他类型的循环)中运行异步操作。让我们来看看如何处理这种情况。

# 按顺序阅读承诺

假设我们有一个文件列表，我们想读取并记录序列中每个文件的内容。我们该怎么做？嗯，我们可以在异步函数中使用循环的**。下面是代码片段。**

```
async function printFiles () {
  let fileNames = ['picard', 'kirk', 'geordy', 'ryker', 'worf'];
  for (const file of fileNames) {
    const contents = await fs.readFile(file, 'utf8');
    console.log(contents);
  }
}
```

> 💡*注意，如果你想按顺序读取文件，你* ***不能*** *使用****forEach****循环。*

让我们用一个简单的例子来详细说明这一点。

```
async function someFunction(items) {
  items.forEach( async(i) => {
     const res = await someAPICall(i);
     console.log('--->', res);
  });
}function someAPICall(param) {
    return new Promise((resolve, reject)=>{
      setTimeout(()=>{
        resolve("Resolved" + param)
      },param);
    })
}someFunction(['3000','8000','1000','4000']);
```

在上面的代码中，我们有一个简单的异步函数，名为 **someFunction，**，它接受一个数组作为参数，迭代数组，并为每一项发出一个 API 请求(通过假的 API 函数)。在这种情况下，我们希望按顺序解析 API 调用。我们希望我们的输出打印如下

```
// expected
3000
8000
1000
4000
```

我们看到的不是这个输出，而是下面的结果

```
// actual
1000
3000
4000
8000
```

forEach 循环不是按顺序运行 API 调用，而是一个接一个地设置 API 调用。它不会等待上一个调用结束。这就是为什么我们会得到先解决的承诺。这是我们不能使用 forEach 循环的主要原因。

相反，我们可以使用一个 **reduce** 函数来迭代数组并依次解析承诺。让我们快速看一个例子。

```
function testPromise(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(`Processing ${time}`);
      resolve(time);
    }, time);
  });
}let result = [3000,2000,1000, 4000].reduce( (accumulatorPromise, nextID) => {
  return accumulatorPromise.then(() => {
    return testPromise(nextID);
  });
}, Promise.resolve());result.then(e => {
  console.log("All Promises Resolved !!✨")
});
```

很整洁不是吗？另一种解决序列承诺的方法是使用**异步生成器。**

```
async function* readFiles(files) {
  for(const file of files) {
    yield await readFile(file);
  }
};
```

生成器和大多数现代浏览器和 Node 10 及以上版本的支持。你可以在这里了解更多关于 Javascript [中的生成器和迭代器。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)

# 并行解决承诺

接下来，让我们看看如何并行解决承诺。让我们回到第一个例子。我们现在想并行读取文件，而不是按顺序读取。在这个场景中，我们不关心内容在控制台中的打印顺序。因此，我们可以简单地使用一个 **Promise.all()** 函数和一个 **map** 。

```
async function printFiles () {
  let fileNames = ['picard', 'kirk', 'geordy', 'ryker', 'worf'];
  await Promise.all(fileNames.map(async (file) => {
    const contents = await fs.readFile(file, 'utf8');
    console.log(contents);
  }));
}
```

每个`async`回调函数调用都返回一个承诺，我们用一个 Prmiss.all()并行地一次存储和解析它们。

我希望这篇快速阅读能让您深入了解如何在循环中使用异步代码。今天到此为止，下次再见。

**参考文献:**

[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Guide/Iterators _ and _ Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)

[https://stack overflow . com/questions/37576685/using-async-await-with-a-foreach-loop](https://stackoverflow.com/questions/37576685/using-async-await-with-a-foreach-loop)

[https://CSS-tricks . com/why-using-reduce-to-sequentially-resolve-promises-works/](https://css-tricks.com/why-using-reduce-to-sequentially-resolve-promises-works/)
# Javascript 中使用 Async/Await 对请求进行分页或循环的简单模式

> 原文：<https://itnext.io/simple-pattern-for-paging-or-looping-through-requests-with-async-await-in-javascript-4089f93678f8?source=collection_archive---------3----------------------->

![](img/ad9ced3ebc4e80c7ee136481fe983521.png)

由 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的 [Becca Tapert](https://unsplash.com/@beccatapert?utm_source=medium&utm_medium=referral) 拍摄的照片

作为提供一些 Api 用法示例的一部分，我编写了一个示例，包括如何使用 async/await 在 API 中分页。因为模式本身就说明了问题，所以我在这里就不赘述了。

代码可以被压缩和改进，但是为了清楚起见，我没有赘述，这不是重点。例如，不做:

```
let response = await reqUsers(token, offset)
records.push.apply(records, response)
```

我们可以这样重构:

```
records.push.apply(records, await reqUsers(token, offset))
```

如果不使用 Async/Await，你将需要一个更沉闷的模式或递归。否则，您将会以并发的方式无休止地调用 API。

更多的注释可以在文章的底部找到。

## 注意事项:

*   示例代码包括一个获取 oAuth 令牌的请求。这并不是示例的主要部分，但是我决定保留它，以防有人发现它有用。
*   Go 函数启动程序并填充顶部的 users 数组。如果这对您不起作用，您可以在函数的底部添加一个简单的 return 语句，并将 users 数组封装在函数中。

```
const go = async () => {let users = []// .... clipped for brevity
// we're all done.  return the users array.return users;
}
```

*   不包括错误处理。在实际应用程序中，您可能希望用 try/catch 块来捕获错误。

```
try {
   let payload = await request(userReq)} catch(err) {
    // handle err }
```
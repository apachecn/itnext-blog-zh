# 带 Firebase 函数的 CSV 或 JSON 导出

> 原文：<https://itnext.io/csv-or-json-export-with-firebase-functions-fa4a4b1416aa?source=collection_archive---------3----------------------->

![](img/4c161f1fc512a9a6dfbb7a64de7de297.png)

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fcsv-or-json-export-with-firebase-functions-fa4a4b1416aa%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

我花时间使用 firebase 函数已经有一段时间了，我非常喜欢使用它。

从更新数据到生成报告，几乎所有事情都可以通过它实现自动化。几天前，我的客户向我要一份 Firestore 的 excel 报告，所以我写了一个函数，帮助他们随时生成新的报告。

基本上，它的工作原理是一个 HTTP GET 请求将发送并触发 firebase 运行一个特定的功能。之后，每当他们打开那个特定的 URL，一个 JSON 将被返回到屏幕或者一个 CSV 文件将被直接下载。

我假设你知道如何写一个 firebase 函数。如果没有，请阅读本[入门指南](https://firebase.google.com/docs/functions/get-started)。

下面是 HTTP 请求函数的结构:

```
exports.csvJsonReport = functions.https.onRequest((request, response) => {
   // Your code goes here
   var report = {'a': 0, 'b': 1}; // your object})
```

您应该知道如何操作 firestore 或实时数据库来获取 JSON 格式的数据。之后，如果您愿意，可以通过使用以下代码将对象返回到屏幕上

```
// Return JSON to screen
  response.status(200).json(report);
```

如果你想在每次调用这个函数的时候得到 CSV，那么你可以使用这个代码

```
// Remember to install json2csv package before you can use it
const json2csv = require("json2csv").parse;// If you want to download a CSV file, you need to convert the object to CSV format
// Please read this for other usages of json2csv - [https://github.com/zemirco/json2csv](https://github.com/zemirco/json2csv) const csv = json2csv(report) response.setHeader(
    "Content-disposition",
    "attachment; filename=report.csv"
  ) response.set("Content-Type", "text/csv")
  response.status(200).send(csv)
```

通过这样做，你可以将 URL 直接复制并粘贴到你的浏览器中，或者将其嵌入到你的应用程序中的一个按钮中。无论哪种方式，您都可以获得您喜欢的数据类型。

请检查 Firebase 函数片段的存储库。我会经常更新它；)

【https://github.com/dalenguyen/firebase-functions-snippets 
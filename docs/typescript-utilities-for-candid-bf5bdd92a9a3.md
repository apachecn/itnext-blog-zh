# 用于偷拍的类型脚本实用程序

> 原文：<https://itnext.io/typescript-utilities-for-candid-bf5bdd92a9a3?source=collection_archive---------3----------------------->

## 与罐智能协定交互时处理可空、日期和 Blob 的函数集合。

![](img/363cd9315d8341d1a2f1c8aa5330ac1e.png)

Georgie Cobbs 在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

为了将我们的 web 编辑器 [DeckDeckGo](https://deckdeckgo.com/) 移植到 [DFINITY](https://dfinity.org/) 的互联网计算机上，我用 TypeScript 开发了几个助手来与我们的罐式智能合同进行交互。

如果这也能让你的生活变得更容易，以下是我最常用的。

# 可空的

为可空类型生成的[坦率的](https://smartcontracts.org/docs/candid-guide/candid-concepts.html)描述并不完全符合我通常在 JavaScript 中为可选类型使用的描述(参见这篇[帖子](https://forum.dfinity.org/t/candid-code-generation-for-nullable-types/6820)了解原因和方法)。

例如，如果我们为这样一个 [Motoko](https://smartcontracts.org/docs/language-guide/motoko.html) 代码片段生成一个接口:

```
actor Example {
  public shared query func list(filter: ?Text) : async [Text] {
    let results: [Text] = myFunction(filter);
    return results;
  };
}
```

可选参数`filter`的定义不会被解释为可能是`undefined`的`string`，而是包含`string`或为空的单元素长度`array`。

```
export interface _SERVICE {
  list: (arg_0: [] | [string]) => Promise<Array<string>>;
}
```

这就是我创建函数来来回转换可选值的原因。

```
export const toNullable = <T>(value?: T): [] | [T] => {
  return value ? [value] : [];
};

export const fromNullable = <T>(value: [] | [T]): T | undefined => {
  return value?.[0];
};
```

`toNullable`将类型为`T`或`undefined`的对象转换为预期与 IC 交互的对象，`fromNullable`执行相反的操作。

# 日期

系统[时间](https://smartcontracts.org/docs/base-libraries/Time.html)(自 1970–01–01 以来的纳秒数)被解析为`bigint`并作为类型`Time`导出。

```
export type Time = bigint;
```

要将 JavaScript `Date`转换成大数，可以通过将秒乘以纳秒来实例化内置对象 [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) 。

```
export const toTimestamp = (value: Date): Time => {
  return BigInt(value.getTime() * 1000 * 1000);
};
```

反过来，首先将大数字转换成原始的[数字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)类型，然后将它分成秒。

```
export const fromTimestamp = (value: Time): Date => {
  return new Date(Number(value) / (1000 * 1000));
};
```

为了支持`Nullable`时间戳值，我还创建了以下助手，它们扩展了转换器并返回适当的可选数组。

```
export const toNullableTimestamp = (value?: Date): [] | [Time] => {
  const time: number | undefined = value?.getTime();
  return value && !isNaN(time) ? [toTimestamp(value)] : [];
};export const fromNullableTimestamp = 
       (value?: [] | [Time]): Date | undefined => {
  return !isNaN(parseInt(`${value?.[0]}`)) ? 
            fromTimestamp(value[0]) : undefined;
};
```

# 一滴

二进制[斑点](https://smartcontracts.org/docs/base-libraries/Blob.html)在 Candid 中被描述为`numbers`的`Array`。为了在智能契约中保存非类型化数据(假设用例允许这样的风险)，同时仍然在前端保留类型，我们可以`stringify`对象，将它们转换成 blobs，并将它们的内容作为二进制数据包含在`ArrayBuffer`中。

```
export const toArray = 
       async <T>(data: T): Promise<Array<number>> => {
  const blob: Blob = new Blob([JSON.stringify(data)], 
                         {type: 'application/json; charset=utf-8'});
  return [...new Uint8Array(await blob.arrayBuffer())];
};
```

要将`numbers`的`Array`转换回特定的对象类型，可以再次使用 [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) 类型，但这一次将使用文本转换来解析结果。

```
export const fromArray = 
       async <T>(data: Array<number>): Promise<T> => {
  const blob: Blob = new Blob([new Uint8Array(data)], 
                         {type: 'application/json; charset=utf-8'});
  return JSON.parse(await blob.text());
};
```

这两种转换都是异步的，因为与 blob 对象交互需要解析 JavaScript 中的承诺。

# 进一步阅读

想了解更多我们的项目吗？以下是自从我们用互联网计算机开始这个项目以来，我发表的博文列表:

*   [再见亚马逊&谷歌，你好 Web 3.0](https://medium.com/geekculture/bye-bye-amazon-google-hello-web-3-0-b01bfe8f8783)
*   [从 CDN 动态导入 ESM 模块](/dynamically-import-esm-modules-from-a-cdn-5a6f741e2a1c)
*   [互联网计算机:Web App 分散数据库架构](https://medium.com/geekculture/internet-computer-web-app-decentralized-database-architecture-8647d1a437b8)
*   [单例&工厂模式带打字稿](https://javascript.plainenglish.io/singleton-factory-patterns-with-typescript-59e5a405531e)
*   [托管在互联网上的电脑](https://javascript.plainenglish.io/getting-started-with-the-internet-computer-web-hosting-b9b748350fc2)
*   [我们收到了一笔赠款，将我们的网络应用移植到互联网计算机上](https://medium.com/geekculture/we-received-a-grant-to-port-our-web-app-to-the-internet-computer-7be64806565a)

# 保持联络

为了跟随我们的冒险，你可以开始观看我们的 [GitHub 回购](https://github.com/deckgo/deckdeckgo) ⭐️和[注册](http://eepurl.com/hKeMLD)加入测试者的名单。

# 结论

我希望这篇简短的博文和几个实用程序对你开始使用互联网计算机有用，这真的是一项有趣的技术。

到无限和更远的地方！

大卫

你可以通过 [Twitter](https://twitter.com/daviddalbusco) 或我的[网站](https://daviddalbusco.com/)联系我。

尝试使用 [DeckDeckGo](https://deckdeckgo.com/) 制作下一张幻灯片。

[![](img/d1292e4858d9cdb834769a6424ed937f.png)](https://deckdeckgo.com)
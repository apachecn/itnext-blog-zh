# 享受在 TypeScript 中解构本地存储的乐趣🤙

> 原文：<https://itnext.io/having-fun-deconstructing-the-localstorage-in-typescript-e5e99d95aa13?source=collection_archive---------2----------------------->

![](img/f4048c31e79b22164af097a165b87dd6.png)

照片由 [Katya Ross](https://unsplash.com/@katya?utm_source=Papyrs&utm_medium=referral) 在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

我最近用 [localstorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) 实现了一些特性。虽然我总是使用接口的 [getItem()](https://developer.mozilla.org/en-US/docs/Web/API/Storage/getItem) 方法读取值，但在我最近的工作中，我用存储对象的解构代替了这种方法。

没有特别的原因。我只是喜欢解构事物，非常喜欢😄。

# 守旧派

回到过去——直到最近几周😉—我可能会实现一个函数来从存储中读取一个字符串化的`object`,如下所示:

```
type MyType = unknown;

const isValid = (value: string | null): value is string => [null, undefined, ""].includes(value)

const oldSchool = (): MyType | undefined => {
  const value: string | null = localStorage.getItem("my_key");

  if (!isValid(value)) {
    return undefined;
  }

  return JSON.parse(value);
};
```

也就是说，在仔细检查其有效性并解析回一个对象之前，我将首先使用`getItem()`获得`string`值(我将保存在存储器中的对象的字符串化`JSON.stringify()`表示)。

# 新学校

虽然我现在继续遵循以前的逻辑(“读取、检查有效性和解析”)，但我现在正在解构存储以读取值。

```
const newSchool = (): MyType | undefined => {
  const { my_key: value }: Storage = localStorage;

  if (!isValid(value)) {
    return undefined;
  }

  return JSON.parse(value);
};
```

同样，没有特别的原因，但是，它不是更闪亮吗？👨‍🎨

这种方法在 TypeScript 中是可行的，因为存储接口——表示[存储 API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)——实际上被声明为`any`类型的键映射。

```
interface Storage {
    readonly length: number;
    clear(): void;
    getItem(key: string): string | null;
    key(index: number): string | null;
    removeItem(key: string): void;
    setItem(key: string, value: string): void;
    // HERE 😃 [name: string]: any;
    [name: string]: any;
}
```

# SSR 和预渲染

`localstorage`是`window`界面的只读属性，即它只存在于浏览器中。为了防止我的 SvelteKit 的静态构建在使用时崩溃，我为 NodeJS 上下文设置了一个`undefined`回退值。

此外，除了解构模式之外，我还喜欢将所有内容(😄).所以，我想出了下面的代码片段来解决我的灵感:

```
import { browser } from "$app/env";

const newSchool = (): MyType | undefined => {
  const { my_key: value }: Storage = browser
    ? localStorage
    : ({ my_key: undefined } as unknown as Storage);

  if (!isValid(value)) {
    return undefined;
  }

  return JSON.parse(value);
};
```

# 一般的

在这一点上，你可能会说“是的，大卫，很好，这很酷，但是，可重用性呢？”。对此，我会回答“拿着我的啤酒，你可以动态解构对象”😉。

```
const newSchool = <T>(key: string): T | undefined => {
  const { [key]: value }: Storage = browser
    ? localStorage
    : ({ [key]: undefined } as unknown as Storage);

  if (!isValid(value)) {
    return undefined;
  }

  return JSON.parse(value);
};
```

# 摘要

返回`undefined`对于演示来说很方便，但是在实际实现中——比如我今天早上在 [Papyrs](https://papy.rs/) (一个 web3 博客平台)中发布的——使用默认的回退值可能更有用。

因此，这是我的泛型函数的最终形式，使用有趣的东西，如解构对象、断言和泛型，读取保存在 TypeScript 的`localstorage`中的项目。

```
import { browser } from "$app/env";

const isValid = (value: string | null): value is string =>
  [null, undefined, ""].includes(value);

const getStorageItem = <T>({
  key,
  defaultValue,
}: {
  key: string;
  defaultValue: T;
}): T => {
  const { [key]: value }: Storage = browser
    ? localStorage
    : ({ [key]: undefined } as unknown as Storage);

  if (!isValid(value)) {
    return defaultValue;
  }

  return JSON.parse(value);
};
```

无限和超越
大卫

更多冒险，请在🖖推特上关注我
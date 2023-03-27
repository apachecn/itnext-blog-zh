# 将 fetch 与 Typescript 和 Todoist 一起使用

> 原文：<https://itnext.io/using-fetch-with-typescript-and-the-todoist-api-5203c5177ed5?source=collection_archive---------0----------------------->

![](img/6485b70727cade8513fb8f8b8b46c97f.png)

这些天我一直在摆弄打字稿。到目前为止，这种体验非常棒。我主要和鲁比·科特林一起工作。我也在一些场合使用过 Javascript。也就是说，我发现对 Typescript 的更改并没有我想象的那么难，事实上，我觉得这很直观。当我将它与 VScode(也是内置的 Typescript)一起使用时，IDE 经常会指出任何缺失/错误的类型，这将阻止无数小时的调试。

在我过去从事的用于发出 HTTP 请求的 javascript 项目中，我会使用像 *Axios* 或*这样的库。ajax* ，这让我想到如果我可以使用最近的 fetch API 会怎么样。使用 fetch 消除了对 Axios 或 jQuery 等外部依赖的需要，尽管目前并非所有浏览器都支持它(看看你的 Internet Explorer)。

所以我决定在一个简单的 react 应用程序中同时尝试 Typescript 和 fetch。对于这个例子，我将使用 Todoist 的任务 API(API Docs:[https://developer.todoist.com/rest/v1/#tasks](https://developer.todoist.com/rest/v1/#tasks))。该示例将包括通过 API 创建一个简单的任务。对于那些不知道的人来说，Todoist 是一个生产力应用程序，可以让你组织、计划和协作完成任务和项目。就我个人而言，我使用它的任务列表来组织我的工作，也可以在我需要去杂货店购物时作为购物清单。

要使用 API，您需要获得一个令牌，该令牌可以通过创建一个帐户来获得。

通过 API 创建任务非常简单，您需要发出一个 POST 请求，带有一些参数，还需要传入几个头，如下所述。

要使用 fetch 设置头，可以使用 HeadersInit 类型，如下所示

```
import { v4 as uuidv4 } from 'uuid';const headers: HeadersInit = {
  'Content-Type': 'application/json',
  'X-Request-Id': uuidv4(),
  'Authorization': `Bearer <API_TOKEN>`
}
```

至于请求体，我已经声明了一个名为 TaskParam 的类型，如下所示

```
export type TaskParam = {
  content: string;
  due_lang: string;
  due_string: string;
  priority: number
};const taskParam: TaskParam = {
  content: 'Buy eggs',
  due_string: 'Tomorrow at 12.00',
  due_lang: 'en',
  priority: 1,
}
```

最后，在这里请求你如何去做:

```
export async function createTask(taskParam: TaskParam): Promise<Response> {
  const url = '[https://api.todoist.com/rest/v1/tasks'](https://api.todoist.com/rest/v1/tasks');
  const opts: RequestInit = {
    method: 'POST',
    headers,
    body: JSON.stringify(taskParam),
  };
  return fetch(url, opts)
}
```

就这样，您应该能够重用这个函数来创建任意多的任务。

相反，如果我们想得到所有活动的任务，代码是非常相似的。

```
export async function getTasks(): Promise<Response> {
  const url = '[https://api.todoist.com/rest/v1/tasks'](https://api.todoist.com/rest/v1/tasks');
  const opts: RequestInit = {
    method: 'GET',
    headers,
  };
  return fetch(url, opts)
}
```

至于测试，您需要将 *fetch-mock* 添加到 package.json 中。

```
import fetchMock from 'fetch-mock';const taskParam: TaskParam = {
  content: 'Buy eggs',
  due_string: 'Tomorrow at 12.00',
  due_lang: 'en',
  priority: 1,
}describe('createTask', () => {
  it('returns ok', async () => {fetchMock.post('[https://api.todoist.com/rest/v1/tasks'](https://api.todoist.com/rest/v1/tasks'), { ok: 'ok' });const response = await createTask(taskParam);
    const result = await response.json()expect(result).toEqual({ ok: "ok" });});
});
```

总的来说，代码可能看起来非常类似于 Javascript，但是当使用 Typescript 时，IDE 可能会指出一些在响应体、头或整体请求配置中可能会丢失的字段。

喜欢这个故事吗？在我的个人网站上查看更多信息[https://www.ricardo-trindade.com/](http://www.ricardo-trindade.com/)
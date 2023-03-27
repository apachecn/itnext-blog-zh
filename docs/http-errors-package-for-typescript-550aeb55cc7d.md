# Typescript 的 Http 错误包

> 原文：<https://itnext.io/http-errors-package-for-typescript-550aeb55cc7d?source=collection_archive---------7----------------------->

Hacktoberfest 5 已经开始了，作为我的第一个贡献，我想做一个带有 HTTP 错误的微型类型脚本库。

每当我开始一个新的基于 Javascript 的项目时，无论是在服务器上，还是在用 [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 编写 API 客户端，我经常发现自己一遍又一遍地做着同样的事情，就是定义一组简单的表示 HTTP 错误的异常，就像这样:

```
class NotFound extends Error {
  httpStatus = 404;
}
```

对此我有点厌倦了，我决定做一个小的包，里面只是 Typescript 的错误列表，还有一些小的实用程序。

![](img/1ec037cb8a89cd574dfc2720c8be15cd.png)

照片由[莫赫达姆·阿里](https://unsplash.com/@mohdali_31?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

**示例:**

```
import { NotFound } from '@curveball/http-errors';throw new NotFound('Article not found');
```

这个想法是，界面实际上就是这样:

```
export interface HttpError extends Error { 
  httpStatus: number;
}
```

这意味着任何带有`httpStatus`属性的错误都会自动遵循这种模式，可以编写通用的中间件来监听它们。

它附带了一个简单的实用函数来查看`Error`是否符合这个模式:

```
import { isHttpError } from '@curveball/http-errors';const myError = new Error('Custom error');
myError.httpStatus = 500;
console.log(isHttpError(myError)); // true
```

# 问题+json

从 HTTP API 发出错误的一个好主意是使用在 [RFC7807](https://tools.ietf.org/html/rfc7807) 中定义的`application/problem+json`格式。该软件包还包含一些实用程序来帮助完成这些任务:

```
export interface HttpProblem extends HttpError { type: string | null;
  title: string; detail: string | null;
  instance: string | null; }
```

这个包附带的每个标准异常也实现了这个接口。大多数属性(除了`title`)默认为`NULL`，因为它们可能是特定于应用程序的。

假设，有人可以编写一个库，用`HttpProblem`属性发出一个错误，而不必依赖这个包，一个通用的应用程序中间件可以神奇地将它转换成正确的格式。

一些 HTTP 响应要求或建议包含额外的 HTTP 头，其中包含关于错误的更多信息。例如，`[405 Method Not Allowed](https://evertpot.com/http/method-not-allowed)`响应应该包括一个`Allow`头，而`503 Service Unavailable`应该有一个`Retry-After`头。内置错误支持这些:

```
import { MethodNotAllowed, ServiceUnavailable } 
  from '@curveball/http-errors'; try {
  throw new MethodNotAllowed('Not allowed', ['GET', 'PUT']);
} catch (e) {
  console.log(e.allow);
} try { 
  throw new ServiceUnavailable('Not open on sundays', 3600*24);
} catch (e) {
  console.log(e.retryAfter);
}
```

我打算再增加几个这样的。例如，`UnsupportedMediaType`类可以包括一个支持的内容类型列表。这可能与 HTTP 响应头不直接相关，但是有人可以用它来自动填充 JSON 响应体。

希望这对其他人有用。以正确的方式做一次，我希望再也不用做了。

链接:

*   [Github 项目](https://github.com/curveballjs/http-errors)
*   [NPM 套餐](https://www.npmjs.com/package/@curveball/http-errors)

*原载于*[](https://evertpot.com/http-errors-package-for-typescript/)**。**
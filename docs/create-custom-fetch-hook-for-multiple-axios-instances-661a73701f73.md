# 为多个 Axios 实例创建自定义提取挂钩

> 原文：<https://itnext.io/create-custom-fetch-hook-for-multiple-axios-instances-661a73701f73?source=collection_archive---------0----------------------->

当您第一次开始将现有的代码库转换成钩子或者开始使用 React 钩子从头开始编写组件时，这可能会很有趣。

经过几个重复的组件后，您将摆脱重复的代码块，尤其是在发出一些网络请求时。每个请求代码块里都有那么多类似的东西。设置加载指示器、处理错误、设置响应数据等。

听起来没必要，直到你感受到那种痛苦。我在项目一开始就有这种感觉，需要找出一个可扩展和可维护的解决方案。

# 定制挂钩！

自定义钩子允许我们使用 React 的钩子构建新的钩子。正如我之前提到的，如果你有一些重复的块和过程，你可以创建类似中间件的钩子来每次处理这些工作。事实上，这是一种效用函数。因此，让我们为 Axios 实例构建自己的定制钩子:

![](img/d57c83e09c8783faafb2be14bb3bf577.png)

[贷项:Unsplash](https://unsplash.com/photos/Wiu3w-99tNg)

首先，让我们定义我们的实例:

```
export const ***contentApi*** = ***axios***.create({
   baseURL: contentApiUrl,
});

export const ***programApi*** = ***axios***.create({
   baseURL: programApiUrl,
});
```

我需要创建这种实例，因为我需要对我的请求进行分组，每个请求需要不同的选项、拦截器、基本 URL 等。因为我有超过 8 个不同的 API。

*更新:非常感谢*[*https://medium.com/@gouaks*](https://medium.com/@gouaks)*我解决了一个恼人的问题，当我为数据或配置传递一个对象时，它会导致一个无限循环。下面是更新后的版本。*

更新 2:我从 Francis Rodrigues 那里得到了一个很好的评论，他用我的文件创建了一个 codesandbox 示例。这是演示:

其次，创建一个名为 ***的文件 useFetch.js:***

```
import { useState, useEffect } from "react";

export default function useFetch({ api, method, url, data = **null**, config = **null** }) {
   const [response, setResponse] = useState(null);
   const [error, setError] = useState("");
   const [isLoading, setIsLoading] = useState(true);

   useEffect(() => {
      const fetchData = async () => {
         try {
            api[method](url, ***JSON***.parse(config), ***JSON***.parse(data))
               .then((res) => {
                  setResponse(res.data);
               })
               .finally(() => {
                  setIsLoading(false);
               });
         } catch (err) {
            setError(err);
         }
      };

      fetchData();
   }, [api, method, url, data, config]);

   return { response, error, isLoading };
}
```

以下是在每个网络请求中避免使用 setLoading 的最简单方法:

```
import useFetch from "../hooks/useFetch";const {response, isLoading} = useFetch({
   api: ***programApi***,
   method: "get",
   url: "/SportsProgram/active_sport_type",
   config: ***JSON***.stringify({ requireAuthentication: true }),
});
```

现在，您可以在 useEffect 中聆听自己的回答，并随心所欲:

```
useEffect(() => {
   if (response !== null) {
      // do more stuff if you wish
   }
}, [response]);
```

保持它易于阅读和理解总是更好的。如果你有任何意见，以改善这个钩，请让我知道，因为这是我第一次自定义钩杆！

以下是 gist 网址:[https://gist . github . com/Murat dogan 17/f 5861 e 82 e 6 db 5545 f 651617823214172](https://gist.github.com/muratdogan17/f5861e82e6db5545f651617823214172)

享受
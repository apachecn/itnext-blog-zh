# 更轻松的服务人员缓存和路由

> 原文：<https://itnext.io/easier-service-worker-caching-with-routing-323d347d95d3?source=collection_archive---------7----------------------->

![](img/431353e86c3a345234a3665b476d189c.png)

https://pxhere.com/en/photo/187030?utm_content=shareClip&UTM _ medium = referral&UTM _ source = px here

服务人员是浏览器工具箱中一个非常强大的工具，但是您可能很难理解它，尤其是以可维护的方式建模和组织不同的流程。使用基于路由的模式(类似于 Node Express 框架中的模式)可以帮助您做到这一点。这就是我在这里做的实验。类似的事情以前也做过。例如，像 Workbox 这样的库将路由作为 API 的一部分来实现。但是这里我将探索一个*路由优先选择*来创建一个更加灵活和轻量级的选项。

为此，我使用了 *itty 路由器*([https://github.com/kwhitley/itty-router](https://github.com/kwhitley/itty-router))。itty router 是一个非常小、轻量级、但仍然强大的路由库，我是在与 Cloudflare Workers 一起试验时偶然发现的。Cloudflare Workers 是一个基于 Service Worker API 的 FaaS 平台，因此 *itty router* 可以在浏览器中使用，无需任何额外的工作。

# itty 路由器简介

使用 *itty 路由器*，单条路由可以这样定义:

```
router.get('/hello', request => {
  return new Response('Hello World!')
})
```

itty router 有两个特别的方面使它在服务人员中非常有用。第一，它使用中间件模式，因此您可以链接处理程序。如果第一个没有返回响应，请求将被转移到下一个:

```
router.get('/hello', handler1, handler2)
```

另一件重要的事情是，虽然请求是处理程序唯一期望的参数，但是可以用将沿着链传递的附加参数注册主路由器处理程序:

```
router.get('/hello', (request, user) => {
  return new Response(`Hello ${ user.name }!`)
})addEventListener('fetch', event =>
  event.respondWith(router.handle(event.request, user))
)
```

# 基本设置

因此，记住这一点，我们可以看到它如何帮助我们实现与服务工作者相关的不同模式/策略，如 CacheFirst 或 NetworkFirst。但首先是基本的设置。可以像这样从 HTML 加载和注册服务工作者:

```
<script type="module">
    const reg = await navigator.serviceWorker.register('/sw.js', {
        type: 'module',
    });
</script>
```

在这个 PoC 中，我使用 ES 模块，这必须反映在脚本标记和服务人员的注册中。服务人员的根源非常简单:

```
import { cacheName, coreAssets } from './config.js';
import router from './router';self.addEventListener('install', function (event) {
    // Cache core assets
    event.waitUntil(caches.open(cacheName).then(function (cache) {
        for (let asset of coreAssets) {
            let req = new Request(asset)
            cache.add(req);
        }
        return cache;
    }));
});self.addEventListener('fetch', async event => {
    event.respondWith(router.handle(event.request, {onLine: navigator.onLine}, event))
})
```

预缓存的一些配置，以及在上下文对象上设置的在线状态。主要工作由导入的路由器完成，可能如下所示:

```
import { Router } from 'itty-router';
import { handler1, handler2 } from './handlers.js';const router = Router()//Add routes
router.get('/assets', handler1 )
router.get('*', handler2)
...export default router;
```

# 基本软件策略

那么，CacheFirst 和 NetworkFirst。让我们首先定义一个从缓存中检索响应的处理程序:

```
const ifCacheRespond = async (request, context, event) => {
    const response = await caches.match(request);
    if (response) {
        return response
    }
}
```

类似于网络响应的处理程序。它在启动提取之前检查在线状态:

```
const ifNetworkRespond = async (request, context, event) => {
    if(context.onLine){
        let response = await fetch(request);
        if (response && response.ok) {
            return response;
        }
    }
}
```

因此，CacheFirst 策略可以表达如下:

```
router.get('/assets/*', ifCacheRespond, ifNetworkRespond, handleError)
```

如果`ifCacheRespond`在缓存中找到一个响应，它将返回这个响应。如果没有，它将请求传递给`ifNetworkRespond`。`handleError`有没有以这样或那样的方式收拾彻底的失败。或者，您可以在末尾使用默认路由来处理这些情况，因为如果有问题的路由没有返回任何响应，路由器会将请求传递到下一个路由。

那么，我们如何表达网络优先战略呢？是的，当然，我们可以交换顺序😁：

```
router.get('/assets/*', ifNetworkRespond, ifCacheRespond, handleError)
```

对于稍微复杂一点的策略，我们需要增加流程。但是它们仍然可以用类似的方式来表达。例如，对于 CacheAsYouGo，您需要在返回响应之前将其添加到缓存中。启用此策略的路线可以定义如下:

```
router.get('/articles/*', ifCacheRespond, getNetworkResponse, addToCache, ifNetworkRespond, handleError)
```

第一个和以前一样。如果您在缓存中有响应，就提供它。如果没有，流程将转到下一个处理程序:

```
const getNetwork = async (request, context, event) => {
    if (context.onLine) {
        const response = context.networkResponse || await fetch(request);
        if (response && response.ok) {
            console.log("response.ok", response)
            context.networkResponse = response;
        }
    }
}
```

这个处理程序不返回网络响应，而是将它添加到上下文中，供链中更下游的处理程序使用，如`addToCache`:

```
const addToCache = async (request, context, event) => {
    const response = context.networkResponse;
    if (response && response.ok) {
        let responseClone = response.clone();
        caches.open(cacheName).then(async (cache) => {
            cache.put(request, responseClone);
        });
    }
}
```

我们需要更改`ifNetworkRespond`来使用上下文对象上的响应(如果可用的话):

```
const ifNetworkRespond = async (request, context, event) => {
    if (context.onLine) {
        const response = context.networkResponse || await fetch(request);
        if (response) {
            context.networkResponse = response;
        }
        if (response && response.ok) {
            return response;
        }
    }
}
```

要启用 StaleWhileRevalidate 这样的策略，我们需要稍微扩展一下。使用 StaleWhileRevalidate 时，您希望使用缓存，但在后台更新缓存而不等待它，以便下一次命中将使用更新的资源:

```
router.get('/', revalidateCache, ifCache, ifNetworkRespond, handleError)
```

在这里，您希望开始重新验证，但如果有缓存，就不要等待。所以`revalidateCache`为它创造了一个承诺，然后继续前进:

```
const revalidateCache = async (request, context, event) => {
    if (context.onLine) {
        const cache = await caches.open(cacheName);
        const networkResponsePromise = fetch(request);
        context.networkResponsePromise = networkResponsePromise;
        event.waitUntil(async function () {
            const networkResponse = await networkResponsePromise;
            await cache.put(request, networkResponse.clone());
        }());
    }
}
```

并且`ifNetworkRespond`也需要处理承诺，所以它不会在没有缓存响应可用的情况下开始新的获取，而是使用重新验证缓存的相同获取:

```
const ifNetworkRespond = async (request, context, event) => {
    if (context.onLine) {
        let response;
        if (context.networkResponsePromise) {
            response = await context.networkResponsePromise
        } else {
            response = context.networkResponse || await fetch(request);
        }
        if (response) {
            context.networkResponse = response;
        }
        if (response && response.ok) {
            return response;
        }
    }
}
```

# 标题过滤

服务工作者使用缓存的另一种典型方式是不使用路由，而是使用头字段。比如用`Content-Type: text/html`缓存一切。这在这里也是可能的。您可以在具有可缓存内容类型的初始上下文对象中添加一个数组，并在缓存之前检查该数组:

```
self.addEventListener('fetch', async event => {
    event.respondWith(router.handle(event.request, {onLine: navigator.onLine, cachableContent: ['text/html']}, event))
})
```

然后，您可以使用一个“全部捕获”路径来缓存所有 HTML:

```
router.get('/*', ifCacheRespond, getNetworkResponse, addToCache, ifNetworkRespond, handleError)
```

对于更细粒度的处理，结合头和路由，过滤器可以改为在流中首先设置一个句柄:

```
const cacheHTMLOnly = async (request, context, event) => {
    context.cachableContent = ['text/html'];
}router.get('/assets/*', cacheHTMLOnly, ifCacheRespond, getNetworkResponse, addToCache, ifNetworkRespond, handleError)
```

# 更高级

这是非常粗略的，一个完整的实现当然需要添加更多的东西，比如处理缓存到期头等。尤其是您需要管理缓存版本控制和删除，这样您就不会在需要更改时陷入糟糕的状态，也不会突然无法更新客户端上的资源。

此外，像 Workbox 这样的库支持更高级的 bookeeping 功能，比如限制将被缓存的请求的数量，保存由于网络故障而失败的 post-requests，添加自己的过期处理等等。为此，您需要像 IndexedDB 这样的东西作为流程的一部分。我不会在这里详细介绍，但我会用下面的场景来描述一下，如果网络中断，您希望能够离线存储新文章:

```
router.post('/article/*', ifNetworkRespond, saveRequestForLater, handleError)
```

如果`ifNetworkRespond`没有返回，`saveRequestForLater`将获取请求并保存到 IndexedDB 中。这是一个非常简化的版本。您可能需要保存更多的请求，比如头、参数等。这很大程度上取决于相关的应用和功能:

```
const saveRequestForLater = async (request, context, event) => {
    if(!context.onLine){
        event.waitUntil(async function () {
            const payload = await request.clone().json();
            const dbrequest = indexedDB.open("articles");
            dbrequest.onsuccess = async (event) => {
                const db = event.target.result;
                const tx = db.transaction('createarticle', 'readwrite');
                const store = tx.objectStore('createarticle');
                store.add(payload)
            } 
        }());       
    }
}
```

当然，这里最重要的是，当应用程序再次上线时，要有某种机制来采取行动，然后重放请求。但这也可能是基于应用程序的要求和特定的功能。

# 缓存过期和边车请求

缓存处理可能需要考虑响应的缓存头，但是如果您不想在服务工作器本身中添加单独的过期处理，该怎么办呢？挑战在于，在将响应放入缓存之前，您不能通过设置自定义头来修改响应，因为它是只读的。一种替代方法是使用 IndexedDB 获取额外的元数据，另一种方法是创建一个 *sidecar 请求*来缓存额外的元数据:

```
const addToCache = async (request, context) => {
    const response = context.networkResponse;
    if (response && response.ok) {
        let responseClone = response.clone();
        caches.open(cacheName).then(async (cache) => {
            cache.put(request, responseClone);
            const sidecarReq = new Request(request.url + '-sidecar', {
                method: "GET",
            });
            const sidecarResp = new Response(null, {
                status: 200,
                headers: {
                    'x-sw-expires': Date.now() + context.cacheMaxAgeMS,
                },
            });
            cache.put(sidecarReq, sidecarResp);
        });
    }
}
```

然后在检查缓存时检查边车中的标题。

***NB！但是要确保这样的 sidecar-routes 不会被其他处理程序无意中缓存，或者用于存储认证或其他敏感的东西，因为它显然会被篡改。***

# 摘要

所以使用像 *itty router* 这样的路由器在编写服务人员时肯定会有所帮助。与使用像 Workbox 这样的库相比，它让您更接近平台工作，并且非常灵活和可定制。但是它仍然让您以一种接近更高级工具的可读性的方式组织流程。并且仍然是非常轻量级的，与它可以帮助您节省的下载字节相比，它只增加了很少的开销。
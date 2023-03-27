# 如何在 AngularJS 中取消 HTTP 请求

> 原文：<https://itnext.io/how-to-cancel-http-requests-in-angularjs-4ccf351319e0?source=collection_archive---------1----------------------->

最近，我们需要开发一个新功能，允许用户取消正在进行的 HTTP 请求。这是一个相当合理的要求，在技术上实现起来并不困难，但是，我们由 AngularJS 和 ngRedux 组成的架构使它更具挑战性。

在这篇文章中，我将提出一种取消 HTTP 请求的方法。

![](img/b5e34d0888b0be3affeb7b9a9648a224.png)

# 承诺链中的问题

AngularJS 提供了一种取消 HTTP 请求的简单方法。`$http`接受可以是承诺的`timeout`选项。当`timeout`承诺被解决时，HTTP 请求将被取消。这里有一个例子:

```
function fetchBooks() {
    const canceller = $q.defer();
    return $http.get('/api/books/', { timeout: canceller.promise });
}
```

在上面的代码片段中，我们可以通过调用`canceller.promise.resolve()`轻松取消 HTTP 请求。然而问题很明显:`fetchBooks()`中没有返回`canceller`。换句话说，`fetchBooks()`的调用者没有取消请求的控制权。

当然，我们可以将`canceller`赋值给结果承诺，就像这样:

```
function fetchBooks() {
    const canceller = $q.defer();
    let promise = $http.get('/api/books/', {
        timeout: canceller.promise
    });
    promise.cancel = () => {
        canceller.resolve();
    }
    return promise;
}// Caller
const books = fetchBooks();// Later..
books.cancel();
```

这看起来不错，但是只有一个问题。如果`fetchBooks()`返回的承诺在承诺链中被进一步使用，这种方法将会失败。例如，以下代码在 redux 操作中非常常用:

```
const action = () => dispatch => {
    return fetchBooks()
        .then(response => response.results)
        .then(books => {
            dispatch(updateBooks(books));
            return books;
        });// Caller
const books = action();
books.cancel();    // ERROR: books.cancel === undefined
```

`.then()`会生成新的承诺，因此`action()`返回的`books`不再是`$http.get()`返回的原承诺。

看到问题了吗？简而言之，我们无法访问`cancel()`方法，因为:

*   `cancel()`能够被暴露的唯一方式是附在结果承诺上；
*   结果承诺在通过承诺链时丢失。

# 在后台

如果我们仔细看看 AngularJS 中的`.then()`方法的实现，我们会发现类似这样的内容:

```
extend(Promise.prototype, {                                                                        
    then: function(onFulfilled, onRejected, progressBack) {                                          
      if (isUndefined(onFulfilled) && isUndefined(onRejected) && isUndefined(progressBack)) {        
        return this;                                                                                 
      }                                                                                              
      var result = new Deferred();                                                                   

      this.$$state.pending = this.$$state.pending || [];                                             
      this.$$state.pending.push([result, onFulfilled, onRejected, progressBack]);                    
      if (this.$$state.status > 0) scheduleProcessQueue(this.$$state);                               

      return result.promise;                                                                         
    },
...
```

我们可以看到旧的承诺对新的承诺有一个引用(`this.$$state.pending[0][0]`)。为了说明这一点，请考虑以下设置:

```
let promiseA = $http.get('/');
promiseB = promiseA.then(response => response);
promiseC = promiseB.then(repsonse => response);
```

然后:

```
promiseA.$$state.pending[0][0].promise === promiseB
promiseB.$$state.pending[0][0].promise === promiseC
```

这里的关键是引用是从最早的到最新的。我们不能从最近的承诺`promiseC`返回到根承诺`promiseA`。然而，我们可以为正在进行的 HTTP 请求创建一个列表，并提供一个方法`cancelPromise(promise)`，该方法接受承诺链中的任何承诺。当`cancelPromise(promise)`被调用时，我们将遍历 HTTP 请求列表并找到相应的承诺链，然后调用根承诺的`.cancel()`方法。

# 履行

我们定义了一个服务来保存 HTTP 请求:

```
class PromiseService { constructor() {
        this._requests = [];
    } register(promise) {
        this._requests.push(promise);
    } unregister(promise) {
        const idx = this._requests.indexOf(promise);
        if (idx >= 0) {
            this._requests.splice(idx, 1);
        }
    } cancelPromise(promise) {
        for (let i = 0; i < this._requests.length; i++) {
            const rootPromise = this._requests[i];
            let p = rootPromise; // Traverse the promise chain to see 
            // if given promise exists in the chain
            while (p !== promise && p.$$state.pending &&
                p.$$state.pending.length > 0) {
                p = p.$$state.pending[0][0].promise;
            } // If this chain contains given promise, then call the
            // cancel method to cancel the http request
            if (p === promise && typeof r.cancel === 'function') {
                r.cancel();
            }
        }
    }
}
```

并且在 API 调用中:

```
function fetchBooks() {
    const canceller = $q.defer();
    let promise = $http.get('/api/books/', {
        timeout: canceller.promise,
    }); promise.cancel = () => {
        canceller.resolve();
    } promiseService.register(promise); // If HTTP requests complete before cancel,
    // then remove the entry from PromiseService
    return promise.then((response) => {
       promiseService.unregister(promise);
       return response;
    });
}
```

完成后，我们可以简单地调用`promiseService.cancel()`:

```
// Caller
let books = action();// Cancel the HTTP request
promiseService.cancel(books);
```
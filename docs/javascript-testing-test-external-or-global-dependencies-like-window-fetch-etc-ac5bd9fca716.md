# Javascript 测试:测试外部或全局依赖，如“窗口”、“获取”等

> 原文：<https://itnext.io/javascript-testing-test-external-or-global-dependencies-like-window-fetch-etc-ac5bd9fca716?source=collection_archive---------6----------------------->

![](img/5e4c4768137a70b0339d6c8ee44b0aa7.png)

照片由[欧文·史密斯](https://unsplash.com/photos/5eBW5GomfhY?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/javascript?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

有多少次你处于下面的位置，你有一个依赖于外部依赖或全局对象的函数(如窗口，获取，文档…)

```
function doSomething(oneParam, callback) {
 oneDependency.doSomethingElse(); 
 fetch('resource/from/somewhere').then(callback)
}
```

现在，当您试图测试这段代码时，您如何控制对`oneDepenency`和`fetch`的两个依赖项的绑定？答案是`dependency injection` + `factoryMethod`下面是怎么回事:

```
import { oneDependency } from './somewhere/somefile';const doSomething = (oneParam, callback) => {
 const generatedValue = oneDependency.doSomethingElse(oneParam);
 // Let your mind to fly
 return fetch('api/' + generatedValue).then(callback);
}export const doSomethingFactory = (oneDependency, fetch) => {
 return doSomething.bind(null, oneDependency, fetch);
}import { **doSomethingFactory** } from './path/to/your/factory';**describe**('DoSomething test suite', () => {
 let service, fakeOneDependency, fakeFetch;**it**('Should be defined', (done) => {
 // Given fake dependencies
 fakeOneDependency = { doSomethingElse: () => ({mock: 'value'})};
 fakeFetch = () => ({ then: () => ({fake: 'object'})});
 **// When** 
 **service = doSomethingFactory(fakeDependency, fakeFetch);** // Then  **service**('the oneParam value', (response) => {
 expect(response.fake).toEqual('object');
 });
 });
})
```

通过这样做，您可以完全控制您的测试，以提供驱动您对代码的正确期望的行为，这是改进和净化您的测试集的极其简单的方法。

**代码覆盖 BTW 也会开心的！；)**

*原载于 2018 年 7 月 3 日 medium.com*[](https://medium.com/@eliasmsedano/javascript-testing-test-external-or-global-dependencies-like-window-fetch-etc-7693e4dab4e4)**。**
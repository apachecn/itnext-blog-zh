# 反应式吊钩的搭便车指南

> 原文：<https://itnext.io/hitchhikers-guide-to-reactive-hooks-92c1a708ccfe?source=collection_archive---------3----------------------->

![](img/50a86d00e70e67e7cff42d3ad0399fd5.png)

反应钩子库连接 RxJS 星系和反应宇宙。

电子商务应用程序前端(FE)开发完全是关于事件管理和 DOM 操作的。RxJS 旨在处理具有竞争条件的复杂异步流。React 有助于最小化昂贵的 DOM 操作。在[经济学院](https://www.reonomy.com)，我们开发了一个名为反应钩子的库来利用两者的力量。

[](https://www.reonomy.com/blog/post/reactive-hooks) [## 反应之路:构建反应钩子库

### 为了使这种转变可行，雷诺米的软件工程师 Dmitry Doronin 创建了一个…

www.reonomy.com](https://www.reonomy.com/blog/post/reactive-hooks) 

## 什么是反应钩子库？

这是一个使用 [React 钩子](https://reactjs.org/docs/hooks-reference.html#usestate)渲染 [RxJS](https://rxjs-dev.firebaseapp.com/) 可观察物体的库。

[](https://www.npmjs.com/package/@reonomy/reactive-hooks) [## @经济/反应-挂钩

### RxJS React 钩子库

www.npmjs.com](https://www.npmjs.com/package/@reonomy/reactive-hooks) 

## 如何使用反应钩？

这个库提供了一组有用的钩子，可以直接导入到 React 组件中，例如:

```
import React from "react";
**import { useRxState } from "@reonomy/reactive-hooks";** import { BehaviorSubject } from "rxjs";const count$ = new BehaviorSubject(0);function Example() {
 **const [count, setCount] = useRxState(count$);**    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
                Click me
            </button>
        </div>
    );
}
```

# API 参考

这里列出了所有的挂钩，最常用的都标有星号。

*   *useRx(内部)*
*   *useRxDebug*
*   ***userxstate***⭐⭐⭐️⭐️⭐️
*   **⭐⭐️**
*   ***用户状态动作***
*   *****userxeffect****⭐️⭐️⭐️***
*   ******userxajax****⭐️⭐️⭐️⭐️****
*   ****用户去抖****
*   *****使用安装效果*** ⭐️**

## **useRx(内部)**

```
**useRx<T>(observable$: Observable<T>, observer: NextObserver<T>):void**
```

**当组件被安装时订阅给定的可观察对象，当组件被卸载时取消订阅，并跟踪所有活动的订阅。当 URL 哈希以**调试**(调试模式)结束时，活动订阅数将被打印到 web 控制台，例如 https://foo.bar# **调试**。这对于性能调优可能很有用。**

**这个钩子是内部的，所以你可能不想用它。但是知道这一点还是有好处的，因为 useRx 是所有其他钩子的基础。**

## ****useRxDebug****

```
**useRxDebug<T>(observable$: Observable<T>, tag: string): void**
```

**订阅给定的可观察值，并将所有更改记录到控制台(调试模式)。**

```
**import React from "react";
import { useRxDebug } from "@reonomy/reactive-hooks";
import { interval } from "rxjs";const interval$ = interval(1000);function Example() {
 **useRxDebug(interval$, "interval$");** return (
        <div>
            URL should end with #debug
        </div>
    );
}**
```

## **用户状态**

**让我们从一个例子开始:**

```
**const [foo, setFoo] = useRxState(foo$);**
```

**这种挂钩的简化版本可以描述如下:**

```
**useRxState<S extends Observable<T>>(subject$: Observable<T>): Out<T>**
```

**订阅 S 类型的给定可观察对象，并基于该类型返回 T 类型的当前值的元组和设置该值的回调。**

```
***+--------------------+-------------------------------------+
| Input Type S       | Output Type                         |* +--------------------+-------------------------------------+
| BehaviorSubject<T> | [T,             (value: T) => void] |
| Subject<T>         | [T | undefined, (value: T) => void] |
| Observable<T>      | [T | undefined ]                    |
+--------------------+-------------------------------------+**
```

**它是用 Typescript 条件类型实现的:**

```
**useRxState<
  S extends Observable<any> | 
  Subject<any> | 
  BehaviorSubject<any>
>(subject$: S): 
  S extends BehaviorSubject<infer T> ? [T, (value: T) => void]: 
  S extends Subject<infer T> ? [T | undefined, (value: T) => void]:  
  S extends Observable<infer T> ? [T | undefined]: 
  never**
```

**输出类型被动态定义。其背后的算法可以表示如下:**

```
**if (S is BehaviorSubject<T>)
    return [T, (value: T) => void]if (S is Subject<T>)
   return [T | undefined, (value: T) => void]if (S is Observable<T>) {
   return [T | undefined]
}**
```

**下面是一个用 useRxState 实现的双向数据绑定的示例:**

```
**import React from "react";
**import { useRxState } from "@reonomy/reactive-hooks";**
import { BehaviorSubject } from "rxjs";const fullName$ = new BehaviorSubject('');function FullName() {
 **const [fullName, setFullName] = useRxState(fullName$);**
    const onChange = e => **setFullName(e.target.value)**;    
    return (
        <div>
            <p>Full Name: {fullName}</p>
            <input 
                type="text" 
                onChange={onChange} 
                value={fullName} />
        </div>
    );
}**
```

## **useRxStateResult**

```
**useRxStateResult<T>(observable$: Observable<T>): T | undefined**
```

**返回当前值的 useRxState 的只读版本。**

```
**import React from "react";
**import { useRxStateResult } from "@reonomy/reactive-hooks";**
import { interval } from "rxjs";const interval$ = interval(1000);function Interval() {
    **const interval = useRxStateResult(interval$);** return (
        <div>
            Interval: {interval}
        </div>
    );
}**
```

## **useRxStateAction**

```
**useRxStateAction<T>(subject$: Subject<T>): (value: T) => void**
```

**返回修改所提供主题的回调。**

**下面是一个将“foo is Foo”打印到 web 控制台的示例。当你点击一个按钮时，它会变成“foo is Bar”。**

```
**import React from "react";
**import { useRxStateAction } from "@reonomy/reactive-hooks";**
import { BehaviorSubject } from "rxjs";const foo$ = new BehaviorSubject('Foo');
foo$.subscribe({
    next: val => console.log("foo is ", val);
})function FooBar() {
    **const setFoo = useRxStateAction(foo$);** return (
        <div>
            <p>{fullName}</p>
            <button type="button" onClick={() => setFoo("Bar")}>
                Make it Bar
            </button>
        </div>
    );
}**
```

## **useRxEffect**

```
 **useRxEffect<T>(
               observable$: Observable<T>, 
               next: (value: T) => void
): void**
```

**类似于 useEffect，但应该用作对可观察到的变化的反应。它让我想起 angularJS 里的“守望”，所以要小心！**

**示例:**

```
**import React from "react";
**import { useRxEffect } from "@reonomy/reactive-hooks";**
import { interval } from "rxjs";const interval$ = interval(1000);function WatchInterval() {
    **useRxEffect(interval$, *interval* => {** // do something
        console.log("interval is ", ***interval***); **});** return (
        <p>
            Look at the console!
        </p>
    );
}**
```

## **useRxAjax**

```
**useRxAjax<Req, Res>(
                    api$: (req: Req) => Observable<Res>,
                    next?: (response: Http<Req, Res>) => void): [Http<Req, Res> | undefined, (req: Req) => void]interface Http<Req, Res> {
  status: Status;  // 'succeeded' | 'failed' | 'pending';
  req: Req;
  res?: Res;
  error?: any;
}**
```

**调用数据 api 调用(获取请求)并返回一个带有响应对象和请求调度程序的元组。还提供了一个回调来对处于挂起、成功或失败状态的请求做出反应。**

**示例:**

```
**import React from 'react';
**import { useRxAjax } from '@reonomy/reactive-hooks';**
import { ajax } from ‘rxjs/ajax’;function weatherStatus({ apiKey }) { 
    return ajax({
        method: "GET",
        url: `https://api.openweathermap.org/data/2.5/weather?zip=10017,us&appid=${apikey}`)
    };
}function WeatherWidget() {
 **const [weather, getWeather] = useRxAjax(api.weather);**
    return (
        <article>
            <h1>Weather Status</h1>
            <button onClick={() => getWeather({apiKey: 'xxxxxx'})}>
                Refresh
            </button>
            <section> { weather && 
                  weather.status === 'pending' && 
                  <p>Pending...</p> } { weather && 
                  weather.status === 'failed' && 
                  <p>Failed: {JSON.stringify(weather.error)}</p> } { weather && 
                  weather.status === 'succeeded' &&
                  <p>Succeeded {JSON.stringify(weather.res)}</p> } </section>     
       </article>
    );
}**
```

## **useRxDebounce**

```
**useRxDebounce<In, Out>(     func$: (input: In) => Observable<Out>, 
    next?: (val: Out) => void, 
    debounce: number = DEBOUNCE): [Out | undefined, (input: In) => void]**
```

**使用给定的去抖超时调用回调。适用于自动建议。**

```
**import React, { useState } from 'react';
**import { useRxDebounce } from '@reonomy/reactive-hooks';**
import { ajax } from 'rxjs/ajax';function addressLookup({ address }) { 
    return ajax({
        method: "GET",
        url: "..."
    };
}function AddressAutosuggest() {
    const [address, setAddress] = useState('');
 **const [suggestions, updateSuggestions] = 
                                      useRxDebounce(addressLookup$);**    const onChange = e => {
        setAddress(e.target.value);
        updateSuggestions(e.target.value);
    };
    return (
        <article>
            <h1>Search by address</h1>
            <input type="text" onChange={onChange} value={address}/>
            { suggestions && 
                <ul>
                { suggestions.map({id, text} => 
                    <li key={id}>{text}</li>
                )}
                </ul> }
       </article>
    );
}**
```

## **useMountEffect**

```
**useMountEffect(onMountCallback: VoidFunction): void**
```

**安装组件时调用回调。当您需要在初始渲染时调用 http 请求时非常有用。**

```
**import React from 'react';
import { useRxAjax, useMountEffect } from '@reonomy/reactive-hooks';
import { ajax } from 'rxjs/ajax';function weatherStatus({ apiKey }) { 
    return ajax({
        method: "GET",
        url: `https://api.openweathermap.org/data/2.5/weather?zip=10017,us&appid=${apikey}`)
    };
}function WeatherWidget() {
    const [weather, getWeather] = useRxAjax(api.weather); **useMountEffect(() => getWeather({apiKey: 'xxxxxx'}));** return (
        <article>
            <h1>Weather Status</h1>
            <button onClick={() => getWeather({apiKey: 'xxxxxx'})}>
                Refresh
            </button>
            <section> { weather && 
                  weather.status === 'pending' && 
                  <p>Pending...</p> } { weather && 
                  weather.status === 'failed' && 
                  <p>Failed: {JSON.stringify(weather.error)}</p> } { weather && 
                  weather.status === 'succeeded' &&
                  <p>Succeeded {JSON.stringify(weather.res)}</p> } </section>     
       </article>
    );
}**
```

# **RxJS 无缝集成**

**React ant RxJS 是帮助处理 DOM 和异步事件的优秀库。有了 Reactive Hooks 库，它们可以顺畅地协同工作，确保整个应用程序的最优 DOM 操作和确定性行为。**
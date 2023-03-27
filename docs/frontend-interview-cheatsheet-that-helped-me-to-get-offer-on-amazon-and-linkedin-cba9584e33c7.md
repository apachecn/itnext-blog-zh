# 🔥帮助我获得亚马逊和 LinkedIn 录用通知的前端面试备忘单

> 原文：<https://itnext.io/frontend-interview-cheatsheet-that-helped-me-to-get-offer-on-amazon-and-linkedin-cba9584e33c7?source=collection_archive---------0----------------------->

如果你正在准备一个前端面试，并且想要快速更新你的前端领域知识，这个备忘单将会节省你很多时间。

![](img/7203285d48cf4a19c5caeb5d359c8840.png)

# 内容

[**简介**](#93de)

[**网络知识**](#f45b)

*   [**1。缓存**](#21a5)
*   [②**。HTTP/2**](#4f67)
*   [**3。安全**](#48af)

[**网页表现**](#59ff)

*   [**1。关键渲染路径**](#cfb1)
*   [**2。回流**](#5178)
*   [**3。预加载、预连接、预取、预渲染**、](#b7ea)
*   [**4。渲染性能**](#3a67)
*   [**5。工人**工人](#88cf)
*   [6**。图像优化**](#d8e2)

[**DOM**](#b93f)

*   [**1。**要素](#6776)
*   [**2。操纵**](#c97c)
*   [**3。文档片段**](#96ac)
*   [**4。事件委托和冒泡**和](#7e69)

[**HTML**](#a4e7)

*   [**1。语义元素**](#fb9d)
*   [2**。无障碍**](#dd22)
*   [**3。响应式 web**](#115c)

[**Javascript**](#071b)

*   **1。** `[**this**](#84ab)`
*   [2**2。**关闭](#84b2)
*   [**3。传承**](#34f3)
*   [**4。异步 Javascript**](#0728)
*   [**5。吊装**](#3cb1)
*   [**6。**谄媚](#59bc)
*   [7**7。高阶函数**](#6b87)

[**设计图案**](#ebeb)

*   [**1。mixin**](#88ca)
*   [2。工厂](#c627)
*   [**3。单例**](#862f)
*   [**4。立面**](#0b77)
*   [**5。MVC，MVVM**](#ca84)
*   [6**。服务器端与客户端渲染**](#829f)

[**结论**](#29b4)

[**你真棒**](#3534) **❤️**

[**了解更多**](#9985)

# 介绍

当然，没有足够的篇幅将所有的前端知识放入一篇文章中。这不是这份备忘单想要达到的目的。这只是前端话题的一个捷径，每个 sr 前端工程师都要熟悉。他们经常在面试中被提到，并帮助我在亚马逊和 LinkedIn 上得到一份工作。享受阅读，并通过点击主题链接随意深入阅读🙌

# 网络知识

***1。缓存***

*   [***Cache-Control***](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control):请求和响应缓存指令；
*   [***Etag***](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)*:<cache _ id>*通过比较 *< cache_id >* 检查资源是否更新，如果没有—更新缓存版本；

***2。*T3[T5*HTTP/2*T8](https://www.sitepoint.com/http2-the-pros-the-cons-and-what-you-need-to-know/)**

*优点:*

*   多个 HTTP 连接调用(HTTP/1 只支持 6 个)；
*   服务器可以将事件推送到客户端；
*   压缩标题；
*   更安全

*缺点:*

*   服务器推送可能被滥用；
*   如果 LB(负载平衡器)支持 HTTP/1 和服务器 HTTP/2，速度可能会慢一些

***3。安全***

*   [](https://www.geeksforgeeks.org/http-headers-transfer-encoding/)**—定义如何加密正文: *chunked* ，*gzip；***
*   **[*访问-控制-允许-起源*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) (跨起源资源共享——CORS)定义了可以访问起源域 API 的域列表；**
*   **[*JSONP*](https://www.w3schools.com/js/js_json_jsonp.asp#:~:text=JSONP%20stands%20for%20JSON%20with,instead%20of%20the%20XMLHttpRequest%20object.) —运行脚本访问跨域数据(旧浏览器)；**
*   **[*X-Frame-Options*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)—防止 iframe 点击劫持；**
*   **[*跨站请求伪造*](https://owasp.org/www-community/attacks/csrf) **(** CSRF)。*攻击*:用户有一个会话(已登录)，攻击者创建链接，用户点击链接并执行请求，攻击者窃取用户会话。*防止:*验证码，从访问过的站点注销；**
*   **[*内容-安全-策略*](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) —阻止有害代码的执行；**
*   **[*X-XSS-保护*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)-启用 XSS 保护旧址；**
*   **[*功能-策略*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy) *—* 禁用不需要的浏览器功能；**
*   **[*Referrer-Policy*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)—当有一个从您的网站到另一个网站的链接时，通过点击它将发送包含一些敏感数据(用户 id、会话)的原始 URL**
*   **不允许用户输入任何 HTML`innerHtml`；**
*   **使用 UI 框架，保持 node_modules 更新，并限制使用第三方服务；**

# **Web 性能**

*****1。*** [***关键渲染路径***](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path) —步骤浏览器制作绘制页面。这些步骤是:**

*   ***DOM* —浏览器编译文档对象模型；**
*   ***CSSOM* —浏览器编译 CSS 对象模型；**
*   ***渲染树* —浏览器结合 DOM 和 CSSOM 渲染树；**
*   ***布局* —浏览器计算每个对象的大小和位置；**
*   ***画图* —浏览器将树转换成屏幕中的像素；**

**优化 ***CRP:*****

*   ***优化资源顺序* —尽快加载关键资源；**
*   ***最小化源数量* —减少数量，加载异步；**

****2*。*** [***重新绘制***](https://developers.google.com/speed/docs/insights/browser-reflow) *—* 浏览器重新绘制后重新计算元素的位置和几何图形。**

**优化 ***回流:*****

*   **减少 DOM 深度；**
*   **避免长的 CSS 选择器，最小化规则；**

*****3。预加载、预连接、预取、预呈现*和****

*   **[*预加载*](https://developer.mozilla.org/en-US/docs/Web/HTML/Link_types/preload)*——*加载需要更快加载的高优先级源`<link rel="preload">`；**
*   **[*pre connect*](https://developer.mozilla.org/en-US/docs/Web/HTML/Link_types/preconnect)*——*如果需要一些资源来加速握手，使用`<link rel="preconnect">`来减少延迟；**
*   **[*预取*](https://developer.mozilla.org/en-US/docs/Glossary/Prefetch) —加载低优先级资源和缓存`<link rel="prefetch">`；**
*   **[*dns 预取*](https://developer.mozilla.org/en-US/docs/Web/Performance/dns-prefetch)—在资源被请求之前减少解析域名的延迟`<link rel="dns-prefetch">`；**
*   **[prerender](https://developer.mozilla.org/en-US/docs/Glossary/prerender) —类似于*预取* +缓存整个页面`<link rel="prerender">`；**

*****4。渲染性能*****

****JS:****

*   **将繁重的任务转移给[网络工作者](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)；**
*   **用`[requestAnimatinFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)`代替`setTimeout`进行动画；**

****样式:****

*   **降低选择器的复杂度；**
*   **减少必须应用样式计算的元素数量；**

****布局:**(元素的位置和大小)**

*   **避免布局变化；**
*   **使用`[flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)`；**
*   **使用`[css-grid](/how-i-learned-css-grid-in-5-min-ec6439d8bf0)`；**

****绘制:**(绘制像素:颜色、阴影；布局改变触发重画)**

*   **使用`[will-change](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change)`优化布局重画；**

*****5。工人*****

*   **[*服务人员*](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker)*——*拦截器搭建离线 app**
*   **[*Web Worker*](https://developer.mozilla.org/en-US/docs/Web/API/Worker)*——*后台执行繁重任务；**

*****6。图像优化*****

****格式** *:***

*   **如果是动画——用`<video>`代替 *gif***
*   **if 高细节和分辨率— *PNG***
*   **如果几何形状— *SVG***
*   **如果是文本徽标—使用字体文本**
*   **如果照片— *JPEG***

****压缩** *:***

*   ***SVG* —使用优化器(像 [SVGO](https://github.com/svg/svgo) )，使用 gzip**
*   ***WebP* —使用优化的网页图像格式；**
*   **从 *SVG 标签中删除元数据属性；***
*   **使用 [*小精灵*](https://www.w3schools.com/css/css_image_sprites.asp)；**

****缓存和延迟加载** *:***

*   **使用 [*CDN*](https://aws.amazon.com/cloudfront/) 分发静态数据；**
*   ***惰性加载*图片和视频——使用`<img loading="lazy"/>`或类似 [*的库 Lazy sizes*](https://github.com/aFarkas/lazysizes)；**

# **数字正射影像图**

****1。元素****

*   ****选择器:** `getElementById`，`getElementByTagName`，`querySelector`，*，*，`querySelectorAll`；**
*   ****导航:** `children`(元素):`childNodes`(节点)、`firstElementChild`、`lastElementChild`、`parentElement`、`previousElementSibling`、`nextElementSibling`；**
*   ****属性:**`classList``clientHeight``clientWidth``childElementCount`*`setAttribute(attrName, value)``removeAttribute(attrName)``removeAttribute(attrName)`；***

*****2*2。操纵*和*****

**`createElement(‘div’)`、`append`、`prepend`、`el.cloneNode(true)`、`remove()`、`insertBefore(newNode, beforeNode)`、`insertAfter(newNode, afterNode);`**

*****3。*** [***文档片段***](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment) *—* 创建文档的虚拟副本，可以存储多个元素。通过将*文件片段*插入 DOM，它变成空的，并只引起一次[回流](#5178)；**

****4*。*** [***事件委托和冒泡***](https://programmingwithmosh.com/javascript/javascript-event-bubbling-and-event-delegation/)**

*   **当我们发出一个*事件时，* ex。`click`，事件通过`parentElement`链接冒泡到`<html>`元素:**

```
**-html (bubble click)
   -body (bubble click)
        -div (bubble click)
            -p 
            -p (click)**
```

*   ***委托*用于提高性能。假设我们有一个结构:**

```
**-div.parent
    -p.child 
    -p.child
    -p.child**
```

**我们希望将一个`addEventListener`分配给`.child`，在这种情况下，我们必须将一个事件附加到 3 个元素上。相反，我们可以只将事件附加到`.parent`并解析逻辑。**

```
**document.querySelector(".parent").addEventListener("click", function(event) {
    if (event.target.classList.contains("child")) {
      // you logic is here
    };
});**
```

# **超文本标记语言**

****1。语义元素**——用名称向开发者和浏览器清晰地描述其含义:`<article>`、`<aside>`、`<details>`、`<figcaption>`、`<figure>`、`<footer>`、`<header>`、`<main>`、`<mark>`、`<nav>`、`<section>`、`<summary>`、`<time>`；**

*****2。*无障碍****

*   **使用割台`<h1>,<h2>,<h3>…`；**
*   **使用`<img alt=””`；**
*   **使用属性`tabindex=”index_position”`通过`Tab`键导航焦点；**
*   **用`roles`像`<ul role=”list”><li role=”listitem”>`、`<dialog role=”dialog”`。在这里 找到全榜[；](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)**
*   **使用`accesskey=”key”`创建键盘快捷键；**
*   **用属性描述元素:`aria-label=”label_text”`或`aria-labelledby=”text_id”`、`aria-describedby=”text_id”`和`<label id="text_id">label_text</label>`；**
*   **使用颜色对比、纹理；**
*   **使用文本大小；**
*   **在视频中使用字幕；**

*****3。响应式 web*****

*   **添加`<meta viewport name=”viewport” content=”width=device-width, initial-scale=1.0"`给浏览器方向缩放；**
*   **使用`<img max-width=”100%”`,图像的缩放不会超过其尺寸；**
*   **使用`<[picture](https://www.w3schools.com/tags/tag_picture.asp)> <source srcset=”” media=”” >`指定不同屏幕尺寸的图像；**
*   **响应式 [**字体大小**](https://css-tricks.com/confused-rem-em/) : `em`和`rem`；**
*   **使用[媒体查询](https://css-tricks.com/a-complete-guide-to-css-media-queries/)；**

# **java 描述语言**

*****1。*****

*   **`this`是对对象的引用，其中函数是**调用**；**
*   **默认`this`上下文为`window`；**
*   **函数将从它调用的地方获取的上下文；**
*   **箭头函数`->`获取**定义的函数的上下文；****
*   **当我们在一个函数中调用另一个函数时，会丢失上下文**

```
**function Foo() {
    console.log(this); 
}
Foo(); // at this line the context is 'window'
// output: 'window'var foo1 = new Foo(); // at this line the context binds to 'foo1'
// output: 'Foo {}'**
```

*   **显式分配`this`的上下文:`foo1.apply(context, arrayOfArgs)`、`foo1.call(context, arg1, arg2, …)`、`var foo2 = foo1.bind(context, arg1, arg2, …)` —返回给定上下文的函数实例；**

*****2。*** [***闭包***](https://javascript.info/closure)***—***函数能够记住并访问作用域，即使是从另一个作用域调用(函数返回函数/块作用域中的块作用域)**

```
**function a(arg1) { // arg1 scoped
    var arg2 = 0; // arg2 scoped return function b(){
        ++arg2;
        return arg1 + arg2;
    }
}var foo = a(2);
foo(); // 3
foo(); // 4
var foo2 = a(4);
foo(); // 5
foo(); // 6**
```

*****3。*** [***继承***](https://javascript.info/prototype-inheritance)**

*   **为了从`obj2`继承`obj1`，你可以将一个对象链接到另一个对象`var obj1 = Object.create(obj2);`**
*   **JS 使用原型继承。每个对象都有一个`__proto__`链接。如果我们访问对象的任何属性，JS 引擎首先检查对象是否有，如果没有—检查原型，并通过`__proto__`链找到属性名，如果没有找到则抛出`undefined`；**

****4*。*** [***异步 Javascript***](/the-evolution-of-asynchronous-patterns-in-javascript-64efc8938b16)**

*   ****事件循环**:JS 中有三种内存:`stack`用于函数调用，`heap`用于每个对象，`queue` — setTimeout。JS 引擎首先执行功能`stack`。如果`stack`为空，则从`queue`弹出事件。如果事件`queue`有另一个函数调用，它会将其推送到`stack`并再次执行，直到它为空。这叫做事件循环；**
*   **JS 使用`callback`、`promise`、async-await 来实现异步模式。你可以在本文中阅读更多关于**异步 JS** 和**事件循环**的内容:**

**[](/the-evolution-of-asynchronous-patterns-in-javascript-64efc8938b16) [## 🔥JavaScript 中异步模式的演变

### 让我们谈谈 JavaScript 中使用的异步模式

itnext.io](/the-evolution-of-asynchronous-patterns-in-javascript-64efc8938b16)** 

*****5。*** [***吊装***](https://javascript.info/var)**

*   **`function`定义在 JS 编译过程中移动到块范围的顶部，然后进入`var`；**
*   **`let`和`const`也被吊起，但在临时死区；**

```
**// Code example              // After hoisting 
1\. console.log('hoisting');  1\. function foo(){
2\. var a;                    2\.    return null;
3\. function foo(){           3\. }
4\.    return null;           4\. var a;
5\. }                         5\. console.log('hoisting');
6\. a = 5;                    6\. a = 5;**
```

*****6。***[](https://javascript.info/currying-partials)***—嵌套函数:*****

```
**function calcABC(a){
    return function(b){
        return function(c){
            return a+b+c;
        }
    }
}console.log(calcABC(1)(2)(3));
// 6**
```

*****7。*** [***高阶函数***](https://www.freecodecamp.org/news/a-quick-intro-to-higher-order-functions-in-javascript-1a014f89c6b/)**

*   **`map`、`reduce`、`filter`、`find`**
*   **您可以将高阶函数链接到`composition`**

```
**[1,2,3,4,5].map((num) => ({age: num})).filter((el) => el.age < 4);
// [{age: 1}, {age: 2}, {age: 3}]**
```

# **设计模式**

*****1。***[***Mixin***](https://javascript.info/mixins)***——***用方法列表扩展一个对象的功能；**

```
**// Option 1
class Person{}let Mixin = {foo(){}};Object.assign(Person.prototype, Mixin);let person = new Person();person.foo();// Option 2let Mixin1 = {foo1(){}};let Mixin2  = {__proto__: Mixin1, foo2(){}};**
```

*****2。*** [***工厂***](https://www.javascripttutorial.net/javascript-factory-functions/)**—一个*类，可以创建一个或多个不同的对象(如果你想在单元测试中生成不同的模拟数据，这很有用)；***

```
**personFactory.one({name: 'John'}); -> Person{name: 'John', age: 5}
personFactory.many(); -> [Person{name: 'Bill', age: 7}, Person{name: 'Anna', age: 3}]**
```

*****3。***[***Singleton***](https://www.sitepoint.com/javascript-design-patterns-singleton/)*—可以直接调用方法的类，不需要创建对象；***

```
***let mySingleton = (function() { let instance = null; function init(){
        return {
           //list all the methods method(){}
        }
    }
    if(instance == null){
        instance = init();
    } return instance;
})();mySingleton.method();***
```

******4。*** [***门面***](https://www.dofactory.com/javascript/design-patterns/facade)***——***抽象更复杂的逻辑，在类中包装。例如，位于组件和 API 层之间的服务:***

```
**ui component - Facade service (complex state object) - API layer (Redux);**
```

*****5。MVC，MVVM —*** [**模型视图控制器**](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) 和 [**模型视图视图模型**](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel) **。****

***React 是 MVC***

*   **状态—`Model`；**
*   **JSX—`View`；**
*   **动作(违反——可以与视图混合)——`Controller`；**

***棱角分明的是 MVVM***

*   **组件— `ModelView`**
*   **模板— `View`(违例—不可重用)**
*   **属性— `Model`**

*****6。*** [***服务器 vs 客户端渲染***](https://www.freecodecamp.org/news/what-exactly-is-client-side-rendering-and-hows-it-different-from-server-side-rendering-bd5c786b340d/)**

****SSR —** 如果一个网站是稳定的，静态的，以 SEO 为重点，可以支付额外的服务器，就使用 SSR；**

***优点***

*   **更快的页面加载(可查看，但不可交互)；**
*   **更适合搜索引擎(更快的索引)；**
*   **最好是有大量静态内容的网站(博客)；**

***缺点***

*   **更多的服务器请求；**
*   **较慢的渲染以进行交互；**
*   **整页重新加载；**

****CSR —** 如果站点正在开发、动态，则使用 CSR；**

***优点***

*   **初始加载后渲染速度更快；**
*   **最适合 web app**

***缺点***

*   **初始加载可能需要更多时间**

# **结论**

**一个开发人员需要知道大量的信息，才能有信心通过大型科技公司的前端面试。虽然更复杂的回合由编码问题和系统设计组成，但是如果你对关于编码问题的单独文章感兴趣，让我们收集 5000 个掌声👏前端知识领域对于回避任何类型的网络概念问题都是非常重要的。别忘了**关注** **和** **订阅** **如果你今天**学到了新的东西(并且想每周获得更多见解)。回头见。😉**

**[](https://medium.com/@easy-web/subscribe) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

medium.com](https://medium.com/@easy-web/subscribe) [](https://medium.com/@easy-web/membership) [## 通过我的推荐链接加入 Medium 维塔利·舍甫琴科

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@easy-web/membership) 

# 你真棒 **❤️**

向最近加入我的博客的所有这些了不起的人致以崇高的敬意。谢谢大家让我保持动力，你们是最棒的！🙌

)(他)(们)(都)(不)(知)(道)(,)(他)(们)(还)(不)(知)(道)(,)(他)(们)(还)(有)(些)(不)(知)(道)(的)(情)(况)(,)(他)(们)(还)(不)(知)(道)(,)(他)(们)(还)(不)(知)(道)(,)(他)(们)(还)(是)(不)(知)(道)(,)(还)(不)(知)(道)(,)(他)(们)(还)(不)(知)(道)(,)(还)(不)(知)(道)(,)(他)(们)(还)(有)(些)(不)(知)(道)(道)(,)(他)(们)(还)(不)(知)(道)(道)(,)(他)(们)(们)(还)(不)(知)(道)(道)(,)(他)(们)(们)(还)(还)(有)(些)( )(他)(们)(都)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(是)(有)(些)(人)(,)(但)(我)(们)(还)(不)(知)(道)(理)(。 )(我)(们)(都)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(,)(我)(们)(还)(不)(知)(道)(。

# 了解更多信息

[](https://easy-web.medium.com/system-design-concepts-that-helped-me-get-sr-frontend-offers-from-amazon-linkedin-9e100f3ce7d2) [## 🔥帮助我从亚马逊和 LinkedIn 获得服务请求前端的系统设计概念

### 如果您刚刚开始您的系统设计之旅，这个概念的汇编将帮助您从基础开始

easy-web.medium.com](https://easy-web.medium.com/system-design-concepts-that-helped-me-get-sr-frontend-offers-from-amazon-linkedin-9e100f3ce7d2) [](/how-to-become-a-sr-frontend-engineer-in-amazon-without-leetcode-9b7ec604a12) [## 没有 LeetCode 如何成为亚马逊高级前端工程师🔥

### 这不是 clickbait，这是一个关于我如何从讨厌 LeetCode 到在大型科技公司获得多个邀请的故事。

itnext.io](/how-to-become-a-sr-frontend-engineer-in-amazon-without-leetcode-9b7ec604a12) [](/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad) [## 🔥微前端如何改变 Angular 的未来？

### 让我们看看为什么 Angular 最适合微前端

itnext.io](/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad)**
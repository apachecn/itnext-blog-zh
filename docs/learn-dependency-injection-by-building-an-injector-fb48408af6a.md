# 通过构建注入器来学习依赖注入

> 原文：<https://itnext.io/learn-dependency-injection-by-building-an-injector-fb48408af6a?source=collection_archive---------4----------------------->

![](img/573015afdb46078c15db2410a95a70b8.png)

如果你用 Angular 构建过任何东西，毫无疑问你对依赖注入(DI)有一些经验，但是并不是每个人都使用 Angular，甚至使用 Angular 的人可能并不真正理解 DI 在做什么。为了真正探索这一点，我们可以用大约 20 行的类型脚本构建我们的 DI 容器(您可以使用普通的 JavaScript，但我觉得类型注释会有所帮助)。

首先，让我们从 DI 的简单定义开始。来自[维基百科](https://en.wikipedia.org/wiki/Dependency_injection)“在[软件工程](https://en.wikipedia.org/wiki/Software_engineering)、**依赖注入**是一种一个对象提供另一个对象的依赖的技术。”虽然这听起来不错，但让我们看一个具体的例子。

```
class HttpService {
  get(url: string) {...}
  post(url: string) {...}
}class UserService {
  constructor(private http: HttpService) {} login(email: string, password: string) {
    return this.http.post('/login', { email, password})
  }
}const userService = new UserService(new HttpService())
```

在上面的例子中，我们不希望我们的用户服务关心如何发出 http 请求，我们希望它关心如何登录。虽然上面的例子现在满足了我们的需求，但随着我们获得更多的依赖，它可能会变得很麻烦。我们不想关心你如何创建一个 HttpService，我们想说“嘿，我需要发出 HTTP 请求，给我 HTTP 服务！！！!"这是注射容器变得有用的时候。前面使用其中一个神秘容器的例子可能看起来像这样。

```
class HttpService { ... }class UserService { ... }// this is the thing we are going to build
const injector = new Injector();const userService = injector.get(UserService);
```

好的，我们希望我们的注射器做一些事情:

1.  它应该为给定的类创建一个新的实例。
2.  它应该创建给定类的依赖关系的新实例。
3.  它应该返回单例对象。这意味着如果先前已经创建了一个实例，它将返回该实例。

## 步骤 1–2:创建实例和依赖关系

首先，我们需要创建一个新实例。在继续之前，停下来想一想，我们需要哪些信息来动态地做这件事……想过吗？我们需要知道依赖关系是什么。为了使事情更明确，我们将静态声明我们的类的依赖关系。

```
class HttpService {}class AuthService {
  static deps = [HttpService];
  ...
}
```

现在我们可以拥有创建新实例所需的一切，并且可以创建我们的注射器的第一个版本了！

```
// This type defines a "Provider" 
// as a class with a static "deps" property
type Provider<T> = {
  deps: Provider<any>[]; new (...args: any[]): T;
};class Injector {
  get<T>(P: Provider<T>): T {
    return new P(...P.deps.map(dep => this.get(dep)));
  }
}
```

就是这样！我们创建了一个新的“P”实例，并递归地调用“Injector.get”来获得每个依赖项的实例，非常简单！每次调用“Injector.get”时，都会创建新的实例。

## 步骤 3:缓存结果并返回单例

既然我们已经成功地创建了服务和服务依赖的新实例，我们需要以某种方式记住它们，以便以后可以返回它们。我们需要一个贮藏处！

```
type Provider<T> = { ... };class Injector {
  // singleton cache
  private cache = new WeakMap<Provider<any>, any>(); get<T>(P: Provider<T>): T {
    // if provided is in cache return it
    if (this.cache.has(P)) {
      return this.cache.get(P);
    } // if no existing instance create a new one
    const instance = new P(...P.deps.map(dep => this.get(dep)));

    // then cache the resulting instance
    this.cache.set(P, instance);

    return instance
  }
}
```

现在我们有了缓存，我们总是可以返回一个 singleton，这意味着如果我们愿意，我们的对象现在可以保存状态。

现在让我们看一个更复杂的例子，看看我们可以用上面的注射器做什么。

就是这样！希望这个简单的阿迪容器的例子能帮助你理解 DI 在做什么，以及它是如何工作的。

您还可以为这个注入器实现添加什么其他特性？如果一个注射器可以有一个父代呢？覆盖一个提供者会是什么样子？试试吧，让我知道你想出了什么！

如果你想看更复杂的实现，请看这里。
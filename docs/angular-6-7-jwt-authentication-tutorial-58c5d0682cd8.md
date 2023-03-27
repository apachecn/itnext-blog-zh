# Angular 6|7: JWT 认证教程

> 原文：<https://itnext.io/angular-6-7-jwt-authentication-tutorial-58c5d0682cd8?source=collection_archive---------1----------------------->

在本教程中，您将通过创建一个可用于处理 JWT 认证的示例 Angular 服务，学习在 Angular 7 应用程序中实现 JWT 认证。

![](img/9f1b51e75f49aebe864c514338af53d2.png)

在本教程中:

*   您将首先安装项目的需求，如 Node.js、npm 和 Angular CLI v7，
*   接下来，您将创建 Angular 7 应用程序，
*   您将在您的应用程序中设置`HttpClient`，
*   您将创建一个 Angular 7 服务来处理 JWT 认证，
*   最后，您将安装并配置`angular-jwt`来将 JWT 访问令牌附加到请求上。

# 了解 JWT

在开始实践之前，让我们简要了解一下什么是 JWT。

JWT 代表 JSON Web Token，它是一个开源标准，规定了如何在计算机系统之间安全地交换信息。

JWT 令牌只是一个紧凑的自包含 JSON 对象，它包含像电子邮件和密码这样的信息。

您可以使用 JWT 在 Angular 7 应用程序中添加身份验证，而无需求助于在 web 应用程序中实现身份验证的传统机制，如会话和 cookies。

以下是 JWT 在您的 web 应用程序中的工作方式。首先，用户登录，您的 web 服务器为用户的凭证创建一个 JWT 令牌，并将其发送回用户的浏览器。之后，JWT 将保存在浏览器的本地存储中，并随每个 HTTP 请求一起发送到服务器，以便能够访问任何受保护的 API 端点。

# 先决条件

为了能够完成本教程，您需要满足一些要求:

*   首先，您需要在系统上安装节点和 NPM。否则，你可以简单地访问[nodejs.org](https://nodejs.org/en/download/)并下载你的系统的二进制文件。
*   接下来，您需要安装 Angular CLI v7。如果没有安装，您只需运行`npm install -g @angular/cli`命令，在您的系统上全局安装 CLI。
*   最后，您需要有一个 Angular 7 项目，或者简单地运行`ng start angular7-jwt-example`命令并回答 CLI 问题来生成您的项目。

安装了这些需求之后，您应该可以开始创建 Angular 7 服务了，该服务封装了在 Angular 应用程序中实现 JWT 认证的所有代码。

# 设置`HttpClient`

在能够向服务器发送 HTTP 请求之前，您需要设置`HttpClient`。我们已经在之前的[教程](https://www.techiediaries.com/angular-httpclient)中做过了。您可以按照该教程获取更多关于使用`HttpClient`的信息，或者通过从`@angular/common/http`包中导入`HttpClientModule`并将其包含在应用程序模块的`imports`数组中来简单地设置`HttpClient`。

打开`src/app/app.module.ts`文件，添加以下代码:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

接下来，您只需要在您的服务和组件中导入和注入`HttpClient`。

在下一部分中，您将创建 JWT 服务。

# 构建身份验证服务

在本节中，您将创建一个封装了 JWT 认证逻辑的 Angular 7 服务。

在您的终端中，运行以下命令以使用 Angular CLI 生成服务:

> *也可以用* `*generate*` *代替* `*g*` *。*

接下来，打开`src/app/jwt.service.ts`文件，导入`HttpClient`类并注入它:

```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
providedIn: 'root'
})

export class JwtService {
    constructor(private httpClient: HttpClient) { }
}
```

现在，您已经准备好添加`login()`、`logout()`和`loggedIn()`方法了。

# 服务器端应用

在本教程中，您不会学习如何创建服务器应用程序。

您的服务器端应用程序需要实现 JWT 身份验证，并公开一些端点:

*   `/auth/login`用于登录用户。此端点应该接受带有用户凭据的 POST 请求，并返回浏览器将接收到的 JWT 访问令牌。
*   `/auth/register`用于注册用户。此端点应该接受包含用户凭证的 POST 请求，并将它们保存在数据库中。

# 添加身份验证方法

现在，在 Angular 7 应用程序中创建了 JWT 服务之后，您需要实现必要的方法，这些方法将用于处理应用程序中的身份验证。

# 添加`.login`方法

让我们从定义`.login`方法开始。它应该接受电子邮件和密码参数，并返回和 RxJS `Observable`。

打开您的`src/app/jwt.service.ts`文件，并在您的服务中添加以下方法:

```
login(email:string, password:string) {
    return this.httpClient.post<{access_token:  string}>('[http://www.your-server.com/auth/login](http://www.your-server.com/auth/login)', {email, password}).pipe(tap(res => {
    localStorage.setItem('access_token', res.access_token);
}))
}
```

那么你做了什么？您首先使用`HttpClient.post`方法向`/auth/login`端点发送一个请求，其中包含作为参数传递的电子邮件和密码。

接下来，您使用了作为 RxJS `Observable`成员的`.pipe`方法来链接操作符，并使用了`tap`函数来执行一个副作用，将服务器返回的 JWT 访问令牌保存在浏览器的本地存储中。

> *确保使用* `*import { tap } from 'rxjs/operators';*`导入 `*tap*` *运算符*

# 添加`.register`方法

就像`.login`方法一样，您还需要添加一个`.register`方法，该方法向服务器发送首次注册用户的请求:

在 Angular 7 服务中，添加以下方法:

```
register(email:string, password:string) {
    return this.httpClient.post<{access_token: string}>('[http://www.your-server.com/auth/register](http://www.your-server.com/auth/register)', {email, password}).pipe(tap(res => {
    this.login(email, password)
}))
}
```

同样，您已经使用了`HttpClient.post`方法向服务器发送一个 POST 请求和注册信息(电子邮件和密码),然后使用`.pipe`和`tap`函数运行一个副作用，调用`.login`方法在注册完成后让用户登录。

# 添加`.logout`方法

接下来，您需要实现一个注销用户的`.logout`方法。这个方法不需要向服务器发送任何请求，它需要做的只是从用户的本地存储中删除 JWT 访问令牌。例如:

```
logout() {
  localStorage.removeItem('access_token');
}
```

您调用`localStorage`的`.removeItem`方法来移除名为`access_token`的键。

# 添加`.loggedIn`属性

最后，您需要创建`.loggedIn`属性来验证用户是否登录。这是通过检查浏览器的本地存储中是否存在 JWT 访问令牌来实现的:

```
public get loggedIn(): boolean{
  return localStorage.getItem('access_token') !==  null;
}
```

你使用了`localStorage`的`.getItem`方法得到了`access_token`物品。如果它不存在，该方法返回一个空对象。

> *你可以通过在方法定义前添加一个* `*get*` *修饰符来创建一个属性。然后，您可以在不使用括号的情况下访问属性。*

现在，您已经实现了将用于登录、注销、注册和检查用户状态的所有身份验证方法。您仍然需要将访问令牌附加到将被发送到服务器以访问受保护端点的每个请求上。并使用防护装置保护视角。

# 安装和设置`angular-jwt`

在 Angular 7 服务中添加实现 JWT 认证所需的方法后。现在让我们看看如何将收到的访问令牌附加到每个请求上。

您可以使用 Auth0 中的`angular-jwt`库来完成这项工作。因此，首先使用以下命令从 npm 安装它:

```
$ npm install @auth0/angular-jwt --save
```

`angular-jwt`库实现了发送访问令牌和每个 HTTP 请求所需的代码，但是它需要一些设置。

打开`src/app/app.module.ts`文件，导入`@auth0/angular-jwt`包中的`JwtModule`:

```
import { JwtModule } from '@auth0/angular-jwt';
```

接下来，您需要将它包含在应用程序模块的`imports`数组中:

```
imports: [
    # [...]
    JwtModule.forRoot({
      config: {
        tokenGetter: function  tokenGetter() {
             return     localStorage.getItem('access_token');},
        whitelistedDomains: ['localhost:3000'],
        blacklistedRoutes: ['[http://localhost:3000/auth/login](http://localhost:3000/auth/login)']
      }
    })
  ],
```

您使用`JwtModule`的`.forRoot`方法来提供具有以下属性的配置对象:

*   `tokenGetter`:该函数用于定制`JwtModule`如何从本地存储器获取 JWT 访问令牌。
*   `whiteListedDomains`:在这个数组中，你可以添加任何允许接收 JWT 的域，比如公共 API。
*   `blackListedRoutes`:在此数组中，您可以添加不允许接收 JWT 令牌的路由。

在这个例子中，您将`localhost:3000` URL 添加到白名单域中，因此只有从这个地址运行的 Angular 应用程序将接收访问令牌。

您还将`localhost:3000/auth/login` URL 列入黑名单，因为它不需要接收任何访问令牌。

# 结论

在本教程中，您学习了如何在 Angular 7 应用程序中实现 JWT 认证。
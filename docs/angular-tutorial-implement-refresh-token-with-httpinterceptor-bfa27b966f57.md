# Angular 教程—使用 HttpInterceptor 实现刷新令牌

> 原文：<https://itnext.io/angular-tutorial-implement-refresh-token-with-httpinterceptor-bfa27b966f57?source=collection_archive---------0----------------------->

![](img/87ccc284e046afbb31aa0a960cb90f8c.png)

> 本文解释了如何在新的 Angular 框架中使用 HttpInterceptor **实现刷新令牌。**

Angular 版本引入了最期待的功能:`HttpInterceptor`界面。在这个版本之前，没有办法全局修改或拦截 http 请求。

HTTP 拦截器为什么有用？HTTP 拦截器用于为日志记录、修改响应、错误处理添加自定义逻辑，但一个常见的情况是自动将身份验证信息附加到请求并刷新令牌，以便保持用户会话活动。

# 创建一个拦截器

> 本教程假设您的应用程序中已经有一个认证服务，并且您正在本地存储中存储 JWT 令牌。

第一步是创建一个拦截器。为此，让我们创建一个实现`*HttpInterceptor*`的`*Injectable*`类，并捕捉可能发生的错误。

```
// src/app/services/token-interceptor.service.ts

import { Injectable } from '@angular/core';
import { HttpRequest, HttpHandler,HttpEvent, HttpInterceptor } from '@angular/common/http';
import { AuthenticationService } from '../authentication.service';
import { Observable } from 'rxjs/Observable';

@Injectable()
export class RefreshTokenInterceptor implements HttpInterceptor {

  constructor(public auth: AuthenticationService) {}

  intercept(request: HttpRequest<any>, next: HttpHandler):                   
  Observable<HttpEvent<any>> {

    return next.handle(request)
       .catch(error => {
           *return* Observable.*throw*(error);
       }); }
}
```

我想提一下，你必须使用从 **@angular/common/http** 导入的新 HttpClient，因为旧的 http 客户端不会触发 HttpInterceptor。

# 寻找未授权的响应

当访问令牌到期时，服务器通常会发送一个`401 Unauthorized`响应。在这种情况下，我们需要再次登录用户，以便使用新的访问令牌继续使用应用程序。

重要的是检查失败的请求是否不是刷新令牌请求本身，以避免递归。此外，我们需要检查刷新令牌请求是否正在进行，因为我们不希望其他调用进来并再次调用 refresh token。代码中注释了另一个重要步骤:

# 结论

新的 HttpInterceptor 是一个强大而有用的特性，它是我们可以拦截和/或改变传出请求或传入响应的更好的地方。
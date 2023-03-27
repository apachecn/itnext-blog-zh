# 用 Angular 中的 HttpInterceptor 和 Toastr 处理 http 响应

> 原文：<https://itnext.io/handle-http-responses-with-httpinterceptor-and-toastr-in-angular-3e056759cb16?source=collection_archive---------0----------------------->

![](img/20bac6d54b22fdba7f590fe2abf2fd34.png)

您是否曾经需要一个集中的系统来处理您的 http 响应？我确信，每当您在一个有大量 http 请求的大项目中工作时，这个问题就会浮出水面。在本文中，我将向您展示如何处理它，通过使用 http 拦截器和`ngx-toastr`插件来显示这些消息。

让 HttpInterceptor 工作非常简单，需要两个步骤:

1.  创建一个实现`HttpInterceptor`接口的类
2.  提供拦截器

开始吧！

**http 接收器**

HttpInterceptor 是一个可以由类实现的接口，它只有一个方法来拦截传出的`HttpRequest`，并有选择地转换它或响应。那个方法叫做`intercept.`

`intercept`方法接受两个参数:`req` (a `HttpRequest`)和`next` (a `HttpHandler`)并返回一个`Observable`

```
intercept( **req: HttpRequest<any>,** **next: HttpHandler**): **Observable<HttpEvent<any>>** {}
```

通常拦截器会在返回`next.handle(req)`之前转换传出的请求。拦截器也可以选择转换响应事件流，方法是对由`next.handle()`返回的流应用额外的 Rx 操作符。

```
return next.handle(req).pipe(
            **tap(evt => {}),
            catchError(error => {})**
);
```

在本文中，我将向您展示如何处理成功和错误响应。如果我们的 http 请求成功，我们希望显示一条祝酒成功消息。另一方面，当 http 请求返回错误时，我们将通过 toast 显示一条错误消息。

在我们开始构建拦截器之前，我们必须导入所有必要的方法、类、服务，并在构造函数中注入`ToastrService`。

```
import { HttpInterceptor, HttpHandler, HttpRequest, HttpEvent, HttpResponse, HttpErrorResponse }   from '@angular/common/http';import { Injectable } from "@angular/core"import { Observable, of } from "rxjs";import { ToastrService } from 'ngx-toastr';
...constructor(**public toasterService: ToastrService**) {}
```

让我们首先处理成功消息。为此，我们必须确定`evt`是`HttpResponse`的一个实例

在我的例子中，我已经设置我的 API 在响应中返回一个包含消息和标题的 success 属性。这些是我想在吐司上展示的数据。也就是说，在这一点上，我们将检查这个响应是否有一个包含这个成功属性的主体。然后我们将显示包含三个参数的成功祝酒词:消息、标题、附加选项。(您可以在此查看所有可用选项[)](https://github.com/scttcper/ngx-toastr)

```
tap(evt => {
                if (**evt instanceof HttpResponse**) {
                    if(**evt.body** && **evt.body.success**)
                        this.toasterService.success(evt.body.success.message, evt.body.success.title, { positionClass: 'toast-bottom-center' });
                }
            }),
```

我还设置了我的 API，以便在发生错误时在响应中返回一个错误属性，该属性还包含一条消息和一个标题。如前所述，我们将检查`err`是否是`HttpErrorResponse`的实例

正如你在下面看到的，我没有检查这个属性是否存在，但是我使用了`try/catch`表达式来显示自定义错误或者一个普通错误。

```
catchError((err: any) => {
 **if(err instanceof HttpErrorResponse) {**
 **try {**
                        this.toasterService.error(err.error.message, err.error.title, { positionClass: 'toast-bottom-center' });
                    **} catch(e) {**
                        this.toasterService.error('An error occurred', '', { positionClass: 'toast-bottom-center' });
 **}**
 **}**
                return of(err);
            }));
```

拦截器的另一个用途是记录错误，而不是显示错误。或者两者都做。取决于项目。

把它们放在一起，这就是我们的最后一节课。将其作为`http.interceptor.ts`保存在您的`app`文件夹中。

```
import { HttpInterceptor, HttpHandler, HttpRequest, HttpEvent, HttpResponse, HttpErrorResponse }   from '@angular/common/http';
import { Injectable } from "@angular/core"
import { Observable, of } from "rxjs";
import { tap, catchError } from "rxjs/operators";
import { ToastrService } from 'ngx-toastr';[@Injectable](http://twitter.com/Injectable)()
export class AppHttpInterceptor implements HttpInterceptor {
    constructor(public toasterService: ToastrService) {}intercept(
        req: HttpRequest<any>,
        next: HttpHandler
      ): Observable<HttpEvent<any>> {

        return next.handle(req).pipe(
            tap(evt => {
                if (evt instanceof HttpResponse) {
                    if(evt.body && evt.body.success)
                        this.toasterService.success(evt.body.success.message, evt.body.success.title, { positionClass: 'toast-bottom-center' });
                }
            }),
            catchError((err: any) => {
                if(err instanceof HttpErrorResponse) {
                    try {
                        this.toasterService.error(err.error.message, err.error.title, { positionClass: 'toast-bottom-center' });
                    } catch(e) {
                        this.toasterService.error('An error occurred', '', { positionClass: 'toast-bottom-center' });
                    }
                    //log error 
                }
                return of(err);
            }));

      }

}
```

**提供拦截器**

这是极其简单的！打开您的`app.module.ts`文件，转到`providers` 部分并在数组中添加以下代码。

```
{provide: **HTTP_INTERCEPTORS**, useClass: **AppHttpInterceptor**, multi: true}
```

您还必须导入`HTTP_INTERCEPTORS`和您的`AppHttpInterceptor`

```
import { ..., **HTTP_INTERCEPTORS** } from '@angular/common/http';import { **AppHttpInterceptor** } from './http.interceptor';
```

就是这样！感谢您的阅读！
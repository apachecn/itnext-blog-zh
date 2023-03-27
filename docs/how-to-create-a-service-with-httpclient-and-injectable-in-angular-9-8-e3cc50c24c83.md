# 如何在 Angular 9/8 中用 HttpClient 和 Injectable 创建服务

> 原文：<https://itnext.io/how-to-create-a-service-with-httpclient-and-injectable-in-angular-9-8-e3cc50c24c83?source=collection_archive---------3----------------------->

在本教程中，我们将学习如何使用 HttpClient 构建一个服务，将您的 Angular 应用程序连接到 REST API 服务器。

我们将看到如何实现一个角度服务来封装处理从 REST API 服务器获取数据的代码。

![](img/a58818f87b50623c93587da4330618e2.png)

角度服务只是一个类型脚本类，您可以使用角度依赖注入器将它注入到其他服务或组件中。

# 先决条件

你需要有节点，NPM 和 Angular CLI 和初始化一个 Angular 项目。

# 步骤 1 —导入 HttpClient 模块

打开`src/app/app.module.ts`文件，更新如下:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HttpClientModule } from '@angular/common/http';@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

# 步骤 2-创建角度服务

打开一个新终端并运行以下命令:

```
$ ng generate service backend
```

# 步骤 3 —导入和注入 HttpClient

打开`src/app/backend.service.ts`文件，然后导入并注入`HttpClient`，如下:

```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';@Injectable({
  providedIn: 'root'
})
export class BackendService { constructor(private httpClient: HttpClient) { }
}
```

我们导入`HttpClient`并通过服务构造器注入它。

# 步骤 4 —定义发送 HTTP 请求的方法

接下来，定义一个`get()`方法，向服务器发送获取数据的请求:

```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';@Injectable({
  providedIn: 'root'
})
export class BackendService { constructor(private httpClient: HttpClient) { } public get(){
    return this.httpClient.get("http://server.com/endpoint");
  }
}
```

我们调用`HttpClient`的`get()`方法向 REST API 服务器发送 GET 请求。

# 步骤 5-创建角度组件

回到您的终端，运行下面的命令来生成一个名为 home 的组件:

```
$ ng generate component home
```

# 步骤 6 —注入角度服务

转到`src/app/home/home.component.ts`文件，导入并注入后端服务，如下所示:

```
import { Component, OnInit } from '@angular/core';
import { BackendService } from '../backend.service';@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit { data= []; constructor(private backendService: BackendService) { }}
```

# 步骤 7 —调用服务方法

最后，我们可以调用服务的`get()`方法来发送 GET 请求，如下所示:

```
ngOnInit() { this.backendService.get().subscribe((ret: any[])=>{
      console.log(ret);
      this.data = ret;
    })  
  }
```

# 步骤 8 —用`ngFor`显示数据

接下来，打开`src/app/home/home.component.html`文件，并按如下方式更新它:

```
<div>
<ul>
    <li *ngFor="let item of data">
        {{item.name}}
    </li>
</ul>           
</div>
```

这里，我们假设我们的数据元素有一个`name`属性。

查看我们的其他[角度教程](https://www.techiediaries.com/angular/)。

您可以通过作者的个人网站、 [Twitter](https://twitter.com/ahmedbouchefra) 、 [LinkedIn](https://www.linkedin.com/in/mr-ahmed/) 和 [Github](https://github.com/techiediaries) 联系或关注作者。

# 结论

在这篇 how-to 帖子中，我们看到了如何在 Angular 中创建一个服务，并使用`HttpClient`向 REST API 后端发送 HTTP 请求，最后使用`ngFor`指令显示从 API 返回的数据。
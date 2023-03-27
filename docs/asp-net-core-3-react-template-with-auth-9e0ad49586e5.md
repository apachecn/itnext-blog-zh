# ASP.NET 核心 3:使用授权对模板做出反应

> 原文：<https://itnext.io/asp-net-core-3-react-template-with-auth-9e0ad49586e5?source=collection_archive---------4----------------------->

[ASP.NET 核心](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-core-3-0-preview-3/)预告 3 于 3 月 6 日发布。此版本添加了在使用微软提供的模板创建 Angular 或 React 应用程序时包含 auth 的选项。我无法表达这个功能让我有多开心。作为一个没有深入研究过 auth 的人，为一个新的应用程序提供一个好的起点是非常有帮助的。

![](img/f1157a16bc9d550aec1db62888ea13c0.png)

# 装置

要获得模板的更新版本，请安装。NET Core 3 预告。你可以在这里找到安装者。

# 项目创建

使用。NET CLI 在命令提示符下，在要创建项目的目录中运行以下命令。

```
dotnet new react --auth Individual
```

项目构建完成后，您应该能够使用下面的命令来运行应用程序。

```
dotnet run
```

# 预览版 3 的问题

我做了上面的操作，导致了下面的错误。

> *微软。AspNetCore.SpaServices:信息:*
> 
> *编译失败。*
> 
> 信息:微软。AspNetCore.SpaServices[0]
> 
> *。/src/components/API-authorization/API authorization constants . js*

似乎 Preview 3 附带的 React 模板存在一些问题。要修复上述错误，请打开 client app/src/components/API-authorization 目录中的**API authorization constants . js**文件，并进行以下更改。

```
Before: ApiAuthorizationPrefix = prefix, After: ApiAuthorizationPrefix: prefix,
```

修复后，您将看到以下错误。

> *。/src/components/API-authorization/API authorization routes . js
> 找不到模块:无法解析。/components/API-authorization/log in ' in ' \ client app \ src \ components \ API-authorization '*

这个修复有点复杂。我在微软提供的[已知问题页面](https://github.com/dotnet/core/blob/master/release-notes/3.0/preview/3.0.0-preview-known-issues.md)中找到了变通方法。

首先，删除**API authorizationroutes . js**文件，该文件与上一个修复程序位于同一目录。然后用下面的内容替换 ClientApp/src 目录中的 **App.js** 的内容。

```
import React, { Component } from 'react'; import { Route } from 'react-router'; import { Layout } from './components/Layout'; import { Home } from './components/Home'; import { FetchData } from './components/FetchData'; import { Counter } from './components/Counter'; import { Login } from './components/api-authorization/Login' import { Logout } from './components/api-authorization/Logout' import AuthorizeRoute from './components/api-authorization/AuthorizeRoute'; import { ApplicationPaths, LoginActions, LogoutActions } from './components/api-authorization/ApiAuthorizationConstants'; export default class App extends Component { static displayName = App.name; render () { return ( <Layout> <Route exact path='/' component={Home} /> <Route path='/counter' component={Counter} /> <AuthorizeRoute path='/fetch-data' component={FetchData} /> <Route path={ApplicationPaths.Login} render={() => loginAction(LoginActions.Login)} /> <Route path={ApplicationPaths.LoginFailed} render={() => loginAction(LoginActions.LoginFailed)} /> <Route path={ApplicationPaths.LoginCallback} render={() => loginAction(LoginActions.LoginCallback)} /> <Route path={ApplicationPaths.Profile} render={() => loginAction(LoginActions.Profile)} /> <Route path={ApplicationPaths.Register} render={() => loginAction(LoginActions.Register)} /> <Route path={ApplicationPaths.LogOut} render={() => logoutAction(LogoutActions.Logout)} /> <Route path={ApplicationPaths.LogOutCallback} render={() => logoutAction(LogoutActions.LogoutCallback)} /> <Route path={ApplicationPaths.LoggedOut} render={() => logoutAction(LogoutActions.LoggedOut)} /> </Layout> ); } } function loginAction(name){ return (<Login action={name}></Login>); } function logoutAction(name) { return (<Logout action={name}></Logout>); }
```

通过以上修复，网站将加载，您应该会看到如下内容。

当你尝试注册时，你会得到以下错误。

> *MissingMethodException:找不到方法:“Microsoft。EntityFrameworkCore . metadata . builders . index builder Microsoft。EntityFrameworkCore . metadata . builders . entitytypebuilder ` 1。HasIndex(系统。linq . expressions . expression ` 1<系统。Func`2 <！0，系统。>对象>)’。*
> 
> *身份服务器 4。entity framework . extensions . modelbuilderextensions+<>c _ _ display class 2 _ 0。<ConfigurePersistedGrantContext>b _ _ 0(EntityTypeBuilder<PersistedGrant>grant)*

再次已知问题页面来拯救。打开你的 **csproj** 文件，替换下面的包引用。

```
Before: <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="3.0.0-preview3-19153-02" /> <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="3.0.0-preview3.19153.1" /> <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="3.0.0-preview3.19153.1" /> After: <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="3.0.0-preview-18579-0056" /> <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="3.0.0-preview.19080.1" /> <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="3.0.0-preview.19080.1" /> <PackageReference Include="Microsoft.EntityFrameworkCore.Relational" Version="3.0.0-preview.19080.1" /> <PackageReference Include="Microsoft.EntityFrameworkCore" Version="3.0.0-preview.19080.1" />
```

并添加下面这段 XML。

```
<PropertyGroup> <NoWarn>$(NoWarn);NU1605</NoWarn> </PropertyGroup>
```

# 包扎

经过上述调整后，一切都正常了。我相信到预览版 4 时，体验会更好。尽管我遇到了一些问题，但让一个基本的新项目进行下去，我还是非常兴奋。谢谢你，ASP.NET 团队，在微软给这些模板添加认证将会非常有帮助。另外，请注意，Angular template 还获得了一个 auth 选项(目前看来这是一个更流畅的体验)。

*原载于 2019 年 3 月 24 日*[*elanderson.net*](https://elanderson.net/2019/03/asp-net-core-3-react-template-with-auth/)*。*
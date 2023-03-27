# Blazor 认证:隐藏导航菜单项

> 原文：<https://itnext.io/blazor-authentication-hide-a-nav-menu-item-1deba40685b9?source=collection_archive---------8----------------------->

在上周的文章[服务器端 Blazor with Authentication](https://elanderson.net/2019/08/asp-net-core-server-side-blazor-with-authentication/) 中，我们介绍了如何创建一个带身份验证的服务器端 Blazor 应用程序，然后使用属性禁止用户在未登录的情况下查看获取数据页面。

虽然 authorize 属性不允许用户查看页面的内容，但它仍然允许用户访问他们无权访问的页面的 nav 菜单项。这将是一个快速的帖子，展示如何使用 AuthorizedView 组件来隐藏用户应该登录才能看到的任何内容(或处于特定角色)。

## 隐藏导航菜单项

在 **Pages/Shared** 目录中打开 **NavMenu.razor** 文件，它是定义导航菜单的文件。下面的代码是呈现 Fetch data 菜单项的代码，如果用户没有登录，这是我们想要隐藏的部分。

```
<li class="nav-item px-3">
    <NavLink class="nav-link" href="fetchdata">
        <span class="oi oi-list-rich" aria-hidden="true"></span> Fetch data
    </NavLink>
</li>
```

为了隐藏菜单项，我们将列表项包装在 **AuthorizeView** 组件中。

```
<AuthorizeView>
    <li class="nav-item px-3">
        <NavLink class="nav-link" href="fetchdata">
            <span class="oi oi-list-rich" aria-hidden="true"></span> Fetch data
        </NavLink>
    </li>
</AuthorizeView>
```

请注意，您仍然应该在需要授权的页面上使用 Authorize 属性，而不是依赖于隐藏的菜单项来阻止用户找到页面。

## 包扎

虽然 Authorize 属性仍然非常有用，但我确信 AuthorizeView 将在 Blazor 应用程序中得到大量使用。AuthrozieView 的优点是不局限于页面组件。

另外，请注意 AuthorizeView 也支持角色和策略。确保并查看官方[授权查看组件文档](https://docs.microsoft.com/en-us/aspnet/core/security/blazor/?view=aspnetcore-3.0&tabs=visual-studio#authorizeview-component)以了解更多详情。如果感兴趣，组件的[代码在 GitHub](https://github.com/aspnet/AspNetCore/blob/master/src/Components/Components/src/Auth/AuthorizeView.cs) 上。

*最初发表于* [*埃里克·安德森*](https://elanderson.net/2019/08/blazor-authentication-hide-a-nav-menu-item/) *。*
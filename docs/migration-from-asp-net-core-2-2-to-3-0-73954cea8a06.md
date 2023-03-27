# 从 ASP.NET 核心 2.2 迁移到 3.0

> 原文：<https://itnext.io/migration-from-asp-net-core-2-2-to-3-0-73954cea8a06?source=collection_archive---------8----------------------->

9 月 23 日[。NET Core 3.0](https://devblogs.microsoft.com/dotnet/announcing-net-core-3-0/) 发布包括【ASP.NET】Core 3.0 和[实体框架 Core 3.0](https://devblogs.microsoft.com/dotnet/announcing-ef-core-3-0-and-ef-6-3-general-availability/) 。这篇文章将采用[ASP.NET 基础系列](https://elanderson.net/category/asp-net-core/asp-net-core-basics/)中使用的联系人项目，并将其从。NET Core 2.2 到。网芯 3.0。本次迁移使用的大部分信息来自微软文档，该文档将涵盖比本文更多的场景。

之前的代码任何改动都可以在这个 [GitHub repo](https://github.com/elanderson/ASP.NET-Core-Basics/tree/134c7fbed806a156aaee92aab90ccd05191c8ff3) 中找到。提醒一下，联系人项目是这篇文章中唯一更新的项目，回购中的项目将暂时保留在 ASP.NET 核心 2.2 中。

## 装置

如果您是 Visual Studio 用户，您可以获得。NET Core 3.0 通过至少安装 [Visual Studio 16.3](https://visualstudio.microsoft.com/downloads/) 。对于不使用 Visual Studio 的用户，可以下载并安装。网芯 3.0 SDK 从[这里](https://dotnet.microsoft.com/download/dotnet-core/3.0)。与以前的版本一样，SDK 适用于 Windows、Linux 和 Mac。

安装完成后，您可以在命令提示符下运行以下命令来查看。你已经安装的 NET Core SDK。

```
dotnet --list-sdks
```

您应该会看到列出的 **3.0.100** 。如果你像我一样，你可能还会看到一些可以卸载的 SDK 预览版。

## 项目文件更改

右键单击项目并选择**编辑项目名称. csproj** 。

![](img/d3212671de441e02435e565a8b5ef875.png)

将 **TargetFramework** 改为 **netcoreapp3.0** 。

```
Before: 
<TargetFramework>netcoreapp2.2</TargetFramework> 

After: 
<TargetFramework>netcoreapp3.0</TargetFramework>
```

包装部分有很多变化。微软。AspNetCore.App 现在已经不存在了，是。NET 核心，而不需要特定的引用。另一件要注意的事情是，实体框架核心不再是“在盒子里”,所以你会看到增加了许多参考，使实体框架核心可用。

```
Before:
<PackageReference Include="Microsoft.AspNetCore.App" />
<PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.2.0" PrivateAssets="All" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="4.0.1" />

After:
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="3.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore" Version="3.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="3.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Identity.UI" Version="3.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="3.0.0" />
<PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="3.0.0" PrivateAssets="All" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="5.0.0-rc4" />
```

最后要注意的是 Swashbuckle 没有为。NET Core 3，所以你必须确保你至少使用版本 5 rc2。

以下是我完整的项目文件供参考。

```
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
    <UserSecretsId>aspnet-Contacts-cd2c7b27-e79c-43c7-b3ef-1ecb04374b70</UserSecretsId>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="3.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore" Version="3.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="3.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Identity.UI" Version="3.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="3.0.0" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="3.0.0" PrivateAssets="All" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="5.0.0-rc4" />
  </ItemGroup>

</Project>
```

## 程序变更

在 **Program.cs** 中，主机的构造方式发生了一些变化。这个版本可能有用，也可能没用，但是我创建了一个新的应用程序，并把它取出来，以确保我使用的是当前的设置。

```
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

namespace Contacts
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                                          {
                                              webBuilder.UseStartup<Startup>();
                                          });
    }
}
```

## 启动更改

在 **Startup.cs** 中，我们有相当多的变化要做。只要您没有在构造函数中做任何定制，您就可以用下面的代码替换它。

```
public Startup(IConfiguration configuration)
{
    Configuration = configuration;
}
```

接下来，他们输入从 **IConfigurationRoot** 更改为 **IConfiguration** 的配置属性。

```
Before:
public IConfigurationRoot Configuration { get; }

After:
public IConfiguration Configuration { get; }
```

转到 **ConfigureServices** 函数，需要做一些更改。第一个是更新到 Swagger 包的新版本的结果，其中的 **Info** 类已经被替换为 **OpenApiInfo** 。

```
Before:
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new Info { Title = "Contacts API", Version = "v1"});
});

After:
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1",  new OpenApiInfo { Title = "Contacts API", Version = "v1" })
});
```

接下来，我们将从使用 **UserMvc** 转移到新的**AddControllersWithViews**，这是一种新的更有针对性的方法，可以添加您需要的框架。

```
Before:
services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

After:
services.AddControllersWithViews();
```

现在，在**配置**功能中，需要更新功能签名并删除日志工厂位。如果你确实需要[配置日志](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-3.0)，那应该作为 **HostBuilder** 的一部分来处理。

```
Before:
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();

After:
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
```

对于下一组更改，我将只显示结果，而不是之前的结果。如果您想要使用新的端点路由功能，则 **UseCors** 可能适用也可能不适用，但添加 **UserRouting** 并用 **UserEndpoint** 替换 **UseMvc** 将适用。

```
app.UseStaticFiles();
app.UseRouting();

app.UseCors(builder =>
            {
                builder.AllowAnyHeader();
                builder.AllowAnyMethod();
                builder.AllowAnyOrigin();
            }
           );

app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
                 {
                     endpoints.MapControllerRoute(
                                                  name: "default",
                                                  pattern: "{controller=Home}/{action=Index}/{id?}");
                     endpoints.MapRazorPages();
                 });
```

## 其他杂项变化

我做的另一个改变是删除了一些与登录相关的 cshtml 文件中的`@using Microsoft.AspNetCore.Http.Authentication`。

## 包扎

从 2.2 到 3.0 的迁移比从 2.1 到 2.2 的迁移要复杂一些，但这并不奇怪这个版本中的所有变化。记得查看[官方迁移指南](https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-3.0)了解更多详情。

上述变更后的联系人项目代码可以在 [GitHub](https://github.com/elanderson/ASP.NET-Core-Basics/tree/1366a31de3604f5146f9fbab6b8768a45b021cd4) 上找到。

*原载于*[](https://elanderson.net/2019/10/migration-from-asp-net-core-2-2-to-3-0/)**。**
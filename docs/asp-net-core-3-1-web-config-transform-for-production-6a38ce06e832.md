# 面向生产的 ASP.NET 核心 3.1 Web.config 转型

> 原文：<https://itnext.io/asp-net-core-3-1-web-config-transform-for-production-6a38ce06e832?source=collection_archive---------4----------------------->

我最近将一个应用程序从 ASP.NET Core 2.2 升级到了 3.1，同时这个应用程序被转移到了一个新的服务器上。旁注，如果可以的话，不要同时做两个大的改变，因为这样会让追踪问题的原因变得更加困难。有问题的应用程序最初是使用 ASP.NET 核心 React 模板创建的。

![](img/ab72ccf3c26a336eefbf6954c4614ef1.png)

## 错误

经过上面的更改后，网站开始返回以下错误。

> HTTP 错误 500.0 — ANCM 进程内处理程序加载失败

为了诊断这个问题，我通过将 **stdoutLogEnabled** 改为 true 来启用标准输出日志。下面是启用了日志记录的应用程序的 web 配置中的一行。

```
<aspNetCore processPath="dotnet" arguments=".\YourApplication.dll" stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout">
```

## 原因

日志显示应用程序试图使用 React 开发服务器启动应用程序的 React 部分，React 开发服务器使用 npm。下面是来自**启动**类的**配置**函数的相关代码。

```
app.UseSpa(spa =>
           {
               spa.Options.SourcePath = "ClientApp";

               if (env.IsDevelopment())
               {
                   spa.UseReactDevelopmentServer(npmScript: "start");
               }
           });
```

从突出显示的区域可以看出，Reac 开发服务器应该只在环境设置为 development 时使用，果然，web.config 具有 ASPNETCORE_ENVIRONMENT 的环境变量，并且值为 Development，如下例所示。

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments="YourApplication.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="InProcess">
        <environmentVariables>
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

## 修复

我的本能反应是找到一种方法，当应用程序以发布模式发布时，改变该变量的值。这帮助我找到了微软的文档。

对于这个修复，我使用了基于构建配置转换。为此，我添加了一个名为 **web 的新文件。发布.配置**。有了这个新文件，文件中的转换将在发布构建运行时执行。下面是我用来将 ASPNETCORE_ENVIRONMENT 设置为生产环境的转换。

```
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <location>
    <system.webServer>
      <aspNetCore>
        <environmentVariables>
          <environmentVariable xdt:Transform="Replace" xdt:Locator="Match(name)" name="ASPNETCORE_ENVIRONMENT" value="Production" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

重要的是，转换的结构要与实际 web.config 中的结构相匹配，否则转换将无法找到需要转换的元素，在本例中，我们在 configuration/location/system . web server/aspNetCore/environment variables 下寻找元素。

在我们的例子中，我们告诉转换我们想要替换元素(xdt:Transform="Replace ")，该元素将名称(xdt:Locator = " Match(name)")ASPNETCORE _ ENVIRONMENT 与值 Production 相匹配。

## 包扎

这只是 web.config 转换的一个小例子。[官方文档](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/transform-webconfig)给了我一个关于转换能做什么的总体高层次的概念，但是对于我需要做的事情不是很有帮助。如果文档没有涵盖您的用例，请准备好进行大量的搜索。

附注在我上面提到的应用程序中，结果是。NET Core 2.1 不包含 ASPNETCORE_ENVIRONMENT 的值，这就是为什么它以前不是一个问题。

*原载于*[](https://elanderson.net/2020/02/asp-net-core-3-1-web-config-transform-for-production/)**。**
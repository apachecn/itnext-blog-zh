# 使用 ngX-Rocket Generator 创建企业角度应用程序

> 原文：<https://itnext.io/creating-enterprise-angular-apps-with-the-ngx-rocket-generator-871ac76ace87?source=collection_archive---------1----------------------->

![](img/b3543594fe6a000a06c979e991bf4034.png)

[https://www.npmjs.com/package/generator-ngx-rocket](https://www.npmjs.com/package/generator-ngx-rocket)

# 介绍

在本文中，我们将深入研究一种新的角度发生器，称为 ngX-Rocket。ngX-Rocket 是一个 Angular 5+企业项目生成器，基于来自社区的最佳实践的 [Angular-CLI](https://github.com/angular/angular-cli) 。它允许开发人员轻松地开发出开箱即用的具有大量功能的应用程序。它的核心特性包括 PWA(渐进式网络应用)和 Cordova-support，一个可扩展的入门模板，编码指南等等。

> 4.2 版是撰写本文时的最新版本。你可以看看[这篇](https://medium.com/ngx-rocket/version-4-0-0-of-ngx-rocket-now-available-fa0c5d090cc1)中篇的变化。我也会建议跟随 ngX-Rocket 的 github-repo。了解进一步的变化和改进。

# 与裸露应用相比的优势`ngular CLI`

你们大多数人可能已经熟悉 Angular CLI 了。像 ngX-Rocket 一样，Angular CLI 是一个命令行界面，用于创建快速启动 Angular 项目和自动化您的开发工作流。那么你为什么会选择 ngX-Rocket 呢？

项目[的 github 页面强调了为什么选择这个生成器而不是 Angular CLI 可能是个好主意的一些原因:](https://github.com/ngx-rocket/generator-ngx-rocket)

> **一个完整的入门模板**:为可伸缩性定制的示例应用程序结构，包含企业项目中所需的每个常见事物的示例和样板代码，例如单元测试、路由、身份验证、HTTPS 服务扩展、带有动态语言更改和自动用户语言检测的 i18n 支持。
> 
> **改进的工具** : SCSS & HTML 林挺，更严格的 TSLint 规则，用于文档的基于 markdown 的本地 wiki 服务器，自动化本地化字符串提取，公司代理支持， [Cordova](https://cordova.apache.org/) 集成。
> 
> **广泛的基础文档** : [打字稿/SCSS/HTML 的编码指南](https://github.com/ngx-rocket/generator-ngx-rocket#coding-guides)，Angular onboarding 指南，企业代理和其他工具[配置和使用](https://github.com/ngx-rocket/generator-ngx-rocket#other-documentation)。
> 
> **现成的 UI 组件**:关注你的应用，而不是堆栈！[在](https://github.com/ngx-rocket/generator-ngx-rocket#libraries) [Bootstrap 4](https://getbootstrap.com/) 、 [Ionic](http://ionicframework.com/) 或 [Angular Material](https://material.angular.io/) 之间选择基于 UI 的界面，该界面具有漂亮的外观和反应灵敏的启动模板。
> 
> **移动应用支持**:选择使用相同代码库的 web 应用、移动应用(使用 [Cordova](https://cordova.apache.org/) )或两者。
> 
> **API 代理示例设置**:使用任何远程服务器更快地开发和调试。
> 
> **生成器输出定制**:借助提供的[附加支持](https://github.com/ngx-rocket/generator-ngx-rocket/tree/master/generators/addon)，通过插入符合您需求的附加组件，例如您的企业主题、SSO 认证、服务集成，更快地启动多个项目。

# 创建新项目

## 先决条件

在使用 ngX -Rocket CLI 之前，您需要在系统上安装 Node.js 6.9.0 和 npm 3.0.0 或更高版本。你可以在 Node 的官方网站上为你的操作系统下载最新版本的 Node.js。

如果已经安装了 Node.js 和 npm，请运行以下命令来验证它们的版本:

```
$ node -v # => displays node version
$ npm -v # => displays npm version
```

一旦安装了 Node.js，就可以使用`npm`命令来安装[类型脚本](http://www.typescriptlang.org/):

```
$ npm install -g typescript
```

从技术上讲，您并不需要 TypeScript，但是 Angular 团队强烈推荐您使用它，所以我建议您安装它，以便尽可能舒适地使用 Angular。

现在您已经安装了 Node.js 和 TypeScript，您可以通过运行以下命令来安装 ngX-Rocket CLI:

```
$ npm i -g generator-ngx-rocket
```

这将在您的系统上全局安装 ngX-Rocket 生成器。您可以使用以下命令来验证安装:

```
$ ngx version # => displays ngx-rocket version
```

## 创建应用程序

我们需要做的第一件事是为我们的新项目创建一个文件夹，因为生成器在默认情况下不会这样做:

```
$ mkdir ngx-starter
```

在新创建的文件夹中导航:

```
$ cd ngx-starter
```

并初始化一个新的 ngx 项目:

```
$ ngx new
```

初始化新项目时，系统会提示您以下选项:

```
$ ngx new$ What's the name of your app?  (ngx app)$ What kind of app do you want to create? 
> (*) Web app
> ( ) Mobile app(using Cordova)$ Do you want a progressive web app? (with manifest and service worker) (Y/n)$ Which UI framework do you want?
> Bootstrap (more website-oriented)
  Angular Material (more website-oriented)
  Ionic (more mobile-oriented)$ Do you want authentication? (Y/n)$ Do you want lazy loading? (y/N)$ Do you want analytics support (with Angularatstics2)? (y/N)
```

这将在你选择的目录中创建一个新的 ngX-Rocket 应用程序。CLI 将下载所有必需的 NPM 软件包并进行设置。

您现在可以运行该应用程序:

> 注意！你需要使用`npm start`而不是`ng serve`来运行应用程序，因为它默认不使用后端代理配置。

```
$ npm start
```

让我们前往`localhost:4200`以确保到目前为止一切都按预期进行。

## 应用程序结构

```
dist/                      => app production build
docs/                      => project docs and coding guides
e2e/                       => end-to-end tests
src/                       => project source code
|- app/                    => app components
|  |- core/                => core module (singleton services and
|  |                          single-use components)
|  |- shared/              => shared module  (common components, 
|  |                          directives and pipes)
|  |- app.component.*       => app root component (shell)
|  |- app.module.ts         => app root module definition
|  |- app-routing.module.ts => app routes
|  +- ...                   => additional modules and components
|- assets/                  => app assets (images, fonts, sounds...)
|- environments/            => values for various build environments
|- theme/                   => app global scss variables and theme
|- translations/            => translation files
|- index.html               => html entry point
|- main.scss                => global style entry point
|- main.ts                  => app entry point
|- polyfills.ts             => polyfills needed by Angular
+- test.ts                  => unit tests entry point
reports/                    => test and coverage reports
proxy.conf.js               => backend proxy configuration
```

## 文档服务器

这些文档全面概述了 Angular 领域的最佳实践。您可以通过运行`npm run docs`命令在单独的服务器中显示文档。您的浏览器将在`http://localhost:4040`打开并显示项目文档。

## HttpClient 扩展

在发出请求之前，您可以通过调用`cache()`来缓存任何 HTTP 请求:

```
this.httpClient
   .cache()
   .get('https://myapi.com');
```

## 代理服务器

为了简化开发，他们安装了一个后端代理，将 API 调用重定向到你想要的任何 URL 和端口。这允许您:

> 开发前端功能，无需在本地运行 API 后端
> 
> 使用没有 CORS 问题的本地开发服务器
> 
> 直接使用来自远程测试平台的数据调试前端代码

# 用户化

我喜欢这个 Angular starter 项目的一个原因是它很容易添加功能。以下是我喜欢做的一些定制的概述。

```
docs/                      
e2e/              
src/                  
|- app/      
|  |- core/
|  |  |- data                => mock services
|  |  |- json                => json files exported for mocks
|  |  |- services            => singleton services
|  |                 
|  |- shared/
|  |  |- directives          => directives used across the app
|  |  |- models              => models used across the app
|  |  |- pipes               => pipes used across the app
|  |  |- components          => components used across the app
|  |
|  |- modules                => collection of all the modules
|  |  |- components          => components which are shared across
|  |  |                         the module
|  |  |- pages               => pages the module consist of
|  |                  
|  |- app.component.*  
|  |- app.module.ts        
|  |- app-routing.module.ts
|  +- ...                 
|- assets/
|- environments/         
|- theme/                  
|- translations/      
|- index.html              
|- main.scss          
|- main.ts            
|- polyfills.ts          
+- test.ts        
reports/                  
proxy.conf.js 
```

# 结论

ngX-Rocket 项目是您开发过程的一个很好的起点。你会得到一大套工具，项目遵循 Angular 团队和社区推荐的最佳实践。该项目也很容易定制，这意味着添加新功能并不是什么大任务。`Happy Coding!`
# 使用角度布线处理角度布线(ngRoute)

> 原文：<https://itnext.io/handling-angularjs-routing-with-angular-route-ngroute-a8e6facaf3f9?source=collection_archive---------2----------------------->

![](img/cfcf9f1f3183ac9af67cb4de4583cc70.png)

我是从 Angular 2+开始的，但是我现在的公司有一些项目需要 AngularJS 1x。所以，我们到了！

在这篇文章中，我将向你展示如何使用 [Angular-Route (ngRoute)](https://docs.angularjs.org/api/ngRoute) 来处理 AngularJS 1x 路由。将会有另一个帖子，我将使用 [UI 路由器](https://github.com/angular-ui/ui-router)来处理 AngularJS 路由。

> 两者的不同之处在于 UI-Router 是一个第三方模块，使用不同的方法来管理路由，UI-Router 具有额外的功能，适合更复杂的项目。

如果你想了解更多他们之间的不同，请查看[这篇来自 stackoverflow](https://github.com/angular-ui/ui-router) 的帖子。

我在这篇文章中使用的 AngularJS 版本是 *1.7.2* ，如果你使用旧版本，它可能会有所不同。

在开始之前，你可以从 [Plunker](https://next.plnkr.co/plunk/UIbEA59H8WPw4YFtfzFM) 看一下这个项目。

 [## 角度路线-角度路线

### 带 n 路由的角路由

next.plnkr.co](https://next.plnkr.co/plunk/UIbEA59H8WPw4YFtfzFM) 

**启动 AngularJS 项目**

很容易建立一个 AngularJS 项目，你只需要在头部嵌入 angular 文件。

```
// index.html<html **ng-app="angularRouting"**>
...<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.7.2/angular.min.js"></script><script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.7.2/angular-route.min.js"></script><script src="app.js"></script>... // Navigation Menu<ul class="uk-nav uk-nav-default">
   <li class="uk-active"> <a href="#">Menu</a></li>
   <li><a **ng-href="#!">Home**</a></li>
   <li><a **ng-href="#!about"**>About</a></li>
   <li><a ng-href="#!params/1">Params</a></li>
   <li><a ng-href="#!auth">You shall not pass!</a></li>
</ul>// Content will go here
<main **ng-view**></main>
```

你的主要逻辑会在 app.js 里面。

```
var app = angular.module('angularRouting', ['**ngRoute**']);
```

**让我们开始路由工作**

我们需要添加 **$routeProvider** 来管理路由。下面的代码显示了主页和 about 页面的路由。您现在可以在浏览器中测试它。

**否则**会将所有其他路线重定向到主页。实际上，您将为此展示一个 404 模板。如果你想分离 HTML 模板，你可以用 **tempateUrl** 代替**模板**。

```
// app.jsapp.config(['$routeProvider', 
  function(**$routeProvider**){
    $routeProvider
      .when('/', {
        template: '<h1>This is home</h1>'
      })
      .when('/about', {
        template: '<h1>This is about</h1>'
      })
      .**otherwise**({
        redirectTo: '/'
      })
  }
]);
```

**从 URL 获取参数**

我创建了一个带参数的菜单。在您的应用程序中，该参数将通过 **{{ id }}** 动态填充。在这个例子中，我将使用数字 1 作为 params 路由的额外参数。

```
// index.html<ul class="uk-nav uk-nav-default">
   <li class="uk-active"> <a href="#">Menu</a></li>
   <li><a ng-href="#!">Home</a></li>
   <li><a ng-href="#!about">About</a></li>
   **<li><a ng-href="#!params/1">Params</a></li>**
   <li><a ng-href="#!auth">You shall not pass!</a></li>
</ul>
```

现在，我们需要在 app.js 中配置路由。我将使用 ParamsController 来获取参数并将其显示给模板。

```
// app.js app.config(['$routeProvider', 
  function($routeProvider, $routeParams){
    $routeProvider
      .when('/', {
        template: '<h1>This is home</h1>'
      })
      .when('/about', {
        template: '<h1>This is about</h1>'
      })
 **.when('/params/:id', {
        template: '<h1>Param: {{ paramValue }}</h1>',
        controller: 'ParamsController'
      })**
      .otherwise({
        redirectTo: '/'
      })
  }
]);app.controller('**ParamsController**', 
  function ParamsController($scope, **$routeParams**) {
    $scope.paramValue = **$routeParams**.id;
  }
);
```

**如何守护你的路线**

所以，有些页面是你不想给公众看的。用户必须登录才能查看它们。

```
// index.html<ul class="uk-nav uk-nav-default">
   <li class="uk-active"> <a href="#">Menu</a></li>
   <li><a ng-href="#!">Home</a></li>
   <li><a ng-href="#!about">About</a></li>
   <li><a ng-href="#!params/1">Params</a></li>
 **<li><a ng-href="#!auth">You shall not pass!</a></li>**
</ul>**{{ message }} //**show authentication message
<main ng-view></main>
```

然后，您将在 app.js 中添加路由和身份验证服务。如果用户没有查看该页面的权限，他们将被重定向到主页。

```
// app.js // Handle **$routeChangeError** 
app.run(['$rootScope', '$location', 
   function($rootScope, $location) {
     $rootScope.$on('**$routeChangeError**', function(event, next, previous, error) {
        if(error == '**AUTH_REQUIRED**') {
            $rootScope.message = 'Sorry, you must log in to access that page';
            $location.path('/');
        } //AUTH_REQUIRED
     }); // $routeChangeError
   }
]);app.config(['$routeProvider', 
  function($routeProvider, $routeParams){
    $routeProvider
      .when('/', {
        template: '<h1>This is home</h1>'
      })
      .when('/about', {
        template: '<h1>This is about</h1>'
      })
      .when('/params/:id', {
        template: '<h1>Param: {{ paramValue }}</h1>',
        controller: 'ParamsController'
      })
 **.when('/auth', {
        template: '<h1>Success</h1>',
        resolve: {
          currentAuth: function(AuthService) {
            return AuthService.authenticate();
          }
        }
      })**
      .otherwise({
        redirectTo: '/'
      })
  }
]);// Authentication Service 
app.factory('**AuthService**', function($q) {
  return {
    **authenticate**: function() {
      // Your authenication logic
      return $q.reject('**AUTH_REQUIRED**');
    }
  }
});
```

我假设用户没有权限查看页面，它将返回一个“AUTH_REQUIRED”错误。

从现在开始，你知道如何用 Angular-Route (ngRoute)处理 AngularJS 路由。我将很快更新路由器版本。

希望这有所帮助:)
# 错误:找不到模块“@angular-devkit/core”

> 原文：<https://itnext.io/error-cannot-find-module-angular-devkit-core-a2609e9f7d41?source=collection_archive---------5----------------------->

## 就在一分钟前，我还在为同样的问题纠结。我的项目是使用 angular-cli 的 1.6.0 版生成的。

![](img/3d0a3c16a3154518debfdef223a0b53b.png)

当我想启动我的 ng 服务器时，我遇到了以下错误:

```
ng s
```

error⬇︎

```
module.js:557
    throw err;
    ^
Error: Cannot find module '@angular-devkit/core'
    at Function.Module._resolveFilename (module.js:555:15)
    at Function.Module._load (module.js:482:25)
    at Module.require (module.js:604:17)
    at require (internal/module.js:11:18)
    at Object.<anonymous> (/workspace/observable-app/node_modules/@angular-devkit/schematics/src/tree/virtual.js:10:16)
    at Module._compile (module.js:660:30)
    at Object.Module._extensions..js (module.js:671:10)
    at Module.load (module.js:573:32)
    at tryModuleLoad (module.js:513:12)
    at Function.Module._load (module.js:505:3)
```

不要担心，这里有一个解决方案:

首先，您需要更改@angular/cli 版本。

编辑我的包

```
"@angular/cli": "1.6.0",
to
"@angular/cli": "^1.6.0",
```

然后

```
npm update
```

这很有效。

![](img/30bc983bbbae58cf72ec274dd73f681e.png)![](img/426d4c9b6c4fab7c222d6c690093d17b.png)

希望对你也有帮助。

享受编码…

![](img/5af42d24f58b7b0d097dc599bda1bb5d.png)![](img/39d030ec9a37d7cbc11138e846925b8b.png)

*最初发表于*[*【qiita.com】*](https://qiita.com/alokrawat050/items/71b39c2f1264229940a6)*。*
# 用 Sage 9 和 UIkit 开发 WordPress 主题

> 原文：<https://itnext.io/wordpress-theme-development-with-sage-9-and-uikit-10752e1d09d2?source=collection_archive---------4----------------------->

![](img/3be2152eb0a0c7cc7ad969f7cca9a841.png)

我作为 Wordpress 开发者已经工作了一段时间，我不得不说 [Sage](https://roots.io/sage/) 是一个很好的 WordPress 入门主题。入门主题包括 Bootstrap 4、Foundation…开箱即用，如果你愿意，你甚至可以集成其他前端框架。

现在，Sage 9 包括了[网络包](https://webpack.github.io/)！虽然是免费的，但这是多么划算啊！)

今天，我将向你展示如何将 [UIkit 框架](https://getuikit.com/)集成到 Sage 9 中。如果你想使用 Sage 8.53，请阅读这篇文章:[我最喜欢的自定义主题开发的 WordPress 启动器](https://medium.com/@dalenguyen/my-favourite-wordpress-starter-for-custom-theme-development-2636657afc58)

如果你想马上使用，我在 Github 上创建了一个 [WordPress Starter。您只需要克隆项目并开始工作:)](https://github.com/dalenguyen/wordpress-sage-9-uikit-starter)

**第一步**:你需要安装 [WordPress](https://wordpress.org/)

**第二步:**进入你的主题文件夹，开始[安装 Sage 主题](https://roots.io/sage/docs/theme-installation/)

```
composer create-project roots/sage your-theme-name
```

当它询问您想要使用哪个框架时，记得选择 **NONE** 。

第三步:安装 UIkit

```
yarn install uikit
```

**第四步:**添加 UIkit 样式表

```
// resources/assets/styles/main.scss// [@import](http://twitter.com/import) "~some-node-module";
[@import](http://twitter.com/import) "~uikit/dist/css/uikit.css";
```

**步骤 4:** 添加 UiKit 脚本和图标

```
// resources/assets/scripts/main.jsimport UIkit from 'uikit';
import Icons from 'uikit/dist/js/uikit-icons';// loads the Icon plugin
UIkit.use(Icons);
```

**第五步:**从现在开始，快乐 WordPress 开发！

```
yarn && yarn build 
```

不要忘记在你的项目中使用 git)

这里是样板文件的存储库:[https://github . com/dalen guyen/WordPress-sage-9-ui kit-starter](https://github.com/dalenguyen/wordpress-sage-9-uikit-starter)
# 如何将基础 6 SCSS 添加到 Sapper

> 原文：<https://itnext.io/how-to-add-foundation-6-scss-to-sapper-fca03a461109?source=collection_archive---------1----------------------->

![](img/a886698e5db91c1236237c1c1bd496d0.png)

工兵+ SCSS +基础 6😍

使用 Rollup 将 SCSS 直接包含到您的 Sapper 应用程序中

Sapper 是一个伟大的，有效的方式来建立您的网站与 SSR 版本的苗条。然而，添加像 Foundation 或 Bootstrap 这样的 SCSS 框架可能会有点痛苦，尤其是如果您不想只包含整个缩小的 CSS 文件，而是挑选您的组件。

一如既往，有志者事竟成，在 Sapper 中，这是以 [rollup-plugin-scss](https://github.com/thgh/rollup-plugin-scss) 的形式出现的。

## **通过 NPM 添加汇总插件 scss**

在您的 sapper repo 中，打开命令提示符/终端并安装 [rollup-plugin-scss](https://github.com/thgh/rollup-plugin-scss)

```
npm install --save-dev rollup-plugin-scss
```

## 添加基础 6

```
npm install --save-dev foundation-sites
```

我们只需要为 dev 保存 foundation，因为生产版本将读取编译的 css 文件，而不是节点模块。

## 创建您的 SCSS 文件

接下来，我们需要创建我们的 SCSS 文件，这将是 rollup 使用的入口点(我们将在稍后配置)，如果您愿意，您可以对结构进行更多的修改，但首先，让我们保持简单。

在你的`src`文件夹中，创建一个新的`scss`文件夹，并在其中创建一个新的`main.scss`文件。

然后，在你的`main.scss`中包括基础，以及你需要的所有比特:

```
// /src/main.scss// Import Foundation from node modules
@import 'node_modules/foundation-sites/scss/foundation.scss'; // Include any components from Foundation as normal
// Global styles
@include foundation-global-styles;
[@](http://twitter.com/include)include foundation-forms;
[@](http://twitter.com/include)include foundation-typography;// Grids (choose one)
[@](http://twitter.com/include)include foundation-xy-grid-classes;
// [@](http://twitter.com/include)include foundation-grid;
// [@](http://twitter.com/include)include foundation-flex-grid;// Generic components
[@](http://twitter.com/include)include foundation-button;
[@](http://twitter.com/include)include foundation-button-group;
[@](http://twitter.com/include)include foundation-close-button;
[@](http://twitter.com/include)include foundation-label;
[@](http://twitter.com/include)include foundation-progress-bar;
[@](http://twitter.com/include)include foundation-slider;
[@](http://twitter.com/include)include foundation-switch;
[@](http://twitter.com/include)include foundation-table;
// Basic components
[@](http://twitter.com/include)include foundation-badge;
[@](http://twitter.com/include)include foundation-breadcrumbs;
[@](http://twitter.com/include)include foundation-callout;
[@](http://twitter.com/include)include foundation-card;
[@](http://twitter.com/include)include foundation-dropdown;
[@](http://twitter.com/include)include foundation-pagination;
[@](http://twitter.com/include)include foundation-tooltip;// Containers
[@](http://twitter.com/include)include foundation-accordion;
[@](http://twitter.com/include)include foundation-media-object;
[@](http://twitter.com/include)include foundation-orbit;
[@](http://twitter.com/include)include foundation-responsive-embed;
[@](http://twitter.com/include)include foundation-tabs;
[@](http://twitter.com/include)include foundation-thumbnail;
// Menu-based containers
[@](http://twitter.com/include)include foundation-menu;
[@](http://twitter.com/include)include foundation-menu-icon;
[@](http://twitter.com/include)include foundation-accordion-menu;
[@](http://twitter.com/include)include foundation-drilldown-menu;
[@](http://twitter.com/include)include foundation-dropdown-menu;// Layout components
[@](http://twitter.com/include)include foundation-off-canvas;
[@](http://twitter.com/include)include foundation-reveal;
[@](http://twitter.com/include)include foundation-sticky;
[@](http://twitter.com/include)include foundation-title-bar;
[@](http://twitter.com/include)include foundation-top-bar;// Helpers
// [@](http://twitter.com/include)include foundation-float-classes;
[@](http://twitter.com/include)include foundation-flex-classes;
[@](http://twitter.com/include)include foundation-visibility-classes;
// [@](http://twitter.com/include)include foundation-prototype-classes;
```

## 通知汇总文件。

现在这已经排序了，我们需要告诉 roll-up 和 sapper 我们有一个新文件要包含，所以在你的`rollup.config.js`中，你在顶部添加一个对 [rollup-plugin-scss](https://github.com/thgh/rollup-plugin-scss) 的引用，然后在插件中包含你的 main.scss。

```
// Add this at the top with the rest of your imports
import scss from 'rollup-plugin-scss';//...// Add scss into the client plugins area, like this...export default {client: {input: config.client.input(),output: config.client.output(),plugins: [replace({'process.browser': true,'process.env.NODE_ENV': JSON.stringify(mode)}),scss({output: 'static/global.css',}),// ... file continues as normal ...
```

注意我们是如何指定输出目录的，这是 Sapper 的默认 css 文件。

如果您想保留这个文件中的 css，您需要在运行代码之前移动它(即创建一个新的 scss 文件，并将其导入 main)，否则当我们运行代码时它将被覆盖。

## 告诉萨帕发生了什么

最后，您现在需要告诉 Sapper 使用这个文件，这样您就可以在`client.js`中导入它，您的文件最终将如下所示:

```
import "./scss/main.scss";import * as sapper from '@sapper/app';sapper.start({target: document.querySelector('#sapper')});
```

## 运行 Dev 并查看结果

如果一切进展顺利，我们可以使用`npm run dev`来看看我们的基础风格的影响。

这种设置非常简单，而且是一种很好的方式来包含不会以苗条为前缀的全球风格。

干得好，你现在是一名工兵 SCSS 向导了👏👏👏。
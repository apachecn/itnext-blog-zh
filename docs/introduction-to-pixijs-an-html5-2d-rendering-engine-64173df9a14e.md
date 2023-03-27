# PixiJS 简介:一个 HTML5 2D 渲染引擎

> 原文：<https://itnext.io/introduction-to-pixijs-an-html5-2d-rendering-engine-64173df9a14e?source=collection_archive---------2----------------------->

JavaScript 变得越来越强大。有许多用 JavaScript 构建的游戏通常使用 2D 或 3D 库。PixiJS 是一个可以在所有设备上运行的 2D 库。这是一个 HTML5 2D 渲染引擎，支持 WebGL API，并在需要时退回到 canvas。

![](img/91c7524bf5673fd8f7ccf3608f50e828.png)

PixiJS

# 先决条件

通过下载预建的 [build](https://github.com/pixijs/pixi.js/wiki/FAQs#where-can-i-get-a-build) 来包含 PixiJS 库！也可以通过 [npm](https://docs.npmjs.com/getting-started/what-is-npm) 安装或者直接嵌入 CDN URL。

通过 NPM 安装:

```
npm install pixi.js
```

然后在脚本文件中:

```
import * as PIXI from 'pixi.js'
```

如果要嵌入内容交付网络(CDN) URL:

```
<script src=" https://cdnjs.cloudflare.com/ajax/libs/pixi.js/5.0.4/pixi.min.js"></script>
```

# 创建精灵

在这篇文章中，我们将创建平铺精灵。创建一个新的 PIXI 应用程序，如下所示:

```
const app = new PIXI.Application();
```

这将为您创建渲染器、跑马灯和根容器。

在下一步中，创建一个 canvas 元素并将其附加到 HTML 文档中。

```
document.body.appendChild(app.view);
```

这里， *app.view* 创建画布元素。让我们创建一个纹理，我们将在平铺精灵中使用。

```
const texture = PIXI.Texture.from('https://i1.wp.com/techshard.com/wp-content/uploads/2017/04/animation.jpg?ssl=1&resize=438%2C438');
```

平铺子画面是渲染平铺图像的一种快速方式。初始化平铺精灵对象，如下所示:

```
const tilingSprite = new PIXI.extras.TilingSprite(
    texture, 
    app.renderer.width,
    app.renderer.height
);
```

我们现在要将平铺精灵对象附加到渲染的根显示容器上。

```
app.stage.addChild(tilingSprite);
```

我们将利用 [Ticker](http://pixijs.download/release/docs/PIXI.Ticker_.html) 类，它为正在监听的对象运行一个更新循环。

```
let count = 0;app.ticker.add(() => {
    count += 0.0005; tilingSprite.tileScale.x = 2 + Math.sin(count);
    tilingSprite.tileScale.y = 2 + Math.cos(count); tilingSprite.tilePosition.x += 1;
    tilingSprite.tilePosition.y += 1;
});
```

这里， *tilingSprite.tileScale* 是缩放正在平铺的图像。*tilingsprite . tile position*是正在平铺的图像的偏移量。

就是这样！运行应用程序。

点击查看完整演示[。](https://codepen.io/swathisprasad/pen/zEmEBx)

PixiJS 演示

# 包扎

与 [PixiJS](https://github.com/pixijs/pixi.js) 一起工作既简单又有趣。它可以让您创建丰富的交互式图形、跨平台游戏和应用程序。它提供了很多很酷的功能。如果您还没有检查过 PixiJS，请检查它们的[文档](http://pixijs.download/release/docs/index.html)和[示例](https://pixijs.io/examples/#/demos-basic/container.js)。

*本文原载于【Techshard.com】[](https://techshard.com/2019/06/10/introduction-to-pixijs-an-html5-2d-rendering-engine/)**。***
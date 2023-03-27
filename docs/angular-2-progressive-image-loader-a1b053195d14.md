# 角度 2+渐进式图像加载器

> 原文：<https://itnext.io/angular-2-progressive-image-loader-a1b053195d14?source=collection_archive---------3----------------------->

你可能已经注意到，某些网站会先加载低质量的图像，当全分辨率图像准备就绪时，它会用全尺寸图像替换低质量图像。在这个媒介上，你可以注意到这个效果。低质量的图像是简单模糊的。

![](img/181fbb5a50e594f1903867fd89615637.png)

媒体的渐进图像加载示例

# 入门指南

在我的项目中，我使用 Amazon s3 存储图像，使用 graphics magick([http://www.graphicsmagick.org/](http://www.graphicsmagick.org/))调整图像大小。当用户上传照片时，我将全尺寸照片上传到一个“临时”桶，然后触发后端从临时桶下载文件，并使用 graphics magick 将图像分成小、中、大尺寸。

我上传到一个临时桶，然后在后端下载它的原因是 graphics magick 只能在后端运行，并且图像通常太大，如果不进行多部分 http POST 就无法通过 http，并且不需要进入我正在进行的项目的太多细节，从前端上传会更容易/更有效。你可以在这里看到我是如何做这个[的，但是要知道它可能不适合你的项目。](https://gist.github.com/lukasasorensen/fdf1b63a614e03fa94dc826d920ad5a1)

[https://gist . github . com/lukasasorensen/FD f1 b 63 a 614 e 03 fa 94 DC 826d 920 ad 5a 1](https://gist.github.com/lukasasorensen/fdf1b63a614e03fa94dc826d920ad5a1)

如果你不太理解承诺，那么不要担心。第 45 行接受 size 数组([50，200，500])，并以 size 作为参数为数组中的每个大小分配一个新的进程函数。因此，变量 actions 本质上变成:[process(50)，process(200)，process(500)]并且第 46 行获取该数组并使其成为一个可执行的承诺，该承诺在数组中的每个承诺被执行之前不会运行 then()。

上面代码中唯一真正重要的部分是第 21 行，它去掉了图像名称的文件扩展名，并替换为“_[size]。jpg " ie image.png = > image _ 50 . jpg，image_200.jpg，image_800.jpg。这是图像加载器获取正确尺寸所必需的。

最后，将图像位置添加到数据库中。

```
post: {
  id: 1,
  body: 'lorem ipsum',
  image: {
    url: {
      sm: s3BucketUrl/image_50.jpg,
      md: s3BucketUrl/image_200.jpg,
      lg: s3BucketUrl/image_800.jpg
    }
  }
}
```

# 图像加载器指令

[https://gist . github . com/lukasasorensen/5e 54474 b 8d 041 cc44 c 456562572 E6 BAC](https://gist.github.com/lukasasorensen/5e54474b8d041cc44c456562572e6bac)

如果你不明白角度指令是如何工作的，我建议你先阅读[。](https://angular.io/guide/attribute-directives)

让我们仔细分析一下这个指令。

第 18 行定义了一个图像源，如果它试图加载的图像失败。而下一行(19)告诉加载程序要加载多大尺寸的图像。接下来，我们有一个事件发射器来告诉父组件加载图像时出错。

以下是如何使用该指令的示例:

```
<img  
  src="{{profileImage?.url?.sm}}"   
  image-loader  
  [imgSize]="lg"  
  [fallback]="/assets/fallback-image.jpg"     
  (emitOnError)="handleImageError()" 
/>
```

# onLoad()

回到指令。第 35–41 行将监听器连接到 image onload 和 onerror 函数，以允许我们在这些事件发生时运行代码。并在我们加载完图像后给我们一些函数来停止监听器。

注意:不要在 img 标签的 html 模板中使用(onload)或(onerror)。这将使您的应用程序面临可能的 XSS 攻击！:'(

让我们看看第 63 行的自定义 onLoad 函数。首先，我们停止应用程序监听加载事件。第 65 行我们在每个“_”处拆分 src 字符串，并将其分配给一个新数组。然后，我们弹出数组中的最后一项，即“_50.jpg ”,因为这是较小的图像大小。并将数组重新连接在一起，放回被 split 函数删除的所有“_”。最后，我们运行 loadLargeImage()。

# loadLargeImage()

首先，在第 44 行我们添加了大图像后缀和扩展名。在本例中是“_800.jpg”。我们定义一个新的图像，并将其命名为 largeImage，给它一个源，这个源应该是我们创建的大图像的 url，然后等待它完成加载。

加载完成后，我们使用 Angular 的 Renderer2 将原生元素(指令所在的 img 元素)源替换为大图像源。瞧啊！有用！

# 一个错误()

如果图像因为任何原因失败。该指令将使用带有 onError 函数的回退映像 src 替换本机元素 src。

# 恩贡德斯特罗伊

当指令不再使用时，移除事件侦听器。

# 结论

渐进式图像加载器允许用户看到较低质量的图像，而您的网站在后台加载较大的图像，并处理图像错误，而不会显示难看的坏图像链接。

你可以进一步学习这个教程，用 html 画布或 graphics magick like Medium 模糊你的小尺寸图像。

感谢阅读！❤

——卢卡斯
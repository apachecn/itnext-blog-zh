# 如何在 Umbraco 9 上提供优化的图像

> 原文：<https://itnext.io/how-to-serve-optimised-images-on-umbraco-9-f86ec06b6344?source=collection_archive---------2----------------------->

![](img/7cd04ebd6623e372a811f3e9be0477bc.png)

关于在 Umbraco 9 上提供优化图像的教程

一个关于在最新版本的 Umbraco 上提供裁剪好的、搜索引擎友好的̶W̶e̶b̶P̶图片的快速教程

*注意:本教程最后一次更新是针对 RC 版本:
T3 9 . 0 . 0-RC(08/07/21)*

在 Umbraco 的引擎盖下有一套强大的媒体管理工具，允许你提供高性能的图片，这些图片是 SEO 友好的，裁剪得很好。

> WEBP 支持[还没有完全准备好](https://github.com/SixLabors/ImageSharp/pull/1552)六个劳动力。ImageSharp.Web，这是一个遗憾，因为它可以作为插件在 8 上的 imageprocessor.web 上获得。

然而，要达到那个阶段，你确实需要一点未记录的知识来让它工作。

## GetCropUrl()简介

输入这个方法，它会根据你的需要进行动态调整，目前有 5 种方法可以用来裁剪你的图片，分为两种主要方法，或者通过传递一个 **IPublishContent** 媒体项，或者一个 url 字符串。我们最感兴趣的是这个:

```
GetCropUrl(this IPublishedContent mediaItem, // Implied by the referring objectIImageUrlGenerator imageUrlGenerator,  // ImpliedIPublishedValueFallback publishedValueFallback,  // ImpliedIPublishedUrlProvider publishedUrlProvider,  // Impliedint? width = null,  // The width to crop to in pixelsint? height = null,  // The height to crop to in pixelsstring propertyAlias = "umbracoFile",  // Property alias of the property containing the Json data. The default is finestring cropAlias = null,  // The crop alias specified in umbraco, we take this implicitly from the IPublished Contentint? quality = null,  // Image compresion quality out of 100ImageCropMode? imageCropMode = null,  // can be left, or specified explicitly based on the modes in Sharps docsImageCropAnchor? imageCropAnchor = null,  // can be left, or specified explicitly based on the modes in Sharps docsbool preferFocalPoint = false,  // Use focal point, to generate an output image using the focal point instead of the predefined cropbool useCropDimensions = false,  // Use crop dimensions to have the output image sized according to the predefined crop sizes, this will override the width and height parameters. So we do not want thisbool cacheBuster = true,  // Add a serialized date of the last edit of the item to ensure client cache refresh when updated. We want thisstring furtherOptions = null,  // Specify extra features to amend the image with. This is where we specify our preferred image format, e.g.  ̶W̶e̶b̶P̶ImageCropRatioMode? ratioMode = null,  //  Use a dimension as a ratio. We'll ignore this as we are specifying the width and height, which will explicitly set the ratio.bool upScale = true // If the image is smaller than the dimensions, attempt to upscale it to fill the space. Ideally this wont be needed, but best to keep it so we dont get black borders or too small images);
```

它给了我们最大的灵活性来改变图像，并允许我们提供图像格式，这是其他变体所缺乏的。

虽然这看起来令人望而生畏，但它实际上对许多不需要的重载来说是多余的。对于我们正在使用的例子，我们可以只关注我们需要的元素。这为我们提供了以下信息:

```
@* /Views/Partials/ResponsiveImage.cshtml *@@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage@using Umbraco.Cms.Core.Media@using Umbraco.Cms.Core.Models.PublishedContent@using Umbraco.Cms.Core.Routing@{var publishedContent =  @ViewData["publishedContent"] as IPublishedContent ?? null;var lazyLoading =  @ViewData["lazyLoading"] ?? false;var imageWebP = publishedContent?.GetCropUrl(width: 600, height: 20, quality:80, furtherOptions: "&format=jpg") ?? "";}
```

然后，我们指定宽度、高度和质量，在进一步的选项中，我们设置了“̶W̶e̶b̶P̶'”格式，这是新网站的最佳格式。

## 用图片包起来

为了使我们的图像更具性能，我们可以指定一个源集，从高性能的 webp 开始，逐渐退化到其他格式。这对于使用老版本浏览器的网站来说很重要，因为它可以让我们退回到 jpeg 或 png 等公认的格式。

为了 SEO 的缘故，我们包括了`loading=”lazy”`标签，用于任何出现在“文件夹下”的东西。即使是空的，也需要指定一个 alt。

我们还指定了宽度和高度，这有助于浏览器提前计算布局。

```
<picture>
 ̶<̶s̶o̶u̶r̶c̶e̶ ̶s̶r̶c̶s̶e̶t̶=̶"̶@̶i̶m̶a̶g̶e̶W̶e̶b̶P̶"̶ ̶t̶y̶p̶e̶=̶"̶i̶m̶a̶g̶e̶/̶w̶e̶b̶p̶"̶>̶
<source srcset="@imageJpeg" type='image/jpeg'>
<img src="@imageOriginal" width="600" loading="@lazyLoading" alt="" />
</picture>
```

## 把所有的放在一起

我们现在可以在任何视图中引用我们的图像生成器部分

```
@Html.Partial("ResponsiveImage",new ViewDataDictionary(ViewData) { { "publishedContent", Model.TestImage.MediaItem }, { "lazyLoading", true } })
```

我们完了。一个超级简单和表演的形象。接下来，我们需要将宽度和高度输入到视图数据中，不过还需要一天的修正。

**问题:使用带有 Crops 的媒体进行裁剪**

根据您使用的测试版，您可能需要采取额外的步骤来获得 IPublishedContent。如果你的媒体以 MediaWithCrops 的形式出现，获取它的 MediaItem 值，这将是 IPublishedContent 类型，所以你可以把它传入。快乐的日子！

```
MediaWithCrops cmsImage = Model.Value("cmsImage");IPublishedContent image = cmsImage.MediaItem;
```

喜欢你读的吗？给我们鼓掌👏👏👏
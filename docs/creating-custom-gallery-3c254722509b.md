# 创建自定义 iOS 图库

> 原文：<https://itnext.io/creating-custom-gallery-3c254722509b?source=collection_archive---------0----------------------->

这不是一个非常常见的功能，但有时您需要为 iPhone gallery 创建一个自定义布局。我和我的团队不得不在我们目前正在进行的项目中对它进行编码。幸运的是，这不是我们必须建立的第一个自定义图库，所以我找到了一个旧代码并稍微改变了布局。这非常有效，我们节省了一天的开发时间。

创建我们想要的布局依赖于用户界面，所以在这个例子中，我们将有一个简单的。获取用户的相册和图片才是真正的问题。我们将为 PHPhotoLibrary、PHAssetCollection 和 PHAsset 创建三个扩展。只需稍作改动，你就可以在需要为你的 iOS 应用创建自定义图库时重复使用它们。

首先，我们需要获得访问用户画廊的许可。

得到它后，我们需要获取相册。在提取之前，我们将选择相册的类型和子类型。相册有三种类型:相册、智能相册和时刻。我们不会显示没有资产的相册。

展示专辑时，我们需要一个标题和封面图片。Apple 的默认封面图片是集合中的最后一张图片，因此您可以在实现代码时更改它。

最后，我们需要为选定的相册获取照片。好的一面是，我们可以创建 fetch 选项谓词，并指定我们想要的媒体类型。

为了在单元格中显示图像，我们将使用 getAssetTumbnail 方法。在这个例子中，我没有使用 getOriginalImage 和 getImageFromPHasset 方法，因为我们没有上传图片。但是，在真正的 app 中，这些方法会很有用。

创建自定义图库非常简单。此外，当您完成编码时，您将在所有其他项目中重用它。我希望这对你有用。下面是代码[示例](https://github.com/pakisha/customGallery)。

## 感谢阅读:)

如果这篇文章是有帮助的，并且你想听到更多类似的话题，请鼓掌，分享，关注或评论。
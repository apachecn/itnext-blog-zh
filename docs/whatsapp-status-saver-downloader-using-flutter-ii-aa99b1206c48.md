# 使用 Flutter II 的 WhatsApp 状态保护程序/下载器

> 原文：<https://itnext.io/whatsapp-status-saver-downloader-using-flutter-ii-aa99b1206c48?source=collection_archive---------0----------------------->

这是早期文章 WhatsApp 状态保护程序/下载器使用 Flutter 的延续。如果你错过了，点击[这里](https://medium.com/@sammytech/whatsapp-status-saver-downloader-using-flutter-ff5ff897e56)。GitHub repo 的链接在本文的底部。

![](img/fc05b14891aeb33deab113258a2dec24.png)

颤振标志

上次，我们停在了仪表板上，它是 UI/dashboard.dart 文件。实现非常简单，因为它在选项卡式视图中调用图像屏幕和视频屏幕。

![](img/6998e0a6bac060b0ae53ed937bf3952e.png)

应用外观

所以我们想在选项卡视图中设计屏幕的图像部分。下一步非常重要，那就是从用户的设备上获取 WhatsApp 的状态。我们需要访问存储状态的目录。这个目录是/storage/emulated/0/WhatsApp/Media/。状态

那么我们如何获取和显示图像呢？这里使用的逻辑是:创建一个列表，查找全部。jpg 文件，并将这些文件附加到该列表中，然后我们在 StaggeredGridView 中使用该列表来显示所有图像。😎 😎 😎

所以说到代码。UI/imageScreen.dart

```
import 'package:flutter/material.dart';
import 'dart:io';
import 'package:flutter_staggered_grid_view/flutter_staggered_grid_view.dart';
import 'package:wa_status_saver/ui/viewphotos.dart';final Directory _photoDir =
    new Directory('/storage/emulated/0/WhatsApp/Media/.Statuses');class ImageScreen extends StatefulWidget {
  @override
  ImageScreenState createState() => new ImageScreenState();
}class ImageScreenState extends State<ImageScreen> {
  @override
  void initState() {
    super.initState();
  } @override
  Widget build(BuildContext context) {
    if (!Directory("${_photoDir.path}").existsSync()) {
      return Container(
        padding: EdgeInsets.only(bottom: 60.0),
        child: Center(
          child: Text(
            "Install WhatsApp\nYour Friend's Status Will Show Here",
            style: TextStyle(fontSize: 18.0),
          ),
        ),
      );
    } else {
      var imageList = _photoDir
          .listSync()
          .map((item) => item.path)
          .where((item) => item.endsWith(".jpg"))
          .toList(growable: false);
      if (imageList.length > 0) {
        return Container(
          padding: EdgeInsets.only(bottom: 60.0),
          child: StaggeredGridView.countBuilder(
            padding: EdgeInsets.all(8.0),
            itemCount: imageList.length,
            crossAxisCount: 4,
            itemBuilder: (context, index) {
              String imgPath = imageList[index];
              return Material(
                elevation: 8.0,
                borderRadius: BorderRadius.all(Radius.circular(8)),
                child: InkWell(
                  onTap: () {
                    Navigator.push(
                        context,
                        new MaterialPageRoute(
                            builder: (context) => new ViewPhotos(imgPath)));
                  },
                  child: Hero(
                      tag: imgPath,
                      child: Image.file(
                        File(imgPath),
                        fit: BoxFit.cover,
                      )),
                ),
              );
            },
            staggeredTileBuilder: (i) =>
                StaggeredTile.count(2, i.isEven ? 2 : 3),
            mainAxisSpacing: 8.0,
            crossAxisSpacing: 8.0,
          ),
        );
      } else {
        return Scaffold(
          body: Center(
            child: new Container(
                padding: EdgeInsets.only(bottom: 60.0),
                child: Text('No Image Found!', style: TextStyle(fontSize: 18.0),)),
          ),
        );
      }
    }
  }
}
```

是的，我们现在可以在屏幕上以一种网格的形式看到 WhatsApp 的状态(图片)😃 😃 😃。但是等等，如果我们想查看一张图片或者下载它呢？我们该怎么做呢？😏 😏 😏

文件:UI/viewphotos.dart

```
import 'dart:io';
import 'dart:typed_data';
import 'package:flutter/material.dart';
import 'package:image_gallery_saver/image_gallery_saver.dart';
import 'package:flutter_fab_dialer/flutter_fab_dialer.dart'; class ViewPhotos extends StatefulWidget {
  final String imgPath;
  ViewPhotos(this.imgPath); @override
  _ViewPhotosState createState() => _ViewPhotosState();
}class _ViewPhotosState extends State<ViewPhotos> {
  var filePath;
  final String imgShare = "Image.file(File(widget.imgPath),)"; final LinearGradient backgroundGradient = new LinearGradient(
    colors: [
      Color(0x00000000),
      Color(0x00333333),
    ],
    begin: Alignment.*topLeft*,
    end: Alignment.*bottomRight*,
  ); void _onLoading(bool t, String str) {
    if (t) {
      showDialog(
          context: context,
          barrierDismissible: false,
          builder: (BuildContext context) {
            return SimpleDialog(
              children: <Widget>[
                Center(
                  child: Container(
                      padding: EdgeInsets.all(10.0),
                      child: CircularProgressIndicator()),
                ),
              ],
            );
          });
    } else {
      Navigator.*pop*(context);
      showDialog(
          context: context,
          barrierDismissible: false,
          builder: (BuildContext context) {
            return Padding(
              padding: const EdgeInsets.all(8.0),
              child: SimpleDialog(
                children: <Widget>[
                  Center(
                    child: Container(
                      padding: EdgeInsets.all(15.0),
                      child: Column(
                        children: <Widget>[
                          Text(
                            "Great, Saved in Gallary",
                            style: TextStyle(
                                fontSize: 20, fontWeight: FontWeight.*bold*),
                          ),
                          Padding(
                            padding: EdgeInsets.all(10.0),
                          ),
                          Text(str,
                              style: TextStyle(
                                fontSize: 16.0,
                              )),
                          Padding(
                            padding: EdgeInsets.all(10.0),
                          ),
                          Text("FileManager > Downloaded Status",
                              style: TextStyle(
                                  fontSize: 16.0, color: Colors.*teal*)),
                          Padding(
                            padding: EdgeInsets.all(10.0),
                          ),
                          MaterialButton(
                            child: Text("Close"),
                            color: Colors.*teal*,
                            textColor: Colors.*white*,
                            onPressed: () => Navigator.*pop*(context),
                          )
                        ],
                      ),
                    ),
                  ),
                ],
              ),
            );
          });
    }
  } @override
  Widget build(BuildContext context) {
    //The list of FabMiniMenuItems that we are going to use
    var _fabMiniMenuItemList = [
      new FabMiniMenuItem.withText(
          new Icon(Icons.*sd_storage*), Colors.*teal*, 4.0, "Button menu",
          () async {
        _onLoading(true, ""); Uri myUri = Uri.*parse*(widget.imgPath);
        File originalImageFile = new File.fromUri(myUri);
        Uint8List bytes;
        await originalImageFile.readAsBytes().then((value) {
          bytes = Uint8List.fromList(value);
          print('reading of bytes is completed');
        }).catchError((onError) {
          print('Exception Error while reading audio from path:' +
              onError.toString());
        });
        final result =
            await ImageGallerySaver.*saveImage*(Uint8List.fromList(bytes));
        print(result);
        _onLoading(false,
            "If Image not available in gallary\n\nYou can find all images at");
      }, "Save", Colors.*black*, Colors.*white*, true),
      new FabMiniMenuItem.withText(new Icon(Icons.*share*), Colors.*teal*, 4.0,
          "Button menu", () {}, "Share", Colors.*black*, Colors.*white*, true),
      new FabMiniMenuItem.withText(new Icon(Icons.*reply*), Colors.*teal*, 4.0,
          "Button menu", () {}, "Repost", Colors.*black*, Colors.*white*, true),
      new FabMiniMenuItem.withText(new Icon(Icons.*wallpaper*), Colors.*teal*, 4.0,
          "Button menu", () {}, "Set As", Colors.*black*, Colors.*white*, true),
      new FabMiniMenuItem.withText(
          new Icon(Icons.*delete_outline*),
          Colors.*teal*,
          4.0,
          "Button menu",
          () {},
          "Delete",
          Colors.*black*,
          Colors.*white*,
          true),
    ]; return Scaffold(
      backgroundColor: Colors.*black12*,
      appBar: AppBar(
        elevation: 0.0,
        backgroundColor: Colors.*transparent*,
        leading: IconButton(
          color: Colors.*indigo*,
          icon: Icon(
            Icons.*close*,
            color: Colors.*white*,
          ),
          onPressed: () => Navigator.*of*(context).pop(),
        ),
      ),
      body: SizedBox.expand(
        child: Stack(
          children: <Widget>[
            Align(
              alignment: Alignment.*center*,
              child: Hero(
                tag: widget.imgPath,
                child: Image.file(
                  File(widget.imgPath),
                  fit: BoxFit.cover,
                ),
              ),
            ),
            new FabDialer(
                _fabMiniMenuItemList, Colors.*teal*, new Icon(Icons.*add*)),
          ],
        ),
      ),
    );
  }
}
```

OnTap，图像调用 UI/viewphotos.dart 文件中的 ViewPhotos 类。因此，我们在一个支架中显示图像，appBar 有一个前导图标(x)来关闭这个特定的图像。我还添加了一个浮动的动作按钮拨号器到这个页面，以防万一你想在图像上执行多个动作。这是一个为默认菜单提供替代选项的小部件。你可以在[酒吧](https://pub.dev/packages/flutter_fab_dialer)查看 flutter_fab_dialer。目前，只有 save 按钮被赋予了 onPressed 方法。其他的都是空的。你可以在它们周围玩耍。😉 😉 😉

这就是关于图像屏幕的全部内容。我将继续在另一篇文章中，我们将处理视频状态的应用程序。这是 https://github.com/mastersam07/wa_status_saver Github 回购[的链接。如果你遇到问题，你可以联系 GitHub 或者在 GitHub 上打开一个问题😀 😀 😀。](https://github.com/mastersam07/wa_status_saver)

敬请关注。
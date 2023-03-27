# Laravel 和 Vue:用 CRUD 管理面板创建作品集网站——第 12 章

> 原文：<https://itnext.io/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-12-f35917a9d5c8?source=collection_archive---------8----------------------->

## 重构组合控制器

我们已经构建了能够执行 CRUD 功能的 PortfolioEditor。

**创建** —我们可以使用 **addItemModal 添加一个投资组合项目🌟**

**READ** —我们可以从 route 中获取投资组合项目并显示到 **PortfolioEditorTable 中🌟**

**更新** —我们可以使用**更新项目模型来更改投资组合项目的详细信息🌟**

**删除** —使用 DELETE 和 **confirmDeleteModal** 删除投资组合项目🌟

我们现在需要重构项目组合控制器。第一件事是删除任何空函数。

## 删除空函数

如果我们看一看**PortfolioController.php**，我们可以看到有空的函数。我们可以去掉这些功能。

```
/̶*̶*̶
̶ ̶ ̶ ̶ ̶ ̶*̶ ̶S̶h̶o̶w̶ ̶t̶h̶e̶ ̶f̶o̶r̶m̶ ̶f̶o̶r̶ ̶c̶r̶e̶a̶t̶i̶n̶g̶ ̶a̶ ̶n̶e̶w̶ ̶r̶e̶s̶o̶u̶r̶c̶e̶.̶
̶ ̶ ̶ ̶ ̶ ̶*̶
̶ ̶ ̶ ̶ ̶ ̶*̶ ̶@̶r̶e̶t̶u̶r̶n̶ ̶\̶I̶l̶l̶u̶m̶i̶n̶a̶t̶e̶\̶H̶t̶t̶p̶\̶R̶e̶s̶p̶o̶n̶s̶e̶
̶ ̶ ̶ ̶ ̶ ̶*̶/̶
̶ ̶ ̶ ̶ ̶p̶u̶b̶l̶i̶c̶ ̶f̶u̶n̶c̶t̶i̶o̶n̶ ̶c̶r̶e̶a̶t̶e̶(̶)̶
̶ ̶ ̶ ̶ ̶{̶
̶ ̶ ̶ ̶ ̶ ̶ ̶ ̶ ̶/̶/̶
̶ ̶ ̶ ̶ ̶}̶
```

我们还可以使注释更具体地针对我们的代码。

```
**/̶*̶*̶
̶ ̶ ̶ ̶ ̶ ̶*̶ ̶D̶i̶s̶p̶l̶a̶y̶ ̶a̶ ̶l̶i̶s̶t̶i̶n̶g̶ ̶o̶f̶ ̶t̶h̶e̶ ̶r̶e̶s̶o̶u̶r̶c̶e̶.̶
̶ ̶ ̶ ̶ ̶ ̶*̶
̶ ̶ ̶ ̶ ̶ ̶*̶ ̶@̶r̶e̶t̶u̶r̶n̶ ̶\̶I̶l̶l̶u̶m̶i̶n̶a̶t̶e̶\̶H̶t̶t̶p̶\̶R̶e̶s̶p̶o̶n̶s̶e̶
̶*̶/̶*****// Display a listing of all the portfolio items.*** *public function* index() {
    *return* PortfolioItem::*all*();
}
```

## 将函数移动到单独的文件中

如果我们查看 api.php 文件，我们可以看到以下与 ProfileEditor 组件相关的路线

```
Route::*get*('/portfolio', 'PortfolioController@index');
Route::*post*('/portfolio', 'PortfolioController@store');
Route::*post*('/portfolio/{id}', 'PortfolioController@update');
Route::*delete*('/portfolio/{id}', 'PortfolioController@destroy');
```

我们应该保留**索引**、**储存**、**更新**和**销毁**。其他函数可以放在单独的文件中。

因此，让我们创建单独的文件

```
**php artisan make:controller PortfolioItemStorer**
```

现在我们在控制器文件夹中有一个名为**PortfolioItemStorer.php**的文件

我们要转移到这个文件中的函数是

*   resizeAndStore
*   makeSmallImage
*   制作大型图像
*   商店详情

PortfolioItemStorer.php 现在👇

```
*<?php

namespace* App\Http\Controllers;

*use* App\PortfolioItem;
*use* Illuminate\Http\Request;

*class* PortfolioItemStorer *extends* Controller
{
    *private function* resizeAndStore($file, $hashName) {
        $img = Image::*make*($file);
        $img->backup();
        $this->makeSmallImage($img, $hashName);
        $img->reset();
        $this->makeLargeImage($img, $hashName);
    }

    *private function* makeSmallImage($img, $hashName) {
        $dir = 'public/itempics/small/';
        $fullpath = $dir . $hashName;
        $contents = $img->resize(320, 240)->encode();
        Storage::*put*($fullpath, $contents);
    }

    *private function* makeLargeImage($img, $hashName) {
        $dir = 'public/itempics/large/';
        $fullpath = $dir . $hashName;
        $contents = $img->resize(640, 480)->encode();
        Storage::*put*($fullpath, $contents);
    }

    *private function* storeDetails($hashName, $item) {
        $item->img_path = $hashName;
        $item->description = request()->description;
        $item->name = request()->name;
        $item->save();
    }
} 
```

我们应该将`***use* Intervention\Image\Image;**`添加到顶部，因为在**resizeAndStore(**$ img =**Image**::*make*($ file)中需要它；)和`***use* Storage**` ，因为在 makeSmallImage 和 makeLargeImage 函数中都需要。

```
*use* App\PortfolioItem;
*use* Illuminate\Http\Request;
***use* Intervention\Image\Image;
*use* Storage**
```

我们还应该删除前两个导入(`**use**`行)，因为它们在 PortfolioItemStorer 中的任何地方都没有使用

```
**u̶s̶e̶ ̶A̶p̶p̶\̶P̶o̶r̶t̶f̶o̶l̶i̶o̶I̶t̶e̶m̶;̶
u̶s̶e̶ ̶I̶l̶l̶u̶m̶i̶n̶a̶t̶e̶\̶H̶t̶t̶p̶\̶R̶e̶q̶u̶e̶s̶t̶;̶**
*use* Intervention\Image\Image;
*use* Storage
```

## 如何在单独的文件中调用方法

![](img/160b328dff2e36a6577a89d4768be154.png)

由[帕万·特里库塔姆](https://unsplash.com/@ptrikutam?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

当我们在同一个文件中调用函数时，我们使用`**$this->**` **。**到使用名为 **PortfolioItemStorer** 的单独文件，我们必须使用`**PortfolioItemStorer::**`。

让我们相应地修改 PortfolioController 的`**store**`函数。

```
*public function* **store**(StorePortfolioItemRequest $request) {
    $file = $request->itempic;
    $hashName = $file->hashName();
    **$̶t̶h̶i̶s̶-̶>̶r̶e̶s̶i̶z̶e̶A̶n̶d̶S̶t̶o̶r̶e̶(̶$̶f̶i̶l̶e̶,̶ ̶$̶h̶a̶s̶h̶N̶a̶m̶e̶)̶;̶**
    **PortfolioItemStorer::*resizeAndStore*($file, $hashName);**
    $item = *new* PortfolioItem();
    **$̶t̶h̶i̶s̶-̶>̶s̶t̶o̶r̶e̶D̶e̶t̶a̶i̶l̶s̶(̶$̶h̶a̶s̶h̶N̶a̶m̶e̶,̶ ̶$̶i̶t̶e̶m̶)̶;̶**
    **PortfolioItemStorer::*storeDetails*($hashName, $item);**
}
```

我们也应该以同样的方式修改`**update**`函数

```
*public function* **update**(UpdatePortfolioItemRequest $request, $id) {
    $item = PortfolioItem::*find*($id);
    *if* ($request->hasFile('itempic')) {
        $file = $request->itempic;
        $hashName = $file->hashName();
        **$̶t̶h̶i̶s̶-̶>̶r̶e̶s̶i̶z̶e̶A̶n̶d̶S̶t̶o̶r̶e̶(̶$̶f̶i̶l̶e̶,̶ ̶$̶h̶a̶s̶h̶N̶a̶m̶e̶)̶;̶**
        **PortfolioItemStorer::*resizeAndStore*($file, $hashName);**
        $item->img_path = $hashName;
    }
    *if* ($request->filled('description')) $item->description = request()->description;
    *if* ($request->description === *null*) $item->description = '';
    $item->name = $request->name;
    $item->save();
}
```

这个函数正在调用**portfolio item store**的`**resizeAndStore**`方法，这是:

```
*private function* resizeAndStore($file, $hashName) {
    $img = Image::*make*($file);
    $img->backup();
    $this->makeSmallImage($img, $hashName);
    $img->reset();
    $this->makeLargeImage($img, $hashName);
}
```

**重要的** : `**private**`函数**无法从其他文件**中调用，我们必须将这些函数改为`**public static**`。

## 将私有静态更改为公共静态

应将 **resizeAndStore** 函数修改为 `**public static**` 函数，而不是`**private**`函数。

```
**p̶r̶i̶v̶a̶t̶e̶** ***public******static*** *function* resizeAndStore($file, $hashName) {
    $img = Image::*make*($file);
    $img->backup();
    $this->makeSmallImage($img, $hashName);
    $img->reset();
    $this->makeLargeImage($img, $hashName);
}
```

**storeDetails** 也是从 store()函数调用的，所以我们也需要使`**public static**` 也是如此。

```
**p̶r̶i̶v̶a̶t̶e̶** ***public******static*** *function* storeDetails($hashName, $item) {
    $item->img_path = $hashName;
    $item->description = request()->description;
    $item->name = request()->name;
    $item->save();
}
```

## 调用同一类的静态函数

我们需要将 resizeAndStore 的`**$this->**` 改为`**self::**`

```
*public static function* resizeAndStore($file, $hashName) {
    $img = Image::make($file);
    $img->backup();
    ***self***::*makeSmallImage*($img, $hashName);
    $img->reset();
    ***self***::*makeLargeImage*($img, $hashName);
}
```

我们应该将 makeSmallImage 和 makeLargeImage 从`**private**`函数改为`**private static**`

```
*private* ***static*** *function* makeSmallImage($img, $hashName) {
    $dir = 'public/itempics/small/';
    $fullpath = $dir . $hashName;
    $contents = $img->resize(320, 240)->encode();
    Storage::*put*($fullpath, $contents);
}

*private* ***static*** *function* makeLargeImage($img, $hashName) {
    $dir = 'public/itempics/large/';
    $fullpath = $dir . $hashName;
    $contents = $img->resize(640, 480)->encode();
    Storage::*put*($fullpath, $contents);
}
```

## 检查使用线路

**PortfolioItemStorer.php**的`**use**`行应该是:

```
*use* Intervention\Image\Facades\Image;
*use* Storage;
```

**PortfolioController.php**的`**use**`线应为:

```
*use* App\Http\Requests\StorePortfolioItemRequest;
*use* App\Http\Requests\UpdatePortfolioItemRequest;
*use* App\PortfolioItem;
```

## 重构 PortfolioController 概述

在这里的重构过程中，我们有:

-移除了空函数
-移除了与一个独立文件间接相关的函数

-使用`**FileNameHere::**` 而不是`**$this**`
调用独立文件中的方法-将函数从私有改为公共静态，以便它们可以从其他文件中调用

-调用同一类的静态函数，即将`**$this**`更改为`**self::**`

-检查了`**use**`线路，确保不多也不少于**条需要的线路**

# 列出清单—第 12 章结束

我们可以从列表中勾掉**重构 PortfolioController** 。

*   前端验证(用于更新 functionality)☑️
*   后端验证(用于更新 functionality)☑️
*   更新和删除功能(阅读是 done)☑️

—

*   重构 PortfolioController☑️

—

*   带文本的图像(Portfolio.vue)
*   具有更大图像和描述的模型(Portfolio.vue)
*   modal (Portfolio.vue)上的关闭按钮

在第 13 章，我们将建立投资组合的公共视图。

[](/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-13-981b0cc36c34) [## Laravel 和 Vue:用 CRUD 管理面板创建作品集网站——第 13 章

### 建立投资组合的公共视图

itnext.io](/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-13-981b0cc36c34)
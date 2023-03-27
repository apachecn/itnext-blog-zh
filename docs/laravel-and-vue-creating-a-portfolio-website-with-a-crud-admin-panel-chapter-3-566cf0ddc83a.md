# Laravel 和 Vue:用 CRUD 管理面板创建作品集网站——第三章

> 原文：<https://itnext.io/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-3-566cf0ddc83a?source=collection_archive---------5----------------------->

## 创建并阅读报头

![](img/bfe8c74d9b613d55a1861b4af37bbaa2.png)

**概述:**我们有能力显示基于数据库的文本和图像。文本可以直接从数据库中读取，但是对于图像，我们获取**图像路径**，并在`img`标签中呈现它以显示图像。

**现在:**我们将把这些能力组合到一个 **ProfileEditor** 组件中。在那之前，我们必须做一些清洁工作。

![](img/b8011398d9d6d78d45156b436ba20d06.png)

[诚信公司](https://unsplash.com/@honest?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# 清洁 App.vue

App.vue 目前有一些不再需要的代码。**{ { image path } }**img 标签下面现在废了。我们只需要它来看看我们在将 data()的 imagepath 放入模板标签时是否有问题。

```
<div>
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + imagepath"/>
    **{̶{̶i̶m̶a̶g̶e̶p̶a̶t̶h̶}̶}̶**
    <image-uploader></image-uploader>
</div>
```

你也可以去掉**图片上传器**标签，因为我们将用 ProfileEditor 组件替换它。

```
<div>
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + imagepath"/>
    **<̶i̶m̶a̶g̶e̶-̶u̶p̶l̶o̶a̶d̶e̶r̶>̶<̶/̶i̶m̶a̶g̶e̶-̶u̶p̶l̶o̶a̶d̶e̶r̶>̶**
</div>
```

从导入和组件对象中删除图像上传程序

```
<script>
    *import* masthead *from* './masthead';
    **i̶m̶p̶o̶r̶t̶ ̶i̶m̶a̶g̶e̶U̶p̶l̶o̶a̶d̶e̶r̶ ̶f̶r̶o̶m̶ ̶'̶.̶/̶I̶m̶a̶g̶e̶U̶p̶l̶o̶a̶d̶e̶r̶'̶;̶**
    *export default* {
        name: "App",
        components: {
            masthead,
            **i̶m̶a̶g̶e̶U̶p̶l̶o̶a̶d̶e̶r̶**
        },
```

# 创建 ProfileEdior 组件

现在应该创建组件文件，导入它并将其包含在组件对象中。

```
<script>
    *import* masthead *from* './masthead';
    ***import* ProfileEditor *from* "./ProfileEditor";**
    *export default* {
        name: "App",
        components: {
            masthead,
            **ProfileEditor**
        },
```

> 确保 components 对象中组件的名称与导入行匹配。大写和小写字母必须相同

在 ProfileEditor 组件中，创建一个带有**文本**字段和**文件**选择字段的表单

```
<form>
    <input *type*="**text**">
    <input *type*="**file**">
    <button>Submit</button>
</form>
```

当你选择一个文件时，一个对话框会打开，允许你上传任何文件。我们想把它限制在图片上。

将文件选择字段修改为仅用于图像文件。

```
<input *type*="file" ***accept*="image/*"**>
```

当您选择文件时，只会显示图像和文件夹，并且只有图像**可以上传。**

以前，我们使用 FormData()附加一个文件，并在选择文件后立即将其发送到服务器。

这一次，我们要输入配置文件名，选择图像，然后提交表单。为此，我们需要查看表单输入绑定。

## 表单输入绑定

我们必须绑定名称，绑定图像文件，并添加一个提交表单的方法。

**绑定名称** —向输入文本字段添加一个`v-model`。v-model 叫做**的名字。**

```
<input *type*="text" ***v-model*="name"**>
```

脚本元素应该有数据()。模型应该反映在脚本标签 data():

```
data() {
    *return* {
        **name: '',**
    }
},
```

当您在名称输入字段中键入一些内容时，您会发现正在发生的变化**实时**！在 VueDevtools 的数据部分。

还可以看张家发生的**现场直播！**将 **{{name}}** 放在页面的某个位置，当您在名称输入字段中键入内容时，观察页面的某个区域发生变化。

```
<form>
    **{{name}}**
    <input *type*="text" *v-model*="name">
    <input *type*="file" *accept*="image/*">
    <button>Submit</button>
</form>
```

一旦你玩够了这个魔法，我们就可以移除 **{{name}}** 并进入**绑定文件**。

**绑定文件—** 添加`**file:null**`到数据()

```
data() {
    *return* {
        name: '',
        **file: *null***}
},
```

因为文件输入不能有 v-model，所以我们不能有动态数据更改。我们要做的是创建一个**方法**，它将在我们选择一个文件时运行。更改文件输入字段以运行该方法。

```
<input *type*="file" *accept*="image/*" ***@change*="selectFile"**>
```

该方法:

```
selectFile(event) {
  *this.file = event.target.files[0];* },
```

Vue Devtools 将从**文件开始:null** ，当您选择文件时，它将显示**文件:文件**

您可以使用{{file}}查看页面上的数据更改

```
<form>
    **{{file}}**
    <input *type*="text" *v-model*="name">
    <input *type*="file" *accept*="image/*">
    <button>Submit</button>
</form>
```

对于您选择的任何文件，您将看到显示[目标文件]的页面。要查看文件的名称，可以更改 selectFile 方法

```
selectFile(event) {
  *this*.file = event.target.files[0]**.name**;
},
```

播放完毕后，将 selectFile 方法改回如下:

```
selectFile(event) {
  *this*.file = event.target.files[0];
},
```

删除 **{{file}}** 和**添加一个提交方法**到按钮。

**添加提交方法** —该方法应该在**点击**时运行

```
<form>
    <input *type*="text" *v-model*="name">
    <input *type*="file" *accept*="image/*" *@change*="selectFile">
    <button ***@click.prevent*="onSubmit"**>Submit</button>
</form>
```

> 必须是 **@click.prevent** 而不是 **@click** 才能阻止你点击后的页面刷新

该方法应该使用 **FormData()** 并附加名称和文件。您可以使用`**this.**`从 data()中获取数据

```
onSubmit() {
    *let* fd = *new* FormData();
    fd.append('name', ***this*.**name);
    fd.append('userpic', ***this*.**file);
}
```

如果想检查追加了什么，可以将这个`for`循环添加到 onSubmit 方法中

```
onSubmit() {
    *let* fd = *new* FormData();
    fd.append('name', *this*.name);
    fd.append('userpic', *this*.file);

    ***for* (*let x* *of* fd.values()) {
        console.log(x);
    }**}
```

单击 submit 后，您将在浏览器控制台中看到附加的数据。

修改 onSubmit 方法。移除`for`回路，增加`axios.post`

```
onSubmit() {
    *let* fd = *new* FormData();
    fd.append('name', *this*.name);
    fd.append('userpic', *this*.file); **axios.post('api/profile', fd).then(response => *console*.log(response));**
}
```

该组件能够发送数据，但是它不会得到响应，因为发送路径尚不存在。为此，我们现在需要在服务器端工作。

## 服务器端

我们需要在 api.php 写路线

```
Route::*post*('/profile', 'ProfileController@create');
```

让我们创建一个**资源控制器**。资源控制器有已经做好的函数，你只需要把它们填进去。当你打开控制器文件时，你就会明白我的意思。

```
php artisan make:controller ProfileController --**resource**
```

像这样填写创建函数:

```
*public function* create() {
    *return* **'ProfileController CreateFunction Reached'**;
}
```

发布到此路由，您将在浏览器控制台和网络选项卡中看到此消息

```
**ProfileController CreateFunction Reached**
```

一旦成功，检查`request()`中有什么。

```
*public function* create() {
    *return* **request()**;
}
```

您将获得一个 JSON，其中包含任何附加内容的详细信息。如果您发布了一个空表单，您将得到:

```
{"name":null,"userpic":"null"}
```

如果您在名称输入中发布@UmarCodes 而没有文件，您将得到

```
{"name":**@UmarCodes**,"userpic":"null"}
```

如果你发布了一个文件和一个名字，你将得到 userpic 的{}

```
{"name":@UmarCodes,"userpic":**{}**}
```

我们希望 create 函数做三件事:

1.  存储名称
2.  存储图像路径
3.  存储图像本身

但是在我们编写函数代码之前，我们实际上需要一个表来存储数据。

让我们使用迁移来创建配置文件表。

```
php artisan make:migration CreateProfileTable
```

我们希望名称和图像在同一个表中，因为我们不再使用 basic_info 表和 image 表，我们可以使用`**Schema::dropIfExists()**`删除它们

让我们将`**Schema::dropIfExists();**` 添加到 up 函数中，这样当您迁移时，它将同时:

a)删除先前的表格

和

b)创建配置文件表

```
*public function* up()
{   **Schema::dropIfExists('basic_info');
    Schema::dropIfExists('image');**
    Schema::create('profile', *function* (Blueprint $table) {
        $table->bigIncrements('id');
        **$table->string('name');
        $table->string('img_path');**
        $table->timestamps();
    });
}
```

运行迁移

```
php artisan migrate
```

现在，如果您查看您的数据库，您会发现 basic_info 表和 image 表不再存在。

您还会发现一个配置文件表。让我们为这个表创建一个模型。

```
php artisan make:model Profile
```

使用`protected **$table**` 指定模型连接到哪个表，并使用 **$fillable** 数组指定哪些列是可填充的

```
*protected* **$table** = 'profile';
*protected* **$fillable** = **['name', 'img_path'];**
```

让我们回到 ProfileController 并填充 create 函数。我们将存储**名称**、**图像路径**和图像**文件**本身。

> 别忘了在控制器顶部加上`use App\Profile;`。

```
*public function* create()
{
    $profile = *new* Profile();
    $profile->**name** = request()->name;
    $profile->**img_path** = request()->file('userpic')->hashName();
    $profile->save();
    return request()->**file**('userpic')->store('/public/userpics');
}
```

让我们来展示您的个人资料。

# 显示您的个人资料

要显示您的个人资料，我们需要遵循与显示您的姓名和上传图像相同的流程。

## **数据库已经勾选，因为我们已经将数据放入数据库。**

![](img/e401a98014179d98571f5b6722e9fa8d.png)

## 途径

注意:已经有一个带有地址/配置文件的路由，但那是一个 post 路由，而这是一个 **get** 路由。

```
Route::***get***('/profile', 'ProfileController@show');
```

**注**:您可以拥有相同地址的路由，只要它们属于不同的类型并且链接到不同的功能。

发布路线链接到 ProfileController 中的**创建**功能，而获取路线链接到**显示**功能。

```
Route::*post*('/profile', 'ProfileController@**create**');
Route::*get*('/profile', 'ProfileController@**show**');
```

显示功能:

```
*public function* show()
{
    $profile = Profile::*all*()[0];
    *return* $profile;
}
```

现在访问路线，您将获得一个表示概要表的 JSON。

![](img/f5ee4180f53a31499033c32e56860bdd.png)

## 脚本

请随意从已安装的()中移除早期的方法。添加我们现在要创建的方法，即 get profile()；

```
mounted() {
    t̶h̶i̶s̶.̶g̶e̶t̶B̶a̶s̶i̶c̶I̶n̶f̶o̶(̶)̶;̶
    t̶h̶i̶s̶.̶g̶e̶t̶I̶m̶a̶g̶e̶P̶a̶t̶h̶(̶)̶;̶
    ***this*.getProfile();**
},
```

getProfile()方法类似于前面使用的 get 方法。

**需要注意的区别:**这次我们要的是**整体** **响应数据**。

> 在第一章和第二章中，我们在寻找位于`**response.data**` 中的`**name**` 和`**imagepath**` 。

在这里，我们想要在 api/ **profile** 路径上可用的所有数据。这样我们就不必向数据中添加更多的东西了()。我们可以一次性获取所有的概要文件数据，将概要文件数据添加到 data()中，然后转到<模板></模板>。

```
getProfile() {
    axios.get('/api/**profile**')
        .then(response => {
            console.log(response.data);
            *this*.**profile** = **response.data**;
        })
        .catch(error => {
            *console*.log(error);
        });
},
```

> 可选:您也可以从方法对象中删除以前使用的方法

将**轮廓**添加到**数据**中()

```
data() {
  *return* {
      n̶a̶m̶e̶:̶ ̶'̶'̶,̶
      i̶m̶a̶g̶e̶p̶a̶t̶h̶:̶ ̶'̶'̶,̶
      **profile**: '',
  }
},
```

![](img/2b3fcd1447942d58ee1cd08d1fdb16dc.png)

在浏览器中重新加载页面，打开 Vue Devtools 你会看到

▶️ **简介:对象**

点击▶️，你可以看到物体里包含了什么；如果您在浏览器中访问该路线，您将获得相同的键值对。

## **模板**

是时候渲染一些数据了。在<masthead>元素上方，键入`**{{profile}}**`。</masthead>

```
<div>
    **{{profile}}**
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + imagepath"/>
    <profile-editor></profile-editor>
</div>
```

现在，在浏览器中加载页面，您将看到一个 JSON 对象，它与我们在 route 中得到的对象相同。

但是我们不想要整个 JSON。我们只想要`name`和`img_path`。为了得到这些，我们必须看一些叫做**点符号**的东西。

**点符号示例—** 假设您有一个名为 **dev** 的对象:

> let**dev**= { name:' Umar Taufiq '，Medium: '@UmarCodes'}

> dev.name 将向您展示 Umar Taufiq
> 
> dev.medium 会得到@UmarCodes

如果你想得到名字，你写下对象的名字，接下来你写一个点，然后是你想访问的项目。

**点符号在我们的例子中:**把`{{profile}}`改成`{{profile.name}}`，重新加载页面就能看到名字了。

```
<div>
    **{{profile.name}}**
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + imagepath"/>
    <profile-editor></profile-editor>
</div>
```

添加`**{{profile.img_path}}**` **，**重新加载页面，你也会看到图片路径。

```
<div>
    {{profile.name}}
    **{{profile.img_path}}**
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + imagepath"/>
    <profile-editor></profile-editor>
</div>
```

我们可以将`**imagepath**`更改为`profile.img_path`，它会向您显示图像。

```
<div>
    {{profile.name}}
    {{profile.img_path}}
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + **profile.img_path**"/>
    <profile-editor></profile-editor>
</div>
```

我们现在可以移除`{{profile.img_path}}`

```
<div>
    {{profile.name}}
    **{̶{̶p̶r̶o̶f̶i̶l̶e̶.̶i̶m̶g̶_̶p̶a̶t̶h̶}̶}̶**
    <masthead *:name*="name"></masthead>
    <img *v-bind:src*="'storage/userpics/' + **profile.img_path**"/>
    <profile-editor></profile-editor>
</div>
```

我们将在报头中使用点符号，但是在我们这样做之前，我们需要**传递配置文件对象**作为道具

**传递概要文件对象-** 我们可以像这样简单地传递概要文件对象:

在 App.vue 的报头标签中，将`**:name="name"**`更改为`**profile="profile"**`

```
<div>
    {{profile.name}}
    <masthead ***:profile*="profile"**></masthead>
    <img *v-bind:src*="'storage/userpics/' + **profile.img_path**"/>
    <profile-editor></profile-editor>
</div>
```

让我们在**报头组件**中有一个相应的道具

```
props: {
            **profile**: String
        }
```

然而，我们最终得到了这个错误:

> [Vue Warn]无效属性:属性“profile”的类型检查失败。应为值为“[object Object]”的字符串，但得到了对象

将`**profile**`作为字符串传递是行不通的，因为它不是一个字符串，而是一个对象。所以我们要把它设置成一个对象。

```
props: {
            **profile**: Object
        }
```

这里有一些好消息和一些坏消息。

好消息是错误已经消失了。

> 😄

坏消息是我们有了一个新的错误:

> 😦
> 
> [Vue Warn]无效属性:属性“profile”的类型检查失败。预期的对象，得到的字符串的值为**“**”。

这意味着道具应该是一个**对象**，但是它得到的数据(got String)是一个**空字符串**。数据是从父节点(App.vue)发送的，所以让我们转到父节点，将`profile: **''**` 更改为`profile: **Object**`

```
data() {
  *return* {
      profile: Object
  }
},
```

> 无效属性:属性“profile”的类型检查失败。预期的对象，得到的函数

为了解决这个问题，我们必须更清楚地告诉子组件什么类型的数据配置文件

```
data() {
  *return* {
      profile: **{
          type: Object
      }**}
},
```

这就清楚了数据`profile`的**类型**是什么。它是一个**物体**。

现在控制台没有 vue 错误了。

在子组件中，移除`{{name}}`(如果有)。并加上`**{{profile}}**`。使用点符号得到**名称**和**图像**

```
<template>
    <header>
        **{{profile.name}}
        <img *v-bind:src*="'storage/userpics/' + profile.img_path"/>**
    </header>
</template>
```

现在子组件正在愉快地渲染一切😃我们在控制台中没有得到 vue 错误。

我们可以将`**{{profile.name}}**` 包装在一个`**h1**`元素中。

```
<template>
    <header>
        **<h1>{{profile.name}}</h1>** <img *v-bind:src*="'storage/userpics/' + profile.img_path"/>    </header>
</template>
```

![](img/6b38bf8bbf98d235976fcb1e7a8db522.png)

# 让我们做一些造型

## 将图像放在名称上方

```
<template>
    <header>
 **<img *v-bind:src*="'storage/userpics/' + profile.img_path"/>
        <h1>{{profile.name}}</h1>**
    </header>
</template>
```

## 将页眉的内容居中

```
<style *scoped*>
header {
    display:flex; 
    justify-content:center; 
    align-items:center; 
}
</style>
```

图像和名称并排出现。我们希望图像出现在文本之上。

## 更改弯曲方向

元素将并排出现，因为默认的`**flex-direction**`是 row。我们把它改成列吧。

```
header {
    display:flex;
    justify-content:center;
    align-items:center;
    **flex-direction:column;**
}
</style>
```

## 应用背景

```
<style *scoped*>
header {
    **background: #04BC92;**
    display:flex;
    justify-content:center;
    align-items:center;
    flex-direction:column;
}
</style>
```

## **给出割台全视口高度**

```
header {
    background: #04BC92;
    display:flex;
    justify-content:center;
    align-items:center;
    flex-direction:column;
    **height:100vh;**
}
</style>
```

这应该会给你一个报头，但是它可能会在报头周围有一个看起来像**的白色边框**。要移除“白色边框”,请访问 App.vue

```
<style>
 **body {
        margin: 0;
    }**
</style>
```

我们现在有一个标题，看起来像这样:

![](img/bfe8c74d9b613d55a1861b4af37bbaa2.png)

# 截断/消除截断的需要

您现在可以通过运行`**TRUNCATE TABLE profile;**`来截断该表。现在创建概要文件，并查看生成的报头。

我们不希望每次更改配置文件时都清空表格，所以下一章将在更新报头的**上。**

# 我们能做什么，不能做什么

这是一个 CRUD(创建、读取、更新、删除)项目。

*   我们**可以**创建，即发布和存储。👍 😃
*   我们**可以**读取，即获取数据并呈现它。👍 😃
*   我们**还不能**更新**。👎**
*   **我们**不能**删除**还不能**。👎**

**所以接下来要做的是**更新**。👷**

**第四章见**

**[](https://medium.com/@UmarCodes/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-four-db24fed8db50) [## Laravel 和 Vue:用 CRUD 管理面板创建作品集网站——第四章

### 更新报头并重构代码

medium.com](https://medium.com/@UmarCodes/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-four-db24fed8db50)**
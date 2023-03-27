# Laravel 和 Vue:用 CRUD 管理面板创建作品集网站——第九章

> 原文：<https://itnext.io/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-nine-7acf3268ee0d?source=collection_archive---------2----------------------->

## 组合编辑器的更新模式

我们从第八章的结尾继续，在那里我们导入了更新项目模式文件

```
<section>
    <h1>Portfolio Editor</h1>
    **<update-item-modal *v-if*="updateItemModal" *:item*="item"/>**
    <portfolio-editor-table *:items*="items"/>
</section>
```

当在浏览器中看到 PortfolioEditor 时，我们需要将 updateItemModal 设置为 FALSE。

当单击编辑按钮时，我们需要将 updateItemModal 设置为 TRUE。

```
data() {
    *return* {
        items: '',
        updateItemModal: *false*,
        item: '',
    }
},
```

这就是 edititempubup(item)函数正在做的事情。

```
editItemPopup(item) {
    *this*.item = item;
    *if* (*this*.updateItemModal === *false*) {
        *this*.updateItemModal = *true*;
    }
    *else if* (*this*.updateItemModal === *true*) {
        *this*.updateItemModal = *false*;
    }
},
```

## 开始编写模型

让我们开始编写 **updateItemModal.vue** 文件

```
<template>
<transition *name*="fade">
        <div *class*="modal">
            <div *class*="modal-body"></div>
        </div>
 </transition>
</template>
```

添加一个关闭模式的方法

```
<div *@click*="closeModal" *class*="modal">
```

在`methods: {}`中编写方法

```
closeModal(event) {
    *if* (event.target.matches('.modal')) {
        *this*.$parent.updateItemModal = *false*;
    }
},
```

> 代码解释:`event.target`指你点击的任何元素。这意味着如果被点击的元素有 modal 类，那么 updateItemModal 将是`**false**`。

## 定位模态

我们还需要将 updateItemModal 放在其余元素的前面(就像一个 Modal)。

这可以通过将模态的`**position**`设置为`**fixed**`并将`**z-index**`设置为`**1**`来实现，同时工作台的 z 索引可以设置为 0。z 索引号决定哪个固定元素出现在前面，即如果有一个 z 索引为 2 的固定元素，它将出现在模态的前面，依此类推。

```
.modal {
    **position: fixed;z-index:1;**
}
```

让我们将顶部、底部、左侧和右侧设置为 0

```
.modal {
    position: fixed;z-index:1;
    **top: 0;
    bottom: 0;
    left: 0;
    right: 0;**
}
```

## 使用 rgba 的背景色

我们应该使用`rgba`设置背景颜色。rgba 允许你设置背景色的透明度，前三个数字组成颜色，第四个数字代表透明度。

例如，`rgba(0,0,0,1)`是可以完全看到的黑色。而`rgba(0,0,0,0)`将不会显示任何黑色。

```
.modal {
    position: fixed;z-index:1;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    **background-color: rgba(0, 0, 0, 0.3);**
}
```

## 使模体居中

现在我们应该使用 modal 类中的`**display:flex; justify-content:center; align-items:center**` 让 modal body div 出现在中间

```
.modal {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.3);
    **display: flex;
    justify-content: center;
    align-items: center;**
}
```

## 时间

在模态体中，我们需要一种形式。该表单将是 updateItemModal.vue 文件的子组件，我们将使用**插槽**。

modalForm.vue👇

```
<template>
    <form *enctype*="multipart/form-data">
        <div *class*="form-inputs">
            <input *type*="file" *@change*="p.selectFile" *name*="itempic">
            <input *type*="text" *@change*="disabled = *false*">
            <textarea *placeholder*="description"/>
        </div>
        **<slot *name*="submit"/>
        <slot *name*="validation" *class*="errorBar"/>**
    </form>
</template>
<script>
    *export default* {
        data() {
            *return* {
                p: *this*.$parent,
            }
        }
    }
</script>
```

在创建表单和更新表单中，我们有相同的输入，但是有不同的提交按钮和不同的验证。这就是**槽**的用途。

我们可以在表单组件和模态组件中设置槽，我们可以填充这些槽。所以在 updateItemModal 组件文件中，我们可以有这样的内容:

```
<modal-form>
    <button ***slot*="submit"** *@click.prevent*="update(item)" *:disabled*="disabled">Change Item Details</button>
    <div ***slot*="validation"**>{{validation}}</div>
</modal-form>
```

`**slot="submit"**`允许您填充提交槽，`**slot="validation"**`允许您填充验证槽。

## 将当前数据输入输入

当您通过更改描述等方式更新项目时，查看您正在更改的描述可能会很有用。我们应该将项目数据输入到表单输入中。

updateItemModal 的数据():

```
data() {
    *return* {
        name: *this*.item.name,
        description: *this*.item.description,}
},
```

在表格中使用 v-model

```
<form *enctype*="multipart/form-data">
    <div *class*="form-inputs">
        <input *type*="text" ***v-model*="p.name"** *@change*="disabled = *false*">
        <textarea ***v-model*="p.description"** *placeholder*="description"/>
    </div>
    <slot *name*="submit"/>
    <slot *name*="validation" *class*="errorBar"/>
</form>
```

现在名称和描述应该出现在表单输入中。

## 更新 Modal 的提交方法

这里的提交方式是`**update(item)**`

```
<button *slot*="submit" *@click.prevent*="update(item)" *:disabled*="disabled">Change Item Details</button>
```

它通过提交按钮运行。

我们需要追加表单数据并发布它。

```
update(item) {
    *let* fd = *new* FormData();
    fd.append('itempic', *this*.file);
    fd.append('name', *this*.name);
    fd.append('description', *this*.description);

    axios.post(`api/portfolio/${item.id}`,fd)
},
```

让我们重构一下。我们可以在单独的函数中追加表单数据并返回`**fd**`。

`**this.appendFormData()**` 将代表返回的`**fd**`

```
appendFormData() {
    *let***fd** = *new* FormData();
    fd.append('itempic', *this*.file);
    fd.append('name', *this*.name);
    fd.append('description', *this*.description);
    ***return* fd;**
},
update(item) {
    *let* fd = ***this*.appendFormData();**
    axios.post(`api/portfolio/${item.id}`,fd)
}
```

我们可以将`**this.appendFormData()**`缩短为`**let**`，用 axios.post 发布

```
update(item) {
    *let* **fd** = *this*.appendFormData()**;**
    axios.post(`api/portfolio/${item.id}`,**fd**)
}
```

submit 按钮将被禁用，直到 computed 中的 validation()通过。

## 前端验证(用于更新)

我们需要检查至少有一个字段与检索到的数据不同。

仅当输入与检索到的数据不同时，才将 this.disabled 设置为 true

```
validation(){
    *let* i = *this*.item;
    *if* (*this*.name !== i.name || *this*.description !== i.description) {
        *this*.disabled = *false* } *else* {
        *this*.disabled = *true* }
}
```

较短的版本

```
validation(){
    *let* i = *this*.item;
    *this*.disabled = !(*this*.name !== i.name || *this*.description !== i.description);
}
```

## api 路线

如果用户通过前端验证，按钮将被启用，新数据将被发布。为此，我们需要在 api routes 文件中键入以下内容。

```
Route::*post*('/portfolio/{id}', 'PortfolioController@update');
```

## 投资组合控制器的更新功能

我们来写更新函数吧。

```
*public function* update(Request $request, **$id**)
{
    $file = $request->itempic;
    $hashName = $file->hashName();
    $this->resizeAndStore($file, $hashName);
    $this->**updateDetails**($hashName, **$id**);
}*public function* store(Request $request) {
    $file = $request->itempic;
    $hashName = $file->hashName();
    $this->resizeAndStore($file, $hashName);
    $this->storeDetails($hashName);
}
```

请注意 update 和 store 函数之间的相似之处和不同之处。两个函数都需要 **itempic** 并且都需要**散列 itempic 文件的名称**。

不同之处在于最后，store 函数需要存储新的细节，而 update 函数需要更新已经存在的项目的细节。

我们还没有`**updateDetails**($hashName, **$id**)`功能。让我们创造它

```
*private function* updateDetails($hashName, **$id**) {
    $item = **PortfolioItem::*find*($id);**
    $item->img_path = $hashName;
    $item->description = request()->description;
    $item->name = request()->name;
    *return* $item->save();
}
```

我们可以看到，除了第一行之外，updateDetails 函数和 storeDetails 函数没有太大的区别。

```
*private function* storeDetails($hashName) {
    $item = *new* PortfolioItem();
    $item->img_path = $hashName;
    $item->description = request()->description;
    $item->name = request()->name;
    $item->save();
}
```

我们可以重构这个。

## 重构存储()和更新()

我们可以移动私有函数的第一行来存储、更新和传递`**$item**`

```
*public function* store(Request $request) {
    $file = $request->itempic;
    $hashName = $file->hashName();
    $this->resizeAndStore($file, $hashName);
    **$item = *new* PortfolioItem();**
    $this->storeDetails($hashName, **$item**);
}*private function* storeDetails($hashName, **$item**) {
    $item->img_path = $hashName;
    $item->description = request()->description;
    $item->name = request()->name;
    $item->save();
}
```

现在这个相同的`**storeDetails()**`函数可以用于`**update()**`函数。

```
$file = $request->itempic;
$hashName = $file->hashName();
$this->resizeAndStore($file, $hashName);
**$item = PortfolioItem::*find*($id);**
$this->**storeDetails**($hashName, **$item**);
```

现在可以删除 updateDetails()

## 勾选清单

我们可以从列表中勾掉前端验证。

*   前端验证(用于更新 functionality)☑️
*   后端验证(用于更新功能)
*   更新和删除功能(读取完成)

—

*   重构组合控制器

—

*   带文本的图像(Portfolio.vue)
*   具有更大图像和描述的模型(Portfolio.vue)
*   modal (Portfolio.vue)上的关闭按钮

在下一章，我们将致力于后端验证(更新功能)

[](/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-ten-a669461c51ad) [## Laravel 和 Vue:用 CRUD 管理面板创建作品集网站——第十章

### 更新功能的后端验证

itnext.io](/laravel-and-vue-creating-a-portfolio-website-with-a-crud-admin-panel-chapter-ten-a669461c51ad)
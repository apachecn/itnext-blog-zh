# 根据以下 7 条建议，充分利用你的 Laravel 模型

> 原文：<https://itnext.io/get-the-most-out-of-your-laravel-models-with-these-7-tips-804f0cf863e4?source=collection_archive---------0----------------------->

![](img/0d256a48d53fa9c06188f555402fe2f8.png)

照片由[凯特琳·贝克](https://unsplash.com/@kaitlynbaker?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

## 优化您使用模型的方式

一旦你开始钻研 Laravel 文档，你会发现很多在文档中很少提到的特性。其他功能使用频率较低，您可能不知道这些功能。

模特也没什么不同。如果您查看 Laravel API 文档，您将会惊讶，并且可能会被模型可用的所有属性和方法所淹没。

有许多功能强大且易于使用的特性，但并不是每个人都知道。这就是为什么我们要回顾这 7 个技巧，每个使用 Laravel 的开发人员都应该知道，以充分利用他们的模型。

# 1.自定义转换类型

Laravel 7 中引入了自定义造型类型。在 Laravel 的早期版本中，当涉及到属性的转换时，您会受到很大的限制。你只能转换成 Laravel 提供的默认设置。

尽管有一些实现定制造型的软件包，但是它们有一个主要的缺点。这些包覆盖了 *getAttribute* 和 *setAttribute* 方法，这意味着它们不能与覆盖这些方法的其他包组合。

在 Laravel 7 之前，您是这样使用 casts 属性的:

```
protected $casts = [
    'is_admin' => 'boolean',
    'date_of_birth' => 'date'
];
```

在 Laravel 7 中，您可以选择添加一个定制的造型类型。您可以通过创建一个实现 *CastsAttributes* 接口的定制 cast 类来实现这一点。这个接口强制您实现一个 *get* 和一个 *set* 方法。

*get* 方法负责将数据库中的原始值转换为转换值。 *set* 方法处理将转换值转换成可以存储在数据库中的原始值。

处理 JSON 的自定义转换类型可能是这样的:

```
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class Json implements CastsAttributes
{
    public function get($model, $key, $value, $attributes)
    {
        return json_decode($value, true);
    }

    public function set($model, $key, $value, $attributes)
    {
        return json_encode($value);
    }
}
```

通过使用自定义转换类型的类名，可以将自定义转换类型附加到模型属性。

```
use App\Casts\Json;// More codeprotected $casts = [
    'is_admin' => 'boolean',
    'date_of_birth' => 'date',
    'settings' => Json::class,
];
```

# 2.自定义时间戳列名

默认情况下，一个模型有一个`created_at`和一个`updated_at`列，它们被自动添加到数据库中。如果您在命令行上使用`-m`标志创建一个模型，那么也会为这个模型创建一个迁移。

这个迁移伴随着将这些列添加到数据库的 *timestamps()* 方法。这种方法的缺点是不能自定义列名。

幸运的是，有一种非常简单的方法可以使用常量`CREATED_AT`和`UPDATED_AT`来更改这些列名。

```
<?php

class Blog extends Model
{
    const CREATED_AT = 'created_date';
    const UPDATED_AT = 'last_update';
}
```

为了使这个工作，你必须删除`$table->timestamps()`，并用这两行代码替换它:

```
$table->timestamp('created_date')->nullable();
$table->timestamp('last_update')->nullable();
```

如果您正在为您的模型使用软删除，那么您可以使用常量`DELETED_AT`来定制 deleted at 列名，如下所示:

```
const DELETED_AT = 'deleted_date';
```

# 3.保存而不触发事件

在某些情况下，您可能希望在不触发任何事件的情况下保存模型。使用这个简单而强大的函数，您可以保存任何模型，而无需触发事件。你所要做的就是将下面的函数添加到你的模型类中。

```
public function saveWithoutEvents()
{
    return static::withoutEvents(function() {
        return $this->save();
    });
}
```

这是在不触发事件的情况下保存模型的方式:

```
$blog = Blog::find($id);
$blog->title = "The updated title";
$blog->saveWithoutEvents();
```

# 4.触摸所有权关系

您可能已经在您的模型上使用过*触摸*方法。该方法更新模型的更新时间戳。

```
$blog = Blog::find($id);
$blog->touch();
```

但是您知道也可以更新模型拥有关系的更新时间戳吗？

可以在模型上调用 *touchOwners* 方法来更新模型的所有权关系。让我告诉你这是如何工作的。

假设我们有一个可以有多个评论的博客模型。我们可以在注释模型中定义一个如下所示的关系:

```
public function blog()
{
    return $this->belongsTo(Blog::class);
}
```

由于我们在评论模型上定义了关系，现在我们可以使用 *touchOwners* 方法来更新博客的更新时间戳。

```
$comment = Comment::create([
    // Some data
]);
$comment->touchOwners();
```

例如，如果我们想在添加新评论时更新博客的更新时间戳，这可能会很有用。

# 5.增加

*increment* 方法提供了一种增加字段值的简单方法。

```
CouponCode::find($id)->increment('times_used');
```

默认情况下，该值递增 1。您可以使用 *increment* 方法的第二个参数来更改值的增量。

下面的代码将把 *times_used* 的值增加 10。

```
CouponCode::find($id)->increment('times_used', 10);
```

除了递增指定的列之外，increment 方法还将更新在记录的时间戳更新的*。*

很高兴知道还有一种*递减*方法，其工作方式与*递增*方法完全相同。

```
CouponCode::find($id)->decrement('times_used');
```

# 6.默认关系值

让我们把前面一个例子的注释模型也用于这个例子。假设用户可以在博客上发表评论。如果允许客人对博客文章发表评论，那么评论模型中的*用户*关系将返回 *null* 。

为了防止用户关系返回空值，我们可以使用带有默认值的方法。

```
 public function user()
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest',
    ]);
}
```

在这个例子中，*用户*关系返回一个名为*客人*的用户模型。通过为关系设置默认值，可以帮助移除代码中的条件检查。

这意味着您可以在您的刀片模板文件中执行`{{ $comment->user->name }}`,因为关系总是返回一个模型。如果您没有使用带有默认的方法的*，您将不得不检查注释是否有用户——这将更加冗长。*

```
@if ($comment->user)
    {{ $comment->user->name }}
@else
    Guest
@end
```

# 7.有

*有*的方法大概不需要过多解释。你可能经常使用它。*有*方法检查关系是否存在。继续前面的例子，我们可以使用下面的代码行来检索所有发布了博客的用户:

```
$users = User::has('blog')->get();
```

我们甚至可以扩大这张支票。假设我们只想得到至少有 5 个博客的用户。我们可以通过向 *has* 方法添加一些参数来做到这一点。

```
$users = User::has('blog', '>=', 5)->get();
```

但是你也知道你可以得到所有至少有一个博客和一个评论的用户吗？

您可以使用点符号嵌套 *has* 语句。

```
$users = User::has('blog.comment')->get();
```

# 一点额外的小费

好了，结束这篇文章的最后一个小技巧。我相信你知道*找*的方法。这是一个经常使用的非常基本的方法。 *find* 方法试图返回一个匹配主键的模型。大多数情况下，您会这样使用它:

```
$blog = Blog::find(1);
```

但是你也知道有可能传入一个有多个 id 的数组吗？

```
$blogs = Blog::find([1,2,3]);
```
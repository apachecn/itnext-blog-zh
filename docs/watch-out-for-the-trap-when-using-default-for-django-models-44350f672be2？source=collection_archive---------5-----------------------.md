# 对 Django 模型使用 Default 时要小心陷阱

> 原文：<https://itnext.io/watch-out-for-the-trap-when-using-default-for-django-models-44350f672be2?source=collection_archive---------5----------------------->

![](img/9b91bfaa2f68166a1ead7d789fb22bce.png)

来源:[保罗·唐尼](https://www.flickr.com/photos/psd/)来自 [Flickr](https://www.flickr.com/photos/psd/506497408/)

我今天花了一整天来解决我们代码库中的一个奇怪问题。问题是，无论何时在我们的系统中创建一个“Book”实例，它都会自动从任何地方获得一些默认值。最令人沮丧的是，这个问题从来没有在我的本地盒子上重现。

我们的系统是用 django 构建的，使用 PostgreSQL 作为数据库。在一些代码阅读和删除之后，我把注意力放在了`Book`模型本身上。模型定义是这样的:

```
from django.contrib.postgres.fields import JSONFieldclass Book(models.Model):
    name = models.CharField(max_length=100)
    detail = JSONField(default={})
```

没错吧。我也没有发现任何问题…在开始的时候。我绝望地再次阅读文件，试图找到任何线索，直到我读到这句话:

> 如果你给这个字段一个`**default**`，确保它是一个可调用的，比如`**dict**`(对于一个空的默认值)或者一个返回字典的可调用的(比如一个函数)。不正确地使用`**default={}**`会创建一个可变缺省值，该值在`**JSONField**`的所有实例之间共享。( [src](https://docs.djangoproject.com/en/2.0/ref/contrib/postgres/fields/#django.contrib.postgres.fields.JSONField) )

![](img/bec21b78d84125dae4b45f8be476ba99.png)

`{}`不应该总还一本新字典吗？但是后来我意识到…

这不是 JAVASCRIPT！

嗯……有道理。为了验证这个问题，我创建了一个测试项目。我将省略模型设置部分(它与上面的代码相同),直接进入测试脚本。

```
from django.core.management.base import BaseCommand
from django_default_trap.demo.models import Bookclass Command(BaseCommand): def handle(self, *args, **options):
      book1 = Book(name='Book 1')
      print('book1.detail.author = %s' % book1.detail.get('author'))
      book1.detail['author'] = 'Steven'
      print('book1.detail.author = %s' % book1.detail.get('author'))
      book1.save()
      print('book1 saved.') book2 = Book(name='Book 2')
      print('book2.detail.author = %s' % book2.detail.get('author'))
      book2.save()
```

运行此测试:

```
$ ./manage.py default_test
book1.detail.author = None
book1.detail.author = Steven
book1 saved.
book2.detail.author = Steven
```

不出所料，`book2.detail`不知从哪里得到了一个值`{ "author": "Steven" }`(我们从来没有给`book2.detail`赋值！).数据库查询也证实了这一点:

```
django_default_trap=# select * from demo_book;
 id |  name  |        detail        
----+--------+----------------------
 12 | Book 1 | {"author": "Steven"}
 13 | Book 2 | {"author": "Steven"}
(2 rows)
```

显然，在创建`book1`时，默认值`{}`被使用并通过引用赋给`book1.detail` **，下一行`book1.detail['author']`实际上是将**全局**对象`{}`改为`{'author': 'Steven'}`。这就是价值的来源。**

在测试脚本中，这个问题只影响当前会话(脚本的生命周期)本身，但是在服务器上，这个奇怪的缺省值将一直存在，直到服务器重新启动。那是一个严重的错误！

解决方案:正如文档中提到的，只需使用:

```
detail = JSONField(default=dict)
```

如果你对测试项目感兴趣，请访问我的 [github](https://github.com/charlee/django-default-trap) 。如果这篇文章有帮助，也请为我鼓掌。感谢您的阅读！
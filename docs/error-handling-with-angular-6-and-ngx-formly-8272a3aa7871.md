# Angular 6 和 ngx-formly 的错误处理

> 原文：<https://itnext.io/error-handling-with-angular-6-and-ngx-formly-8272a3aa7871?source=collection_archive---------0----------------------->

![](img/839d2fab6599d91d034d9ea635556a5f.png)

ngx-formly 提供了一种在 Angular 6 中创建表单的简单且更好的方法，但其官方文档中的错误处理部分已经过时。本帖用`ngx-formly`总结错误处理技巧。

在本帖中，我们假设一个表单字段配置如下:

```
fields: FormlyFieldConfig[] = [
    {
        key: 'name',
        type: 'input',
        templateOptions: {
            type: 'text',
            label: 'Name',
        },
    }
];
```

我们使用的软件包版本有:

*   `@angular/core@^6.0.3`
*   `@ngx-formly/core@^4.7.2`，`@ngx-formly/material@^4.7.2`
*   `@angular/material@^6.4.1`

# 前端验证

ngx-formly 有内置的前端验证，非常容易使用。它的官方文件[提到了解决方案](https://formly-js.github.io/ngx-formly/guide/validation)但是似乎过时了，而且字段与实际类型不匹配。

基本上有两个步骤来进行前端验证:

1.  定义验证器，
2.  为验证器定义错误消息。

假设我们要验证的名字不应该超过 100 个字符。首先，添加一个`validators`配置到配置字段:

```
validators: {
    maxlength: ctrl => ctrl.value && ctrl.value <= 100,
},
```

一个`validators` config 是一个对象，其中的键是一个任意的名称，值是一个验证函数。验证功能应该是:

*   参数:`(ctrl: FormControl)`，正在验证的表单控件通过
*   返回值:`boolean`，返回验证是否成功

接下来，添加一个`validation`配置来定义验证失败时的错误消息:

```
validation: {
    messages: {
        maxlength: 'The max length is 100 characters',
    }
},
```

`validation.messages`是为每个验证器定义错误消息的对象。它的键应该是在`validators`中定义的验证器名称，它的值是作为错误消息的`string`。

因此，最终的表单配置应该如下所示:

```
fields: FormlyFieldConfig[] = [
    {
        key: 'name',
        type: 'input',
        templateOptions: {
            type: 'text',
            label: 'Name',
        },
        validators: {
            maxlength: ctrl => ctrl.value && ctrl.value <= 100,
        },
        validation: {
            messages: {
                maxlength: 'The max length is 100 characters',
            }
        },
    }
];
```

一个特殊的内置验证器是`required`。它应该在`templateOptions`中设置为`required: true`，并且需要一个`validation.messages`对象中的`required: 'errmsg'`。因此，要使上述示例成为必需的，您应该编写:

```
fields: FormlyFieldConfig[] = [
    {
        key: 'name',
        type: 'input',
        templateOptions: {
            type: 'text',
            label: 'Name',
            required: true,
        },
        validators: {
            maxlength: ctrl => ctrl.value && ctrl.value <= 100,
            required: 'This field is required.',
        },
        validation: {
            messages: {
                maxlength: 'The max length is 100 characters',
            }
        },
    }
];
```

最后，要在表单无效时禁用提交按钮，应该在提交按钮中使用`[disabled]="!form.valid"`(其中`form`是`ngx-formly`使用的`FormGroup`对象)。

# 后端验证

并非所有的字段标准都可以在前端进行验证，在某些情况下，后端会返回一些错误。

假设后端返回的错误具有以下格式(这是 [django-rest-framework](http://www.django-rest-framework.org/) 的标准格式):

```
{
    "name": [
        "Name contains invalid characters."
    ]
}
```

由于后端不返回错误类型(`name`是字段名)，我们假设后端返回的所有错误都是`other`类型。在前端，在错误处理程序中使用以下代码将错误设置为表单:

```
_.forEach(this.form.controls, (ctrl, name) => {
    if (errors[name]) {
        ctrl.setErrors({ other: errors[name] });
    }
}
```

这里的`_`是[洛达什](https://lodash.com/)图书馆。我们通过调用`_.forEach(this.form.controls)`迭代所有的表单控件，然后为服务器返回错误的表单控件调用`ctrl.setErrors({ error_type: error_message })`。

接下来，我们需要让 formly 知道如何显示`other`类型的错误信息。就像我们对`maxlength`类型错误所做的一样，我们向`validation.messages`添加另一个字段:

```
validation: {
    messages: {
        maxlength: 'The max length is 100 characters',
        **other: (err, field) => err,**
    }
},
```

为了显示服务器返回的内容，我们使用了一个用于类型的函数，而不是像在`maxlength`中那样使用固定的字符串。函数签名是:

*   参数:`(error_message, error_field)`。`error_message`是`ctrl.setErrors({ error_type: error_message })`传递的消息，`error_field`是对表单控件的引用。
*   返回值:一个字符串，将显示在字段下方的实际错误消息。

现在后端错误消息可以正确显示了。

# 全局错误消息

到目前为止，我们只在每个字段中设置错误消息，当多个字段具有相同的错误消息配置时，这可能会很麻烦。我们可以将常见的错误信息移动到 formly 的全局配置中，这样就不需要逐个字段地设置它们。

在您的`app.module.ts`(即导入`FormlyModule`的地方)中添加以下代码，这样就不需要为每个字段设置错误信息:

```
FormlyModule.forRoot({
    validationMessages: [
        { name: 'required', message: 'This field is required' },
        { name: 'other', message: (err, field) => err },
    ],
}),
```

就是这样！

这篇文章来自我自己的经历，希望它能帮助那些也在使用 formly 并且因为缺少文档而苦苦挣扎的人。喜欢就给我鼓掌。
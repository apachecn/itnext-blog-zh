# 处理 ReactJS 中的表单数据

> 原文：<https://itnext.io/easy-form-handling-in-react-be5490a1284?source=collection_archive---------1----------------------->

![](img/e146478a892257187c53156666559e9c.png)

我将分享如何在 React 中轻松处理表单数据的技巧。

您需要 3 个步骤来轻松管理表单数据。

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Feasy-form-handling-in-react-be5490a1284) 上分享这篇文章

# 步骤 1:初始化状态

假设您有一个带有 ***电影*** & ***价格*** 字段的表单，将它添加到您的初始状态:

```
constructor () {
  this.state = {
    movie: '',
    price: ''
  }
}
```

> 这将是你的数据来源。在 JQuery 中，您必须从 html 输入字段中获取数据。在 React 中，您现在可以在这个州获得您的数据。

# 步骤 2:将[name]属性添加到字段中

将[ `name` ]属性添加到 ***电影*** & ***价格*** 字段。

对于 ***电影*** 字段，将`movie`作为`name]`属性的值。

对于 ***价格*** 字段，将`price`作为`name]`属性的值。

```
<input name='movie' />
<input name='price' />
```

# 步骤 3:创建一个函数来更新您的状态

创建一个函数，姑且称之为`handleChange`。

```
handleChange (event) {
  this.setState( [event.target.name]: event.target.value )
}
```

表单字段应该用`onChange`事件调用`handleChange`函数。

```
<input name=’movie’ onChange={event => this.handleChange(event)} />
<input name='price' onChange={event => this.handleChange(event)} />
```

> 那就这样吧！现在，每当您更改表单输入字段的值时，您的状态都会更新。

通过这三个步骤，无论你在字段中做了什么改变，它都会自动更新你的状态。特别是如果你的表单中有超过 10 个字段的话，这是非常方便的。)所以当你准备好表单提交的时候，数据就处于就绪状态了。不需要从表单的每个字段获取它。

![](img/9c6ea2d4fc76418637f037229e19c6cf.png)

> 您可以按住“鼓掌”按钮来留下许多掌声！\m/谢谢！

![](img/f0d7d42ff86cc64c3df42345b5d52bfd.png)
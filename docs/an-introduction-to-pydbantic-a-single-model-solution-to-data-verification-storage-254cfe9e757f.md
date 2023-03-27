# Pydbantic 简介——数据验证和存储的单一模型解决方案

> 原文：<https://itnext.io/an-introduction-to-pydbantic-a-single-model-solution-to-data-verification-storage-254cfe9e757f?source=collection_archive---------1----------------------->

![](img/0aeac1aad5b3955a5d190bda879f5132.png)

Pydbantic 从 [**pydantic**](https://pydantic-docs.helpmanual.io/) 继承了它的名字，这是一个“使用 Python 类型提示进行数据解析和验证”的库。

简而言之，pydantic 提供了一个框架来验证接口之间的输入，以确保满足正确的输入数据(类型、结构、必需、可选)，从而消除了添加逻辑来捕捉和验证错误输入的需要。

Pydantic 负责确保数据类型/格式的正确性，但是这种数据在创建后并不能永久保存。开发人员的一项常见任务是以某种方式将这些模型转换成“数据库原生”格式以便持久化。

Pydantic 为这种序列化提供了许多有用的方法。但是，大多数 pydantic 模型并不是平等的。这意味着对于每个模型，通常需要单独的逻辑来将给定的字段转换为“数据库本地”列类型。因此，每当数据被序列化时，都必须再次反序列化才能再次使用。

数据存储在哪里以及如何存储，通常与编写和管理序列化逻辑一样令人望而生畏。python 生态系统提供了许多选择，通常没有“错误的选项”，但是直到最近，每个选项都需要单独的数据验证模型&数据库表。

# 创建模型

理解和欣赏“单一模型”模型的最好方法，就是看它的实际应用。下面是一个简单的模型，它映射了公司可能存储的关于员工的一些非常基本的信息，我们将继续对其进行扩展:

```
# Application Structure
├── app.py
└── models.py
```

这个模型可以马上使用，就像任何 pydantic 模型一样，开始看到它的潜力。

```
$ python app.pyid='0e2eeddd-bfa2-4e81-9d38-87b098039b3d' salary=20000.0 is_employed=True date_employed='2021-11-15T09:21:02.358015'
```

到目前为止，应用程序只创建了一个 Employee 实例，只提供了`salary`和`is_employed`。其他字段在实例化时使用提供的“零参数”函数生成。这个实例可以像任何其他 pydantic 模型一样使用，带有一些可用于持久性的附加方法(如果插入到数据库中)。

# 保存模型

```
├── app.py
├── db.py
├── models.py
```

`Employee`模型通过首先调用`Database.create(`工厂方法连接到数据库，工厂方法将模型连接到新的或现有的数据库。如果是新的，将创建该表，如果是现有的，将检查该表是否有修改&如果需要，将迁移该表。

在`pydbantic`、`.insert()`、`.save()`和`.create(`中有 3 种保存模型的有效方法，上面演示了后两种方法。

# **查询模型**

```
├── app.py
├── company.db
├── db.py
├── models.py
```

```
$ python app.py[
    Employee(id='0877c661-0d53-4435-a800-425b64c08c43', salary=100000.0, is_employed=True, date_employed='2021-11-15T09:37:51.105720'), 
    Employee(id='67c476d9-3830-434a-9dfe-8ad3cc914129', salary=100000.0, is_employed=True, date_employed='2021-11-15T09:37:51.118808')
]
```

## 主关键字

```
$ python app.py
employee is id='0877c661-0d53-4435-a800-425b64c08c43' salary=100000.0 is_employed=True date_employed='2021-11-15T09:37:51.105720'
```

**过滤**

```
$ python app.py[
    Employee(id='0877c661-0d53-4435-a800-425b64c08c43', salary=100000.0, is_employed=True, date_employed='2021-11-15T09:37:51.105720'), 
    Employee(id='67c476d9-3830-434a-9dfe-8ad3cc914129', salary=100000.0, is_employed=True, date_employed='2021-11-15T09:37:51.118808')
]
```

# 更新模型

```
$ python app.py[
    Employee(id='0877c661-0d53-4435-a800-425b64c08c43', salary=102000.0, is_employed=True, date_employed='2021-11-15T09:37:51.105720'), 
    Employee(id='67c476d9-3830-434a-9dfe-8ad3cc914129', salary=102000.0, is_employed=True, date_employed='2021-11-15T09:37:51.118808')
] 
```

通过模型实例的`.save()`或`.update()`方法可以更新模型。

# 从模型中删除数据

```
$ python app.py[]
```

就像更新一样，从一个模型中删除数据需要通过一个模型的实例通过`.delete()`来完成。

# 模型关系

这里定义了一个名为`Positions`的新`DataBaseModel`，它通过一个新字段`position`与`Employees`相关联。

请特别注意`position`的默认值，如果在创建员工时未指定，将使用该默认值**，如果字段尚不存在，则使用**进行迁移。

```
$ python app.py11-15 10:23 pydb         WARNING  Migrations may be required. Will attempt in order: [<class 'models.Employee'>]11-15 10:23 pydb         WARNING  Migration Required: ['New Column: position'][
    Employee(id='c61e1114-76fd-4356-a167-ffc802c7ad93', salary=100000.0, is_employed=True, date_employed='2021-11-15T11:23:03.047574', position=Positions(name='Manager', department='HR'))
]
[    
    Positions(name='Manager', department='HR')
]
```

在`Employees.create()`创建时，`Positions(name=’Manager’, department=’HR’)`被创建，因为还没有名为`Manager`的`Positions`对象存在。

还要注意应用程序启动前的额外日志。`Migrations may be required. Will attempt in order: [<class 'models.Employee'>]`该日志和以下日志指示检测到的对模型`Employee`的更改，以及将进行迁移的通知。

聪明的人可能会注意到`Positions`从未直接插入`db.py`。`DatabaseModel`具有相关`DatabaseModel`的对象将确保相关的表也在运行时插入数据库。`DatabaseModel`如果定义被另一个已经输入到`Database`的`DatabaseModel`取消引用，则需要手动输入到`Database`创建中。

**相关车型变更**

在上述模型示例中，新的`BaseModel`被定义为表示新的`DataBaseModel` `Department`位置。现有`Positions`模型的`department`字段被替换为`positions_department`，默认值为`Department(name=’HR’, company=’FOOBAR’)`。

```
$ python app.py11-15 12:38 pydb         WARNING  Migrations may be required. Will attempt in order: [<class 'models.Positions'>]11-15 12:38 pydb         WARNING  Migration Required: ['New Column: position_department', 'Deleted Column: department'][
    Employee(id='c61e1114-76fd-4356-a167-ffc802c7ad93', salary=100000.0, is_employed=True, date_employed='2021-11-15T11:23:03.047574', position=Positions(name='Manager', position_department=Department(name='HR', company='FOOBAR', location=Coordinates(latitude=52.299387, longitude=4.976949))))
][
    Positions(name='Manager', position_department=Department(name='HR', company='FOOBAR', location=Coordinates(latitude=52.299387, longitude=4.976949)))
]
[
    Department(name='HR', company='FOOBAR', location=Coordinates(latitude=52.299387, longitude=4.976949))
]
```

在运行时，新模型被导入并添加到`Database`，迁移需求被检测并执行。与此同时，`Employee`和`Positions`模型的现有对象继续引入相关的模型变更。

## 删除相关数据

在首次更新/删除相关对象之前，从相关`DataBaseModels`中删除对象时，应特别考虑模型。

在上面的例子中，所有的`Positions`对象都被删除了，而对这些被删除对象的引用存在于`Employees`对象中。查询`Employees`的结果缺少`Positions`数据，即`None`。`None`然而，类型不是模型`Employee`属性`position`的允许类型。

对此的简单解决方法是允许`NoneType`。

这里，`Employee`的属性`position`用`Union[Positions, None]`标注，因此允许属性为 None 值。使用现有的`Employee`对象实例，如果需要，可以分配和更新新的`Positions`对象。

# **使用 Pydbantic 进行缓存**

只需将`cache_enabled=True`添加到`Database.create()`以及定义的`redis_url`即可启用缓存。

当启用缓存时，db 读取响应被缓存&在更改时自动失效。

# **使用 Pydbantic 进行测试**

使用 Pydbantic 测试很容易，只需将`testing=True`添加到`database.create()`fixture，链接的数据库表&缓存将在启动前自动清空。

# 帮助 Pydbantic 变得更好！

*   喜欢图书馆吗？星叉 [me](https://github.com/codemation/pydbantic) ！
*   有想法，改进，Bug，提交问题[这里](https://github.com/codemation/pydbantic/issues)！
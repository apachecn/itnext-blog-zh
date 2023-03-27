# Python & GraphQL。提示、技巧和性能改进。

> 原文：<https://itnext.io/python-graphql-tips-tricks-and-performance-improvements-beede1f4adb6?source=collection_archive---------1----------------------->

![](img/164238f897d81991202189a72ce794cd.png)

最近我用 GraphQL 完成了另一个后端，但现在是在 Python 上。在这篇文章中，我想告诉你我所面临的所有困难和可能影响表现的狭窄地方。

技术栈:[石墨烯](https://graphene-python.org/) + [烧瓶](http://flask.pocoo.org/)和 [sqlalchemy](https://github.com/graphql-python/graphene-sqlalchemy) 集成。这里有一段`requirements.txt`:

```
graphene
graphene_sqlalchemy
flask
flask-graphql
flask-sqlalchemy
flask-cors
injector
flask-injector
```

这允许我将数据库实体直接映射到 GraphQL。

看起来是这样的:

模型。

```
class Color(db.Model):
  """color table"""
  __tablename__ = 'colors'

  color_id = Column(BigInteger().with_variant(sqlite.INTEGER(), 'sqlite'), primary_key=True)
  color_name = Column(String(50), nullable=False)
  color_r = Column(SmallInteger)
  color_g = Column(SmallInteger)
  color_b = Column(SmallInteger)
```

节点。

```
class ColorNode(SQLAlchemyObjectType):
  class Meta:
    model = colours.Color
    interfaces = (relay.Node,)

  color_id = graphene.Field(BigInt)
```

一切都简单美好。

但是有什么问题呢？

# 烧瓶环境。

在写这篇文章的时候，我无法将我的上下文发送到 GraphQL。

```
app.add_url_rule('/graphql',
                 view_func=GraphQLView.as_view('graphql',
                 schema=schema.schema,
                 graphiql=True,
                 context_value={'session': db.session})
                 )
```

这东西对我不起作用，因为视图在 [flask-graphql](https://github.com/graphql-python/flask-graphql) 集成中被 flask request 取代。

也许现在这个问题已经解决了，但是我必须子类化 GrqphQLView 来保存上下文:

```
class ContexedView(GraphQLView):
  context_value = None

  def get_context(self):
    context = super().get_context()
    if self.context_value:
      for k, v in self.context_value.items():
        setattr(context, k, v)
    return context
```

# CORS 支持

总是我忘记补充的东西:)

对于 Python Flask，只需在需求中添加 [flask-cors](https://flask-cors.readthedocs.io/en/latest/) ，并通过`CORS(app)`在 create_app 方法中设置它。仅此而已。

# Bigint 类型

我必须创建自己的 bigint 类型，因为我在数据库中使用它作为一些列的主键。当我尝试发送 int 类型时，出现了石墨烯错误。

```
class BigInt(Scalar):
  @staticmethod
  def serialize(num):
    return num

  @staticmethod
  def parse_literal(node):
    if isinstance(node, ast.StringValue) or isinstance(node, ast.IntValue):
      return int(node.value)

  @staticmethod
  def parse_value(value):
    return int(value)
```

# 复合主键

此外，graphene_sqlalchemy 不支持开箱即用的复合主键。我有一个带有(Int，Int，Date)主键的表。为了通过 Relay 的节点接口按 id 进行解析，我必须覆盖 get_node 方法:

```
@classmethod
def get_node(cls, info, id):
  import datetime
  return super().get_node(info, eval(id))
```

`datetime` import 和`eval`在这里非常重要，因为没有它们，日期字段将只是一个字符串，在查询数据库时什么都不会起作用。

# 授权突变

为查询授权真的很容易，我所需要的就是添加查看器对象并编写`get_token`和`get_by_token`方法，就像我以前在 java 中多次做的那样。

但是突变被称为绕过`Viewer`，这是 GraphQL 的天性。

我不想在每个变异的头中添加授权代码，因为这会导致代码重复，这有点危险，因为我可能会因为忘记添加这些代码而创建一个后门。

所以我有子类突变和重新实现它是这样的`mutate_and_get_payload`:

```
class AuthorizedMutation(relay.ClientIDMutation):
  class Meta:
    abstract = True

  @classmethod
  @abstractmethod
  def mutate_authorized(cls, root, info, **kwargs):
    pass

  @classmethod
  def mutate_and_get_payload(cls, root, info, **kwargs):
    # authorize user using info.context.headers.get('Authorization')
    return cls.mutate_authorized(root, info, **kwargs)
```

我的所有突变都继承了`AuthorizedMutation`的子类，并在`mutate_authorized`中实现了它们的业务逻辑。仅当用户被授权时才调用它。

# 可排序和可过滤的连接

为了让我的数据通过 query in connection 自动排序(将排序选项添加到模式中)，我必须子类化 relay 的 connection 并实现 get_query 方法(它在 graphene_sqlalchemy 中被调用)。

```
class SortedRelayConnection(relay.Connection):
  class Meta:
    abstract = True

  @classmethod
  def get_query(cls, info, **kwargs):
    return SQLAlchemyConnectionField.get_query(cls._meta.node._meta.model, info, **kwargs)
```

然后我决定在每个字段上添加动态过滤。也具有扩展模式。

开箱即用的石墨烯做不到这一点，所以我不得不再加一个 [PR](https://github.com/graphql-python/graphene-sqlalchemy/pull/164) 和子类连接一次:

```
class FilteredRelayConnection(relay.Connection):
  class Meta:
    abstract = True

  @classmethod
  def get_query(cls, info, **kwargs):
    return FilterableConnectionField.get_query(cls._meta.node._meta.model, info, **kwargs)
```

《T2》是在哪里被引入 PR 的。

# 哨兵中间件

我们使用 sentry 作为错误通知系统，很难让它与石墨烯一起工作。Sentry 有很好的烧瓶集成，但石墨烯的问题是——它会吞掉异常，并在响应中将它们作为错误返回。

我不得不使用我自己的中间件:

```
class SentryMiddleware(object):

  def __init__(self, sentry) -> None:
    self.sentry = sentry

  def resolve(self, next, root, info, **args):
    promise = next(root, info, **args)
    if promise.is_rejected:
      promise.catch(self.log_and_return)
    return promise

  def log_and_return(self, e):
    try:
      raise e
    except Exception:
      traceback.print_exc()
      if self.sentry.is_configured:
      if not issubclass(type(e), NotImportantUserError):
        self.sentry.captureException()
    return e
```

它在 GraphQL 路由创建中注册:

```
app.add_url_rule('/graphql',
                 view_func=ContexedView.as_view('graphql',
                 schema=schema.schema,
                 graphiql=True,
                 context_value={'session': db.session},
                 middleware=[SentryMiddleware(sentry)]
                )
```

# 低绩效与关系

一切都很好，测试是绿色的，我很高兴，直到我的应用程序进入开发环境，有了大量的数据。一切都超级慢。

问题出在 sqlalchemy 的关系上。他们因[默认](https://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html)而懒惰。

意思是——如果你有一个有 3 个关系的图:主人->宠物->食物，并且查询它们，第一个查询将接收所有主人(`select * from masters``)。你收到了 20 个。然后，对于每个主节点，将会有查询(`select * from pets where master_id = ?`)。20 个查询。最后，基于宠物返回的 N 个食物查询。

我在这里的建议是——如果你有复杂的关系和大量的数据(我正在为大数据世界写后端)，你必须让所有的关系都变得热切。查询本身会更难，但它只有一个，大大减少了响应时间。

# 自定义查询的性能改进

在我让我的关键关系变得热切(不是所有关系，我必须研究前端应用程序，以了解他们查询什么以及如何查询)之后，一切都工作得更快了，但还不够。我看着生成的查询，有点害怕——它们太可怕了！我必须为一些节点编写自己的优化查询。

例如，如果我有一个有几个`OrderColorDistributions`的`PlanMonthly`实体，每个实体有一个`Order`。

我可以使用子查询来限制数据(记住，我正在为大数据编写后端)，并用现有数据填充关系(无论如何，我在查询中有这些数据，所以没有必要使用 ORM 生成的急切连接)。这将有助于请求。

步骤:

1.  标记子查询`with_labels=True`

2.使用 root(对于此请求)实体作为返回实体:

```
Order.query \
  .filter(<low level filtering here>) \
  .join(<join another table, which you can use later>) \
  .join(ocr_query, Order.order_id == ocr_query.c.order_color_distribution_order_id) \
  .join(date_limit_query,
        and_(ocr_query.c.order_color_distribution_color_id == date_limit_query.c.plans_monthly_color_id,
             ocr_query.c.order_color_distribution_date == date_limit_query.c.plans_monthly_date,
             <another table joined previously> == date_limit_query.c.plans_monthly_group_id))
```

3.在所有一级关系上使用 contains_eager。

```
query = query.options(contains_eager(Order.color_distributions, alias=ocr_query))
```

4.如果你有第二层关系(`Order`->-`OrderColorDistribution`->-`PlanMonthly`)链`contains_eager`:

```
query = query.options(contains_eager(Order.color_distributions, alias=ocr_query)
             .contains_eager(OrderColorDistribution.plan, alias=date_limit_query))
```

# 减少对数据库的调用次数

除了数据呈现层，我还有我的服务层，它对 GraphQL 一无所知。我不打算在这里介绍它，因为我不喜欢高耦合。

但是每项服务都需要提取几个月的数据。为了只使用一次所有的数据并在所有的服务中使用它们，我使用了带有`@request`作用域的 injector。记住这个作用域，它是你在 GraphQL 中的朋友。

它像单例一样工作，但是只在对`/graphql`的一个请求中。在我的连接中，我只是用通过 GraphQL 查询找到的计划填充它(包括来自前端的所有自定义过滤器和范围):

```
app.injector.get(FutureMonthCache).set_months(found)
```

然后在所有需要访问这些数据的服务中，我只使用这个缓存:

```
@inject
def __init__(self,
             prediction_service: PredictionService,
             price_calculator: PriceCalculator,
             future_month_cache: FutureMonthCache) -> None:
  super().__init__(future_month_cache)
  self._prediction_service = prediction_service
  self._price_calculator = price_calculator
```

另一件好事是——我所有的服务，操作数据和形成请求，也有`@request`作用域，所以我不需要计算每个月的预测。我从缓存中取出它们，进行一次查询并存储结果。此外，一个服务可以依赖其他服务的计算数据。请求范围在这里很有帮助，因为它允许我只计算所有数据一次。

在节点端，我通过解析器调用我的请求范围服务:

```
def resolve_predicted_pieces(self, _info):
  return app.injector.get(PredictionCalculator).get_original_future_value(self)
```

只有在 GraphQL 查询中指定了 predicted_pieces 时，它才允许我运行繁重的计算。

# 总结

这就是我所面临的所有困难。我没有尝试过 websocket 订阅，但是根据我所了解的情况，我可以说 Python 的 GraphQL 比 Java 的更灵活。因为 Python 本身的灵活性。但是如果我打算在高负载后端工作，我宁愿不使用 GraphQL，因为它更难优化。
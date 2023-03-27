# 理解 Ruby on Rails 中的委托

> 原文：<https://itnext.io/understanding-delegate-in-ruby-on-rails-i-wish-i-knew-before-5edd341bad47?source=collection_archive---------0----------------------->

在本文中，我的重点是解释 Ruby on Rails 的`**delegate**`方法——它是什么，为什么，以及如何用实际例子来说明。

![](img/b4b04660eecf546285ee482aa999706e.png)

https://www.robcarol.com/effectively-delegate/

# 什么？

根据 [API 文档](https://apidock.com/rails/Module/delegatehttps://apidock.com/rails/Module/delegate)【1】—it "*提供了一个* [*委托*](https://apidock.com/rails/Module/delegate) *类方法，可以轻松地将包含对象的公共方法公开为自己的方法。”*方法的签名如下—

`**delegate**(*methods, to: nil, prefix: nil, allow_nil: nil)` *公共*

如果使用，这个委托类方法将把属于由**指定的目标对象的一组***方法**暴露给。**

我将通过一些例子来解释其余的论点。

# **为什么？**

[**德米特里定律**](https://en.wikipedia.org/wiki/Law_of_Demeter)【2】或**最少知识原则**是一个软件设计指南，根据维基百科，它被总结为—

*   每个单元应该只对其他单元有有限的了解:只有与当前单元“密切”相关的单元。
*   每个单位应该只和自己的朋友说话；不要和陌生人说话。
*   只和你最亲近的朋友说话。

Rails 中的委托方法通过只暴露包含对象的必要方法来帮助执行这一规则，从而使它们更容易被访问。

# 如何？

现在，对于“如何做”部分，我将通过一组示例进行讲解。大多数例子都是从书[3]——“拿我的钱:在网上接受支付”中收集的，但当然是为了本文的目的而编辑的。

假设我们有两个由 has_many 关联关联的 ActiveRecord 模型，如下所示—

```
class Event < ActiveRecord::Base
  has_many :performances
endclass Performance < ActiveRecord::Base
  belongs_to :event
end
```

我将启动 **rails 控制台**并检查这两个模型的成员—

> [3]撬(主)>事件。**列名**
> = > ["id "，"名称"，"描述"，"图像 url "，"创建时间"，"更新时间"]
> 
> [4]撬(主)>性能。**column _ names**
> =>[" id "，" event_id "，" start_time "，" end_time "，" created_at "，" updated_at"]

现在要从**性能**对象**、**中访问一个**事件**对象的名称和描述，我们通常可以这样做

> 性能=性能.查找(459352183)
> 
> [8]撬(主)>性能。 **event.name**
> = >"漂白剂流浪者"
> 
> [9]撬(主)>性能。**event . description**
> =>“一部关于露天看台流浪汉的戏”

因此，我们在这里浏览**:事件**关联。

但是如果我们使用**委托**方法，我们可以更方便地公开**名称**和**描述**方法。让我们修改我们的**性能**模型来做到这一点。

```
class Performance < ApplicationRecord
  belongs_to :event

  delegate :name, :description, to: :event
end
```

现在，我已经暴露了**名称**和**描述**事件**对象的**方法，好像它们属于**性能**对象。从 rails 控制台，现在我们可以访问**事件的**名称和描述，如下所示—

> [10]pry(main)> performance . name
> =>"漂白剂 Bums "
> 
> 【11】撬(主)>性能描述
> = >《一出关于露天看台流浪汉的戏》

我们不再需要通过**:事件**关联。

但是如果**性能**模型本身有一个**名为**的方法呢？这就是**前缀**的作用。那样的话，我可以修改**的性能**模型——

```
class Performance < ApplicationRecord 
  belongs_to :event

  delegate :name, :description, to: :event, **prefix: true** **def name
    "performance"
  end**
end
```

如果我们指定**前缀:true，**我们可以访问**性能的**名称和**事件的**名称，如下所示—

> [21]撬(主)>性能。**名称**
> = >【性能】
> 
> [22]撬(主)>性能**。event_name**
> = >"漂白剂流浪者"

我可以进一步指定我们想要的前缀，如下所示—

```
class Performance < ApplicationRecord
  belongs_to :event

  delegate :name, :description, to: :event, **prefix: :show**
end
```

通过指定前缀，我们可以访问**事件**的名称和描述，如下所示—

> [25]撬(主)>性能。**显示**_ 名称
> = >“漂白布”
> 
> [26]撬(主)>性能。**秀**_ 描述
> = >“一部关于露天看台流浪汉的戏”

如果由**到**指定的对象是 **nil** ，它将引发一个异常，如下所示—

```
class Performance < ApplicationRecord

  belongs_to :event

  delegate :name, :description, to: :event
end
```

> [33]pry(main)> performance . new . event
> =>**nil**
> 
> [34]pry(main)> Performance . new . name
> **Module::delegateion error**:Performance # name 委托给 event.name，但 event 为零:# < Performance id: nil，event_id: nil，start_time: nil，end_time: nil，created_at: nil，updated_at: nil >

我收到了一个**模块::DelegationError** 。为了抑制这个错误并得到一个 **nil** 响应，我可以使用下面的 **allow_nil** 参数—

```
class Performance < ApplicationRecord

  belongs_to :event

  delegate :name, :description, to: :event, **allow_nil: true**
end
```

> [36]pry(main)> performance . new . name
> =>**nil**

顺便提一下，如果接收对象是 **not nil** 但是方法不存在，我们将收到如下的 **NoMethodError**

```
class Performance < ApplicationRecord

  belongs_to :event

  delegate :name, :description, **:time**, to: :event, allow_nil: true
end
```

[43]撬(主)>性能。**时间**
**NoMethodError:未定义的# <事件的方法“时间”:0x007fd776780730 >**

# 一个更实际的例子

下面的节选也来自这本书[3] —

```
class StripeToken

  attr_accessor :credit_card_number, :expiration_month, :expiration_year, :cvc

  def initialize(credit_card_number:, expiration_month:, expiration_year:, cvc:)
    @credit_card_number = credit_card_number
    @expiration_month = expiration_month
    @expiration_year = expiration_year
    @cvc = cvc
  end

  def token
    @token ||= Stripe::Token.create(
      card: {
        number: credit_card_number, exp_month: expiration_month,
        exp_year: expiration_year, cvc: cvc})
  end

  delegate :id, to: :token

  def to_s
    "STRIPE TOKEN: #{id}"
  end

  def inspect
    "STRIPE TOKEN #{id}"
  end
end
```

在这里，我试图创建一个条带令牌对象。delegate 命令将 **id** 转换为 **token.id** 。让我们在 rails 控制台上体验一下—

> [102]pry(main)> stripe token . new(credit _ card _ number:" 424242424242 "，expiration_month: 12，expiration_year: 2020，cvc: 111) **。id**
> =>" Tok _ 1 gmrobbn 4 gtwm 2 epi 6 ody kka "

id 委托提供了一种直接访问令牌 id 的好方法。从上面的代码可以看出，我可以很容易地使用 **id** 委托来定义方便的 **to_s** 和 **inspect** 方法。从 rails 控制台—

> [103]pry(main)> stripe token . new(credit _ card _ number:" 424242424242 "，expiration_month: 12，expiration_year: 2020，cvc: 111)。**to _ s**
> =>" STRIPE TOKEN:Tok _ 1 gmrquebn 4 gtwm 2 epbdehwfud "
> 
> [104]pry(main)> stripe token . new(credit _ card _ number:" 424242424242 "，expiration_month: 12，expiration_year: 2020，cvc: 111)。**inspect**
> =>" STRIPE Tok _ 1 gmrqkbn 4 gt WM 2 epu qbcp 55 o "

在本文中，我试图阐明 Rail 的**委托**方法，以及读者如何将它付诸实际使用。我希望至少对某些人来说是愉快的:)

*更多详细和深入的未来技术帖子请关注我这里或* [*推特*](https://twitter.com/meraj_enigma) *。*

# 参考资料:

1.  [https://API dock . com/rails/Module/delegate https://API dock . com/rails/Module/delegate](https://apidock.com/rails/Module/delegate)
2.  [http://en.wikipedia.org/wiki/Law_of_Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter)
3.  [https://www . Amazon . com/Take-My-Money-Accepting-Payments/DP/1680501992](https://www.amazon.com/Take-My-Money-Accepting-Payments/dp/1680501992)
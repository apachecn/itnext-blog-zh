# ActiveInteractor

> 原文：<https://itnext.io/activeinteractor-8557c0dc78db?source=collection_archive---------6----------------------->

![](img/fc805285bc0171916ff5741c0bff5b2b.png)

这个周末，我发布了 [ActiveInteractor](https://rubygems.org/gems/activeinteractor/versions/1.0.0) 的 1.0.0 版本，这是一个基于 [interactor](https://github.com/collectiveidea/interactor) gem 的带有 [ActiveModel::Validations](https://api.rubyonrails.org/classes/ActiveModel/Validations.html) 的 Ruby 的[命令模式](https://en.wikipedia.org/wiki/Command_pattern)的实现，具有对属性、回调、验证和线程安全性能方法的丰富支持。

我想复习一下宝石的一些基本用法。交互器是一个简单的、单一用途的服务对象。交互器可以用来减少控制器、工人和模型的责任，并封装应用程序的业务逻辑。每个交互器代表应用程序做的一件事。

ActiveInteractor 的主要组件称为 Interactor，每个 interactor 都有自己的不可变上下文，其中包含 interactor 完成工作所需的一切。当一个交互体完成它的单一目的时，它会影响它给定的上下文。ActiveInteractor 内置了两种交互器:基本交互器和组织器。基本交互器是一个从`ActiveInteractor::Base`继承的类，它定义了一个`#perform`方法。

# 基本用法:

大多数时候，你的应用程序会从它的控制器中使用它的交互器。以下控制器:

```
class SessionsController < ApplicationController
  def create
    if user = User.authenticate(session_params[:email], session_params[:password])
      session[:user_token] = user.secret_token
      redirect_to user
    else
      flash.now[:message] = "Please try again."
      render :new
    end
  end

  private

  def session_params
    params.require(:session).permit(:email, :password)
  end
end
```

可以重构为:

```
# app/interactors/authenticate_user_context.rb
class AuthenticateUserContext < ActiveInteractor::Context::Base
  attributes :email, :password, :user, :token
  validates :email, presence: true,
                    format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :password, presence: true
  validates :user, presence: true, on: :called
end# app/interactors/authenticate_user.rb
class AuthenticateUser < ActiveInteractor::Base
  def perform
    context.user = User.authenticate(
      context.email,
      context.password
    )
    context.token = context.user.secret_token
  end
end# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def create
    result = AuthenticateUser.perform(session_params)

    if result.success?
      session[:user_token] = result.token
      redirect_to result.user
    else
      flash.now[:message] = t(result.errors.full_messages)
      render :new
    end
  end

  private

  def session_params
    params.require(:session).permit(:email, :password)
  end
end
```

# 组织者:

组织者是基本交互者的一个重要变体。它的唯一目的是运行其他交互器。

```
class CreateOrder < ActiveInteractor::Base
  def perform
    ...
  end
end

class ChargeCard < ActiveInteractor::Base
  def perform
    ...
  end
end

class SendThankYou < ActiveInteractor::Base
  def perform
    ...
  end
end

class PlaceOrder < ActiveInteractor::Organizer::Base

  organize :create_order, :charge_card, :send_thank_you
end
```

在控制器中，您可以像运行任何其他交互器一样运行`PlaceOrder`管理器:

```
class OrdersController < ApplicationController
  def create
    result = PlaceOrder.perform(order_params: order_params)

    if result.success?
      redirect_to result.order
    else
      @order = result.order
      render :new
    end
  end

  private

  def order_params
    params.require(:order).permit!
  end
end
```

组织者将其上下文传递给它所组织的交互者，一次一个，按顺序传递。每个交互器都可以在上下文传递给下一个交互器之前改变上下文。

## 有条件地组织互动者

我们还可以通过向`.organize`方法传递一个块来向我们的组织者添加条件语句:

```
class PlaceOrder < ActiveInteractor::Organizer::Base
  organize do
    add :create_order, if :user_registered?
    add :charge_card, if: -> { context.order }
    add :send_thank_you, if: -> { context.order }
  end

  private

  def user_registered?
    context.user&.registered?
  end
end
```

# 使用轨道

如果您正在使用 rails 项目，ActiveInteractor 附带了一些有用的生成器来帮助加快开发速度。您应该首先使用以下命令运行安装生成器:

```
rails generate active_interactor:install
```

在某些情况下，您可能想要使用一个`ActiveRecord`模型作为交互器的上下文。您可以通过在任何`ActiveRecord`模型上调用`.acts_as_context`方法来实现这一点，然后简单地在您的交互器或组织器上调用`.contextualize_with`方法来将它指向适当的类。

```
# app/models/user.rb
class User < ApplicationRecord
  acts_as_context
end

# app/interactors/create_user.rb
class CreateUser < ApplicationInteractor
  contextualize_with :user

  def perform
    context.email&.downcase!
    context.save
  end
end

CreateUser.perform(email: 'HELLO@AARONMALLEN.ME')
#=> <#User id=1 email='hello@aaronmallen.me'>
```

我希望 ruby 社区发现这个宝石是有用的，并且我喜欢任何反馈，问题，或者你愿意给出的[库](https://github.com/aaronmallen/activeinteractor)上的星星。宝石的详细用法可以在[维基](https://github.com/aaronmallen/activeinteractor/wiki)上找到。gem 的技术文档可在 [rubydoc](https://www.rubydoc.info/gems/activeinteractor) 上找到。
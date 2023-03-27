# rails 应用程序的 RESTful 投票 gem

> 原文：<https://itnext.io/restful-voting-for-rails-app-b76076fb7cac?source=collection_archive---------0----------------------->

![](img/d35e5356f7533a58a6ad2e57ea9912ba.png)

资料来源:blog.codelation.com

*RailsRestVote* 是一个 Ruby Gem，它为 rails 应用程序的任何模型添加了投票功能，并公开了它的 RESTful APIs。

如果你使用的是 Angular 这样的前端框架，React 在你的应用中真的很有帮助。

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Frestful-voting-for-rails-app-b76076fb7cac)

# 先决条件

## 扶持 CORS

如果你正在构建一个公共 API，你可能想要启用跨源资源共享(CORS ),以使跨源 AJAX 请求成为可能。

rack-cors gem 使这变得非常简单。就像这样把它放在你的 Gemfile 里:

```
gem 'rack-cors'
```

并将类似下面的代码放入 Rails 应用程序的 config/application.rb 中。例如，这将允许来自任何资源的 GET、POST 或 OPTIONS 请求。

```
module YourApp
  class Application < Rails::Application # ... config.middleware.insert_before 0, "Rack::Cors" do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end end
end
```

> *更多详细配置选项请参见 gem* [*文档*](https://github.com/cyu/rack-cors)

# 装置

将这一行添加到应用程序的 Gemfile 中:

```
gem 'rails_rest_vote'
```

然后执行:

```
$ bundle install
```

在项目文件夹中运行以下命令:

```
$ rails g rails_rest_vote MODEL
```

在上面的命令中，您需要用应用程序用户的类名替换`MODEL`(通常是`User`，但也可能是`Admin`)

它会为你做三件事:

*   在 db/migrate/ folder 中创建投票表的迁移文件。
*   在用户模型中插入关联，即 has_many :votes
*   创建投票模型，即 app/models/vote.rb

之后，使用以下命令迁移数据库:

```
$ rake db:migrate
```

现在你可以走了。

# 使用

将以下行添加到您想要使用向上投票/向下投票或相似/不相似功能的模型中:

```
has_many :votes, :as => :votable
```

它有两种用法，我在下面的例子中使用的是`Post`模型:

> ***用作 upvote/downvote(堆栈溢出/Quora):***
> 
> ***原料药***

## /API/票数/上升

以下 API 用于模型的向上投票:

```
method: POST 
body: {"votable_id":"1","votable_type":"Post","user_id":"1"}
content-type: application/json
response: {"status":200,"message":"upvoted successfully."}
```

## /API/票数/下降

以下 API 用于否决模型:

```
method: POST 
body: {"votable_id":"1","votable_type":"Post","user_id":"1"}
content-type: application/json
response: {"status":200,"message":"downvoted successfully."}
```

## /API/选票/用户？user_id=1

以下 API 返回特定用户完成的向上投票和向下投票计数:

```
method: GET 
content-type: application/json
response: {"status":200,"upcount":1,"upvotes":[{"id":1,...}], "downcount":1,"downvotes":[{"id":3,...}]}
```

## /API/票数/车型？votable_id=1&votable_type=Post

以下 API 返回特定用户完成的向上投票和向下投票计数。

```
method: GET 
content-type: application/json
response: {"status":200,"upcount":1,"upvotes":[{"id":1,...}], "downcount":1,"downvotes":[{"id":3,...}]}
```

> ***用* *作为喜欢/不喜欢(脸书):***
> 
> ***原料药***

## /API/喜欢

相同的 API 用于相似和不相似的功能。如果你第一次点击这个 API，它将像“喜欢”一样工作，如果你第二次发送相同的参数，它将从投票表中删除记录，即“不喜欢”

```
method: POST 
body: {"votable_id":"1","votable_type":"Post","user_id":"1"}
content-type: application/json
response: {"status":200,"message":"liked successfully."}
```

## /API/赞/用户？user_id=1

以下 API 返回特定用户完成的计数:

```
method: GET 
content-type: application/json
response: {"status":200,"likecount":1,"likes":[{"id":1,...}]}
```

## /api/likes/model？votable_id=1&votable_type=Post

下面的 API 返回类似于在特定模型上完成的计数。

```
method: GET 
content-type: application/json
response: {"status":200,"likecount":1,"likes":[{"id":1,...}]}
```

## 注意:

`user_id`内部请求体可以是`admin_id`，这取决于应用程序的用户模型(通常是`User`，但也可能是`Admin`)

> 根据您的使用情况，这两种功能可以在同一应用程序中使用。

# 基于令牌的认证

如果您使用基于令牌的身份验证来授权请求，您可以在应用程序的 ApplicationController 中进行。

```
# app/controllers/application_controller.rb 
class ApplicationController < ActionController::API 
    before_action :authenticate_request 
    attr_reader :current_user 

    private 
    def authenticate_request
       # Here you can read headers and check for current user. this example is for `jwt` check below link for more.
       @current_user = AuthorizeApiRequest.call(request.headers).result render json: { error: 'Not Authorized' }, status: 401 unless @current_user 
    end 
end
```

> *阅读*[*token-based-authentic ation-with-ruby-on-rails-5-API*](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api)*博文进行整合。*

# 检查 git repo。

[](https://github.com/tixdo/rails_rest_vote) [## thecodinghouse/rails_rest_vote

### 用于 rails 应用程序的 RESTful 投票 gem

github.com](https://github.com/tixdo/rails_rest_vote) 

(帮助其他人找到我在 Medium 上的文章👏🏽下面。)
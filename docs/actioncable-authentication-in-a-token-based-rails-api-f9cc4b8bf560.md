# 基于令牌的 Rails API 中的 ActionCable 身份验证

> 原文：<https://itnext.io/actioncable-authentication-in-a-token-based-rails-api-f9cc4b8bf560?source=collection_archive---------0----------------------->

![](img/f95ab9b4c25d65a634766e0ad4c32133.png)

[Action Cable](http://guides.rubyonrails.org/action_cable_overview.html) 是 Rails 应用中的 websocket。如果你有一个 Rails API 并且刚开始使用 actioncable，你将需要设置它的认证。

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Factioncable-authentication-in-a-token-based-rails-api-f9cc4b8bf560) 上分享这篇文章

大多数教程使用会话或 cookies 来验证 ActionCable 连接中的`User`。但是在基于令牌的 Rails API 中，我们使用 J [WT 令牌](https://jwt.io/)，所以我们必须设置不同的认证。对于 React 应用程序，幸运的是有一个名为`[action-cable-react-jwt](https://github.com/zekedran/action-cable-react-jwt)`的 npm 库，我们可以用它来连接 ActionCable。`action-cable-react-jwt`与其他*(如:* `*action-cable*` *或* `*action-cable-react*` *)不同的是，*有传递 JWT 令牌的选项。

# [安装](https://github.com/zekedran/action-cable-react-jwt)

在 React 应用程序中:

```
yarn add action-cable-react-jwtornpm install action-cable-react-jwt
```

# 使用

## 在 React 应用程序中:

***导入*** `***action-cable-react-jwt***` ***首先***

```
import ActionCable from 'action-cable-react-jwt'
```

***连接到动作电缆***

```
componentDidMount () { 
  this.createSocket()
}const createSocket = () => { // get your JWT token
  // this is an example using localStorage
  const yourToken = localStorage.Token let App = {}
  App.cable = ActionCable.createConsumer(‘ws://localhost:3000/cable', yourToken) const subscription = App.cable.subscriptions.create({channel: 'YourChannelName'}, {
    connected = () => {},
    disconnected = () => {},
    received = (data) => { 
      console.log(data) 
    }
  })
}
```

## 在您的 Rails 应用程序中:

***认证***

```
# inside app/channels/application_cable/connection.rbmodule ApplicationCable        
  class Connection < ActionCable::Connection::Base 
    identified_by :current_user

    def connect                
      self.current_user = find_verified_user
    end

    private
      def find_verified_user   
        begin
          token = request.headers[:HTTP_SEC_WEBSOCKET_PROTOCOL].split(' ').last
          decoded_token = JsonWebToken.decode(token)
          if (current_user = User.find(decoded_token["user_id"]))
            current_user
          else                 
            reject_unauthorized_connection
          end
        rescue
          reject_unauthorized_connection  
        end
      end
  end
end
```

就是这样！现在应该可以用 JWT 令牌安全地连接到 actioncable 了。

![](img/9c6ea2d4fc76418637f037229e19c6cf.png)

> 谢谢！我欣赏一两次鼓掌。。。\m/
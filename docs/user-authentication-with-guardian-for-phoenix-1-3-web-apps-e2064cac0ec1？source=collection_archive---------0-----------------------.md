# 使用 Guardian for Phoenix 1.3 Web 应用程序进行用户认证

> 原文：<https://itnext.io/user-authentication-with-guardian-for-phoenix-1-3-web-apps-e2064cac0ec1?source=collection_archive---------0----------------------->

![](img/39af1bc2b52a3a3f6912330d66598b6d.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fuser-authentication-with-guardian-for-phoenix-1–3-web-apps-e2064cac0ec1)

身份验证是大多数 web 应用程序不可或缺的一部分。在本文中，我将展示如何使用 Guardian 库和 Phoenix 1.3 基于用户名/密码对用户进行身份验证。登录的用户信息将保存在会话中，以便管理员可以轻松检查和保护敏感资源，防止未经授权的用户访问。

本文中我将使用的示例项目基于 Chris McCord 的书——编程凤凰——高效| >可靠| >快速。我已经根据书中的内容编写了 **rumbl** 项目，并对其进行了相应的转换以支持 Phoenix 1.3。只有身份验证部分被修改为使用 Guardian。完整的项目代码和测试可以在我的 github [这里](https://github.com/imeraj/Phoenix_rumbl)找到。

用于 Guardian 的 github [文档](https://github.com/ueberauth/guardian)非常好，但是对于一个新手来说，完成所有这些并进行认证可能是一项艰巨的任务。本文的目的是解读其中的一些文本，并提供工作代码来轻松引导项目的认证部分。

# **设置依赖关系**:

在 **mix.exs** 中添加以下依赖关系:

```
defp deps **do** [
    {:guardian, "~> 1.0"},
    {:comeonin, "~> 4.0"},
    {:bcrypt_elixir, "~> 1.0"}
  ]
**end**
```

这里—

i. guardian:是认证库

二。c [omeonin](https://github.com/riverrun/comeonin) :是仙丹的密码哈希库

三。bcrypt_elixir:是 comeonin 使用的密码散列算法

# **实施监护人回调**

创建下面的模块来设置 Guardian 回调( **guardian.ex** ) —

```
defmodule Rumbl.Auth.Guardian **do** @moduledoc **false** use Guardian, otp_app: :rumbl

  alias Rumbl.Accounts

  def subject_for_token(user, _claims) **do** sub = to_string(user.id)
    {:ok, sub}
  **end** def resource_from_claims(claims) **do** id = claims["sub"]
    user = Accounts.get_user(id)
    {:ok, user}
  **end
end**
```

这里，我们想在**帐户**上下文中使用**用户**模式来认证**用户**。(*你可以通读本节在 Guardian github 上的评论，更好地理解回调*)。

# **配置监护人:**

在 **config.exs** 中添加以下内容以配置 Guardian —

```
# Configures Guardian
config :rumbl, Rumbl.Auth.Guardian,
  issuer: "rumbl",
  secret_key: "HNinpKh9Ne3tr8BpjCpAEh0xzCqTIG3PWsfkR2AtzvUaRIpbs6oIQ9RcmjmGPekJ"
```

# **注册时创建密码哈希:**

完成上述设置后，我们就可以提供特定于应用程序的更改，以便在数据库中存储散列密码，并在登录请求发出时对用户进行身份验证。

我们在 **user.ex** 中的注册变更集如下所示

```
def registration_changeset(%User{} = user, attrs) **do** user
  |> cast(attrs, [:name, :username, :password])
  |> validate_required([:name, :username, :password])
  |> validate_length(:username, min: 3, max: 10)
  |> validate_length(:password, min: 5, max: 10)
  |> unique_constraint(:username)
  |> put_password_hash()
**end**defp put_password_hash(changeset) **do** case changeset **do** %Ecto.Changeset{valid?: **true**, changes: %{password: pass}} ->
      put_change(changeset, :password_hash, Comeonin.Bcrypt.hashpwsalt(pass))

    _ ->
      changeset
  **end
end**
```

在这里，在注册期间，我们使用 bcrypt 计算一个密码，并将其放入变更集，以便在用户成功注册时保存在 DB 中。

# 验证用户/登录:

每当用户尝试登录时，就会调用一个 sessioncontoller#create 操作，其中会写入身份验证代码—

```
def create(conn, %{"session" => %{"username" => user, "password" => password}}) **do** case **Rumbl.Auth.authenticate_user**(user, password) **do** {:ok, user} ->
      conn
      |> **Rumbl.Auth.login(user)**
      |> put_flash(:info, "Welcome back!")
      |> redirect(to: user_path(conn, :index))

    {:error, _reason} ->
      conn
      |> put_flash(:error, "Invalid username/password combination")
      |> render("new.html")
  **end
end**
```

这里，重要的行用粗体显示。 **Rumbl。Auth.authenticate_user** 从 params 获取用户名和密码，并对用户进行身份验证

```
def authenticate_user(username, given_password) **do** query = Ecto.Query.from(u in User, where: u.username == ^username)

  Repo.one(query)
  |> check_password(given_password)
**end** defp check_password(**nil**, _), **do**: {:error, "Incorrect username or password"}

defp check_password(user, given_password) **do** case Bcrypt.checkpw(given_password, user.password_hash) **do
    true** -> {:ok, user}
    **false** -> {:error, "Incorrect username or password"}
  **end
end**
```

代码非常简单。这里，首先使用用户名从数据库中检索用户，并将其与 **check_password** 中存储的数据库散列密码进行匹配。

如果认证成功， **Rumbl。Auth.login(user)** 被调用( **auth.ex** ) —

```
def login(conn, user) **do** conn
  |> Guardian.Plug.sign_in(user)
  |> assign(:current_user, user)
**end**
```

这基本上是一个在会话中存储用户信息并将用户分配给 **current_user** 的插件。

# **注销用户:**

注销代码非常简单。每当用户注销时，就会调用 sessioncontroller#delete 操作，并注销用户——

```
def delete(conn, _) **do** conn
  |> **Rumbl.Auth.logout()**
  |> redirect(to: page_path(conn, :index))
**end**
```

代码中突出显示的粗体部分如下所示( **auth/auth.ex** ) —

```
def logout(conn) **do** conn
  |> Guardian.Plug.sign_out()
**end**
```

# **封堵管线保护线路:**

在基本认证工作之后，我们需要保护路由免受未授权用户的访问。为此，我们需要定义下面的管道(**auth _ access _ pipeline . ex)**—

```
defmodule Rumbl.Auth.AuthAccessPipeline **do** @moduledoc **false** use Guardian.Plug.Pipeline, otp_app: :rumbl

  plug(Guardian.Plug.VerifySession, claims: %{"typ" => "access"})
  plug(Guardian.Plug.EnsureAuthenticated)
  plug(Guardian.Plug.LoadResource)
**end**
```

这里—

一.**监护人。Plug.VerifySession** —在会话中查找令牌并验证它

二。**守护者。Plug.EnsureAuthenticated** —确保找到了一个令牌并且它是有效的

三。**守护者。Plug.LoadResource —** 如果找到了一个令牌，就为它加载资源

接下来，我们需要将管道和错误处理程序添加到 **config.exs** 中的配置中

```
config :rumbl, Rumbl.Auth.AuthAccessPipeline,
  module: Rumbl.Auth.Guardian,
  error_handler: **Rumbl.Auth.AuthErrorHandler**
```

错误处理器是一个实现`auth_error`功能的模块。Rumbl。Auth.AuthErrorHandler 在 **auth_error_handler.ex** 中定义

```
defmodule Rumbl.Auth.AuthErrorHandler **do** @moduledoc **false** import Plug.Conn

  def auth_error(conn, {type, _reason}, _opts) **do** body = Poison.encode!(%{message: to_string(type)})
    send_resp(conn, 401, body)
  **end
end**
```

接下来，我们需要修改 **router.ex** 来使用上面提到的管道来保护被认证的路由—

```
pipeline :auth **do** plug(Rumbl.Auth.AuthAccessPipeline)
**end**scope "/", RumblWeb **do** pipe_through([:browser, :auth])

  resources("/users", UserController, only: [:index, :show])
  resources("/videos", VideoController)
  resources("/sessions", SessionController, only: [:delete])
  get("/watch/:id", WatchController, :show)
**end**
```

# **获取控制器中的当前用户:**

在控制器中，我们经常需要当前登录的用户信息。为此，我写了下面的插件—

```
def load_current_user(conn, _) **do** conn
  |> assign(:current_user, Guardian.Plug.current_resource(conn))
**end**
```

你可以把它放在任何需要 current_user 的控制器上面—

```
import Rumbl.Auth, only: [load_current_user: 2]

plug(:load_current_user when action in [:show, :index])
```

这应该让你开始使用 Guardian 在 Phoenix web app 中进行身份验证。完整的项目代码和测试可以在我的 github [这里](https://github.com/imeraj/Phoenix_rumbl)获得。

*更多详细和深入的未来技术帖子请关注我这里或点击* [*twitter*](https://twitter.com/meraj_enigma) *。*
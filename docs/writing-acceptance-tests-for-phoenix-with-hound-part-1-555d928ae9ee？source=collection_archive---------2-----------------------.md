# 为 Phoenix with Hound 编写验收测试

> 原文：<https://itnext.io/writing-acceptance-tests-for-phoenix-with-hound-part-1-555d928ae9ee?source=collection_archive---------2----------------------->

![](img/b66692ffc3305d6f4aeb78d45ac1e6ff.png)

在本文中，我将使用 Hound 对 Phoenix framework 进行端到端的验收测试。我将使用的运行测试的例子来自 Chris McCord 的书——编程凤凰——高效| >可靠| >快速。我已经从书上写了 **rumbl** 项目，并对其进行了相应的转换以支持 Phoenix 1.3。完整的项目代码和测试可以在我的 github [这里](https://github.com/imeraj/Phoenix_rumbl)找到。

我已经用 Safari 和 Chrome 浏览器测试了测试用例。对于 Safari 浏览器，需要进行一些额外的设置，如下面的**先决条件**部分所述。

它应该是直接运行项目，除非你需要配置 Mysql 数据库正确。

```
config :rumbl, Rumbl.Repo,
  adapter: Ecto.Adapters.MySQL,
  username: "root",
  password: "root",
  database: "rumbl_dev",
  hostname: "localhost",
  pool_size: 10
```

如果您使用其他用户名/密码，请在 config/dev.exs 文件中相应更新。

config/test.exs 也是如此。

# **什么是猎犬**

正如 github 页面上提到的，Hound 是一个为 Elixir/Phoenix 编写端到端自动化测试的库。同一页中提到的一些主要特征是—

*   可以同时运行多个浏览器会话。
*   支持 Selenium (Firefox，Chrome)，ChromeDriver 和 PhantomJs。
*   支持大量使用 Javascript 的应用。在报告错误之前重试几次。
*   实现 WebDriver Wire 协议。

你可以在这里找到库的 API 参考—[https://hexdocs.pm/hound/readme.html](https://hexdocs.pm/hound/readme.html)

# **前提条件**

1.  **网络驱动**

为了让浏览器自动化测试正常运行，有必要安装并正常运行一个 webdriver 服务器。如果您不熟悉什么是 webdriver 服务器或如何运行它，请查看互联网上的各种资源。如需快速入门，请点击此处的[链接](https://github.com/HashNuke/hound/wiki/Starting-a-webdriver-server)。深入研究 webdriver 如何工作超出了本文的范围。

在 mac 平台上，你可以很容易地用自制软件安装 selenium webdriver

```
# brew install selenium-server-standalone
```

安装后，在终端中运行命令—

```
# selenium-server
```

如果您的服务器启动时没有任何错误，您应该会看到下面的输出消息—

```
2018-02-10 22:45:39.241:INFO:osjs.Server:main: Started @668ms22:45:39.241 INFO - Selenium Server is up and running
```

2.对于 Safari，从**开发**菜单中启用“允许远程自动化”。

# **猎犬设置**

一旦你的先决条件得到解决，是时候设置猎犬了，包括以下步骤

1.  将依赖性添加到您的 Phoenix 项目中——

```
{:hound, "~> 1.0"}
```

2.在 test/test_helper.exs 文件中的 ExUnit.start()行之前启动 Hound:

```
{:ok, _} = Application.ensure_all_started(:hound)
ExUnit.start()
```

3.在 config/test.exs 中，将端点的服务器选项更改为 **true** :

```
config :rumbl, RumblWeb.Endpoint,
  http: [port: 4001],
  server: **true**
```

# **猎犬配置**

要配置 hound，请将下面一行放在 config/config.exs 中—

```
config :hound, browser: "safari"
```

如果你更喜欢用“chrome”来代替，用上面的“Chrome”来代替“safari”。更多选项，请点击查看[。](https://github.com/HashNuke/hound/blob/master/notes/configuring-hound.md)

# **运行测试**

一旦设置和配置完成，selenium-server 运行，您应该能够使用下载的 **rumbl** 项目文件夹中的以下命令运行测试

```
# mix test test/rumbl_web/acceptance/hound_test.ex
```

如果一切顺利，你应该看到类似下面的视频，所有的测试都应该通过。

用猎犬进行测试

```
Finished in 8.2 seconds3 tests, 0 failures
```

# **测试用例解释**

有三个简单的测试用例都写在 test/rumbl _ web/acceptance/hound _ test . ex 文件中。在这一节中，我将回顾测试用例并提供一些解释。

# **测试 1:主页加载成功**

检查主页是否加载成功。

```
test "homepage loads successfully", _meta **do** navigate_to(@page_url)
  assert page_title() == "Hello Rumbl!"
**end**
```

naviage_to 将导航到@page_url —

```
@page_url page_url(RumblWeb.Endpoint, :index)
```

这转化为主页。

断言是检查页面标题是否与预期的标题匹配——“Hello Rumbl！”。

# 测试 2:注册成功

该测试将检查使用名称、用户名和密码作为注册字段的用户注册是否成功。

```
test "successfull registration", _meta **do** navigate_to(@page_url)

  element = find_element(:link_text, "Register")
  element |> click()

  name = find_element(:xpath, ~s|//*[@id="user_name"]|)
  name |> fill_field("Meraj")

  username = find_element(:xpath, ~s|//*[@id="user_username"]|)
  username |> fill_field("meraj")

  password = find_element(:xpath, ~s|//*[@id="user_password"]|)
  password |> fill_field("password")

  submit = find_element(:xpath, ~s|/html/body/div/main/form/div[4]/button|)
  submit |> click()

  assert current_url() == @user_url
  assert page_source() =~ "Listing Users"
  assert page_source() =~ "meraj"
**end**
```

这里—

*   find_element(:link_text，" Register ")在主页上找到**注册**链接并点击它。这将加载**注册**表单。
*   接下来，我使用三个 xpath 字段来检索名称、用户名和密码字段，并用适当的输入值填充。
*   接下来，我使用 xpath 找到 submit 按钮并点击它
*   这将把我们带到@user_url 页面，其中列出了所有用户(成功登录后显示相同的页面)。
*   接下来，一些断言检查注册用户是否在用户列表中

# 测试 3:登录成功

最后一个测试是检查注册用户是否可以成功登录。

```
test "successful login", _meta **do** user = insert_user()
 navigate_to(@page_url)

 element = find_element(:link_text, "Log In")
 element |> click()

 username = find_element(:xpath, ~s|//*[@id="session_username"]|)
 username |> fill_field(user.name)

 password = find_element(:xpath, ~s|//*[@id="session_password"]|)
 password |> fill_field(user.password)

 submit = find_element(:xpath, ~s|/html/body/div/main/form/button|)
 submit |> click()

 assert current_url() == @user_url
 assert page_source() =~ "Listing Users"
 assert page_source() =~ "meraj"
**end**
```

在这个阶段，它应该是非常简单明了的。这里-

*   使用 insert_user()插入一个用户。这方面的代码可以在**test/support/test _ helpers . ex**中找到。
*   然后，用户名和密码用于测试成功登录。

# **一些观察**

对于运行并发浏览器测试，我得到了以下错误

```
** (DBConnection.OwnershipError) cannot find ownership process for #PID<0.475.0>
```

快速搜索把我带到了这个页面。

要解决这个问题，请按照以下步骤操作—

*   在 config/config.exs 或 config/test.exs 中设置一个标志以启用沙盒

```
config :rumbl, sql_sandbox: **true**
```

*   使用标志有条件地将插件添加到 lib/rumbl_web/endpoint.ex 中的**插件(RumblWeb)之前。【T21 路由器】。**

```
if Application.get_env(:your_app, :sql_sandbox) do
  plug Phoenix.Ecto.SQL.Sandbox
end
```

*   将以下内容添加到测试用例或用例模板中

```
setup do  
  :ok = Ecto.Adapters.SQL.Sandbox.checkout(YourApp.Repo)
  metadata = Phoenix.Ecto.SQL.Sandbox.metadata_for(YourApp.Repo, self())
  Hound.start_session(metadata: metadata)
  :ok
end
```

我希望这篇文章能帮助一些人使用 Hound for Phoenix 平台编写验收测试(端到端自动化测试)。

*更多详细和深入的未来技术帖子请关注我这里或点击* [*twitter*](https://twitter.com/meraj_enigma) *。*

# 高效| >可靠| >快速
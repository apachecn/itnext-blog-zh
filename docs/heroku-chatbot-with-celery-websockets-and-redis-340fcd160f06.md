# Django 聊天机器人与芹菜，网络插座，和 Redis。

> 原文：<https://itnext.io/heroku-chatbot-with-celery-websockets-and-redis-340fcd160f06?source=collection_archive---------2----------------------->

让我们创建一个聊天机器人，它可以使用 Celery 异步执行任务，并通过 WebSockets 进行通信。

![](img/3575021144dffb4ddcd6c18700ddc519.png)

詹姆斯·罗亚尔·劳森的聊天机器人

## TL；博士

这里有一个 [Github repo](https://github.com/slyapustin/django-chatbot) ，点击`[Deploy to Heroku](https://heroku.com/deploy?template=https://github.com/inoks/django-chatbot)`即可。

# 姜戈

我们将使用 Django 作为我们应用程序的基础:

*   HTTP 请求处理程序
*   应用程序配置
*   模板引擎

# 雷迪斯

我们需要在几个地方使用 Redis:

*   高速缓冲存储
*   Celery 消息代理和结果后端
*   WebSocket 通信的通道层

我们将使用 Heroku 进行部署，因此不需要手动安装和配置 Redis。Redis DSN 将作为环境变量`REDIS_URL`可用，因此我们可以在任何需要的地方使用它。

## Redis 作为高速缓存存储

Django 内部不支持 Redis，所以我们需要使用额外的包。我们将使用`[django-redis](https://github.com/niwinz/django-redis)`。这是 Django 的全功能 Redis 缓存后端。

```
pip install django-redis
```

这将安装更多的依赖项，包括`[redis-p](https://github.com/andymccurdy/redis-py)y`——Redis 的 Python 接口。

我们只需要用`CACHES`设置更新我们的 Django 项目配置。

## 芹菜与 Redis 作为消息代理

我们将有一些任务，这可能需要一段时间。例如，从远程服务器获得响应。因此，我们需要在后台执行这些操作，并在结果可用时将其发送回客户端。

芹菜的配置非常简单，我们将重用芹菜的`REDIS_URL``BROKER_URL`和`RESULT_BACKEND`。

让我们在`project/celery.py`中定义芹菜实例:

我们需要在项目中导入 Celery 实例，以确保在 Django 启动时加载应用程序。

所以现在我们的芹菜准备好了，我们可以向我们的项目添加后台任务。比如这个:

# 带有 Django 通道的 WebSockets

当结果可用时，我们希望立即将结果从发送到客户端。而不需要通过预定的 AJAX 调用每隔几秒查询一次。

WebSocket 是一种计算机通信协议，通过单一 TCP 连接提供全双工通信通道。因此，用户可以向后端发送数据，后端也可以向用户发送数据。

[如果你需要 WebSockets，Django 频道](https://github.com/django/channels)是必经之路。现在有两个可用的版本 1.x 和 2.x。它们有重大的变化，如果您有一个相当大的应用程序，从 1.x 升级到 2.x 可能是一个大项目。不再支持 1.x 版本，所以对于新项目，应该使用 2.x 分支。

Django-Channels 在官方网站上有非常好的[文档](https://channels.readthedocs.io/en/latest/)，所以我鼓励你去看看。

关键时刻是——我们需要改变应用程序中的一些东西来支持 WebSockets。

## 达芙尼

为了使用 WebSockets，我们需要一个支持它的 web 服务器。所以我们需要用 Daphne 来代替 [Gunicorn](https://docs.djangoproject.com/en/2.2/howto/deployment/wsgi/gunicorn/) ，我们通常用它来服务 Django 驱动的网站。

Daphne 是一个 HTTP、HTTP2 和 WebSocket 协议服务器，用于 ASGI 和 ASGI-HTTP。

我们的应用程序不需要配置任何东西，我们可以像运行 WSGI 应用程序一样运行它，但是我们需要将它指向 ASGI 应用程序，而不是 WSGI:

```
daphne project.asgi:application
```

ASGI 配置看起来和 WSGI 一样，但是指向 Django Channels 应用程序:

## Django 频道和 Redis

Django 通道是某种通信系统，它允许多个消费者实例相互交谈，并与 Django 的其他部分交谈。

通道层提供了以下抽象:

*   一个**通道**是一个可以接收消息的邮箱。每个频道都有一个名称。任何拥有频道名称的人都可以向该频道发送消息。
*   **组**是一组相关的频道。一个团体有一个名字。任何拥有群组名称的人都可以通过名称向群组添加/移除频道，并向群组中的所有频道发送消息。

每个消费者实例都有一个自动生成的唯一通道名，因此可以通过通道层进行通信。

所以让我们为这个项目定义`CHANNEL_LAYERS`。为此，我们将使用`[channels_redis](https://github.com/django/channels_redis/)`。所以我们需要定义后端并提供 Redis DSN:

## WebSocket 消费者和路由

现在我们需要定义我们将如何与我们的客户交互——前端 JS 调用。因此，我们将接收来自客户端的每条消息(以 JSON 字符串的形式)并对其进行处理。

下面是它可能的样子:

由于我们的应用程序可以有多个消费者(就像我们可能有多个 Django 视图一样),我们需要类似于 Django `urls.py`文件的 URL 路由:

因此，当客户端向 WebSocket `/ws/chat/`路径发送消息时，它将由我们的`ChatConsumer`处理。

最后，我们需要将我们的应用程序 URL 包含到主路由器— `project/routing.py`:

```
# project/routing.py
from channels.routing import ProtocolTypeRouter, URLRouter
import chat.routing application = ProtocolTypeRouter({    
    # (http->Django views is added by default)    
    'websocket':    
        URLRouter(
            chat.routing.websocket_urlpatterns
        ),
})
```

并从我们的`project/settings.py`文件中指向它:

```
ASGI_APPLICATION = "project.routing.application"
```

# 部署到 Heroku

我将使用 Heroku 进行部署，因为这是一步完成所有部署的最快方式。而且——这是免费的(尽管他们需要账户验证)。

## Procfile

该文件包含进程类型定义:

在这里，我们定义了在应用程序`release`阶段需要做什么，并配置了两个应用程序实例:

*   `web` — Daphne 服务器，它将监听`$PORT`并处理我们的`https://`和`wss://`请求。
*   `worker` — Celery worker 实例处理我们的异步任务队列。

## Heroku app.json 清单

这个文件将用于从头开始创建一个新的 Heroku 网站实例。

*   我们需要为我们的应用程序设置环境变量(如 Django `SECRET_KEY`值)
*   我们需要哪些服务(PostgreSQL、Redis)
*   我们想要哪种类型的虚拟机`heroku/python`。

该文件也可以用来创建一个漂亮的`Deploy to Heroku`按钮。所以下次可以一键部署。

[![](img/9cb85dd7004fa644a18ad0e2a80cfd4d.png)](https://heroku.com/deploy?template=https://github.com/inoks/django-chatbot)

这是一个简单的解释，没有涉及太多的细节(将在下一篇文章中介绍)。

Github 上可用的项目:[https://github.com/slyapustin/django-chatbot](https://github.com/slyapustin/django-chatbot)

试玩聊天机器人:[https://django-chat-bot.herokuapp.com/](https://django-chat-bot.herokuapp.com/)
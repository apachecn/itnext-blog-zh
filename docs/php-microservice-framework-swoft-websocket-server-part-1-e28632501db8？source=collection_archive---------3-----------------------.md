# PHP 微服务框架 Swoft: WebSocket 服务器第 1 部分

> 原文：<https://itnext.io/php-microservice-framework-swoft-websocket-server-part-1-e28632501db8?source=collection_archive---------3----------------------->

![](img/0191dd32621835a91d010137a124911b.png)

我们要学习的这篇文章是:如何安装和运行 swoft websocket 服务器。

> *本文是 Swoft HTTP Server 系列文章之一。来了解一下 Swoft 吧！*

# 什么是 Swoft？

Swoft 是一个 PHP 高性能微服务协程框架。已经出版多年，已经成为 php 的不二之选。它可以像 Go 一样，内置协同 web 服务器和通用协同客户端，驻留在内存中，独立于传统的 PHP-FPM。还有类似的 Go 语言操作，类似于 Spring Cloud 框架的灵活注解。

Swoft 通过三年的积累和方向探索，将 Swoft 打造成 PHP 界的春天云，是 PHP 高性能框架和微服务管理的不二之选。

# 开源代码库

[](https://github.com/swoft-cloud/swoft) [## 软件云/软件

### PHP 微服务协程框架 Swoft 是一个基于 Swoole 扩展的 PHP 微服务协程框架…

github.com](https://github.com/swoft-cloud/swoft) 

# 创建新项目

使用`swoft-cli`工具为 Websocket 创建新项目。

```
php swoftcli.phar create:app --type ws swoft-ws-app
cd swoft-ws-app
composer install
```

# 启动服务器

用`php bin/swoft ws:start`命令启动 Websocket 服务器，可以看到下图:

```
$ php bin/swoft ws:start Information Panel
  ********************************************************************
  * WebSocket | Listen: 0.0.0.0:18308, type: TCP, mode: Process, worker: 8
********************************************************************
```

> *web socket 服务器的端口是*T2

# 创建一个模块

使用`swoft-cli`工具创建新的 websocket 模块。

```
php swoftcli.phar gen:ws-mod echo --prefix /echo
```

回送模块(`app/WebSocket/EchoModule.php`)的代码如下:

```
<?php declare(strict_types=1);namespace App\WebSocket;use Swoft\Http\Message\Request;
use Swoft\Http\Message\Response;
use Swoft\WebSocket\Server\Annotation\Mapping\WsModule;
use Swoft\WebSocket\Server\Annotation\Mapping\OnOpen;
use Swoft\WebSocket\Server\Annotation\Mapping\OnHandshake;
use Swoft\WebSocket\Server\Annotation\Mapping\OnMessage;
use Swoole\WebSocket\Server;/**
 * Class EchoModule - This is an module for handle websocket
 *
 * @WsModule("/echo")
 */
class EchoModule
{
    /**
     * @OnHandshake()
     * @param Request $request
     * @param Response $response
     * @return array
     */
    public function checkHandshake(Request $request, Response $response): array
    {
        // some validate logic ...
        return [true, $response];
    } /**
     * @OnOpen()
     * @param Server $server
     * @param Request $request
     * @param int $fd
     * @return mixed
     */
    public function onOpen(Server $server, Request $request, int $fd)
    {
        $server->push($fd, 'hello, welcome! :)');
    }

    /**
     * @OnMessage()
     * @param Server $server
     * @param Frame  $frame
     */
    public function onMessage(Server $server, Frame $frame): void
    {
        $server->push($frame->fd, 'Recv: ' . $frame->data);
    }
}
```

# 访问 Websocket 服务器

这里使用`swoft-devtool`连接 WebSocket 服务器。

使用`swoft-devtool`组件中的`php bin/swoft dclient:ws /echo`命令连接 WebSocket 服务器，可以看到如下连接成功消息。

```
Begin connecting to websocket server: 127.0.0.1:18308 path: /echo
Success connect to websocket server. Now, you can send message
INTERACTIVE
================================================================================
server> ?Opened, welcome #1!
client> hi
server> Recv: hi
client>
```

# 开源代码库

[](https://github.com/swoft-cloud/swoft) [## 软件云/软件

### PHP 微服务协程框架 Swoft 是一个基于 Swoole 扩展的 PHP 微服务协程框架…

github.com](https://github.com/swoft-cloud/swoft)
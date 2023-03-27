# PHP 微服务框架——Swoft v 2 . 0 . 7 发布

> 原文：<https://itnext.io/php-microservice-framework-swoft-v2-0-7-released-f935a278266d?source=collection_archive---------7----------------------->

![](img/99fdd256a81ee0bc9c190474bc1cf4ea.png)

# 什么是 Swoft？

Swoft 是一个 PHP 高性能微服务协程框架。已经出版多年，已经成为 php 的不二之选。它可以像 Go 一样，内置协同 web 服务器和通用协同客户端，驻留在内存中，独立于传统的 PHP-FPM。还有类似的 Go 语言操作，类似于 Spring Cloud 框架的灵活注解。

Swoft 通过三年的积累和方向探索，将 Swoft 打造成 PHP 界的春天云，是 PHP 高性能框架和微服务管理的不二之选。

# Swoft v2.0.7

Swoft `v2.0.7`继续在`v2.0.6`上航行，已经在大量的生产作业中使用，得到了众多用户的认可和支持。正式版做了很多改进和优化，性能更好。

*   增加了 http 会话功能组件，提供 Http 会话管理，支持多种存储驱动
*   增强的 TCP 服务器请求支持添加全局或相应的方法中间件
*   增强的 Websocket 服务器消息请求支持，用于添加全局或相应的方法中间件

## 开源代码库

[https://github.com/swoft-cloud/swoft](https://github.com/swoft-cloud/swoft)

# Http 会话

使用 Composer 安装 swoft/session 组件

*   在 project composer.json 所在的目录下执行`composer require swoft/session`。
*   将`Swoft\Http\Session\SessionMiddleware`中间件添加到全局中间件

在`app/bean.php`:

```
'httpDispatcher'    => [
        // Add global http middleware
        'middlewares'      => [
            \Swoft\Http\Session\SessionMiddleware::class,
        ],
    ],
```

> *默认是基于本地文件的驱动程序，保存在* `*runtime/sessions*` *目录下*

更多关于驱动只需要配置相应的`handler`。例如，配置`Redis`驱动程序:

```
'sessionHandler' => [
    'class'    => RedisHandler::class,
    // Config redis pool
    'redis' => bean('redis.pool')
],
```

# Websocket 消息中间件

*   全球中间件

在`app/bean.php`中配置:

```
/** @see \Swoft\WebSocket\Server\WsMessageDispatcher */
    'wsMsgDispatcher' => [
        'middlewares' => [
            \App\WebSocket\Middleware\GlobalWsMiddleware::class
        ],
    ],
```

*   作用于控制器

```
/**
 * Class HomeController
 *
 * @WsController(middlewares={DemoMiddleware::class})
 */
class TestController
{}
```

# TCP 请求中间件

*   全球中间件

在`app/bean.php`中配置:

```
/** @see \Swoft\Tcp\Server\TcpDispatcher */
    'tcpDispatcher' => [
        'middlewares' => [
            \App\Tcp\Middleware\GlobalTcpMiddleware::class
        ],
    ],
```

*   作用于控制器

```
/**
 * Class DemoController
 *
 * @TcpController(middlewares={DemoMiddleware::class})
 */
class DemoController
{
    // ....
}
```

## 开源代码库

[https://github.com/swoft-cloud/swoft](https://github.com/swoft-cloud/swoft)

# 更新日志

> *升级提示:*

*   `Swoole\WebSocket\Server::push`第四个参数`$finish`在 swoole `4.4.12`后改为 int 类型。
*   tcp 服务器的`TcpServerEvent::CONNECT`事件参数保持与接收和关闭相同。`$fd, $server`互换位置。

**固定:**

*   修复配置注入时，如果找不到值，将使用相应类型的默认值来覆盖属性，导致属性的默认值被覆盖 [d84d50a7](https://github.com/swoft-cloud/swoft-component/pull/522/commits/d84d50a76c4c7ff19dc0896868745cfe8f0d93c9)
*   修复了在 ws 服务器中使用消息调度时，没有空数据被过滤，导致多一个响应。避免方法[swoft-cloud/swoft # 1002](https://github.com/swoft-cloud/swoft/issues/1002)d84d 50 a 7
*   修复了在 tcp 服务器中使用消息调度时，没有空数据被过滤，导致多一个响应。 [07a01ba1](https://github.com/swoft-cloud/swoft-component/pull/522/commits/07a01ba1e6ff52baffbc7b2baf997e0e6a07ae04)
*   修复了独立使用控制台组件时丢失的 swoft/stdlib 库依赖关系 [c569c81a](https://github.com/swoft-cloud/swoft-component/pull/529/commits/c569c81ae15c0b2b73db3a15c457d7b982a06d7f)
*   修正了`ArrayHelper::get`当输入键为整数时，参数 parameter 不正确 [a44dcad](https://github.com/swoft-cloud/swoft-component/pull/528/commits/a44dcad42cbdd20cb4078351a8deb3b966b1ca09)
*   使用表格修复控制台渲染，计算 int 值时，计算宽度报告类型错误 [74a835ab](https://github.com/swoft-cloud/swoft-component/pull/528/commits/74a835abd78ed58c081668129e723d5e83429398)
*   修复了组件用户不能自定义默认错误处理级别的错误 [4c78aeb](https://github.com/swoft-cloud/swoft-component/pull/530/commits/4c78aeb3326bfb333227f07b675a2bdfc3b95f0f)
*   修复启用和禁用组件设置`isEnable()`不起作用 [da8c51e56](https://github.com/swoft-cloud/swoft-component/pull/530/commits/da8c51e561074f94ec29a8c5055f563c6ed13cc0)
*   在 cygwin 环境中使用`uniqid()`方法的修复必须将第二个参数设置为 true [c7f688f](https://github.com/swoft-cloud/swoft-component/pull/530/commits/c7f688f4ef07bb7c7d59a0c03888d0c79f77d4c7)
*   修复了无法在 cygwin 环境中设置进程标题并导致错误 [c466f6a](https://github.com/swoft-cloud/swoft-component/pull/530/commits/c466f6a3841b6fb58e581d517c2151699ac4cf3e)
*   修复了无法使用 http `response->delCookie()` [8eb9241](https://github.com/swoft-cloud/swoft-component/pull/530/commits/8eb9241329e46dcab78e4f18ea4771808a1f034e) 删除浏览器 cookie 数据的问题
*   修复了 ws 服务器消息调度，收到的 ext 数据不一定是导致错误的数组 [ff45b35](https://github.com/swoft-cloud/swoft-component/pull/530/commits/ff45b356c709d97bc2937dab8d467e4680d58cf0)
*   修复按时间分割的日志文件 [c195413](https://github.com/swoft-cloud/swoft-component/pull/533/commits/08c42449cc5ca7922e5bf54d6523b0d8799ba910)
*   修复日志`JSON`格式小问题 [a3fc6b9](https://github.com/swoft-cloud/swoft-component/pull/535/commits/a3fc6b94a8afff873f7f3ac8248432ff2a409d13)
*   修正`rpc`服务商`getList`呼叫两次 [fd03e71](https://github.com/swoft-cloud/swoft-component/pull/535/commits/fd03e71f525c265add11e9334cc6dd505daf62ec)
*   修复`redis cluster`不支持`auth`参数 [7a678f](https://github.com/swoft-cloud/swoft-component/pull/525/commits/7a678fd866e3ec842b00f734f7f6c3b7b1a03a9b)
*   修正模型查询`json`类型，不支持`array` [6023a9](https://github.com/swoft-cloud/swoft-component/pull/525/commits/6023a99aad06d9e87ff0a38bb4f37242f331a771)
*   固定 redis `multi`操作未及时连接 [e5f698](https://github.com/swoft-cloud/swoft-component/pull/525/commits/e5f69802947ec3d9b56b7d56ec2dc4b1d70b4995)
*   Fix redis 不支持`expireAt`，`geoRadius` [749241](https://github.com/swoft-cloud/swoft-component/pull/525/commit/749241561dbb5a6b94c659b2642e255900cb6b69)
*   修正了`crontab`时间戳检测偏差问题 [eb08a46](https://github.com/swoft-cloud/swoft-ext/commit/eb08a46833188f49e05b1ea3c041c8a60ff53606)

**更新:**

*   在呈现帮助消息`ConsoleEvent::SHOW_HELP_BEFORE` [d3f7bc3](https://github.com/swoft-cloud/swoft-component/pull/522/commits/d3f7bc3c5093a11a1de3710fd239c4375b835160) 之前，更新控制台还会发出一个事件
*   简化统一 http、ws、tcp、rpc 服务器管理命令逻辑 [f202c826](https://github.com/swoft-cloud/swoft-component/pull/522/commits/f202c826b74972775fe97ad91b2c38e5c7d97014)
*   更新 ws 和 tcp 连接类，添加`newFromArray`和`toArray`方法，以便于通过第三方存储(`redis`)【a 8 b 0 b 7 c】([swoft-cloud/swoft](https://github.com/swoft-cloud/swoft)-component/pull/528/commits/a 8 b 0 b 7 c 77d 56d 4392 EBA 75d 13 a 911816 b 9 DC 0 CEE)导出信息和恢复连接
*   优化服务器添加统一的 swoole pipe 消息事件处理程序，使用 ws、tcp 中的 swowt 事件处理进程间消息 [1c51a8c](https://github.com/swoft-cloud/swoft-component/pull/530/commits/1c51a8c82357f52b6f858bcaa58396c521080695)

**增强:**

*   现在 tcp 请求支持添加全局或者对应的方法中间件，流程和用法和 http 中间件差不多。*仅在使用系统调度*时有用 [6b593877](https://github.com/swoft-cloud/swoft-component/pull/528/commits/6b593877acc5cb78bbd863e08c0559454fb0b59c)
*   现在 websocket 消息请求支持添加全局或者对应的方法中间件，流程和用法和 http 中间件类似。*仅在使用系统调度*时有用 [9739815](https://github.com/swoft-cloud/swoft-component/pull/530/commits/973981568df4bca18a4858efe1ef7730d903353e)
*   事件管理允许设置`destroyAfterFire`在每次事件调度 [50bf43d3](https://github.com/swoft-cloud/swoft-component/pull/530/commits/50bf43d39857fea7328c83a790c941e83847b82b) 后清理事件中携带的数据
*   添加了数据库错误异常`code`返回 [fd306f4](https://github.com/swoft-cloud/swoft-component/pull/533/commits/fd306f470ba171f556bc05682aae58cab217cacc)
*   协程文件操作`writeFile`新写入失败异常 [08c4244](https://github.com/swoft-cloud/swoft-component/pull/533/commits/08c42449cc5ca7922e5bf54d6523b0d8799ba910)
*   RPC 新参数验证 [8646fc5](https://github.com/swoft-cloud/swoft-component/pull/533/commits/8646fc5c64bcfeb84c0c38adb31ff2759a20726d)

[](https://github.com/swoft-cloud/swoft) [## 软件云/软件

### PHP 微服务协程框架 Swoft 是一个基于 Swoole 扩展的 PHP 微服务协程框架…

github.com](https://github.com/swoft-cloud/swoft)
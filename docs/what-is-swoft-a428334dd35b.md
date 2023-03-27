# PHP 微服务框架— Swoft

> 原文：<https://itnext.io/what-is-swoft-a428334dd35b?source=collection_archive---------4----------------------->

![](img/53dff9c78f5c202931505f8bf059e8f2.png)

Swoft 是一个 PHP 高性能微服务协程框架。已经出版多年，已经成为 php 的不二之选。它可以像 Go 一样，内置协同 web 服务器和通用协同客户端，驻留在内存中，独立于传统的 PHP-FPM。还有类似的 Go 语言操作，类似于 Spring Cloud 框架的灵活注解。

# 开源代码库

[T3【https://github.com/swoft-cloud/swoft】T5](https://github.com/swoft-cloud/swoft)

# 特征

# 完整协程框架

Swoft 是第一个驻留在内存注释框架中的 PHP，具有比配置设计概念更大的 Spring Boot 约定，以及一组开发规范。

# 面向切面编程

AOP 是一种面向对象的编程，它使得分离业务代码、提高代码质量和增加代码可重用性变得更加容易。

```
/**
 * @Aspect(order=1)
 * @PointBean(include={"App\Http\Controller\TestExecTimeController"})
 */
class CalcExecTimeAspect
{
    protected $start; /**
     * @Before()
     */
    public function before()
    {
        $this->start = microtime(true);
    } /**
     * @After()
     */
    public function after(JoinPoint $joinPoint)
    {
        $method = $joinPoint->getMethod();
        $after = microtime(true);
        $runtime = ($after - $this->start) * 1000; echo "{$method} cost: {$runtime}ms\n";
    }
}
```

# Http 服务

Http 服务简单灵活，只需使用`@Controller()`和`@RequestMapping(route="index")`注释来定义服务。

```
/**
 * @Controller()
 */
class IndexController
{
    /**
     * @RequestMapping(route="index")
     */
    public function index(): string
    {
        return "test";
    }
}
```

# Websocket 服务

Swoft 为开发者提供了一个完整的 Websocket 来快速构建服务

```
/**
 * @WsModule(
 *     "/chat",
 *     messageParser=TokenTextParser::class,
 *     controllers={HomeController::class}
 * )
 */
class ChatModule
{
    /**
     * @OnOpen()
     * @param Request $request
     * @param int     $fd
     */
    public function onOpen(Request $request, int $fd): void
    {
        server()->push($request->getFd(), "Opened, welcome!(FD: $fd)");
    }
}
```

# RPC 服务

Swoft RPC 可以像 Dubbo 一样调用原生函数。

```
/**
 * @Controller()
 */
class RpcController
{
    /**
     * @Reference(pool="user.pool", version="1.0")
     *
     * @var UserInterface
     */
    private $userService; /**
     * @RequestMapping("getList")
     *
     * @return array
     */
    public function getList(): array
    {
        $result  = $this->userService->getList(12, 'type');
        return [$result];
    }
}
```

# TCP 服务

Swoft 还提供功能丰富的 TCP 服务支持。

```
<?php declare(strict_types=1);namespace App\Tcp\Controller;use Swoft\Tcp\Server\Annotation\Mapping\TcpController;
use Swoft\Tcp\Server\Annotation\Mapping\TcpMapping;
use Swoft\Tcp\Server\Request;
use Swoft\Tcp\Server\Response;/**
 * Class DemoController
 *
 * @TcpController()
 */
class DemoController
{
    /**
     * @TcpMapping("echo", root=true)
     * @param Request  $request
     * @param Response $response
     */
    public function echo(Request $request, Response $response): void
    {
        // $str = $request->getRawData();
        $str = $request->getPackage()->getDataString(); $response->setData('[echo]hi, we received your message: ' . $str);
    }
}
```

# 连接池

Swoft 定义高性能连接池很简单，如下所示:

```
return [
    'xxx.pool' => [
        'class'       => \Swoft\xxx\Pool::class,
        'minActive'   => 10,
        'maxActive'   => 20,
        'maxWait'     => 0,
        'maxWaitTime' => 0,
        'maxIdleTime' => 60,
    ]
];
```

# 与 Laravel ORM 兼容

Swoft 数据库与 Laravel ORM 高度兼容，PHP 开发人员可以很容易地在 Swoft 中使用。

```
// Model
$user = User::new();
$user->setName('name');
$user->setSex(1);
$user->setDesc('this my desc');
$user->setAge(mt_rand(1, 100));
$user->save();
$id = $user->getId();// Query
$users = DB::table('user')->get();
foreach ($users as $user) {
    echo $user->name;
}// Transaction
DB::beginTransaction();
$user = User::find($id);
$user->update(['name' => $id]);DB::beginTransaction();
User::find($id)->update(['name'=>'sakuraovq']);
DB::rollBack();DB::commit();
```

# 微服务

Swoft 提供了一套快速构建的微服务治理组件，便于开发者使用。

*   服务注册和发现
*   服务经纪人
*   集中式配置
*   服务节流能力

```
/**
 * @Bean()
 */
class Test
{
    /**
     * @Breaker(fallback="funcFallback")
     *
     * @return string
     * @throws Exception
     */
    public function func(): string
    {
        // Do something throw new Exception('Breaker exception');
    } /**
     * @RequestMapping()
     * @RateLimiter(key="request.getUriPath()")
     *
     * @param Request $request
     *
     * @return array
     */
    public function requestLimiter(Request $request): array
    {
        $uri = $request->getUriPath();
        return ['requestLimiter', $uri];
    }
}
```

# 开源代码库

[**https://github.com/swoft-cloud/swoft**](https://github.com/swoft-cloud/swoft)

# 基准

![](img/54011bedda4d88ac1290383871de9ff5.png)[](https://github.com/swoft-cloud/swoft) [## 软件云/软件

### PHP 微服务协程框架 Swoft 是一个基于 Swoole 扩展的 PHP 微服务协程框架…

github.com](https://github.com/swoft-cloud/swoft)
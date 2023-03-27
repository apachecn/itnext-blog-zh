# PHP 微服务框架 Swoft:使用数据库第 1 部分

> 原文：<https://itnext.io/php-microservice-framework-swoft-use-database-part-1-137cb861e30a?source=collection_archive---------2----------------------->

![](img/3e94a6f3f07f36a5f3b09b8b40486574.png)

我们要学习的这篇文章是:如何安装和运行 Swoft 数据库。

> *本文是关于 Swoft 数据库 ORM 的系列文章之一。来了解一下 Swoft 吧！*

# 什么是 Swoft？

Swoft 是一个 PHP 高性能微服务协程框架。已经出版多年，已经成为 php 的不二之选。它可以像 Go 一样，内置协同 web 服务器和通用协同客户端，驻留在内存中，独立于传统的 PHP-FPM。还有类似的 Go 语言操作，类似于 Spring Cloud 框架的灵活注解。

Swoft 通过三年的积累和方向探索，将 Swoft 打造成 PHP 界的春天云，是 PHP 高性能框架和微服务管理的不二之选。

# 开源代码库

[](https://github.com/swoft-cloud/swoft) [## 软件云/软件

### PHP 微服务协程框架 Swoft 是一个基于 Swoole 扩展的 PHP 微服务协程框架…

github.com](https://github.com/swoft-cloud/swoft) 

# 安装

这里我们使用 composer 来安装 db 组件

```
composer require swoft/db
```

Swoft 数据库基于 PDO 扩展驱动，请确保有 mysqld 和 PDO 扩展。

# 配置

数据库的配置放在`app\bean.php`文件中，您可以将配置的 db 看作一个`bean`对象。

```
return [
    'db'         => [
       'class'     => Database::class,
       'dsn'       => 'mysql:dbname=test;host=127.0.0.1',
       'username'  => 'root',
       'password'  => '123456',
       'charset'   => 'utf8mb4',
    ],
];
```

配置类似于`yii2`对象属性注入方法的配置，可以用`\bean('db')`得到当前配置的`Database`对象

*   `class`指定用于当前 bean 容器的类。当然，您也可以指定自己的数据库类实现。
*   `dsn`PDO 需要使用的连接配置信息
*   `username`数据登录用户名
*   `password`数据库登录密码
*   `charset`数据库字符集

如果要配置主从/集群，请参考 [Swoft 数据库 D](https://www.swoft.org/docs/2.x/en/db/setting.html) oc

# 简单易用

# 挑选

```
/**
 * @Controller()
 */
class UserController {

    /**
     * Displays a list of all users of the application
     *
     * @RequestMapping()
     * @return array
     */
    public function index()
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return (array)users;
    }
}
```

`select`方法总是会返回一个数组，数组中的每个结果都是一个数组，结果值可以这样访问:

```
foreach ($users as $user) {
    echo $user['name'];
}
```

# 交易

您可以使用 DB 的 transaction 方法在数据库事务中运行一组操作。如果在事务关闭时发生异常，事务将被回滚。如果事务关闭 closure 执行成功，事务将被自动提交。一旦使用了事务，您就不再需要担心手动回滚或提交问题:

```
DB::transaction(function () {
    DB::table('users')->update(['status' => 1]); DB::table('posts')->delete();
});
```

# 开源代码库

[](https://github.com/swoft-cloud/swoft) [## 软件云/软件

### PHP 微服务协程框架 Swoft 是一个基于 Swoole 扩展的 PHP 微服务协程框架…

github.com](https://github.com/swoft-cloud/swoft)
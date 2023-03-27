# 提示:将最新的 TLS 版本与 Azure Cache 一起用于 Redis

> 原文：<https://itnext.io/tip-using-the-latest-tls-version-with-azure-cache-for-redis-91a4b02fabb4?source=collection_archive---------7----------------------->

[Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-overview?WT.mc_id=medium-blog-abhishgu) 提供基于开源软件 Redis 的内存数据存储。

作为全行业推动专用传输层安全(`TLS`)版本`1.2`或更高版本的一部分，Redis 的 Azure Cache 将不支持`TLS`版本 1.0 和 1.1，即您的应用程序将被要求使用`TLS` 1.2 或更高版本来与您的缓存通信

> *阅读详细内容，* [*请参考本页来自*](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-remove-tls-10-11?WT.mc_id=devto-blog-abhishgu) 的产品文档

了解这将如何体现在你的围棋应用中可能会有所帮助(我用`[go-redis](https://github.com/go-redis/redis)` [客户端](https://github.com/go-redis/redis)作为例子)

# 如果您根本没有指定`TLS`

例如

```
c := redis.NewClient(&redis.Options{Addr: endpoint, Password: password})err := c.Ping().Err()
    if err != nil {
        log.Fatal(err)
    }
defer c.Close()
```

..你会遇到这个错误`i/o timeout`(可能没什么帮助)

# 如果指定的`TLS`版本低于`1.2`

例如

```
**tlsConfig := &tls.Config{MaxVersion: tls.VersionTLS11, MinVersion: tls.VersionTLS10}**c := redis.NewClient(&redis.Options{Addr: endpoint, Password: password, TLSConfig: tlsConfig})err := c.Ping().Err()
 if err != nil {
    log.Fatal(err)
 }defer c.Close()
```

..您将最终导致一个`tls: DialWithDialer timed out`错误(同样，不是那么明显)

# 不过，解决方案很明显

如果不设置`MaxVersion`或`MinVersion`，即使用`tlsConfig := &tls.Config{}`，它将从`MaxVersion`默认到`TLS1.3`开始工作(参见[https://golang.org/pkg/crypto/tls/#Config](https://golang.org/pkg/crypto/tls/#Config))

为了清楚起见，最好是明确的，即

```
tlsConfig := &tls.Config{MinVersion: tls.VersionTLS12}
```

如果您在使用 Go 连接到 Azure Cache for Redis 时遇到任何问题，我希望这能对您有所帮助

干杯！
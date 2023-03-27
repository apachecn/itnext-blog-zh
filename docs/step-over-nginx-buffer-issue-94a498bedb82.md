# 跳过 nginx 缓冲区问题

> 原文：<https://itnext.io/step-over-nginx-buffer-issue-94a498bedb82?source=collection_archive---------0----------------------->

![](img/b375838b997685fb94aa1afac51513fe.png)

# 背景

在我的日常开发工作中，我一直使用 nginx 作为反向代理。今天升级 nginx 后，它不知何故停止了正常工作:从 webpack 构建的一个 vendor.js 文件没有得到正确的服务。我从 chrome 控制台得到以下错误:

```
GET http://nginx.example.com/dist/js/vendor.js net::ERR_CONTENT_LENGTH_MISMATCH
```

我快速环顾四周，发现:

*   网络选项卡显示 vendor.js 请求的状态代码为 200。
*   回复头好像没问题。不过 chrome 网络标签里没有什么可以预览的。
*   vendor.js 的内容长度与文件的实际大小相匹配。
*   `# ls -l : get the size in byte -rw-r--r-- 1 michaelzheng staff 4844863 Jul 2 17:00 vendor.js # ls -lh : get the size in human readable format -rw-r--r-- 1 michaelzheng staff 4.6M Jul vendor.js`
*   直接通过浏览器或者`curl`加载 vendor.js 效果很好。

# 调试

*   找到 nginx 配置文件的位置(即 nginx.conf)
*   `nginx -t`
*   找到错误日志的位置，它通常在 nginx.conf 中定义
*   检查错误
*   `# tail -f nginx 2018/07/02 22:29:27 [crit] 14586#0: *3641 open() "/usr/local/var/run/nginx/proxy_temp/1/08/0000000081" failed (13: Permission denied) while reading upstream, client: 127.0.0.1, server: nginx.example.com, request: "GET /dist/js/vendor.js HTTP/1.1", upstream: "http://127.0.0.1:8080/dist/js/vendor.js", host: "nginx.example.com", referrer: "http://nginx.example.com/testabc"`
*   现在我们可以清楚地看到这与缓冲目录`proxy_temp`的权限问题有关。

# 解决方法

*   解决方案 1 —授予权限
*   `chown -R nobody:admin proxy_temp`
*   您需要首先运行命令`ps aux | grep nginx`来找出 nginx 进程的所有者，通常是`nobody`
*   解决方案 2 —禁用缓冲，将 [proxy_buffering](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffering) 设置为`off`
*   解决方案 3 —使用 [proxy_temp_path](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_temp_path) 指令，将缓冲区目录更改为 nginx 流程所有者有权访问的其他目录。

# 通知；注意

*   如果您想关注我的博客系列的最新消息/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。
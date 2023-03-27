# Nginx Docker 和环境变量

> 原文：<https://itnext.io/nginx-docker-and-environment-variables-9753dfb5d41?source=collection_archive---------0----------------------->

## 使用 perl 模块读取环境变量

![](img/191ca4a5e5a789257da44795d45a10bd.png)

如果您在 google 上搜索 Nginx docker 和环境变量，您将最终得到 [**envsubst**](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html#:~:text=The%20envsubst%20program%20substitutes%20the%20values%20of%20environment%20variables.&text=Output%20the%20variables%20occurring%20in%20shell%2Dformat%20.&text=In%20normal%20operation%20mode%2C%20standard,replaced%20with%20the%20corresponding%20values.) 解决方法来将环境变量传递给 docker 容器。尽管这种变通方法可行，但它并不那么灵活，也不容易操作。

## docker 图像:

我喜欢**基于阿尔卑斯山的**图像，因为它们重量轻且安全。你可能想知道为什么我使用支持 **Perl** 的镜像，答案很简单:为了能够读取 Nginx 中的环境变量，需要运行在 **Perl** 上的`nginx-mod-http-perl`模块！

```
FROM nginx:1.18.0-alpine-perlRUN apk add --no-cache nginx-mod-http-perlCOPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.confEXPOSE 80/tcp
EXPOSE 443/tcpCMD ["/bin/sh", "-c", "exec nginx -g 'daemon off;';"]
WORKDIR /usr/share/nginx/html
```

## Nginx 配置

Nginx 主配置文件是`nginx.conf`，位于`/etc/nginx/nginx.conf`。我们使用这个文件来加载 Nginx 模块和配置环境变量。

要加载该模块，只需使用以下代码行:

```
load_module "modules/ngx_http_perl_module.so";
```

为您需要的每个环境变量定义一个变量:

```
env ENV;
env BACKEND;
env FRONTEND;
env FOO_BAR;
```

为您定义的每个变量定义一个 perl-set:

```
perl_set $ENV 'sub { return $ENV{"ENV"}; }';
perl_set $BACKEND 'sub { return $ENV{"BACKEND"}; }';
perl_set $FRONTEND 'sub { return $ENV{"FRONTEND"}; }';
```

如果你想了解更多关于`perl_set` r [的信息，请阅读 Nginx](http://nginx.org/en/docs/http/ngx_http_perl_module.html#perl_set) 的这篇文档。变量可以在任何其他。Nginx 中的 conf 文件。要使用该变量，只需使用`${VARIABLE_NAME}`即可。

要将变量传递给 docker 容器，请使用`-e`参数来传递它们。

```
docker run --rm -it --name foo -p 8080:80 -e "ENV=PROD" -e "FRONTEND=[h](https://crow-asfalt-impuls-tooling-app-test.azurewebsites.net)ttp://example.com" -e "BACKEND=http://api.example.com" foo
```

配置文件的完整工作示例:

## nginx.conf

```
worker_processes  1;error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;load_module "modules/ngx_http_perl_module.so";env ENV;
env BACKEND;
env FRONTEND;events {
    worker_connections 1024;
}http {
    default_type application/octet-stream;log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
              '$status $body_bytes_sent "$http_referer" '
              '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log  main;

    sendfile on;
    keepalive_timeout 65;perl_set $ENV 'sub { return $ENV{"ENV"}; }';
    perl_set $BACKEND 'sub { return $ENV{"BACKEND"}; }';
    perl_set $FRONTEND 'sub { return $ENV{"FRONTEND"}; }';include /etc/nginx/conf.d/*.conf;
}
```

## 默认. conf

```
server {    
    listen       80;
    server_name  localhost;resolver 1.1.1.1;location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        proxy_pass ${FRONTEND}$request_uri;
    }location /api {        
        proxy_pass ${BACKEND}$request_uri; 
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host  $host;
        proxy_set_header X-Forwarded-Port  $server_port;
        proxy_ssl_session_reuse off;
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
    }
```

注意:在上面的例子中，我使用`resolver: 1.1.1.1`作为 DNS 解析器，因为我在 Nginx 中为反向代理设置传递动态变量。
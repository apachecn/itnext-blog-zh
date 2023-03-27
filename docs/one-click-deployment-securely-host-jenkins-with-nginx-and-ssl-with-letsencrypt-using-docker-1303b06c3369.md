# 一键式部署:使用 Docker Compose 通过 Nginx 安全托管 Jenkins，通过 Letsencrypt 安全托管 SSL

> 原文：<https://itnext.io/one-click-deployment-securely-host-jenkins-with-nginx-and-ssl-with-letsencrypt-using-docker-1303b06c3369?source=collection_archive---------2----------------------->

![](img/df86865d7692017022589de0d580ddea.png)

您是否希望部署一个具有代理和 SSL 支持的安全 Jenkins 实例？不要再看了！在本教程中，我们将指导您使用 Letsencrypt 和 Docker Compose 通过 Nginx 代理和 SSL 设置 Jenkins 的过程。无论您是希望实现 CI/CD 管道自动化的开发人员，还是希望管理基础设施的系统管理员，本教程都能满足您的需求。只需几个简单的步骤，您马上就可以启动并运行一个全功能的 Jenkins 实例。所以让我们开始吧！

# 为什么需要让 Jenkins 使用 Nginx 反向代理

有几个原因可能会让 Jenkins 使用 Nginx 反向代理。其中一个主要原因是为了提高 Jenkins 实例的安全性。通过将 Jenkins 置于 Nginx 代理之后，您可以为 Jenkins 服务器增加一层额外的保护。Nginx 可以配置为阻止某些类型的流量，如恶意请求或易受攻击的 HTTP 方法，这有助于防止对 Jenkins 实例的攻击。此外，使用 Nginx 反向代理可以让您轻松地为 Jenkins 实例配置 SSL，这对于加密 Jenkins 服务器和客户端之间的流量至关重要。这有助于保护密码和 API 密钥等敏感数据不被第三方截获。总的来说，使用 Nginx 反向代理可以显著提高 Jenkins 设置的安全性，强烈推荐用于任何生产部署。

**先决条件**:

*   Docker 和 Docker Compose 安装在您的机器上
*   指向服务器 IP 地址的域名

# **步骤 1:为 Jenkins 和 Nginx 代理**创建一个目录

让我们为 Jenkins 和 Nginx 代理文件创建一个目录:

```
mkdir jenkins-nginx
cd jenkins-nginx
```

# 步骤 2:创建 Docker 合成文件

接下来，让我们创建一个 Docker 合成文件来定义 Jenkins 和 Nginx 容器:

```
nano docker-compose.yml
```

将以下内容粘贴到文件中:

```
version: '3.7'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    volumes:
      - jenkins_data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - VIRTUAL_HOST=${VIRTUAL_HOST}
      - LETSENCRYPT_HOST=${LETSENCRYPT_HOST}
      - LETSENCRYPT_EMAIL=${LETSENCRYPT_EMAIL}
      - VIRTUAL_PORT=8080
    networks:
      - jenkins
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - cert:/etc/nginx/certs
      - /var/run/docker.sock:/tmp/docker.sock:ro
    networks:
      - jenkins
  acme-companion:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: acme-companion
    restart: always
    environment:
      - NGINX_PROXY_CONTAINER=nginx-proxy
      - NGINX_DOCKER_GEN_CONTAINER=nginx-proxy
      - NGINX_DOCKER_GEN_NETWORK=jenkins_default
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - cert:/etc/nginx/certs
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - nginx-proxy
    networks:
      - jenkins

volumes:
  jenkins_data:
  conf:
  vhost:
  html:
  cert:

networks:
  jenkins:
    driver: bridge
```

在上面的 Docker Compose 文件中，我们定义了三个服务:

*   **Jenkins** :这是将运行 Jenkins 服务器的 Jenkins 容器。
*   Nginx 代理:这是 Nginx 代理容器，将充当 Jenkins 的反向代理。
*   Letsencrypt-nginx-proxy-companion:这是一个使用 lets encrypt 来帮助我们的域生成和更新 SSL 证书的容器。

我们还定义了容器将使用的一些卷和网络。

# 步骤 3:创建一个。环境文件

接下来，让我们创建一个。env 文件来存储我们的环境变量:

```
nano .env
```

将以下内容粘贴到文件中，用您的实际域名替换`your-domain.com`:

```
VIRTUAL_HOST=your-domain.com
LETSENCRYPT_HOST=your-domain.com
LETSENCRYPT_EMAIL=your@email.com
```

# 步骤 4:启动容器

现在，我们可以使用以下命令启动 Jenkins、Nginx 代理和 Letsencrypt 容器:

```
docker-compose up -d
```

这将提取所需的图像并在后台启动容器。您可以使用以下命令检查容器的状态:

```
docker-compose ps
```

# 步骤 5:完成 Jenkins 设置

一旦 Jenkins 容器启动并运行，您就可以通过在 web 浏览器中打开您的域名来完成设置。您将看到 Jenkins 安装向导，它将指导您完成其余的安装过程。

**获取詹金斯管理员密码:**

```
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

# 结论:

在本教程中，我们向您展示了如何使用 docker-compose 通过 Nginx 代理和 SSL ready 通过 Letsencrypt 轻松部署 Jenkins。这种设置允许您使用您的域名轻松访问 Jenkins，并确保所有流量都使用 SSL 加密。我希望本教程对你有所帮助，并且你能够在你的服务器上成功设置 Jenkins。

【github.com】Github Repo:[omerkarabacak/one-click-Jenkins-deployment](https://github.com/omerkarabacak/one-click-jenkins-deployment)

**支持媒体和我:)**[https://omerkarabacak.medium.com/membership](https://omerkarabacak.medium.com/membership)
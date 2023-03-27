# SonarQube:在 Docker 中运行来自 Jenkins 管道的测试

> 原文：<https://itnext.io/sonarqube-running-tests-from-jenkins-pipeline-from-docker-7740702b6f42?source=collection_archive---------1----------------------->

![](img/78429a8eb528e8d0b65bc692cb92085d.png)

任务是使用来自 Jenkins 管道作业的 SonarQube 运行我们的后端 PHP 测试。

Jenkins 运行在 Docker 中，它的所有构建也使用 Docker。

在这个设置过程中，我面临的主要问题是，使用 sonar cube 的容器运行两个服务:一个是 sonar cube 本身，另一个是 Elasticesearch(而 containers concept 说“*每个容器一个服务*”)。

在此期间，Elasticsearch 无法从 root 用户启动，所以我不得不使用 uid 和挂载点。

## 内容

*   [码头工人为 SonarQube](https://rtfm.co.ua/en/sonarqube-running-tests-from-jenkins-pipeline-from-docker/#Docker_Compose_for_SonarQube) 作曲
*   [NGINX](https://rtfm.co.ua/en/sonarqube-running-tests-from-jenkins-pipeline-from-docker/#NGINX)
*   [詹金斯码头工人作曲](https://rtfm.co.ua/en/sonarqube-running-tests-from-jenkins-pipeline-from-docker/#Jenkins_Docker_Compose)
*   [詹金斯配置](https://rtfm.co.ua/en/sonarqube-running-tests-from-jenkins-pipeline-from-docker/#Jenkins_configuration)
*   [sonar cube 中的项目配置](https://rtfm.co.ua/en/sonarqube-running-tests-from-jenkins-pipeline-from-docker/#A_project_configuration_in_SonarQube)

## 为 SonarQube 编写 Docker

检查 SonarQube Docker 映像中默认用户的 UID:

```
root@jenkins-dev:/opt/sonarqube# docker run -ti sonarqube id
uid=999(sonarqube) gid=999(sonarqube) groups=999(sonarqube)
```

创建目录以保存 SonarQube 的数据:

```
root@jenkins-dev:~# mkdir -p /data/sonarqube/{conf,logs,temp,data,extensions,bundled_plugins,postgresql,postgresql_data}
```

创建新用户并更改这些目录的所有者:

```
root@jenkins-dev:~# adduser sonarqube
root@jenkins-dev:~# usermod -aG docker sonarqube
root@jenkins-dev:~# chown -R sonarqube:sonarqube /data/sonarqube/
```

查找上面创建的用户的 UID:

```
root@jenkins-dev:/opt/sonarqube# id sonarqube
uid=1004(sonarqube) gid=1004(sonarqube) groups=1004(sonarqube),999(docker)
```

使用`user`中的 UID 创建一个 Docker 合成文件:

```
version: "3"

networks:
  sonarnet:
    driver: bridge

services:
  sonarqube:
    // use UID here
    user: 1004:1004
    image: sonarqube
    ports:
      - "9000:9000"
    networks:
      - sonarnet
    environment:
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar
    volumes:
      - /data/sonarqube/conf:/opt/sonarqube/conf
      - /data/sonarqube/logs:/opt/sonarqube/logs
      - /data/sonarqube/temp:/opt/sonarqube/temp
      - /data/sonarqube/data:/opt/sonarqube/data
      - /data/sonarqube/extensions:/opt/sonarqube/extensions
      - /data/sonarqube/bundled_plugins:/opt/sonarqube/lib/bundled-plugins

  db:
    image: postgres
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - /data/sonarqube/postgresql:/var/lib/postgresql
      - /data/sonarqube/postgresql_data:/var/lib/postgresql/data
```

检查一下:

```
root@jenkins-dev:/opt/sonarqube# docker-compose -f sonarqube-compose.yml up
Starting sonarqube_db_1 … done
Recreating sonarqube_sonarqube_1 … done
…
sonarqube_1 | 2019.06.14 15:33:46 INFO app[][o.s.a.SchedulerImpl] Process[ce] is up
sonarqube_1 | 2019.06.14 15:33:46 INFO app[][o.s.a.SchedulerImpl] SonarQube is up
```

## NGINX

现在，需要配置 NGINX 和 SSL 来访问 SonarQube 的 web 界面。

停止 NGINX，安装 Let's Encrypt client，获取一个证书(更多细节参见 AWS EC2 上的 [Bitwarden:一个组织的密码管理器自托管版本安装):](https://rtfm.co.ua/en/bitwarden-an-organizations-password-manager-self-hosted-version-installation-on-an-aws-ec2/#Lets_Encrypt)

```
root@jenkins-dev:/opt/sonarqube# systemctl stop nginx
root@jenkins-dev:/opt/sonarqube# /opt/letsencrypt/letsencrypt-auto certonly -d sonar.example.com
root@jenkins-dev:/opt/sonarqube# systemctl start nginx
```

创建一个虚拟主机配置，这里只是通过复制已经存在的:

```
root@jenkins-dev:/opt/sonarqube# cp /etc/nginx/conf.d/ci.example.com.conf /etc/nginx/conf.d/sonar.example.com.conf
```

更新它，使它看起来像 next(在 SSL 部分可能有点过时，但仍然给出+结果):

```
upstream sonar {
    server 127.0.0.1:9000;
}

server {

    listen 80;
    server_name  dev.sonar.example.com;

    # Lets Encrypt Webroot
    location ~ /.well-known {

    root /var/www/html;
        allow all;
    }

    location / {
        return 301 [https://dev.sonar.example.com;](https://dev.sonar.example.com;)
    }
}

server {

    listen       443 ssl;
    server_name  dev.sonar.example.com;

    access_log  /var/log/nginx/dev.sonar.example.com-access.log proxy;
    error_log /var/log/nginx/dev.sonar.example.com-error.log warn;

    ssl_certificate /etc/letsencrypt/live/dev.sonar.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dev.sonar.example.com/privkey.pem;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/nginx/dhparams.pem;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:ECDHE-RSA-AES128-GCM-SHA256:AES256+EECDH:DHE-RSA-AES128-GCM-SHA256:AES256+EDH:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
    ssl_session_timeout 1d;
    ssl_stapling on;
    ssl_stapling_verify on;

    location / {
        proxy_http_version 1.1;
        proxy_request_buffering off;
        proxy_buffering off;

        proxy_redirect          off;
        proxy_set_header        Host            $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto   $scheme;
        proxy_pass [http://sonar$request_uri;](http://sonar$request_uri;)
    }
}
```

检查语法并重新加载 NGINX 的配置:

```
root@jenkins-dev:/home/admin# nginx -t && systemctl start nginx
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

## 詹金斯·道克尔作曲

现在是时候将 sonar cube 添加到 Jenkins Docker Compose 文件中了(尽管可以作为一个专用服务使用)—将 sonar cube 和 PostgreSQL 移动到这个文件中:

```
version: '3'

networks:
 jenkins:

services:

  jenkins:
    user: root
    image: jenkins/jenkins:2.164.3
    networks:
      - jenkins
    ports:
      - '8080:8080'
      - '50000:50000'
    volumes:
      - /data/jenkins:/var/lib/jenkins
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /usr/lib/x86_64-linux-gnu/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7
    environment:
      - JENKINS_HOME=/var/lib/jenkins
      - JAVA_OPTS=-Duser.timezone=Europe/Kiev
      - JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dhudson.model.DirectoryBrowserSupport.CSP=\"default-src 'self' 'unsafe-inline' 'unsafe-eval'; img-src 'self' 'unsafe-inline' data:;\""
    logging:
      driver: "journald"

  sonarqube:
    user: 1004:1004
    image: sonarqube
    ports:
      - "9000:9000"
    networks:
      - jenkins
    environment:
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar
    volumes:
      - /data/sonarqube/conf:/opt/sonarqube/conf
      - /data/sonarqube/logs:/opt/sonarqube/logs
      - /data/sonarqube/temp:/opt/sonarqube/temp
      - /data/sonarqube/data:/opt/sonarqube/data
      - /data/sonarqube/extensions:/opt/sonarqube/extensions
      - /data/sonarqube/bundled_plugins:/opt/sonarqube/lib/bundled-plugins
    logging:
      driver: "journald"

  db:
    image: postgres
    networks:
      - jenkins
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - /data/sonarqube/postgresql:/var/lib/postgresql
      - /data/sonarqube/postgresql_data:/var/lib/postgresql/data
    logging:
      driver: "journald"
```

重启 Jenkins 服务(参见[Linux:systemdсервсдляdocker 撰写帖子](https://rtfm.co.ua/linux-systemd-servis-dlya-docker-compose/) ( *Rus* )):

```
root@jenkins-dev:/opt/jenkins# systemctl restart jenkins
```

检查容器:

```
root@jenkins-dev:/home/admin# docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
fc2662391c45 sonarqube “./bin/run.sh” 48 seconds ago Up 46 seconds 0.0.0.0:9000->9000/tcp jenkins_sonarqube_1
3ac2bb5f0e87 postgres “docker-entrypoint.s…” 48 seconds ago Up 46 seconds 5432/tcp jenkins_db_1
113496304b0f jenkins/jenkins:2.164.3 “/sbin/tini — /usr/…” 48 seconds ago Up 46 seconds 0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp jenkins_jenkins_1
```

进入声纳的网络界面:

![](img/54f2249b3a9221dc4918ea9ec3989cb6.png)

用 *admin:admin* 登录。

## 詹金斯构型

通常，对于 Jenkins，使用的是[sonar cube Scanner](https://plugins.jenkins.io/sonar)插件，但是我们将从 Docker 容器运行 Scanner，所以不需要安装这个插件。

## 作业配置

创建管道作业:

![](img/0b2252c57dd20f6edca8c745045662f8.png)

创建管道脚本:

```
node {

    stage('Clone repo') {
        git branch: "develop", url: "git@github.com:example-dev/example-me.git", credentialsId: "jenkins-github"
    }

    stage('SonarTests') {
        docker.image('newtmitch/sonar-scanner').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
            sh "--version"
        }
    }
}
```

再次面临入口点错误:

> $ docker top d 2269 Fe 30970490 ba 9957 AC 57701 EC 6091 f 3c CBF 78 f 957 e 903 a 3319 fa 1445 BD-EO PID，comm
> 错误:容器已启动，但没有运行预期的命令。请仔细检查您的入口点是否按照官方 docker 映像的要求，执行了作为 docker run 参数传递的命令(请参见[https://github . com/docker-library/official-images # consistency](https://github.com/docker-library/official-images#consistency)了解入口点一致性要求)。
> 或者，您可以通过添加选项`— entrypoint= ' ' '来强制禁用图像入口点。

让我们使用来自 [Jenkins 的解决方案:通过 Github 和 Allure-reports](https://rtfm.co.ua/en/jenkins-running-phpunit-from-codeception-by-a-pull-reguest-in-github-and-allure-reports/#Jenkins_Docker_Plugin_The_container_started_but_didnt_run_the_expected_command_Please_double_check_your_ENTRYPOINT) 帖子中的 Pull 请求从 Codeception 运行 PHPUnit。

运行容器，将`entrypoint`设置为`bash`:

```
docker run -ti — entrypoint=”bash” newtmitch/sonar-scanner
root@20baaae7de9a:/usr/src#
```

找到扫描仪的可执行文件:

```
root@20baaae7de9a:/usr/src# which sonar-scanner
/usr/local/bin/sonar-scanner
```

更新 Jenkins 脚本—在`sh` - `/usr/local/bin/sonar-scanner`中设置`--entrypoint=""`和完整路径:

```
node {

    stage('Clone repo') {
        git branch: "develop", url: "git@github.com:example-dev/example-me.git", credentialsId: "jenkins-github"
    }

    stage('SonarTests') {
        docker.image('newtmitch/sonar-scanner').inside('-v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""') {
            sh "/usr/local/bin/sonar-scanner --version"
        }
    }
}
```

运行构建，检查结果:

![](img/2a7698c5b316aa25efce58cffd3fddb6.png)

很好——很有效。

但现在另一个问题来了:

1.  詹金斯在*詹金斯 _ 詹金斯*网络中作为码头集装箱运行
2.  SonarQube 在 *jenkins_jenkins* 网络中作为 Docker 容器运行
3.  Jenkins 创建了一个带有`sonar-scanner`的新容器，它必须能够访问 SonarQube 容器，而 SonarQube 容器又运行在一个“外部”网络中 *jenkins_jenkins* (从扫描器的容器角度来看)

检查 Jenkins 主机上 Docker 中的现有网络:

```
root@jenkins-dev:/home/admin# docker network ls
NETWORK ID NAME DRIVER SCOPE
67900babbbc4 bridge bridge local
d51bc8ee54d0 host host local
30b091d801d6 jenkins_jenkins bridge local
16ab0c37234e none null local
```

解决方案是更新扫描仪的容器设置—将其网络设置为 *jenkins_jenkins* ( `--net jenkins_jenkins`):

```
node {

    stage('Clone repo') {
        git branch: "develop", url: "git@github.com:example-dev/example-me.git", credentialsId: "jenkins-github"
    }

    stage('SonarTests') {
        docker.image('newtmitch/sonar-scanner').inside('-v /var/run/docker.sock:/var/run/docker.sock --entrypoint="" --net jenkins_jenkins') {
            sh "/usr/local/bin/sonar-scanner -Dsonar.host.url=http://sonarqube:9000 -Dsonar.sources=."
        }
    }
}
```

运行它:

![](img/51911e92ff2379fcb6a6ca8c94aef8cb.png)

很好—声纳扫描仪正在运行，并且能够连接到 SonarQube 容器。

## SonarQube 中的项目配置

添加新项目:

![](img/2cb3a3dad7916eb73bb64b6edf266dce.png)![](img/c67eb4403a1f34ade8bc9d6bf6c1bf90.png)

创建其令牌:

![](img/cb842b5b7561ebead8cf5fd5ebe231a4.png)

选择平台，SonarQube 将显示必要的设置:

![](img/58105e88e7ed02c226997c10c6bf61f0.png)

更新脚本，运行作业，然后:

> 错误:SonarQube 扫描仪执行期间出错
> 错误:没有找到质量配置文件，您可能没有安装任何语言插件。

好吧…

回到 SonarQube 到*管理>市场*并安装 SonarPHP:

![](img/fb653c832c18210b91e5c2ed12b674ca.png)

重启声纳:

![](img/3770046758b5078b58b5157c7de6ef5d.png)

立即检查已安装的插件:

```
root@jenkins-dev:/home/admin# curl localhost:9000/api/plugins/installed
{“plugins”:[{“key”:”php”,”name”:”SonarPHP”,”description”:”Code Analyzer for PHP”,”version”:”3.0.0.4537",”license”:”GNU LGPL v3",”organizationName”:”SonarSource and Akram Ben Aissi”,”editionBundled”:false,”homepageUrl”:”http://redirect.sonarsource.com/plugins/php.html","issueTrackerUrl":"http://jira.codehaus.org/browse/SONARPHP","implementationBuild":"026dee08c29a3689ab1228552e14bfefda9ae57e","updatedAt":1560775404315,"filename":"sonar-php-plugin-3.0.0.4537.jar","sonarLintSupported":true,"hash":"c80c0d053f074a9147d341cf1357d994"}]}
```

> " name":"SonarPHP "，" description ":" PHP 代码分析器"

好，安装完毕，再次运行作业:

![](img/bc47046540375a8768caa08b2bd3b40e.png)

并导致 SonarQube:

![](img/ae4abc0260376b31683cbb3116de7a87.png)

## 声纳扫描仪:索引了 0 个文件

但是为什么 *0 扫描了*？

> …
> 信息:索引文件…
> 信息:项目配置:
> 信息:索引 0 个文件
> …

从 Jenkins 到其扫描仪容器的目录映射有问题。

让我们尝试显式设置`sonar.sources`和`sonar.projectBaseDir`:

```
...
    stage('SonarTests') {
        docker.image('newtmitch/sonar-scanner').inside('-v /var/run/docker.sock:/var/run/docker.sock --entrypoint="" --net jenkins_jenkins') {
            sh "/usr/local/bin/sonar-scanner -Dsonar.host.url=http://sonarqube:9000 -Dsonar.sources=/var/lib/jenkins/workspace/SonarTest -Dsonar.projectBaseDir=/var/lib/jenkins/workspace/SonarTest -Dsonar.projectKey=ProjectName -Dsonar.login=91a691e7c91154d3fee69a05a8fa6e2b10bc82a6 -Dsonar.verbose=true"
        }
    }
```

运行它:

![](img/70a762c29f7b7f4431d0c0aa989a6ec3.png)

作品。

再一次 SonarQube 的结果:

![](img/20b0c921828dcaef8802c819e8538c3a.png)

实际上——就这些了，现在需要把这一切弄得更干净。

所以这个问题的出现是因为 Jenkins Docker 插件将一个当前的工作目录映射到一个同名的容器中，然后将其设置为一个`--workdir`:

> …
> $ docker 运行[…]-w/var/lib/Jenkins/workspace/SonarTest
> …

当`sonar-scanner`试图在基于默认*的目录*中找到它时:

> …
> 09:08:56.666 信息:基本目录:/usr/src
> …

让我们尝试将`sonar.projectBaseDir`设置为“`.`”值，即-当前目录。

现在脚本看起来像下面这样:

```
node {

    stage('Clone repo') {
        git branch: "develop", url: "git@github.com:example-dev/example-me.git", credentialsId: "jenkins-github"
    }

    stage('SonarTests') {
        docker.image('newtmitch/sonar-scanner').inside('-v /var/run/docker.sock:/var/run/docker.sock --entrypoint="" --net jenkins_jenkins') {
            sh "/usr/local/bin/sonar-scanner -Dsonar.host.url=http://sonarqube:9000 -Dsonar.projectBaseDir=. -Dsonar.projectKey=ProjectName -Dsonar.login=91a691e7c91154d3fee69a05a8fa6e2b10bc82a6"
        }
    }
}
```

运行它:

> …
> 信息:基本目录:/var/lib/Jenkins/workspace/SonarTest
> 信息:工作目录:/var/lib/Jenkins/workspace/SonarTest/。扫描器工作…
> 信息:执行成功

项目的设置可以移动到项目的 Github 资源库根目录下的`sonar-project.properties`文件中:

```
sonar.host.url=http://sonarqube:9000
sonar.projectBaseDir=.
sonar.projectKey=ProjectName
sonar.login=91a691e7c91154d3fee69a05a8fa6e2b10bc82a6
```

并更新我们的脚本以删除所有参数——扫描仪将寻找`sonar-project.properties`来使用它的设置:

```
node {

    stage('Clone repo') {
        git branch: "develop", url: "git@github.com:example-dev/example-me.git", credentialsId: "jenkins-github"
    }

    stage('SonarTests') {
        docker.image('newtmitch/sonar-scanner').inside('-v /var/run/docker.sock:/var/run/docker.sock --entrypoint="" --net jenkins_jenkins') {
            sh "/usr/local/bin/sonar-scanner"
        }
    }
}
```

完成了。

*最初发布于* [*RTFM: Linux、DevOps 和系统管理*](https://rtfm.co.ua/en/sonarqube-running-tests-from-jenkins-pipeline-from-docker/) *。*
# 在弹簧靴中切换构建配置

> 原文：<https://itnext.io/switching-build-configurations-in-spring-boot-e7a607c6af82?source=collection_archive---------1----------------------->

![](img/3a21a7b6058476b40484b5e368bbcf5d.png)

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fswitching-build-configurations-in-spring-boot-e7a607c6af82%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

**总结**

*   设置环境变量$SPRING_PROFILES_ACTIVE
*   任何应用-{something}。{something}匹配的 yml 将优先于 application.yml

**示威游行**

*我不会在这里展示任何应用程序逻辑，但如果有人想初始化一个简单的 spring-boot 应用程序，这里是一个很好的地方。

[https://start.spring.io/](https://start.spring.io/)

构建目标通常至少由开发/生产来分隔，下面是如何使用 spring-boot 和 application.yml 来做到这一点。

例如，让我们根据构建配置选择不同的数据库。我将使用“demodb”数据库进行生产，使用“tst_demodb”进行开发。

应用程序. yml

```
**spring:
  datasource:
    url:** jdbc:mysql://127.0.0.1:3306/{DB_name} //demodb
    **username:** hoge
    **password:** hoge
    **hikari:
      minimum-idle:** 10
      **maximum-pool-size:** 10
      **connection-timeout:** 3000
```

应用程序开发人员

```
**spring:
  datasource:
    url:** jdbc:mysql://127.0.0.1:3306/{DB_name} //tst_demodb
    **username:** fuga
    **password:** fuga
    **hikari:
      minimum-idle:** 10
      **maximum-pool-size:** 10
      **connection-timeout:** 3000
```

两个 yamls 都放在

```
{project_root}/src/main/resources/config
```

现在创建一个简单的部署脚本，如下所示:

初始化. sh

```
#!/bin/bash**export SPRING_PROFILES_ACTIVE='dev'**./gradlew bootRun
```

很抱歉这么晚才提到使用 gradle，但是只要您的应用程序可以启动，maven 或任何东西都可以。

这里的关键点用粗体显示。

通过设置 SPRING_PROFILES_ACTIVE，任何应用-{something}。将使用 xml。

在本例中，一个名为 application- **dev** 的文件。将读取 yml 而不是 application.yml

以下是输出结果:

```
.   ____          _            __ _ _/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \\\/  ___)| |_)| | | | | || (_| |  ) ) ) )'  |____| .__|_| |_|_| |_\__, | / / / /=========|_|==============|___/=/_/_/_/:: Spring Boot ::        (v2.0.0.RELEASE) 2018-04-07 19:09:06.764  INFO 17839 --- [           main] c.e.mybatisdemo.MybatisdemoApplication   : **The following profiles are active: dev**
```

检查应用程序中的任何 d b-insert 或 updates 只发生在 tst_demodb 上，这是在 application-dev.yml 中指定的

**总结**

*   设置环境变量$SPRING_PROFILES_ACTIVE
*   任何应用-{something}。{something}匹配的 yml 将优先于 application.yml
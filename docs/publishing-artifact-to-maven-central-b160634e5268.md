# 将工件发布到 Maven Central

> 原文：<https://itnext.io/publishing-artifact-to-maven-central-b160634e5268?source=collection_archive---------4----------------------->

![](img/7a03f141a519fb4f6f3eb2170381a872.png)

照片由[切瓦农摄影](https://www.pexels.com/@chevanon?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)从[派克斯](https://www.pexels.com/photo/art-blur-cappuccino-close-up-302899/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

最近，我着手向 Maven Central 发布我的第一个工件。虽然有许多优秀的文章，但我发现它们并不完全适合我。我想分享我的经历，我在哪里遇到了问题，我学到了什么。

# 先前技术

我将跳过几个细节。这篇文章是作为一个起点，而不是重复其他伟大作品中已经说过的话。

首先，我要感谢这篇文章 [Seun Matt](https://medium.com/u/355ff06a8792?source=post_page-----b160634e5268--------------------------------) 的大力帮助:

*   [如何将开源 Java 库上传到 Maven Central](https://medium.freecodecamp.org/how-to-upload-an-open-source-java-library-to-maven-central-cac7ce2f57c)

![](img/7d22e6053d0167e0f0fcc5603ecdd7e0.png)

来自 [Pexels](https://www.pexels.com/photo/hand-fruits-rural-gardening-92354/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 的 [mali maeder](https://www.pexels.com/@mali?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 的照片

# 初始设置

显然，我们需要部署一些东西。对我来说，我正在上传我制作的一堆 maven 瓷砖的人工制品。

[鹅峰路/美文](https://github.com/efenglu/maven)

确保设置您的 pom *groupid* 、 *artifactid* 和*版本*。工件和版本可以是你想要的任何东西，但是 *groupid* 应该是你控制的 URL 域或者类似于你的 github 组织。没有 Github org？好吧，如果你是 Github 用户，好消息！**你们**是一个组织。

对我来说，我使用:

io.github.efenglu

## Pom 项目描述

设置您的 pom 定义，以包括:

*   名称—通常为*$ { project . groupid }:$ { project . artifact id }*
*   描述
*   URL—git 存储库的 URL
*   许可证
*   开发商
*   单片机
*   分布式管理— Sonatype

![](img/d1ee21eeb6d3375c5f96920e7e41dba0.png)

照片由 [Pexels](https://www.pexels.com/photo/man-standing-in-front-of-counter-1833399/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 的 [Lisa Fotios](https://www.pexels.com/@fotios-photos?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 拍摄

# Sonatype 设置

你需要一个 Sonatype 的账户。前往 sonatype.org[注册账户。](https://oss.sonatype.org/)

您还需要创建一个项目。

详情请参考本指南: [OSSRH 导轨](https://central.sonatype.org/pages/ossrh-guide.html)

这可能需要一两天的时间。

## GPG 设置

我们需要在所有的艺术品上签名。为此，我们需要一把 GPG 钥匙。

```
gpg --full-generate-key
```

别忘了我们需要发布 Ubuntu 的密钥库的密钥。

```
gpg --keyserver keyserver.ubuntu.com --send-keys ${KEY_ID}
```

## Maven 设置

我们需要将所有凭据存储在 pom 之外。我们可以通过 settings.xml 文件做到这一点。

*~/.m2/settings.xml:*

您可以将它们直接存储在 settings.xml 中。但是，有些人可能不喜欢以纯文本形式存储密码。另一种方法是让 settings.xml 引用一个环境变量，只有在实际发布时才设置这个环境变量。

要让 maven 引用环境变量，以“env”开始变量引用。后跟环境变量的名称。

例如:

```
<gpg.passphrase>${env.GPG_PASSPHRASE}</gpg.passphrase>
```

![](img/244f2befce01bd9bf467996d11826dc2.png)

照片由[切瓦农摄影](https://www.pexels.com/@chevanon?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)从[像素](https://www.pexels.com/photo/beverage-business-cafeteria-caffeine-302894/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

## Maven 构建插件

我们需要添加几个插件:

*   javadoc —创建包含 javadoc 的 jar
*   源 jar —创建包含源的 jar
*   gpg 签名插件——为所有没有 GPG 签名的工件签名
*   nexus 暂存插件—部署到 nexus 暂存和发布存储库

我把它们放在一个配置文件中，所以它们只在发布期间执行。

![](img/72ca58518e1e3590a4d23f3247a13044.png)

照片由 [bruce mars](https://www.pexels.com/@olly?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从 [Pexels](https://www.pexels.com/photo/man-holding-white-teacup-in-front-of-gray-laptop-842567/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 拍摄

# 释放；排放；发布

现在我们已经准备好实际执行发布了。

为发布选择一个新版本。最好将它设置为一个环境变量，因为我们会经常用到它:

```
export newVersion=1.0.0
```

## ***设置发布版本***

将 pom 的版本设置为发布工件版本

```
./mvnw versions:set -DnewVersion=${newVersion} \
   -DgenerateBackupPoms=false
```

## ***构建和测试***

构建并测试版本，以确保一切都是正确的

```
./mvnw clean install -Prelease
```

## ***验证 GPG 签名***

验证是否使用了正确的签名来签署工件

```
gpg --verify target/*.pom.asc
```

您应该会在结果中看到您的电子邮件，如下所示:

```
gpg: assuming signed data in '***********'
gpg: Signature made Sun Mar 31 22:34:40 2019 CDT
gpg:                using RSA key
**************************
gpg: Good signature from "Erik Englund <[*****@gmail.com](mailto:englundGiant@gmail.com)>" [ultimate]
```

## ***部署发布***

将发布部署到 Sonatype

```
./mvnw clean deploy -Prelease
```

假设 Maven 成功了，您的工件应该可以在一个小时左右的时间内从 Maven Central 获得。你的藏物可能几个小时内不会出现在搜索中心。

## ***提交和标签释放***

至此，我们完成了，但是我们还应该在 Github 上标记我们的发布。

```
git add .
git commit -m **"Release ${newVersion}"** git tag ${newVersion}
git push --tags
```

不要忘记回滚您的更改，它们不会被添加到主代码库中:

```
git reset HEAD~1
git checkout .
git clean -fd
```

![](img/1f4a5697f12e41fa7063f2d005303f68.png)

照片由 [Pixabay](https://www.pexels.com/@pixabay?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从 [Pexels](https://www.pexels.com/photo/cheerful-close-up-coffee-cup-208165/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 拍摄

# 使自动化

在这一点上，我们的发布过程是手动的。让我们自动化它。

使用 shell 脚本:

```
**#!/bin/bash -e** DIR="**$**( cd "**$**( dirname "**$**{BASH_SOURCE[0]}" )" *>*/dev/null 2*>*&1 **&&** pwd )"

export VERSION=$1

**if [[ -z** "$VERSION" **]]**; **then** echo "ERROR: Version is undefined"
  exit 1
**fi
if [[ -z** "**$**{SONATYPE_USERNAME}" **]]**; **then** echo "ERROR: SONATYPE_USERNAME is undefined"
  exit 1
**fi
if [[ -z** "**$**{SONATYPE_PASSWORD}" **]]**; **then** echo "ERROR: SONATYPE_PASSWORD is undefined"
  exit 1
**fi
if [[ -z** "**$**{GPG_KEYNAME}" **]]**; **then** echo "ERROR: GPG_KEYNAME is undefined"
  exit 1
**fi
if [[ -z** "**$**{GPG_PASSPHRASE}" **]]**; **then** echo "ERROR: GPG_PASSPHRASE is undefined"
  exit 1
**fi** mvn versions:set -DnewVersion=**$**{VERSION} -DgenerateBackupPoms=false
mvn clean deploy -B -P release --settings **$**{DIR}/settings.xml
git checkout .
git tag $VERSION
git push tags
```

像这样运行脚本:

```
./release.sh 1.0.2
```

要查看完整的示例，请访问 Github 上的 my repository:

[](https://github.com/efenglu/maven) [## 埃丰卢/马文

### 包含带有许多插件的 pom 的示例项目

github.com](https://github.com/efenglu/maven) 

# 参考

[](https://central.sonatype.org/pages/apache-maven.html) [## 阿帕奇 Maven

### Apache Maven 通过将其所有组件和所需的依赖项发布到中央存储库来启动中央存储库…

central.sonatype.org](https://central.sonatype.org/pages/apache-maven.html)  [## 生成新的 GPG 密钥- GitHub 帮助

### GitHub 支持几种 GPG 密钥算法。如果您尝试添加由不受支持的算法生成的密钥，您可能会…

help.github.com](https://help.github.com/en/articles/generating-a-new-gpg-key) [](http://maven.apache.org/plugins/maven-gpg-plugin/usage.html) [## 阿帕奇马文 GPG 插件-用法

### Apache > Maven >插件> Apache Maven GPG 插件>用法

maven.apache.org](http://maven.apache.org/plugins/maven-gpg-plugin/usage.html)  [## GnuPrivacyGuardHowto -社区帮助 Wiki

### 本页描述如何使用 OpenPGP 密钥。有关 OpenPGP 的简要描述，请参见下一节。的…

help.ubuntu.com](https://help.ubuntu.com/community/GnuPrivacyGuardHowto) [](https://medium.freecodecamp.org/how-to-upload-an-open-source-java-library-to-maven-central-cac7ce2f57c) [## 如何将开源 Java 库上传到 Maven Central

### 本文从头到尾是一个全面的指南，介绍如何将 Java 库部署到 Maven Central，以便…

medium.freecodecamp.org](https://medium.freecodecamp.org/how-to-upload-an-open-source-java-library-to-maven-central-cac7ce2f57c)
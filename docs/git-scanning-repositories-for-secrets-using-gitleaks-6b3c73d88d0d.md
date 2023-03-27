# Git:使用 Gitleaks 扫描仓库中的秘密

> 原文：<https://itnext.io/git-scanning-repositories-for-secrets-using-gitleaks-6b3c73d88d0d?source=collection_archive---------1----------------------->

![](img/685fb56ad1c23b41006e89a20771945e.png)

机密数据泄露，比如对 Git 存储库的 RDS 密钥或密码，即使它是一个私有的 Github 存储库，也是一件非常糟糕的事情，最好检查一下您的存储库，了解是否有开发人员对这些数据进行了提交。

# **内容**

*   [扫描实用程序](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Scanning_utilities)
*   [策划](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Planning)
*   [Gitleaks —手动运行](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Gitleaks_-_manual_run)
*   [Github 令牌](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Github_token)
*   [詹金斯的工作](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Jenkins_job)
*   [管道脚本](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Pipeline_script)
*   [Groovy 中的循环](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Loops_in_Groovy)
*   [詹金斯 Docker 插件](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Jenkins_Docker_plugin)
*   [忽略詹金斯阶段的错误{}](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Ignoring_errors_in_a_Jenkins_stage)
*   来自詹金斯的时差通知
*   [创建一个松弛机器人](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Create_a_Slack_Bot)
*   [詹金斯的证件](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Jenkins_credentials)
*   [Jenkins . plugins . slack . standardslackservice postToSlack 响应代码:404](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#jenkinspluginsslackStandardSlackService_postToSlack_Response_Code_404)
*   [文件上传到 Slack](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#File_upload_to_Slack)
*   [Gitleaks 配置](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Gitleaks_configuration)
*   [提交检查](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Commits_to_check)
*   [Gitleaks 配置文件](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/#Gitleaks_configuration_file)

# 扫描实用程序

乍一看，要检查 Git 存储库的泄漏，有很多实用程序:

*   Gittyleaks —看起来很有趣，但上次更新是在 2 年前
*   [回购主管](https://github.com/auth0/repo-supervisor) —有一个 WebUI，使用 AWS Lambda，与 Github 完全集成，可以稍后检查
*   块菌猪 —只有 CLI，看起来还不错
*   [Git Hound](https://github.com/ezekg/git-hound)—`git`的插件，只能执行提交前扫描，不能执行远程存储库
*   最近一次更新是在三年前
*   瞭望塔——看起来很有趣，甚至有一个 WebUI，但他们甚至没有在网站上公布他们的定价，所以退出了比赛
*   [git guardian](https://www.gitguardian.com/)——一个非常好的解决方案，但是价格过高
*   gitleaks —只有 CLI，我们将在本文中使用

因此，从上面的列表来看，值得一试[松露猪](https://github.com/trufflesecurity/truffleHog)和 [gitleaks](https://github.com/zricethezav/gitleaks) ，但是我不喜欢[松露猪](https://github.com/trufflesecurity/truffleHog)的文档。

[回购主管](https://github.com/auth0/repo-supervisor)看起来也很有希望，会在下面的帖子中查看。

从那两个:

*   Gitleaks:只是一个扫描器——给出了一个存储库的 URL，它将生成一个包含发现的 JSON 报告
*   回购主管:有两种用途:
*   只是为了扫描本地目录
*   在 PullRequest/push/etc 上扫描远程存储库

因此，对于 Gitleaks，我们可以在 Jenkins 或 Kubernetes 中创建一个 cronjob，它将接受要检查的存储库列表，然后将报告发送到 Slack 通道。

还有，Gitleaks 可以和 [Github 动作](https://rtfm.co.ua/github-obzor-github-actions-i-deploj-s-argocd/)一起使用，更多[见这里> > >](https://github.com/marketplace/actions/gitleaks) ，但是并不是我们所有的开发者团队都使用动作。另一种方式可以是[预提交钩子](https://pre-commit.com/)。

# 规划

所以，现在，让我们用 Jenkins 来尝试一个解决方案，尽管有各种方法来运行它:

*   使用 [GitHub 拉取请求生成器](https://rtfm.co.ua/jenkins-github-pull-request-builder-plagin/)触发作业
*   使用через *GitHub hook 触发器为 GITScm 轮询* или *轮询 SCM* 触发作业
*   作为 crontask 运行

首先，我们将创建一个按计划运行的简单作业，然后将检查其他解决方案。

我们的项目中有什么:

*   大约 200 个 Github 库
*   大约 10 个开发团队——后端、前端、分析、iOS 和 Android 移动应用、游戏、devops。

我们能对 Gitleaks 做什么:

*   为每个团队创建一个 Jenkins 工作
*   该作业将接受一个带有团队存储库列表的参数
*   将为每个团队创建一个专用的空闲通道
*   每天将运行一次扫描，并将报告发送到相应的空闲通道

首先，让我们手动运行 Gitleaks，看看它是如何工作的，然后将进行自动化工作。

# Gitleaks —手动运行

安装它。在 Arch Linux 上，可以从 AUR 安装:

```
$ yay -S gitleaks
```

## Github 令牌

接下来，需要创建一个令牌来访问 Github 组织的存储库。

转到您的 Github 用户设置，创建一个令牌:

![](img/77748f8774b56a40ca990b6c90913cdd.png)

给它`repo`权限:

![](img/aae7ce3594da4d558e406cfe1c98bbc3.png)

用令牌运行 Gitleaks，一个存储库的 URL，添加`--verbose`，将结果保存到一个文件:

```
$ gitleaks — access-token=ghp_C6h***3z5 — repo-url=https://github.com/example/BetterBI — verbose — report=analytics-repo.json
…\
INFO[0036] scan time: 32 seconds 756 milliseconds 672 microseconds\
INFO[0036] commits scanned: 1893
WARN[0036] leaks found: 111
```

检查报告:

```
$ less analytics-repo.json
```

调查结果中有一个例子:

```
...
 {
  "line": "  \"private_key\": \"-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADA***CCaM=\\n-----END PRIVATE KEY-----\\n\",",
  "lineNumber": 5,
  "offender": "-----BEGIN PRIVATE KEY-----",
  "offenderEntropy": -1,
  "commit": "0f047f0cca3994b3465821ef133dbd3c8b55ee7a",
  "repo": "BetterBI",
  "repoURL": "https://github.com/example/BetterBI",
  "leakURL": "https://github.com/example/BetterBI/blob/0f047f0cca3994b3465821ef133dbd3c8b55ee7a/adslib/roas_automation/example-service-account.json#L5",
  "rule": "Asymmetric Private Key",
  "commitMessage": "DT-657 update script for subs and add test\n\nDT-657 create test for check new json (add new subs)",
  "author": "username",
  "email": "example@users.noreply.github.com",
  "file": "adslib/roas_automation/example-service-account.json",
  "date": "2021-05-11T19:46:46+03:00",
  "tags": "key, AsymmetricPrivateKey"
 },
...
```

这里:

*   `line`:到底发现了什么
*   `offender`:该发现触发的规则
*   `commit`:带秘密的提交 ID

使用正则表达式执行查找，如`[default.go](https://github.com/zricethezav/gitleaks/blob/master/config/default.go)`中所述。

此外，您可以创建自己的配置文件并将其传递给 Gitleaks。

例如，上面的私有 RSA 密钥是通过[非对称私有密钥](https://github.com/zricethezav/gitleaks/blob/master/config/default.go#L76:)规则找到的:

```
[[rules]]
    description = "Asymmetric Private Key"
    regex = '''-----BEGIN ((EC|PGP|DSA|RSA|OPENSSH) )?PRIVATE KEY( BLOCK)?-----'''
    tags = ["key", "AsymmetricPrivateKey"]
```

因此，我们可以为每个团队或存储库创建一个专用的配置文件，并通过 Kubernetes ConfigMap 或作为 Jenkins 作业中的一个文件传递它们。

# 詹金斯工作

现在，当我们看到 Gitleaks 是如何启动的，让我们添加一个 Jenkins 作业来定期运行它。

## 管道脚本

因此，对于每个团队，我们将创建一个专门的 Jenkins 作业，该作业将包含一个带有团队存储库列表的参数。

## Groovy 中的循环

不久前，我用 Golang 做了一个类似的解决方案，查看 Github 中的 [Go:检查公共库列表。Go slices comparison](https://rtfm.co.ua/en/go-checking-public-repositories-list-in-github-go-slices-comparison-the-first-golang-experience/) 贴出细节，在列表上运行一个循环稍微简单一点。用 Groovy，必须谷歌一下。

创建一个新的 Jenkins 作业，将其类型设置为 Pipeline:

![](img/e4b9b62e21476e473a93a0580d98851b.png)

在作业的设置中，创建一个字符串参数，其中包含团队的存储库列表，这里只使用了两个:

![](img/b778724a45932b9c5714fbf4165ca749.png)

接下来，转到 Jenkins 脚本。

设置一个名为`$repos_list`的变量，它将接受一个环境变量`$TEAM_REPOS`，然后通过使用`split()`方法划分列表的对象。

然后，通过使用`for`循环对它们进行积分:

![](img/bc1de8b4446b75532b372c23ba641827.png)

```
node('master') {

  def repos_list = "${env.TEAM_REPOS}".split(',')

  for (repo in repos_list) {
    println repo
  }

}
```

运行作业:

![](img/17fb90103b3e6d8902b9f34fc88de2da.png)

## Jenkins Docker 插件

我们运行 Jenkins 构建的默认方法是使用 Docker 容器来保持主机系统的整洁。

用*密码*类型添加另一个参数，在这里保存 Github 令牌:

![](img/441016c6da0c0e72fed170e3ff0b5647.png)

通过使用 [Jenkins Docker 插件](https://www.jenkins.io/doc/book/pipeline/docker/)创建一个包含 Gitleaks 的 Docker 容器，传递令牌、URL 和报告文件。注意，报告文件将包含一个存储库的名称:

```
node('master') {

  def repos_list = "${env.TEAM_REPOS}".split(',')

  for (repo in repos_list) {
    stage("Repository ${repo}") {
      docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
        sh "gitleaks --access-token=${GITHUB_TOKEN} --repo-url=https://github.com/example/${repo} --verbose --report=analytics-${repo}-repo.json"
      }
    }
  }
}
```

这里，对于来自`repos_list`列表的每个存储库名称，我们将创建一个专用的 Jenkins 管道阶段，它也将使用存储库的名称。

运行并检查:

![](img/37e26dfc61d7b52e03d0b4901844becb.png)

嗯…这里有一个问题:扫描将在第一次扫描的 repo 中的第一个结果之后立即停止，因为 Geatleaks 发现了一个泄漏，返回了 exit 1 代码，并且作业立即停止:

![](img/df1de0f7aad691671f5ad306e5d05ef3.png)

## 忽略 Jenkins 阶段中的错误{}

要解决这个问题，我们可以使用`try/catch`解决方案:每个阶段将在其`try`中运行，如果出现错误，我们将使用`catch`来捕捉它们，并继续构建:

```
node('master') {

  def repos_list = "${env.TEAM_REPOS}".split(',')
  def build_ok = true

  for (repo in repos_list) {
    try {
      stage("Repository ${repo}") {
        docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
          sh "gitleaks --access-token=${GITHUB_TOKEN} --repo-url=https://github.com/example/${repo} --verbose --report=analytics-${repo}-repo.json"
        }
      }
    } catch(e) {
        currentBuild.result = 'FAILURE'
    }
  }
}
```

运行它:

![](img/695a2aad4d022d5afcb1bc41e9c480d6.png)

好—现在所有阶段都在运行，尽管有前一个阶段的结果。

## 来自 Jenkins 的时差通知

我们的下一步是配置向松弛工作区发送警报。

为此，我们使用[松弛通知](https://plugins.jenkins.io/slack)插件。此处> > > 见其文档。

## 创建一个 Slack Bot

转到[松弛应用](https://api.slack.com/apps)，创建一个新应用:

![](img/f4fcf4c509fbb5a5580c2210399df263.png)![](img/ea119c92bdb90b74134422ba7a1228d7.png)

转到*权限*:

![](img/bbd8bdfda4613e5c84bfa27571e0e0df.png)

添加以下内容:

*   `files:write`
*   `chat:write`

![](img/b765cfab0391641566f209317d908bed.png)

转到 *OAuth &权限*，将 bot 安装到 Slack 工作区:

![](img/59fe7ae1ff2192cb37eb44f364c3ccd4.png)

保存令牌:

![](img/b4066594b6bb144ed81dab0b99ac04bf.png)

## 詹金斯证书

向 Jenkins 添加令牌—转到*管理 Jenkins >管理凭证*:

![](img/d2442ff1769c075d3ab054dbb42a675c.png)

添加一个新的:

![](img/a8ef334e1957d2a9eb04929859da4bd7.png)

将其类型设置为*秘密文件*:

![](img/4e18e867f8a24981ce6975e17aa1a86f.png)

在松弛工作区中，创建一个新频道:

![](img/f40350b9c1c4cfe00d61fca5c92ddca4.png)

邀请机器人加入频道:

![](img/0426b0f926c4a4cfff2dc57d0840ad51.png)![](img/5d6cd128ffc848beb3ab7f63960cd412.png)

向 Jenkins 脚本添加一个新函数— `notifySlack()`，并从`catch{}`运行它，以便在扫描过程中发现任何秘密时发送警报:

```
def notifySlack(String buildStatus = 'STARTED') {

    // Build status of null means success.
    buildStatus = buildStatus ?: 'SUCCESS'

    def color
    //change for another slack chanel
    def token = 'gitleaks-slack-bot'

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"
    slackSend(color: color, message: msg, tokenCredentialId: token, channel: "#devops-alarms-gitleaks-analytics")
}

node('master') {

  def repos_list = "${env.TEAM_REPOS}".split(',')

  for (repo in repos_list) {
    try {
      stage("Repository ${repo}") {
        docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
          sh "gitleaks --access-token=${GITHUB_TOKEN} --repo-url=https://github.com/example/${repo} --verbose --report=analytics-${repo}-repo.json"
        }
      }   
    } catch(e) {
        currentBuild.result = 'FAILURE'
        notifySlack(currentBuild.result)
    } 
  }     
}
```

## Jenkins . plugins . slack . standardslackservice postToSlack 响应代码:404

运行构建，并得到以下错误:

> 12:42:39 错误:时差通知失败。有关详细信息，请参阅 Jenkins 日志。

查看 Jenkin 在*https://<JENKINS _ URL>/log/all*上的日志:

![](img/0595d2d9c9c0f044fb7cd5aea7491f7e.png)

进入*管理 Jenkins >配置系统*，找到 Slack 插件的选项，设置*自定义 slack app bot 用户*:

![](img/b39ec4fcfa48dc7bea43d23817a99ffe.png)

这里的凭证是默认的，我们将从管道中覆盖它们。

在*高级*中删除*覆盖 URL，如果设置了*:

![](img/8aa4722f8b92bcc701ae5f0f37d3866b.png)

再次运行，现在一切正常:

![](img/58f5d95b0000c8e6561ea28c9b13f672.png)

## 文件上传到松弛时间

现在，让我们通过使用`slackUploadFile()`函数向 Slack 通道中的消息添加一个报告文件 upload 和发现:

```
def notifySlack(String buildStatus = 'STARTED', reportFile) {

    // Build status of null means success.
    buildStatus = buildStatus ?: 'SUCCESS'

    def color
    //change for another slack chanel
    def token = 'gitleaks-slack-bot'

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"
    slackSend(color: color, message: msg, tokenCredentialId: token, channel: "#devops-alarms-gitleaks-analytics")
    slackUploadFile(credentialId: token, channel: "#devops-alarms-gitleaks-analytics", filePath: "${reportFile}")
}

node('master') {

  def repos_list = "${env.TEAM_REPOS}".split(',')

  for (repo in repos_list) {
    try {
      stage("Repository ${repo}") {
        docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
          sh "gitleaks --access-token=${GITHUB_TOKEN} --repo-url=https://github.com/example/${repo} --verbose --report=analytics-${repo}-repo.json"
        }
      }   
    } catch(e) {
        currentBuild.result = 'FAILURE'
        notifySlack(currentBuild.result, "analytics-${repo}-repo.json")
    } 
  }     
}
```

这里的通道可以在以后移动到作业的参数中。

这里，在`notifySlack()`中，我们添加了另一个参数——T9，然后在`notifySlack()`函数调用期间，我们将报告文件作为第二个参数传递给函数。

运行作业，检查松弛通道:

![](img/57072d06056e58de1fd254843ed0ae80.png)

最后一件事是设置运行作业的时间表:

![](img/12aec128688449d0d58c9366df00f1be.png)

# Gitleaks 配置

## 提交检查

此时，Gitleasks 将对存储库进行全面扫描——所有提交，所有历史。

如果我们每天都运行它，那么每天我们都会收到一些关于旧的有问题的提交的消息。

作为一种减轻风险的方法，我们可以创建两个作业:在第一个作业中，我们将执行完全扫描，在第二个作业中，我们将对过去 24 小时内发生的更改执行一种增量扫描。

也就是说,“增量”作业将在每天中午 12:00 运行，此时所有开发人员都在办公室，该作业将只检查最后一天的提交。

为此，Gitleaks 提供了`--commit-since`选项。让我们添加一个名为`yesterday`的新变量，它带有由`Date()`类的`previous()`方法获取的昨天的日期，然后这个日期将被传递给`--commit-since`:

```
...

node('master') { 

  def repos_list = "${env.TEAM_REPOS}".split(',')
  def yesterday = new Date().format( 'yyyy-MM-dd' ).previous()

  println yesterday

  for (repo in repos_list) {
    try {
      stage("Repository ${repo}") {
        docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
          sh "gitleaks --access-token=${GITHUB_TOKEN} --repo-url=https://github.com/example/${repo} --verbose --report=analytics-${repo}-repo.json --commit-since=${yesterday}"
        }
      }
    } catch(e) {
        currentBuild.result = 'FAILURE'
        notifySlack(currentBuild.result, "analytics-${repo}-repo.json")
    }
  }
}
```

## Gitleaks 配置文件

另一件事是为 Gitleaks 创建一个专用的规则文件。

这可以用`--repo-config-path`来完成，在每个存储库中，我们可以添加它自己的配置文件。

添加一些默认规则，另外我想检查作为明文传递给提交的密码:

```
...

[[rules]]
    description = "Plaintext password"
    regex = '''(?i)pass*[a-z]{5}[:|=]? +["|'](.*)["|']'''
    tags = ["password", "PlainTextPassword"]

[allowlist]
    description = "Allowlisted files"
    files = ['''^\.?gitleaks.config$''']
```

使用`(?i)pass*[a-z]{5}[:|=]? +["|'](.*)["|']`正则表达式，我们寻找一个以 *pass* 开始的字符串，然后是一个":"或" = "符号，然后它可以包含或不包含一个空格，然后是一个引号，然后是任何文本，再次是引号。

看来肯定是管用的:

![](img/b88e43e82ca44987b8e09a8c1aa0f757.png)

将其作为`.github/gitleaks.config`保存到存储库中，并通过使用该文件在作业中添加另一个参数:

```
...
        docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
          sh "gitleaks --access-token=${GITHUB_TOKEN} --repo-url=https://github.com/example/${repo} --verbose --report=analytics-${repo}-repo.json --commit-since=${yesterday} --repo-config-path=.github/gitleaks.config"
        }
...
```

目前就这些。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/git-scanning-repositories-for-secrets-using-gitleaks/) *。*
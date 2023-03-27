# 松弛时间+代码部署= ❤️

> 原文：<https://itnext.io/slack-codedeploy-%EF%B8%8F-6c177a88c730?source=collection_archive---------5----------------------->

![](img/92ff58eec2b02678b11e1a30679ed9b0.png)

我们都知道懈怠，对吧？你有没有想过用它来部署你的代码，只需输入`/deploy`？我做了，这是我为一个团队设计和开发的，这个团队厌倦了处理设计糟糕的 AWS Codedeploy UI。

# 语境

我们的堆栈在 AWS 上运行，我们在每个环境中都有一个帐户，这意味着`Staging`和`Production`运行在两个完全独立的帐户上；每个用户都必须登录我们称之为`Main`的账户，然后扮演他们想要与之互动的环境的角色。为了更好地理解我所说的，下面是从`git push`到实际部署的过程:

1.  开发人员将代码推送到 git
2.  CircleCI 构建代码，将所有内容打包到一个`tar.gz`文件中，并使用以下文件夹结构将其推送到一个 S3 桶(在`Main`帐户上:`prefix/productname/branchname/commitid.tar.gz`
3.  开发人员访问 Github 并复制提交 ID
4.  登录到`Main` AWS 帐户
5.  承担`Staging`角色
6.  打开 Codedeploy 控制台并选择应用程序名称
7.  手动输入 S3 存储桶 URL**，粘贴提交 ID 并在字符串末尾添加`.tar.gz`**
8.  **点击部署按钮，希望他/她没有犯任何错误组成 S3 桶网址(他/她很可能犯了！)**

**我的想法是用一个简单的 [Slack 的斜杠命令](https://api.slack.com/slash-commands)替换步骤#2 之后的任何内容。**

# **斜线命令**

**当 slash 命令被调用时，Slack 向您指定的请求 URL 发送一个 HTTP POST。该请求包含一个数据有效载荷，其报头设置为`application/x-www-form-urlencoded`。请求的正文如下所示:**

```
token=gIkuvaNzQIHg97ATvDxqgjtO
team_id=T0001
team_domain=example
enterprise_id=E0001
enterprise_name=Globular%20Construct%20Inc
channel_id=C2147483705
channel_name=test
user_id=U2147483697
user_name=Steve
command=/weather
text=94070
response_url=https://hooks.slack.com/commands/1234/5678
trigger_id=13345224609.738474920.8088930838d88f008e0
```

**在开始编码之前，我通过 Slack 仪表板创建了以下斜杠命令，并将它们的标记保存在一个安全的地方:**

# **该应用程序**

**现在我已经准备好了斜杠命令，我需要点击一些端点，有趣的部分开始了。**

**我开发了一个叫做 [node-slack-deployer](https://github.com/darkraiden/node-slack-deployer) (以前叫做`codedeploy-helper`)的 Express NodeJS 应用来完成这项工作。服务器为这些斜杠命令提供了两个端点(`/slack/get-deployments`和`/slack/deploy`)和一个用于健康检查的端点(`/`，它返回一个`OK`消息)。**

**但是我们如何知道基于`POST`的主体我们需要寻找或部署什么呢？这非常简单:在`/command`本身作为主体的`text`以`string`格式传递之后添加的任何东西，我们只需要把它分离出来。**

**部署命令的论据:**

```
 if (req.body.text.split(' ').length === 2) {
        // Store product name and branch in product object
        res.locals.product = {
            service: req.body.text.split(' ')[0].toLowerCase(),
            env: req.body.text.split(' ')[1].toLowerCase(),
            description: 'Deploying via Slack'
        };
    } else {
        // Store body information in product object
        res.locals.product = {
            service: req.body.text.split(' ')[0].toLowerCase(),
            env: req.body.text.split(' ')[1].toLowerCase(),
            branch: req.body.text.split(' ')[2],
            hash: req.body.text.split(' ')[3],
            description: 'Deploying via Slack'
        };
    }
```

***工件命令的参数*:**

```
 //Params length must have at least 2 elements
    if (params.length &lt; 2) {
        res.json(
            slackHandler.payloadToSlack(
                'failure',
                'Code Artifacts',
                400,
                'Invalid number of arguments\nUsage: `/artifact [service-name] [branch]`'
            )
        );
        return;
    } res.locals.bucketPath = awsHandler.generateBucketPath({
        service: params[0],
        branch: params[1],
        hash: params[2]
    });
```

**出于安全原因，我们还会检查 Slack 发送的令牌是否与我们使用环境变量传递给应用程序的令牌相匹配:**

```
//Check if the deployment token passed matches the env vars one
exports.checkDeploymentToken = token => {
    return token === process.env.SLACK_DEPLOYMENT_TOKEN;
};
```

**看起来不错，现在怎么办？哦，对了，让我们看看单个端点能为我做些什么！**

# **工件命令**

**`/artifacts`命令向用户返回给定产品及其分支的最后 5 个构建工件的列表:**

```
/artifacts api master
```

**既然说了 S3 斗活在`Main`账号上，这里就不需要什么`STS`魔法了，只需要查询 S3 斗！**

```
exports.getDeployments = async (req, res) => {
    const data = await awsHandler.s3GetObjects(res.locals.bucketPath);
    let max = 5;
    let arr = [];// Check if data is empty
    if (data.length &lt; 1) {
        res.json(
            slackHandler.payloadToSlack(
                'warning',
                'Code Artifacts',
                404,
                'Elements not found'
            )
        );
        return;
    }// Check length of the object and returns max 5 items
    if (data.length &lt; 5) {
        max = data.length;
    }
    for (let i = 0; i &lt; max; i++) {
        arr.push(utilsHandler.getHash(data[i].Key));
    }res.json(
        slackHandler.payloadToSlack(
            'success',
            'Code Artifacts',
            'Artifacts List:',
            slackHandler.generateHashesText(arr, res.locals.params[0])
        )
    );
};
```

**您可以注意到，`data`正在获取`awsHandler.s3GetObjects`方法的输出，它只不过是这个 AWS API 调用:**

```
exports.s3GetObjects = async bucketPath => {
    const listObjects = promisify(S3.listObjects, S3);
    const data = await listObjects({
        Bucket: this.bucketParams.Bucket,
        Prefix: bucketPath
    });//Order data from newest to oldest
    if (data.Contents.length &gt; 1) {
        data.Contents.sort((a, b) =&gt; {
            return new Date(b.LastModified) - new Date(a.LastModified);
        });
    }return data.Contents;
};
```

**还有，最后一件事，`slackHandler.generateHashesText`看起来像这样:**

```
//Generate a formatted string of hashes with ordered list
exports.generateHashesText = (data, service) => {
    let string = '';
    let max = 5;
    if (data.length &lt; 5) {
        max = data.length;
    }
    for (let i = 0; i &lt; max; i++) {
        string += `${i + 1}. &lt;[https://github.com/${](https://github.com/${)
            process.env.GITHUB_USER
        }/${service}/commit/${data[i]}|${data[i]}&gt;`;
        if (i &lt; max - 1) {
            string += '\n';
        }
    }
    return string;
};
```

**排序:我们的应用程序正在返回 slack 一个有效负载，在其主体中包括一个将用户链接到他们的 Github 提交 URL 的最后 5 个工件的列表，以防你想确保你看到的是正确的提交！**

# **部署命令**

**`/deploy`命令触发一个 Codedeploy 部署，它接受 4 个参数:**

*   **产品名称**
*   **环境**
*   **树枝**
*   **提交 ID**

```
/deploy api production master 5d0e80e4152eef31f64c7b8026277894a541f166
```

**这里的事情有点棘手，因为我们需要:**

1.  **检查该命令是否是从右松弛通道调用的；**
2.  **检查工件是否存在；**
3.  **承担 AWS 角色；和**
4.  **最后部署代码。**

**让我们从第 1 点和第 2 点开始，然后:**

```
// ...
    if (!this.isDeploymentChannel(req.body.channel_id)) {
        console.log('Invalid Slack channel');
        res.json(
            slackHandler.payloadToSlack(
                'failure',
                'Slack Deploy',
                401,
                "Oooops! It looks like you can't run this command from this channel!"
            )
        );
        return;
    }
// ... const isS3Element = await awsHandler.s3GetSingleObject(bucketPath); if (!isS3Element) {
        res.json(
            slackHandler.payloadToSlack(
                'warning',
                'Slack Deploy',
                404,
                'Element not found'
            )
        );
        return;
    }// ...
```

**其中`isDeploymentChannel`简单来说就是:**

```
//Check Slack Channel ID
exports.isDeploymentChannel = channelId => {
    return channelId === process.env.SLACK_DEPLOYMENTS_CHANNEL_ID;
};
```

**然后，我们准备好承担客户角色:**

```
exports.assumeRole = async (req, res, next) => {
    params.RoleSessionName = res.locals.product.env;
    params.RoleArn = awsHandler.awsRoles[res.locals.product.env];// If the environment received doesn't exist, throw an error
    if (!params.RoleArn) {
        res.json(
            slackHandler.payloadToSlack(
                'failure',
                'Slack Deploy',
                400,
                `${params.RoleSessionName} is not a valid environment`
            )
        );
        return;
    }res.locals.awsRole = await awsHandler.stsAssumeRole(params);next();
};
```

**其中`stsAssumeRole`为:**

```
exports.stsAssumeRole = async params => {
    const assumeRole = promisify(STS.assumeRole, STS);
    return await assumeRole(params);
};
```

**最后但同样重要的是，让我们部署代码—这些是部署控制器调用的方法:**

```
// Initialise codedeploy object
exports.codedeployInit = params => {
    return new AWS.CodeDeploy({
        accessKeyId: params.Credentials.AccessKeyId,
        secretAccessKey: params.Credentials.SecretAccessKey,
        sessionToken: params.Credentials.SessionToken
    });
};// Deploy using codedeploy
exports.codedeployDeploy = async (awsRole, product, bucketPath) => {
    const CodeDeploy = this.codedeployInit(awsRole); const params = {
        applicationName: product.service,
        deploymentGroupName: product.service,
        description: product.description,
        revision: {
            revisionType: 'S3',
            s3Location: {
                bucket: this.bucketParams.Bucket,
                bundleType: process.env.AWS_CODEDEPLOY_BUNDLE_TYPE,
                key: bucketPath
            }
        }
    }; const codedeploy = promisify(CodeDeploy.createDeployment, CodeDeploy); return await codedeploy(params);
};
```

**done:slack 上的用户将收到一条好消息，说明 Codedeploy 启动了部署并提供了部署 ID。**

# **观察**

**这是一个简单的快速应用程序的快速运行，到处粘贴随机片段，只是为了给出更多的上下文。它是根据我们团队的需求设计和编写的，所以我知道可能很难将其应用于任何使用 Codedeploy 部署代码的*团队/公司。但是，请记住，它的发展还没有结束。我最近对它进行了开源，我将花一些时间对它进行改进，添加/删除一些与其他用例相关或不相关的特性。***

# **贡献**

**非常感谢您的任何贡献，我将非常乐意在 Github [问题](https://github.com/darkraiden/node-slack-deployer/issues)版块查看您的 PR 或讨论改进。这个项目，再一次，可以在这里找到。**

***原载于 2018 年 8 月 13 日*[*https://www.darkraiden.com/blog/slack-codedeploy-love/*](https://www.darkraiden.com/blog/slack-codedeploy-love/)*。***
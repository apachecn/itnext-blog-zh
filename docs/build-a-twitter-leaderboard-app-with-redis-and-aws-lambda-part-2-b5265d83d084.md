# 用 Redis 和 AWS Lambda 构建一个 Twitter 排行榜应用程序(第 2 部分)

> 原文：<https://itnext.io/build-a-twitter-leaderboard-app-with-redis-and-aws-lambda-part-2-b5265d83d084?source=collection_archive---------1----------------------->

## AWS CDK 代码演练

这是这个由两部分组成的系列的第二篇博文，它使用一个实际的应用程序来演示如何将 Redis 与 AWS Lambda 集成。第一部分是关于解决方案概述和部署，希望您能够端到端地进行测试。

[](/build-a-twitter-leaderboard-app-with-redis-and-aws-lambda-part-1-670e7a8c6a91) [## 用 Redis 和 AWS Lambda 构建一个 Twitter 排行榜应用程序(第 1 部分)

### 你好，欢迎👋🏼这个由两部分组成的博客系列使用一个简单而实用的应用程序来演示如何…

itnext.io](/build-a-twitter-leaderboard-app-with-redis-and-aws-lambda-part-1-670e7a8c6a91) 

正如所承诺的，第二部分(这一部分)将涵盖基础设施方面(具体来说是 IaaC)，它由三个(CDK) [堆栈](https://docs.aws.amazon.com/cdk/v2/guide/stacks.html)(在单个 [CDK 应用](https://docs.aws.amazon.com/cdk/v2/guide/apps.html)的上下文中)组成。

我将提供一个用 Go 编写的 CDK 代码的演示，感谢 [CDK Go 支持](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-go.html)(在撰写本文时是开发者预览版)。

# CDK 代码遍历

让我们一次拿一叠

> *请注意，为了简洁起见，一些代码已经被编辑/省略——您可以随时在*[*GitHub repo*](https://github.com/abhirockzz/twitter-leaderboard-app)中查阅完整的代码

## 从基础架构堆栈开始

```
stack := awscdk.NewStack(scope, &id, &sprops)
vpc = awsec2.NewVpc(stack, jsii.String("demo-vpc"), nil)authInfo := map[string]interface{}{"Type": "password", "Passwords": []string{memorydbPassword}}
user = awsmemorydb.NewCfnUser(stack, jsii.String("demo-memorydb-user"), &awsmemorydb.CfnUserProps{UserName: jsii.String("demo-user"), AccessString: jsii.String(accessString), AuthenticationMode: authInfo})
acl := awsmemorydb.NewCfnACL(stack, jsii.String("demo-memorydb-acl"), &awsmemorydb.CfnACLProps{AclName: jsii.String("demo-memorydb-acl"), UserNames: &[]*string{user.UserName()}})*//snip .....*subnetGroup := awsmemorydb.NewCfnSubnetGroup(stack, jsii.String("demo-memorydb-subnetgroup"), &awsmemorydb.CfnSubnetGroupProps{SubnetGroupName: jsii.String("demo-memorydb-subnetgroup"), SubnetIds: &subnetIDsForSubnetGroup})memorydbSecurityGroup = awsec2.NewSecurityGroup(stack, jsii.String("memorydb-demo-sg"), &awsec2.SecurityGroupProps{Vpc: vpc, SecurityGroupName: jsii.String("memorydb-demo-sg"), AllowAllOutbound: jsii.Bool(true)})memorydbCluster = awsmemorydb.NewCfnCluster(*//... details omitted)**//...snip*twitterIngestFunctionSecurityGroup = awsec2.NewSecurityGroup(*//... details omitted)*
twitterLeaderboardFunctionSecurityGroup = awsec2.NewSecurityGroup(*//... details omitted)*memorydbSecurityGroup.AddIngressRule(*//... details omitted)*
memorydbSecurityGroup.AddIngressRule(*//... details omitted)*
```

总结一下:

*   创建 VPC 和相关组件的一行代码！
*   我们为 MemoryDB 集群创建 ACL、用户、子网组，并在使用`awsmemorydb.NewCfnCluster`创建集群时引用它们
*   我们还创建所需的安全组——它们的主要作用是允许 Lambda 函数访问 MemoryDB(我们指定显式的入站规则来实现这一点)
*   一个用于内存数据库集群
*   两个 Lambda 函数各一个

## 下一个堆栈部署了 tweets 摄取 Lambda 功能

```
*//....*memoryDBEndpointURL := fmt.Sprintf("%s:%s", *memorydbCluster.AttrClusterEndpointAddress(), strconv.Itoa(int(*memorydbCluster.Port())))lambdaEnvVars := &map[string]*string{"MEMORYDB_ENDPOINT": jsii.String(memoryDBEndpointURL), "MEMORYDB_USER": user.UserName(), "MEMORYDB_PASSWORD": jsii.String(getMemorydbPassword()), "TWITTER_API_KEY": jsii.String(getTwitterAPIKey()), "TWITTER_API_SECRET": jsii.String(getTwitterAPISecret()), "TWITTER_ACCESS_TOKEN": jsii.String(getTwitterAccessToken()), "TWITTER_ACCESS_TOKEN_SECRET": jsii.String(getTwitterAccessTokenSecret())}awslambda.NewDockerImageFunction(stack, jsii.String("lambda-memorydb-func"), &awslambda.DockerImageFunctionProps{FunctionName: jsii.String(tweetIngestionFunctionName), Environment: lambdaEnvVars, Timeout: awscdk.Duration_Seconds(jsii.Number(20)), Code: awslambda.DockerImageCode_FromImageAsset(jsii.String(tweetIngestionFunctionPath), nil), Vpc: vpc, VpcSubnets: &awsec2.SubnetSelection{Subnets: vpc.PrivateSubnets()}, SecurityGroups: &[]awsec2.ISecurityGroup{twitterIngestFunctionSecurityGroup}})*//....*
```

和之前的栈相比还是挺简单的。我们定义 Lambda 函数所需的环境变量(包括 Twitter API 凭证),并将其部署为 Docker 映像。

对于要打包成 Docker 映像的函数，我使用了 [Go:1.x 基础映像](https://gallery.ecr.aws/lambda/go)。但是，你也可以探索其他选择。在部署期间，Docker 映像在本地构建，被推送到一个[私有 ECR 注册表](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html)，最后 Lambda 函数被创建——所有这一切，只需几行代码！

> *请注意，MemoryDB 集群和安全组是从以前的堆栈中自动引用/查找的(不是重新创建的！).*

## 最后，第三个堆栈负责排行榜 Lambda 函数

它与前一个非常相似，除了添加了 Lambda 函数 URL ( `awslambda.NewFunctionUrl`)，我们使用它作为堆栈的输出:

```
*//....*
memoryDBEndpointURL := fmt.Sprintf("%s:%s", *memorydbCluster.AttrClusterEndpointAddress(), strconv.Itoa(int(*memorydbCluster.Port())))lambdaEnvVars := &map[string]*string{"MEMORYDB_ENDPOINT": jsii.String(memoryDBEndpointURL), "MEMORYDB_USERNAME": user.UserName(), "MEMORYDB_PASSWORD": jsii.String(getMemorydbPassword())}function := awslambda.NewDockerImageFunction(stack, jsii.String("twitter-hashtag-leaderboard"), &awslambda.DockerImageFunctionProps{FunctionName: jsii.String(hashtagLeaderboardFunctionName), Environment: lambdaEnvVars, Code: awslambda.DockerImageCode_FromImageAsset(jsii.String(hashtagLeaderboardFunctionPath), nil), Timeout: awscdk.Duration_Seconds(jsii.Number(5)), Vpc: vpc, VpcSubnets: &awsec2.SubnetSelection{Subnets: vpc.PrivateSubnets()}, SecurityGroups: &[]awsec2.ISecurityGroup{twitterLeaderboardFunctionSecurityGroup}})funcURL := awslambda.NewFunctionUrl(stack, jsii.String("func-url"), &awslambda.FunctionUrlProps{AuthType: awslambda.FunctionUrlAuthType_NONE, Function: function})awscdk.NewCfnOutput(stack, jsii.String("Function URL"), &awscdk.CfnOutputProps{Value: funcURL.Url()})
```

这篇博文到此为止。以 AWS Go CDK v2 参考的链接结束:

*   对于 memory db—[https://pkg . go . dev/github . com/AWS/AWS-CDK-go/AWS CDK/v2/AWS memory db](https://pkg.go.dev/github.com/aws/aws-cdk-go/awscdk/v2@v2.24.1/awsmemorydb)
*   对于 Lambda—[https://pkg . go . dev/github . com/AWS/AWS-CDK-go/AWS CDK/v2/AWS Lambda](https://pkg.go.dev/github.com/aws/aws-cdk-go/awscdk/v2@v2.24.1/awslambda)
*   对于 VPC 等。—[https://pkg . go . dev/github . com/AWS/AWS-CDK-go/AWS CDK/v2/AWS ec2](https://pkg.go.dev/github.com/aws/aws-cdk-go/awscdk/v2@v2.24.1/awsec2)
*   https://pkg.go.dev/github.com/aws/aws-cdk-go/awscdk/v2 AWS CDK V2

这个两部分系列到此结束。请继续关注，一如既往，快乐编码！
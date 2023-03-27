# nuxt Express-模板和 Azure

> 原文：<https://itnext.io/nuxt-express-template-azure-e3fce9458de1?source=collection_archive---------3----------------------->

内容:
- context
-配置 azure
-env vars
-web . config
-kudu
-整理笔记

![](img/6a75f051d5e66ae40795f82d1dc60b4b.png)

# **语境**

我想把这个故事放在这里，以帮助任何其他在 Azure 上遇到 Nuxt Express 生产部署问题的可怜人。

想象一下，你有一个有两个需求的项目:
1)你需要部署一个 node.js 应用程序——一个闪亮的新应用程序，可以满足你所有的 API 和 MVVM 需求，nuxt-community 包‘express-nuxt’可以愉快而高效地完成这项工作。这个应用程序必须部署在 Azure(微软云)基础设施中。

第二个要求是你将在哪里遇到一些困难，我将在下面给你答案！

注意，这不是一个循序渐进的教程。

# **azure 的配置**

**环境变量**

在 Azure >你的应用程序>应用程序设置中的应用程序设置内定义你的节点和 npm 版本是非常值得的，而不是在 package.json 或 iisnode.yml 文件内。

```
WEBSITE_NODE_DEFAULT_VERSION : 8.11.1
WEBSITE_NPM_DEFAULT_VERSION : 5.6.0
```

如果您决定在 package.json 中添加引擎版本:

*package.json*

```
"engines": {
    "npm": "5.6.0",
    "node": "8.11.1"
}
```

注意，Azure 对待所有这些定义引擎版本的方法是不同的。如果使用 iisnode.yml 文件定义引擎，它应该覆盖 package.json 和应用程序设置定义。

在应用程序设置中添加以下内容将打开调试

**web.config**

web 配置文件是应用程序在 Azure 上运行所必需的，它允许您定义 IISnode 模块应该通过管道将请求发送到的入口点。这对于设置来说可能有点挑剔，基本前提是您希望将所有请求定向到 nuxt 应用程序的 main.app。

下面是我当前的 web.config 文件的例子，包括注释。

*web.config*

```
<configuration>
  <system.webServer>
    <webSocket enabled="false" />
    <handlers>
      <!-- Indicates that the main.js file is a node.js site to be handled by the iisnode module -->
      <add name="iisnode" path="main.js" verb="*" modules="iisnode"/>
    </handlers>
    <rewrite>
      <rules>
        <!-- Do not interfere with requests for node-inspector debugging -->
        <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">
          <match url="^main.js\/debug[\/]?" />
        </rule>

        <!-- <rule name="Redirect to https" stopProcessing="true">
          <match url="(.*)"/>
          <conditions>
            <add input="{HTTPS}" pattern="Off"/>
            <add input="{REQUEST_METHOD}" pattern="^get$|^head$" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}"/>
        </rule> -->

        <!-- First we consider whether the incoming URL matches a physical file in the /public folder -->
        <rule name="StaticContent">
          <action type="Rewrite" url="public{REQUEST_URI}"/>
        </rule>

        <!-- All other URLs are mapped to the node.js site entry point -->
        <rule name="DynamicContent">
          <conditions>
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="True"/>
          </conditions>
          <action type="Rewrite" url="main.js"/>
        </rule>

      </rules>
    </rewrite>

    <!-- 'bin' directory has no special meaning in node.js and apps can be placed in it -->
    <security>
      <requestFiltering>
        <hiddenSegments>
          <remove segment="bin"/>
        </hiddenSegments>
      </requestFiltering>
    </security>

    <!-- Make sure error responses are left untouched -->
    <httpErrors existingResponse="PassThrough" />

    <!--
      You can control how Node is hosted within IIS using the following options:
        * watchedFiles: semi-colon separated list of files that will be watched for changes to restart the server
        * node_env: will be propagated to node as NODE_ENV environment variable
        * debuggingEnabled - controls whether the built-in debugger is enabled
    -->
  </system.webServer>
</configuration>
```

**库杜**

关于 kudu 的一个说明是完全公开的:微软支持技术人员强烈建议我不要运行定制部署或构建脚本，我忽略了他们。Microsoft 的建议是预构建应用程序并提交所有/build 和。nuxt 到应用程序存储库。

Kudu 为你的 Azure 应用程序执行预构建到后构建脚本，这很容易操作。添加一个. deployment 文件，该文件只包含要运行的 bash 脚本的定义。

*。部署*

```
[config]
command = bash deploy.sh
```

这告诉 kudu 将 deployment.sh 文件作为自定义部署脚本运行，而不是 Azure 默认使用的标准脚本。

deploy.sh 内容取决于您的具体构建，我的配置如下。

*deploy.sh*

```
#!/bin/bash

# ----------------------
# KUDU Deployment Script
# Version: 1.0.17
# ----------------------

# Helpers
# -------

exitWithMessageOnError () {
  if [ ! $? -eq 0 ]; then
    echo "An error has occurred during web site deployment."
    echo $1
    exit 1
  fi
}

# Prerequisites
# -------------

# Verify node.js installed
hash node 2>/dev/null
exitWithMessageOnError "Missing node.js executable, please install node.js, if already installed make sure it can be reached from current environment."

# Setup
# -----

SCRIPT_DIR="${BASH_SOURCE[0]%\\*}"
SCRIPT_DIR="${SCRIPT_DIR%/*}"
ARTIFACTS=$SCRIPT_DIR/../artifacts
KUDU_SYNC_CMD=${KUDU_SYNC_CMD//\"}

if [[ ! -n "$DEPLOYMENT_SOURCE" ]]; then
  DEPLOYMENT_SOURCE=$SCRIPT_DIR
fi

if [[ ! -n "$NEXT_MANIFEST_PATH" ]]; then
  NEXT_MANIFEST_PATH=$ARTIFACTS/manifest

  if [[ ! -n "$PREVIOUS_MANIFEST_PATH" ]]; then
    PREVIOUS_MANIFEST_PATH=$NEXT_MANIFEST_PATH
  fi
fi

if [[ ! -n "$DEPLOYMENT_TARGET" ]]; then
  DEPLOYMENT_TARGET=$ARTIFACTS/wwwroot
else
  KUDU_SERVICE=true
fi

if [[ ! -n "$KUDU_SYNC_CMD" ]]; then
  # Install kudu sync
  echo Installing Kudu Sync
  npm install kudusync -g --silent
  exitWithMessageOnError "npm failed"

  if [[ ! -n "$KUDU_SERVICE" ]]; then
    # In case we are running locally this is the correct location of kuduSync
    KUDU_SYNC_CMD=kuduSync
  else
    # In case we are running on kudu service this is the correct location of kuduSync
    KUDU_SYNC_CMD=$APPDATA/npm/node_modules/kuduSync/bin/kuduSync
  fi
fi

# Node Helpers
# ------------

selectNodeVersion () {
  if [[ -n "$KUDU_SELECT_NODE_VERSION_CMD" ]]; then
    SELECT_NODE_VERSION="$KUDU_SELECT_NODE_VERSION_CMD \"$DEPLOYMENT_SOURCE\" \"$DEPLOYMENT_TARGET\" \"$DEPLOYMENT_TEMP\""
    eval $SELECT_NODE_VERSION
    exitWithMessageOnError "select node version failed"

    if [[ -e "$DEPLOYMENT_TEMP/__nodeVersion.tmp" ]]; then
      NODE_EXE=`cat "$DEPLOYMENT_TEMP/__nodeVersion.tmp"`
      exitWithMessageOnError "getting node version failed"
    fi

    if [[ -e "$DEPLOYMENT_TEMP/__npmVersion.tmp" ]]; then
      NPM_JS_PATH=`cat "$DEPLOYMENT_TEMP/__npmVersion.tmp"`
      exitWithMessageOnError "getting npm version failed"
    fi

    if [[ ! -n "$NODE_EXE" ]]; then
      NODE_EXE=node
    fi

    NPM_CMD="\"$NODE_EXE\" \"$NPM_JS_PATH\""
  else
    NPM_CMD=npm
    NODE_EXE=node
  fi
}

##################################################################################################################################
# Deployment
# ----------

echo Handling node.js deployment.

# 1\. KuduSync
if [[ "$IN_PLACE_DEPLOYMENT" -ne "1" ]]; then
  "$KUDU_SYNC_CMD" -v 50 -f "$DEPLOYMENT_SOURCE" -t "$DEPLOYMENT_TARGET" -n "$NEXT_MANIFEST_PATH" -p "$PREVIOUS_MANIFEST_PATH" -i ".git;.hg;.deployment;deploy.sh"
  exitWithMessageOnError "Kudu Sync failed"
fi

# 2\. Select node version
selectNodeVersion

# 3\. Install npm packages and build
if [ -e "$DEPLOYMENT_TARGET/package.json" ]; then
  cd "$DEPLOYMENT_TARGET"

  echo "NPM_CMD is: $NPM_CMD"
  echo "NODE_EXE is: $NODE_EXE"

  echo "Running $NPM_CMD install --production"
  eval $NPM_CMD install --production
  exitWithMessageOnError "npm install failed"

  echo "Running $NPM_CMD run build"
  eval $NPM_CMD run build
  exitWithMessageOnError "build failed"

  #Copy build back to root
  eval cp build/* .

  # echo "Running $NPM_CMD run dev"
  # eval $NPM_CMD run dev
  # # eval npm run build
  # exitWithMessageOnError "dev failed"

  cd - > /dev/null
fi

##################################################################################################################################
echo "Finished successfully."
```

还值得一提的是，在 Azure 中运行 npm install 和 npm build 非常慢——一个几乎没有依赖关系的小应用程序很容易就花了 400 多秒来部署。

# 整理笔记

根据提供的细节，你应该能够在 Azure 上启动并运行 nuxt。老实说，虽然我自己也经历过这一旅程(没有任何其他选择)，但我还是强烈建议使用另一种云服务进行托管。还有许多其他供应商****具有健壮和简化的流程，它们只需工作*** ，无需任何定制配置或 hastle。如果 Azure 是你的项目的一个需求，至少你现在可以用这些知识来应对挑战了！*
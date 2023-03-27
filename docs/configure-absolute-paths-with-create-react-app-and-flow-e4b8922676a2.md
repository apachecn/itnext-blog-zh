# 使用 Create React App 和 Flow 配置绝对路径

> 原文：<https://itnext.io/configure-absolute-paths-with-create-react-app-and-flow-e4b8922676a2?source=collection_archive---------0----------------------->

![](img/5908ad34f0683a1177a3eacdf182bcec.png)

我花了一些时间来收集信息，用 flow 配置 CRA 项目，以处理导入语句中的绝对路径，所以我决定记录我使其工作的步骤。

我有以下版本的相关软件包:

```
react-scripts v1.1.4
flow v0.72.0
eslint-plugin-import v2.10.0
```

# TL；速度三角形定位法(dead reckoning)

创造。项目根目录中的 env 文件，包含以下内容

```
NODE_PATH=src/
```

给你补充以下内容。eslintrc 文件(确保您的 devDependencies 中安装了[eslint-plugin-importy](https://github.com/benmosher/eslint-plugin-import))

```
{
... "settings": { "import/resolver": { "node": { "moduleDirectory": ["node_modules", "src/"] } } }}
```

将以下内容添加到您的。流程配置文件

```
[options]
module.system.node.resolve_dirname=node_modules
module.system.node.resolve_dirname=src
```

# 虚拟代码

要使 VSCode 理解绝对路径，请创建 jsconfig.js 并添加以下内容

```
{ "compilerOptions": { "baseUrl": ".", "paths": { "*": ["src/*"] } }}
```

# 智能理念

要让 Intellij Idea 停止抱怨绝对路径，右键单击 src 文件夹，将目录标记为源根目录。
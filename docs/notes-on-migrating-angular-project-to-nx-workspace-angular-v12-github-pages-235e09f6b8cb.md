# 关于将 Angular 项目迁移到 Nx Workspace & Angular v12 & Github 页面的说明

> 原文：<https://itnext.io/notes-on-migrating-angular-project-to-nx-workspace-angular-v12-github-pages-235e09f6b8cb?source=collection_archive---------1----------------------->

![](img/39139207d7165a7e978e603a15dae12d.png)

[https://unsplash.com/photos/hq96V-dazC4](https://unsplash.com/photos/hq96V-dazC4)

我接触投资组合网站已经有一段时间了。因此，今天我决定升级这个项目，并在这个过程中利用更好的技术。这些变化背后的原因如下:

*   Monorepos architect 可以轻松扩展和管理
*   Angular Lazyload 模块->更多的项目不会影响现有的性能。
*   快速部署和构建流程+ CI / CD 支持
*   项目间共享库、模型和服务
*   JavaScript 开发人员友好
*   更大的

事不宜迟，这是将现有的 Angular 项目迁移到 Nx Workspace & Angular v12 并将其部署到 Github 页面的过程

# 将现有 Angular 升级到最新版本(Angular v 12——在撰写本文时)

我试图跟踪 Nx 工作区，并希望通过运行一行代码就可以将现有项目迁移到 Nx 工作区，但这并没有发生:(

所以我必须遵循手动迁移流程。我不想将现有的 Angular 迁移并更新到最新版本。最好先升级，然后再进行迁移。

您可以查看[角度更新指南](https://update.angular.io/)进行迁移。然而，只要运行这行代码，它就会告诉你该怎么做。

```
ng update
```

# 迁移到 Nx 工作空间

升级到 Angular 的最新版本后，就到了[迁移到 Nx 工作空间](https://nx.dev/latest/angular/migration/migration-angular)的时候了。最佳实践是创建另一个分支来测试手动过程，这样就不会与您当前的分支混淆。

```
git checkout -b nx 
```

如果幸运的话，这个命令有助于将 Angular CLI 工作空间转换为 Nx 工作空间。

```
ng add @nrwl/workspace
```

如果你不幸运——像我一样，你可以一直遵循手动流程。

**第一步:将现有文件夹移动到临时文件夹**

你所需要的就是 **src** 文件夹，其他文件如果不需要可以删除。

**第二步:生成一个分支新的 Nx 工作空间**

```
npx create-nx-workspace myorg --preset=angular --linter eslint
```

当提示输入`application name`时，输入当前`angular.json`文件中的*项目名称*。

第三步:复制申请文件

您的应用程序代码自包含在 Angular CLI 工作区的`src`文件夹中。

*   将 Angular CLI 项目中的`src`文件夹复制到`apps/<app name>`文件夹，覆盖现有的`src`文件夹。
*   将任何特定于项目的文件，如`browserslist`或服务人员配置文件复制到`apps/<app name>`文件夹下的相对路径中。
*   将`assets`、`scripts`、`styles`和特定于构建的配置(如服务人员配置)从您的 Angular CLI `angular.json`传输到 Nx workspace `angular.json`文件。

**步骤 4:运行并测试新项目**

通过运行以下命令来验证您的应用程序是否正常运行:

```
ng serve <app name>
```

然后继续检查你的测试&林挺。确保一切正常。如果不是，请遵循 [Nx 指令](https://nx.dev/latest/angular/migration/migration-angular)。

**配置 Github 动作，将项目部署到 Github 页面**

我已经为 Github 动作设置了 CI/CD。有了这个结构，我必须做一些改变，以适应新的目标输出。现在，我的应用程序将位于:

```
dist/apps/<app-name>
```

这是 Github 操作的 YAML 文件

```
# .github/workflows/ci.yamlname: Pull request or push to Dev Branchon:
  push:
    branches:
      - devjobs:
  build-and-deploy:
    timeout-minutes: 30
    name: Deploy website and sentry
    runs-on: ubuntu-lateststeps:
      - name: Checkout
        uses: actions/checkout@v1- uses: actions/setup-node@v1
        with:
          node-version: '14.x'- uses: actions/cache@v2
        with:
          path: '**/node_modules'
          key: ${{ runner.os }}-modules-${{ hashFiles('**/yarn.lock') }}- name: Install dependencies
        if: steps.yarn-cache.outputs.cache-hit != 'true'
        run: yarn install- name: Build
        run: yarn build- name: Generate 404 files
        run: |
          echo "---" > "./dist/apps/portfolio/404.html"
          echo "permalink: /404.html" >> "./dist/apps/<app-name>/404.html"
          echo "---" >> "./dist/apps/<app-name>/404.html"
          cat "./dist/apps/<app-name>/index.html" >> "./dist/apps/<app-name>/404.html"- name: Deploy
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
          BRANCH: master
          FOLDER: dist/apps/<app-name>
```

这里有一些关于 YAML 档案的笔记

安装依赖项时，使用操作缓存以节省时间。

```
- uses: actions/cache@v2
        with:
          path: '**/node_modules'
          key: ${{ runner.os }}-modules-${{ hashFiles('**/yarn.lock') }}
```

确保构建完成后，创建一个与`index.html`相同的`404.html`文件。否则，如果您直接从 URL 手动访问[页面，它将不会解析路由。](https://angular.io/guide/deployment#deploy-to-github-pages)

```
- name: Generate 404 files
        run: |
          echo "---" > "./dist/apps/portfolio/404.html"
          echo "permalink: /404.html" >> "./dist/apps/<app-name>/404.html"
          echo "---" >> "./dist/apps/<app-name>/404.html"
          cat "./dist/apps/<app-name>/index.html" >> "./dist/apps/<app-name>/404.html"
```

在新的设置之后，CI/CD 时间显著减少，从大约 7 分钟减少到不到 1 分钟。您可以看一下运行示例。

[](https://github.com/dalenguyen/dalenguyen.github.io/actions/runs/867843285) [## dalen guyen/dalen guyen . github . io

### 个人网站与 Github 页面和 Angular。通过创建……为 dalen guyen/dalen guyen . github . io 开发做出贡献

github.com](https://github.com/dalenguyen/dalenguyen.github.io/actions/runs/867843285) 

这是我的笔记的结尾，希望你会觉得有用。
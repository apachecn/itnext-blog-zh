# 什么是 Jenkins 共享库，为什么你应该使用它们

> 原文：<https://itnext.io/jenkins-shared-libraries-part-1-5ba3d072536a?source=collection_archive---------2----------------------->

## Jenkins 共享库系列文章的第一部分

![](img/2bd26feac6a62d72b43262eceec17ed8.png)

[https://jenkins.io/](https://jenkins.io/)

Jenkins 是目前最常用的持续集成/部署服务器之一。Jenkins 可以将持续集成或部署任务表示为由一个或多个“阶段”组成的“管道”,并且通常被写入一个通常名为“Jenkinsfile”的文件中。Jenkinsfile 包含阶段执行的顺序，并且可以包含这些阶段的逻辑。Jenkinsfile 也可以签入源代码管理。

编写 Jenkinsfile 时，它很容易变得臃肿，最终可能会变成数百行代码。Jenkinsfile 没有层次或继承的概念，因此任何公共函数或逻辑都必须从一个文件复制到另一个文件，以便共享该逻辑。希望这一切听起来很可怕。别担心，我将向您展示如何使用 Jenkins 共享库来解决这个问题。

# 什么是詹金斯共享库？

Jenkins 共享库是用 [Groovy](https://groovy-lang.org/) 编写的，允许你创建通用的逻辑集，并在团队/项目/组织之间共享。您不必从其他 Jenkins 文件中“复制和粘贴”代码，只需将一个库加载到 Jenkins 中，Jenkins 主服务器上的每个管道作业都可以访问该共享库。

当编写共享库时，代码必须以一种[特定的方式](https://jenkins.io/doc/book/pipeline/shared-libraries/#directory-structure)构建。

![](img/b9230e460052ec78c59867a5ced5285a.png)

[https://Jenkins . io/doc/book/pipeline/shared-libraries/# directory-structure](https://jenkins.io/doc/book/pipeline/shared-libraries/#directory-structure)

“vars”目录包含 groovy 脚本，按照 Jenkins 的说法，这些脚本被称为“全局 vars”。这些是您希望在 Jenkinsfile 文件中公开使用的步骤。假设您希望创建一个共享步骤来处理一些代码的部署，您可以将名为“deploy.groovy”的文件添加到“vars”目录中。

```
(root)
src/
    ...
vars/
    deploy.groovy
    deploy.txt
```

这将允许像这样调用部署步骤

```
node() {
    stage(‘deploy’){
        deploy()  // deploy var from shared library
    }
}
```

# 为什么应该在管道中使用 Jenkins 共享库

假设您的团队中有三个项目，它们都以相同的方式部署代码。你不会想把这个逻辑写三遍。这违反了 DRY(不要重复 youreslf)原则。这意味着，如果过程发生了变化，那么每个团队都必须更新他们的 Jenkinsfile 来适应这些变化。这通常包括复制其他人所做的，然后将代码粘贴到他们的项目中，然后进行小的调整以适应他们的项目。

使用共享库，部署项目的代码只需编写一次，然后作为库版本的简单更新提供给所有其他团队。这种关注点的分离将允许团队将他们所有的注意力放在编写代码上，而不是担心如何编写代码来部署代码、进行自动发布等，这反过来节省了时间和金钱。

共享库的另一个优点是它们促进了团队之间的协作。有时候团队会低着头，不知道其他人在做什么。这通常会导致相同的代码被多次编写。共享库可以在做类似事情的团队之间架起一座桥梁，并允许这些团队在一段共享的代码上一起工作，这对他们以及任何其他团队都有好处。

让我们看一下部署 javascript 应用程序的 Jenkinsfile。

## 原始 Jenkinsfile

```
node {
    checkout scm stage('Install') {
        sh 'npm install'
    } stage('Test') {
        sh 'npm test'
    } stage('Deploy') {
        if (deploy == true) {
            sh 'npm publish'
        }
    }
}
```

现在单单这一点看起来还不算太糟。当您的团队有 3 个不同的项目，并且都以相同的方式部署一个 javascript 应用程序时，问题就来了。因此，我们可以利用共享库，而不仅仅是在 3 个项目之间复制和粘贴代码。

我们可以从 Jenkinsfile 中取出代码，并将其包装在一个全局变量中，这样整个管道就可以作为 Jenkins 的一个名为“buildJavascriptApp”的步骤，供其他任何人在他们自己的 Jenkinsfile 中使用。

```
def call() {
    node {
        checkout scm stage('Install') {
            sh 'npm install'
        } stage('Test') {
            sh 'npm test'
        } stage('Deploy') {
            if (deploy == true) {
                sh 'npm publish'
            }
        }
    }
}
```

使用这个全局变量将原来的 Jenkinsfile 变成:

## 新 Jenkinsfile

```
buildJavascriptApp deploy: true
```

我们刚刚把一个 14 行的 Jenkinsfile 变成了一个 1 行的 Jenkinsfile。这在现在看起来可能没什么，但是当 Jenkinsfiles 开始变大时，共享库的优势就变得很明显了。

另一种方法是利用 [Groovy 闭包](http://www.groovy-lang.org/closures.html)。这允许一个项目使用全局共享管道，同时也增加了它。因此，如果您的项目执行 javascript 构建过程，同时还向 slack 发送一条消息来通知构建成功或失败，那么您可以扩展库步骤，并在公共逻辑完成后传递您想要执行的逻辑。

```
def call(Closure body) {
    node {
        checkout scm stage('Install') {
            sh 'npm install'
        } stage('Test') {
            sh 'npm test'
        } stage('Deploy') {
            if (deploy == true) {
                sh 'npm publish'
            }
        } body()
    }
}
```

我们看到这个全局变量接受闭包作为参数。这是您希望在公共阶段之后执行的额外代码块。有了这样的共享变量，我们可以像这样使用它:

```
buildJavascriptApp deply: true {
    stage('Notify') {
        slackSend(...)  // send a message to slack
    }
}
```

希望这些例子显示了共享库的强大，以及它们为团队提供的灵活性和使用 Jenkins 的持续集成工作。

# 警告

编写 Jenkins 共享库的一个缺点是代码是专门为 Jenkins 编写的。理由是，如果在某个时候您或您的团队想要转移到另一个 CI 系统，就需要重写包含在共享库中的代码。如果转移到一个新的系统，这个过程很可能会发生。有一种观点认为，编写共享逻辑的方式更容易被其他系统使用，比如 shell 脚本、Ansible playbooks 或 CLI 工具。

# 结论

Jenkins 共享库是帮助保持 Jenkinsfile 简洁易读的好方法。当某个过程中可能发生变化时，这些库减少了手动更新多个 Jenkinsfiles 的麻烦和时间。它们还允许每个项目的 Jenkinsfile 关注“什么”需要发生，并让共享库负责“如何”发生。

感谢您阅读我的 Jenkins 共享库系列的第 1 部分。

在我的 Jenkins 共享库系列的第二部分中，我将通过一步一步的教程向你展示如何创建你自己的共享库。

# 我的詹金斯共享图书馆系列

[](https://medium.com/@werne2j/jenkins-shared-libraries-part-1-5ba3d072536a) [## 什么是 Jenkins 共享库，为什么你应该使用它们

### Jenkins 共享库系列文章的第 1 部分

medium.com](https://medium.com/@werne2j/jenkins-shared-libraries-part-1-5ba3d072536a) [](https://medium.com/@werne2j/how-to-build-your-own-jenkins-shared-library-9dc129db260c) [## 如何构建自己的 Jenkins 共享库

### 我的 Jenkins 共享库系列的第二部分

medium.com](https://medium.com/@werne2j/how-to-build-your-own-jenkins-shared-library-9dc129db260c) [](https://medium.com/@werne2j/unit-testing-a-jenkins-shared-library-9bfb6b599748) [## Jenkins 共享库的单元测试

### Jenkins 共享库系列文章的第三部分

medium.com](https://medium.com/@werne2j/unit-testing-a-jenkins-shared-library-9bfb6b599748) [](https://medium.com/@werne2j/collecting-code-coverage-for-a-jenkins-shared-library-c2d8f502732e) [## 收集 Jenkins 共享库的代码覆盖率

### 我的 Jenkins 共享库系列的第四部分

medium.com](https://medium.com/@werne2j/collecting-code-coverage-for-a-jenkins-shared-library-c2d8f502732e)
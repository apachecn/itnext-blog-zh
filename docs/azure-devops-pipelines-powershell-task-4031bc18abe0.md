# Azure DevOps 管道:PowerShell 任务

> 原文：<https://itnext.io/azure-devops-pipelines-powershell-task-4031bc18abe0?source=collection_archive---------7----------------------->

这将是一个展示在管道中使用 PowerShell 任务的快速帖子。帖子中没有什么是真正针对我们在过去几周使用的 Azure DevOps 项目的，但以防万一你对 Azure DevOps 和/或本系列完全陌生，你可以使用以下帖子开始。

[Azure devo PS 入门](https://elanderson.net/2020/02/getting-started-with-azure-devops/)
[Azure devo PS 中的管道创建](https://elanderson.net/2020/03/pipeline-creation-in-azure-devops/)
[Azure DevOps 为 ASP.NET 核心发布工件](https://elanderson.net/2020/03/azure-devops-publish-asp-net-core/)
[Azure DevOps 管道:YAML 的多个作业](https://elanderson.net/2020/03/azure-devops-pipelines-multiple-jobs-in-yaml/)
[Azure DevOps 管道:可重用的 YAML](https://elanderson.net/2020/03/azure-devops-pipelines-reuseable-yaml/)
[Azure DevOps 管道:跨 Repos 使用 YAML](https://elanderson.net/2020/04/azure-devops-pipelines-use-yaml-across-repos/)
[Azure devo PS 管道:YAML 的条件](https://elanderson.net/2020/04/azure-devops-pipelines-conditionals-in-yaml/)

![](img/80f42590e7beded1705152cfc55b9616.png)

## PowerShell 任务

PowerShell 任务将允许您做几乎任何事情。如果没有现有的 DevOps 任务更符合您的需求，您可以找到一种方法，使用 PowerShell 任务在运行该任务的计算机甚至外部计算机的上下文中完成您的需求，这取决于您的网络和安全设置。下面是我添加到管道中的一个示例任务，它将所有环境变量输出到日志中。这是一个内联脚本，但是您也可以从文件运行脚本。另外，请注意，这在 Windows 和 Linux 代理上都有效。

```
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: 'Get-ChildItem -Path Env:\'
```

虽然这个脚本对于生产管道不是非常有用，但我经常在设置管道时使用它，以便更好地了解变量方面的可用内容。此外，请记住，根据运行的触发条件，这些变量可能会有所不同。例如，如果一个运行是由一个拉请求触发的，那么您将拥有许多与拉请求相关的变量。下面是这个命令在我的测试项目上的输出，它是通过一个 pull 请求触发的，因此包含了一组 SYSTEM_PULLREQUEST_x 变量，这些变量包含了关于 pull 请求的信息。代理运行的是 Linux。

```
Name                           Value
----                           -----
AGENT_ACCEPTTEEEULA            True
AGENT_BUILDDIRECTORY           /home/vsts/work/1
AGENT_DISABLELOGPLUGIN_TESTFI… true
AGENT_DISABLELOGPLUGIN_TESTRE… true
AGENT_HOMEDIRECTORY            /home/vsts/agents/2.165.2
AGENT_ID                       9
AGENT_JOBNAME                  Build WebApp1
AGENT_JOBSTATUS                Succeeded
AGENT_MACHINENAME              fv-az563
AGENT_NAME                     Hosted Agent
AGENT_OS                       Linux
AGENT_OSARCHITECTURE           X64
AGENT_READONLYVARIABLES        true
AGENT_RETAINDEFAULTENCODING    false
AGENT_ROOTDIRECTORY            /home/vsts/work
AGENT_TEMPDIRECTORY            /home/vsts/work/_temp
AGENT_TOOLSDIRECTORY           /opt/hostedtoolcache
AGENT_VERSION                  2.165.2
AGENT_WORKFOLDER               /home/vsts/work
agent.jobstatus                Succeeded
ANDROID_HOME                   /usr/local/lib/android/sdk
ANDROID_SDK_ROOT               /usr/local/lib/android/sdk
ANT_HOME                       /usr/share/ant
AZURE_EXTENSION_DIR            /opt/az/azcliextensions
AZURE_HTTP_USER_AGENT          VSTS_08ccc6b2-4e5e-4621-8f5b-3fe0de2efa22_build…
BOOST_ROOT_1_69_0              /usr/local/share/boost/1.69.0
BOOST_ROOT_1_72_0              /usr/local/share/boost/1.72.0
BUILD_ARTIFACTSTAGINGDIRECTORY /home/vsts/work/1/a
BUILD_BINARIESDIRECTORY        /home/vsts/work/1/b
BUILD_BUILDID                  73
BUILD_BUILDNUMBER              merge_20200422.1
BUILD_BUILDURI                 vstfs:///Build/Build/73
BUILD_CONTAINERID              3453972
BUILD_DEFINITIONNAME           Playground
BUILD_DEFINITIONVERSION        4
BUILD_QUEUEDBY                 Microsoft.VisualStudio.Services.TFS
BUILD_QUEUEDBYID               00000002-0000-8888-8000-000000000000
BUILD_REASON                   PullRequest
BUILD_REPOSITORY_CLEAN         False
BUILD_REPOSITORY_GIT_SUBMODUL… False
BUILD_REPOSITORY_ID            ff7a6325-1129-42e3-b095-6a39ef6a6bd3
BUILD_REPOSITORY_LOCALPATH     /home/vsts/work/1/s
BUILD_REPOSITORY_NAME          Playground
BUILD_REPOSITORY_PROVIDER      TfsGit
BUILD_REPOSITORY_URI           https://ericlanderson@dev.azure.com/ericlanders…
BUILD_REQUESTEDFOR             Eric Anderson
BUILD_REQUESTEDFOREMAIL        ericlanderson@outlook.com
BUILD_REQUESTEDFORID           45247cb1-8f49-4c03-a4c5-b03ac3286c99
BUILD_SOURCEBRANCH             refs/pull/10/merge
BUILD_SOURCEBRANCHNAME         merge
BUILD_SOURCESDIRECTORY         /home/vsts/work/1/s
BUILD_SOURCEVERSION            3e2b77c27f31a4c729a5f195b49d2e108500399d
BUILD_SOURCEVERSIONAUTHOR      Eric Anderson
BUILD_SOURCEVERSIONMESSAGE     Merge pull request 10 from docChanges into mast…
BUILD_STAGINGDIRECTORY         /home/vsts/work/1/a
BUILDCONFIGURATION             Release
BUILDWEBAPP2                   false
CHROME_BIN                     /usr/bin/google-chrome
CHROMEWEBDRIVER                /usr/local/share/chrome_driver
COMMON_TESTRESULTSDIRECTORY    /home/vsts/work/1/TestResults
CONDA                          /usr/share/miniconda
DEBIAN_FRONTEND                noninteractive
DOTNET_SKIP_FIRST_TIME_EXPERI… 1
ENDPOINT_URL_SYSTEMVSSCONNECT… [https://dev.azure.com/ericlanderson/](https://dev.azure.com/ericlanderson/)
GECKOWEBDRIVER                 /usr/local/share/gecko_driver
GIT_TERMINAL_PROMPT            0
GOROOT                         /usr/local/go1.14
GOROOT_1_11_X64                /usr/local/go1.11
GOROOT_1_12_X64                /usr/local/go1.12
GOROOT_1_13_X64                /usr/local/go1.13
GOROOT_1_14_X64                /usr/local/go1.14
GRADLE_HOME                    /usr/share/gradle
HOME                           /home/vsts
ImageOS                        ubuntu18
ImageVersion                   20200406.2
INPUT_ARGUMENTS                
INVOCATION_ID                  3e6abb812a484ab39fadc9e8721258ee
JAVA_HOME                      /usr/lib/jvm/zulu-8-azure-amd64
JAVA_HOME_11_X64               /usr/lib/jvm/zulu-11-azure-amd64
JAVA_HOME_12_X64               /usr/lib/jvm/zulu-12-azure-amd64
JAVA_HOME_7_X64                /usr/lib/jvm/zulu-7-azure-amd64
JAVA_HOME_8_X64                /usr/lib/jvm/zulu-8-azure-amd64
JOURNAL_STREAM                 9:30085
LANG                           C.UTF-8
LEIN_HOME                      /usr/local/lib/lein
LEIN_JAR                       /usr/local/lib/lein/self-installs/leiningen-2.9…
M2_HOME                        /usr/share/apache-maven-3.6.3
MSDEPLOY_HTTP_USER_AGENT       VSTS_08ccc6b2-4e5e-4621-8f5b-3fe0de2efa22_build…
PATH                           /opt/microsoft/powershell/7:/usr/share/rust/.ca…
PIPELINE_WORKSPACE             /home/vsts/work/1
POWERSHELL_DISTRIBUTION_CHANN… Azure-DevOps-ubuntu18
PSModulePath                   /home/vsts/.local/share/powershell/Modules:/usr…
RUNNER_TOOLSDIRECTORY          /opt/hostedtoolcache
SELENIUM_JAR_PATH              /usr/share/java/selenium-server-standalone.jar
SWIFT_PATH                     /usr/share/swift/usr/bin
SYSTEM                         build
SYSTEM_ARTIFACTSDIRECTORY      /home/vsts/work/1/a
SYSTEM_COLLECTIONID            08ccc6b2-4e5e-4621-8f5b-3fe0de2efa22
SYSTEM_COLLECTIONURI           [https://dev.azure.com/ericlanderson/](https://dev.azure.com/ericlanderson/)
SYSTEM_CULTURE                 en-US
SYSTEM_DEFAULTWORKINGDIRECTORY /home/vsts/work/1/s
SYSTEM_DEFINITIONID            5
SYSTEM_DEFINITIONNAME          Playground
SYSTEM_ENABLEACCESSTOKEN       SecretVariable
SYSTEM_HOSTTYPE                build
SYSTEM_ISSCHEDULED             False
SYSTEM_JOBATTEMPT              1
SYSTEM_JOBDISPLAYNAME          Build WebApp1
SYSTEM_JOBID                   98395c9e-7365-5c3f-03de-ec42b09a8a98
SYSTEM_JOBIDENTIFIER           WebApp1.__default
SYSTEM_JOBNAME                 __default
SYSTEM_JOBPARALLELISMTAG       Private
SYSTEM_JOBPOSITIONINPHASE      1
SYSTEM_PHASEATTEMPT            1
SYSTEM_PHASEDISPLAYNAME        Build WebApp1
SYSTEM_PHASEID                 a142d6c6-ff80-5cff-8292-5044e2c5b0ef
SYSTEM_PHASENAME               WebApp1
SYSTEM_PIPELINESTARTTIME       2020-04-22 06:11:44-05:00
SYSTEM_PLANID                  726fda14-a3a2-45b1-b745-bef8cf17bdaa
SYSTEM_PULLREQUEST_ISFORK      False
SYSTEM_PULLREQUEST_PULLREQUES… 10
SYSTEM_PULLREQUEST_PULLREQUES… 1
SYSTEM_PULLREQUEST_SOURCEBRAN… refs/heads/docChanges
SYSTEM_PULLREQUEST_SOURCECOMM… ba11cb768bc75ae65ff6b7ac6afb8a2950063f07
SYSTEM_PULLREQUEST_SOURCEREPO… https://ericlanderson@dev.azure.com/ericlanders…
SYSTEM_PULLREQUEST_TARGETBRAN… refs/heads/master
SYSTEM_SERVERTYPE              Hosted
SYSTEM_STAGEATTEMPT            1
SYSTEM_STAGEDISPLAYNAME        __default
SYSTEM_STAGEID                 96ac2280-8cb4-5df5-99de-dd2da759617d
SYSTEM_STAGENAME               __default
SYSTEM_TASKDEFINITIONSURI      [https://dev.azure.com/ericlanderson/](https://dev.azure.com/ericlanderson/)
SYSTEM_TASKDISPLAYNAME         PowerShell
SYSTEM_TASKINSTANCEID          6417fa85-e8cf-55f9-817e-d698bd79d6f7
SYSTEM_TASKINSTANCENAME        PowerShell
SYSTEM_TEAMFOUNDATIONCOLLECTI… [https://dev.azure.com/ericlanderson/](https://dev.azure.com/ericlanderson/)
SYSTEM_TEAMFOUNDATIONSERVERURI [https://dev.azure.com/ericlanderson/](https://dev.azure.com/ericlanderson/)
SYSTEM_TEAMPROJECT             Playground
SYSTEM_TEAMPROJECTID           7550ca2f-9ffe-45b7-abd5-c4e92a4a5f4e
SYSTEM_TIMELINEID              726fda14-a3a2-45b1-b745-bef8cf17bdaa
SYSTEM_TOTALJOBSINPHASE        1
SYSTEM_WORKFOLDER              /home/vsts/work
TASK_DISPLAYNAME               PowerShell
TF_BUILD                       True
USER                           vsts
VCPKG_INSTALLATION_ROOT        /usr/local/share/vcpkg
VSTS_AGENT_PERFLOG             /home/vsts/perflog
VSTS_PROCESS_LOOKUP_ID         vsts_54420f58-c41f-4a43-8ce8-bbbac5023620
```

我不知道你是怎么想的，但是能够看到内置路径变量实际映射到什么路径对我很有帮助，特别是当文件需要移动的时候。

## 包扎

如上所述，您可以使用 PowerShell 任务做任何事情。从读取 JSON 文件到为 QA 构建 VM，我已经用它做了很多事情。如果你以前没有使用过这个任务，我希望这篇文章可以帮助你开始，让你看到 PowerShell 任务可以做的大量事情。

*原载于*[](https://elanderson.net/2020/05/azure-devops-pipelines-powershell-task/)**。**
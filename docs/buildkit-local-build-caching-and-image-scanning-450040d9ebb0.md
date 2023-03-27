# Buildkit:本地构建、缓存和图像扫描

> 原文：<https://itnext.io/buildkit-local-build-caching-and-image-scanning-450040d9ebb0?source=collection_archive---------6----------------------->

在之前的文章[第 1 部分](/jenkins-k8s-building-docker-image-without-docker-d41cffdbda5a)和[第 2 部分](/jenkins-k8s-buildkit-life-behind-the-corporate-proxy-cb052bd7f969)中，我们讨论了在 Jenkins 管道中使用 Buildkit 构建 Docker 映像。所描述的方法有一个缺陷。

![](img/d9dcd3857a41a7fde22e982151e99de2.png)

看下面这个阶段:

```
stage('Build Docker Image') {
            container('buildkit') {
                sh """
                  buildctl build \
                      --frontend dockerfile.v0 \
                      --local context=. \
                      --local dockerfile=. \
                      --output type=image,name=${image},push=true
                  buildctl build \
                      --frontend dockerfile.v0 \
                      --local context=. \
                      --local dockerfile=. \
                      --output type=image,name=${repository}:${tag},push=true
                """
                milestone(1)
            }
        }
```

在这种情况下，`buildctl`执行了两次，这两次都将运行完整的构建，从而使阶段执行时间加倍。

这个可以改进吗？

是的，Buildkit 的缓存效率很高，我们只是在上面描述的阶段没有使用它。

为了利用 Buildkit 缓存层的能力，我们必须在第一次执行`buildctl`时将其导出到本地容器文件系统，然后在第二次运行时将其导入。

要将图层导出到本地缓存，添加以下选项:`--export-cache type=local,dest=/tmp/buildkit/cache`。

要稍后导入它，请添加以下选项:`--import-cache type=local,src=/tmp/buildkit/cache`。

如果管道中有两个以上对`buildctl`工具的调用，那么除了第一个调用之外，对所有调用都使用`--export-cache`和`--import-cache`选项是有意义的。

因此，要在上面的示例阶段利用缓存，我们必须将其修改如下:

```
stage('Build Docker Image') {
            container('buildkit') {
                sh """
                  buildctl build \
                      --frontend dockerfile.v0 \
                      --local context=. \
                      --local dockerfile=. \
                      --export-cache type=local,dest=/tmp/buildkit/cache \
                      --output type=image,name=${image},push=true
                  buildctl build \
                      --frontend dockerfile.v0 \
                      --local context=. \
                      --local dockerfile=. \
                      --import-cache type=local,src=/tmp/buildkit/cache \
                      --output type=image,name=${repository}:${tag},push=true
                """
                milestone(1)
            }
        }
```

我还想讨论另外两个使用案例。

其中之一是在 Docker 映像构建期间编译源代码。历史上，我们首先编译结果 executble 或 archive(在 Java 世界中),然后以某种方式将其提供给 Docker 映像。

然而，作为 Docker 映像构建的一部分，没有什么能真正阻止我们这样做。一个小问题阻止了我在管道中使用这种方法多年。

你看，我通常会将测试报告发布到 Jenkins 作业运行中，这样就可以通过 Jenkins fronetend 访问它们。从图像中提取这些报告并不简单。

当使用 plain Docker 构建映像时，我必须首先构建映像，并在构建阶段结束时停止。然后，我必须从该映像创建一个容器，并将报告和其他文件从映像复制到主机文件系统。

然后，我必须再次构建它，直到获得最终将被推送到注册表的最终图像。

Buildkiy 简化了这项任务。它支持结果图像的多种输出格式。它可以作为目录结构刷新到磁盘，或者作为 OCI 映像存储。tar 文件。因此，在这种情况下，我可能更喜欢将映像构建为本地目录，然后将文件复制出来。

假设我们有以下构建 Spring Boot 应用程序的 Dockerfile 文件:

```
FROM eclipse-temurin:17.0.4.1_1-jdk as sourceARG version="1.0.0"
ARG build_numberENV BUILD_NUMBER=${build_number}RUN mkdir /appCOPY .gradle.properties /root/.gradle/gradle.propertiesCOPY gradle /app/gradle
COPY build.gradle gradle.properties gradlew settings.gradle sonar-project.properties /app/WORKDIR /appRUN ./gradlew cleanFROM source as buildARG version="1.0.0"
ARG build_numberENV BUILD_NUMBER=${build_number}COPY src /app/src
COPY test /app/testRUN ./gradlew buildFROM eclipse-temurin:17.0.4.1_1-jre as appARG version="1.0.0"
ARG build_numberRUN groupadd -g 1001 devops \
    && useradd -u 1001 -g 1001 devops \
    && mkdir -p /app/config \
    && chown -R devops /appENV BUILD_NUMBER=${build_number}COPY --from=build /app/build/libs/service-${version}.${build_number}.jar /app/service-${version}.${build_number}.jar
RUN ln -s /app/service-${version}.${build_number}.jar /app/service.jarRUN mkdir -p /app/config \
 && chown -R devops /appEXPOSE 8080WORKDIR /app
USER devopsFROM app as debugCMD ["java", "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005", "-jar", "/app/service.jar"]FROM appCMD java $JAVA_OPTS -jar /app/service.jar
```

因此，要构建它，我需要首先将它运行到构建目标。在这种情况下,“构建 Java 代码”阶段应该如下所示:

```
container('buildkit') {
            stage('Build Java Code') {
                try {
                    sh """
                        mkdir -p /root/.gradle; cat /etc/.gradle/gradle-new.properties > /root/.gradle/gradle.properties
                        cp /etc/.gradle/gradle-new.properties .gradle.properties
                        chmod 755 ./gradlew
                        buildctl build \
                             --frontend dockerfile.v0 \
                             --local context=. \
                             --local dockerfile=. \
                             --export-cache type=local,dest=/tmp/buildkit/cache \
                             --output type=local,dest=/tmp/app \
                             --opt network=host \
                             --opt build-arg:version=${version} \
                             --opt build-arg:build_number=${env.BUILD_NUMBER} \
                             --opt target=build
                        cp -R /tmp/app/app/build ./
                    """
                } catch (error) {
                    step([$class: 'Mailer',
                        notifyEveryUnstableBuild: true,
                        recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                                        [$class: 'DevelopersRecipientProvider']]),
                        sendToIndividuals: true])
                    throw error
                } finally {
                    step([$class: 'JUnitResultArchiver', testResults: 'build/test-results/test/*.xml'])
                    step([$class: 'JacocoPublisher',
                        execPattern: 'build/jacoco/*.exec',
                        classPattern: 'build/classes',
                        sourcePattern: 'src/main/java',
                        exclusionPattern: 'src/test*'
                    ])
                }
            }
```

注意，在执行 buildctl 之后，我们将构建目录复制回工作区:`cp -R /tmp/app/app/build ./`。

然后在最后一个模块中，我们将测试结果和 Jacoco 测试结果发布给 Jenkins job。

此外，我们现在可以从 sonar cube 容器中运行 sonar cube 扫描，因为所有必需的文件都在工作区中。

请注意，我们正在缓存层，因此，如果我们现在运行构建到结束，Buildkit 将不会再次编译代码，而只是使用缓存层。

我想在管道中做的另一件事是，我想扫描结果图像的漏洞。

历史上，我们使用 Anchore 服务器扫描图像的漏洞。使用这种方法，图像应该在扫描之前被推送到注册表，这并不理想，因为如果我们在图像中发现漏洞，它将不会被使用，而只是挂在注册表中，占用磁盘空间。

因此，理想情况下，我们希望在推送图像之前，先对其进行本地扫描。此外，我们希望推动图像只有当它是从主分支机构建立的，但我们希望扫描每个分支机构和每个公关。

Grype 的出现让我们可以轻松地在本地扫描图像，但是在使用 Buildkit 方法时，如前所述，我们无法访问有问题的图像:我们构建并立即推送它。

因此，为了扫描生成的图像，我们希望构建它并将其存储在本地，这次可能是作为 OCI 图像。tar 文件。

这一阶段将如下所示:

```
stage('Build Docker Image') {
    try {
        sh """
            buildctl build \
               --frontend dockerfile.v0 \
               --local context=. 
               --local dockerfile=. \
               --export-cache type=local,dest=/tmp/buildkit/cache \
               --import-cache type=local,src=/tmp/buildkit/cache \
               --output type=oci,dest=/tmp/image.tar \
               --opt network=host \
               --opt build-arg:version=${version} \
               --opt build-arg:build_number=${env.BUILD_NUMBER}
        """
    } catch (error) {
        step([$class: 'Mailer',
            notifyEveryUnstableBuild: true,
            recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                            [$class: 'RequesterRecipientProvider']]),
            sendToIndividuals: true])
            throw error
    }
}
```

请注意，我们在 stage 中同时使用了`--export-cache`和`--import-cache`，生成的图像被存储为`/tmp/image.tar`文件。

现在我可以对。tar 文件，因为 grype 支持 OCI 图像。我将使用以下阶段来扫描图像:

```
stage('Scan Docker Image') {
            try {
                sh """
     wget [https://raw.githubusercontent.com/anchore/grype/main/install.sh](https://raw.githubusercontent.com/anchore/grype/main/install.sh)
     chmod 755 install.sh./install.sh
     mv bin/grype /usr/local/bin/

                    GRYPE_MATCH_GOLANG_USING_CPES=false \
                          /usr/local/bin/grype \
                                  oci-archive:/tmp/image.tar \
                                  -f high \
                                  --scope all-layers \
                                  -o template \
                                  --file report.html \
                                  -t grype.tmpl \
                                  --only-fixed
                """
            } catch (error) {
                step([$class: 'Mailer',
                    notifyEveryUnstableBuild: true,
                    recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                                    [$class: 'RequesterRecipientProvider']]),
                    sendToIndividuals: true])
                    throw error
            } finally {
                publishHTML (target : [allowMissing: false,
                 alwaysLinkToLastBuild: true,
                 reportDir: '',
                 keepAll: true,
                 reportFiles: 'report.html',
                 reportName: 'Grype Scan Report',
                 reportTitles: 'Grype Scan Report'])
            }
            milestone()
            }
```

以上阶段假设`grype.tmpl`文件在工作区中可用。此文件是为 grype 扫描生成 html 报告并将其发布到 Jenkins 作业运行所必需的。更多关于在 grype 中使用模板的细节可以在[这里](https://github.com/anchore/grype#using-templates)找到。

因此，现在如果所有阶段都成功了，并且如果正在讨论的构建是针对主分支的，那么我们可以像以前一样构建和推送结果映像。

```
if (env.BRANCH_NAME == "master") {
            //Release stage is only executed from the 'master' branch
            stage('Tagging Source Code') {
                values = version.tokenize(".")
                def repositoryCommitterEmail = "[jenkins@iktech.io](mailto:jenkins@iktech.io)"
                def repositoryCommitterUsername = "jenkinsCI"sh "git config user.email ${repositoryCommitterEmail}"
                sh "git config user.name '${repositoryCommitterUsername}'"
                sh "git tag -d v${values[0]} || true"
                sh "git push origin :refs/tags/v${values[0]}"
                sh "git tag -d v${values[0]}.${values[1]} || true"
                sh "git push origin :refs/tags/v${values[0]}.${values[1]}"
                sh "git tag -d v${version} || true"
                sh "git push origin :refs/tags/v${version}"sh "git tag -a v${values[0]} -m \"passed CI\""
                sh "git tag -a v${values[0]}.${values[1]} -m \"passed CI\""
                sh "git tag -a v${version} -m \"passed CI\""
                sh "git tag -a v${version}.${env.BUILD_NUMBER} -m \"passed CI\""
                sh "git push --tags"
            }
            milestone()stage('Push Docker Image to the registry') {
                container('buildkit') {
                    try {
                        sh """
                            wget [https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.6.0/linux-amd64/docker-credential-ecr-login](https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.6.0/linux-amd64/docker-credential-ecr-login) -O /usr/local/bin/docker-credential-ecr-login
          chmod 755 /usr/local/bin/docker-credential-ecr-loginmkdir -p /root/.docker
          cp /tmp/docker/config.json /root/.docker/
                            buildctl build \
                                 --frontend dockerfile.v0 \
                                 --local context=. \
                                 --local dockerfile=. \
                                 --output type=image,name=${image},push=true \
                                 --export-cache type=local,dest=/tmp/buildkit/cache \
                                 --import-cache type=local,src=/tmp/buildkit/cache \
                                 --opt network=host \
                                 --opt build-arg:version=${version} \
                                 --opt build-arg:build_number=${env.BUILD_NUMBER}
                            buildctl build \
                                --frontend dockerfile.v0 \
                                --local context=. \
                                --local dockerfile=. \
                                --output type=image,name=${repository}:latest,push=true \
                                --export-cache type=local,dest=/tmp/buildkit/cache \
                                --import-cache type=local,src=/tmp/buildkit/cache \
                                --opt network=host \
                                --opt build-arg:version=${version} \
                                --opt build-arg:build_number=${env.BUILD_NUMBER}
                        """
                    } catch (error) {
                        step([$class: 'Mailer',
                            notifyEveryUnstableBuild: true,
                            recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                                            [$class: 'RequesterRecipientProvider']]),
                            sendToIndividuals: true])
                            throw error
                    }
                }
            }stage('Publish Service docker image version to the artifactz.io') {
                publishArtifact name: 'service',
                                description: 'Test Service',
                                type: 'DockerImage',
                                stage: 'Development',
                                flow: 'Simple',
                                version: "${version}.${env.BUILD_NUMBER}"
            }stage('Push Service docker image version to the Automated Integration Testing stage') {
                pushArtifact name: 'service', stage: 'Development'
            }
        } else {
            echo 'Skipping release for branch [' + env.BRANCH_NAME + ']. Release are only executed from the master.'
            stage('Notify') {
                node('master') {
                    if(!hudson.model.Result.SUCCESS.equals(currentBuild.getPreviousBuild()?.getResult())) {
                        step([$class: 'Mailer',
                            notifyEveryUnstableBuild: true,
                            recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                                            [$class: 'RequesterRecipientProvider']]),
                            sendToIndividuals: true])
                    }
                }
            }
        }
```

如您所见，Buildkit 为我们在管道中的工作提供了很大的灵活性。
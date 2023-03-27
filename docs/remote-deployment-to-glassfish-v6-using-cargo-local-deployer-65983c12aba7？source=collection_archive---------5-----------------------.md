# 使用 Cargo Local Deployer 远程部署到 Glassfish v6

> 原文：<https://itnext.io/remote-deployment-to-glassfish-v6-using-cargo-local-deployer-65983c12aba7?source=collection_archive---------5----------------------->

![](img/bbe525b66addc9069a161aebd3e74370.png)

> *这是对现有* [*的补充，使用 Cargo maven 插件*](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/docs/deploy-cargo.md) *将 Jakarta EE 9 应用程序部署到 Glassfish v6。*

[Cargo maven 插件](https://codehaus-cargo.github.io) 1.8.3 将为新的 Glassfish v6 包含一个`glasfish6x` *containerId* 。在 1.8.2 或之前的版本中，它允许您使用基于 JSR88 规范(部署)的远程部署器和*运行时*配置来将应用程序部署到正在运行的 Glassfish 服务器。

由于 Jakarta EE 9 和 Glassfish v6 中发生的变化，当切换到使用`glassfish6x`容器时，这将停止工作。

JSR88 在更远的 Jakarta EE 9 中被移除，检查 [6.1.4。删除了 Jakarta EE 9 规范中的 Jakarta Technologies](https://jakarta.ee/specifications/platform/9/jakarta-platform-spec-9.html#a2333) 部分。Glassfish v5 发布系列中存在的`deployment-client`构件在 Glassfish v6 中不可用。

相应地，cargo `glassfish6x`容器将不包含远程部署器，并且您不能像以前一样配置运行时配置。但是您可以使用本地部署器将应用程序部署到远程服务器。

```
<profile>
    <id>glassfish-local-deployer</id>
    <properties>
        <cargo.hostname>localhost</cargo.hostname>
        <cargo.servlet.port>8080</cargo.servlet.port>
        <cargo.glassfish.admin.port>4848</cargo.glassfish.admin.port>
        <cargo.zipUrlInstaller.downloadDir>${project.build.directory}/../installs
        </cargo.zipUrlInstaller.downloadDir>
    </properties> <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven2-plugin</artifactId>
                <configuration>
                    <container>
                        <containerId>glassfish6x</containerId>
                        <zipUrlInstaller>
                            <url>
                               [https://download.eclipse.org/ee4j/glassfish/glassfish-${glassfish.version}.zip](https://download.eclipse.org/ee4j/glassfish/glassfish-${glassfish.version}.zip)
                            </url>
                            <downloadDir>${cargo.zipUrlInstaller.downloadDir}</downloadDir>
                        </zipUrlInstaller>
                        <!-- or use artifactInstaller-->
                        <!--<artifactInstaller>
                                    <groupId>org.glassfish.main.distributions</groupId>
                                    <artifactId>glassfish</artifactId>
                                    <version>${glassfish.version}</version>
                                </artifactInstaller>-->
                    </container>
                    <configuration>
                        <!-- the configuration use to deploy -->
                        <home>${project.build.directory}/glassfish6x-home</home>
                        <properties>
                            <cargo.hostname>${cargo.hostname}</cargo.hostname>
                            <cargo.servlet.port>${cargo.servlet.port}</cargo.servlet.port>
                            <cargo.glassfish.admin.port>${cargo.glassfish.admin.port}
                            </cargo.glassfish.admin.port>
                            <cargo.remote.username>admin</cargo.remote.username>
                            <cargo.remote.password></cargo.remote.password>
                        </properties>
                    </configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

> *以上配置同样适用于现有的* `*glassfish5x*` *containerId。*

副作用是您必须下载 Glassfish dist 的副本，本地部署人员使用 Glassfish 内置的`asadmin`工具命令来执行部署。

> *注意:* `*cargo.hostname*` *是要部署的目标 Glassfish 服务器的主机名/地址。*

执行以下命令，将应用程序打包并部署到目标服务器。

```
mvn clean package cargo:deploy -Pglassfish-local-deployer
```

要取消部署应用程序，请执行以下命令。

```
mvn  cargo:undeploy -Pglassfish-local-deployer
```

查看货物官方指南[远程部署到 Glassfish v6](https://codehaus-cargo.github.io/cargo/Remote+deployments+to+GlassFish+6.x.html) 和[货物 glassfish6x containerid](https://codehaus-cargo.github.io/cargo/GlassFish+6.x.html) 的变更。
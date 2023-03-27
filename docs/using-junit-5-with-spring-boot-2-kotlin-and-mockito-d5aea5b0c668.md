# 将 JUnit 5 与 Spring Boot 2、科特林和莫克托一起使用

> 原文：<https://itnext.io/using-junit-5-with-spring-boot-2-kotlin-and-mockito-d5aea5b0c668?source=collection_archive---------0----------------------->

有点拗口，但不是*那种*难搞的工作；-)

JUnit 5 仍然是“最近的”，不同的框架/库正在缓慢地增加对它的支持。问题是，当你迫不及待地想玩闪亮的新玩具时，等待稳定的版本就太无聊了。

这里有一个简短的指南，解释如何将 JUnit 5+与 Spring Boot 2(目前是 M7)、Kotlin 和 Mockito 一起使用。

在你更进一步之前:是的，这有点“在流血的边缘”，拼图块仍然在移动(例如，Surefire 支持，Java 9+兼容性，…)。

我们将经历的步骤:

*   构建配置
*   为 Mockito 实现一个 JUnit 5+扩展类
*   调整现有测试

# 构建配置

我还在用 Maven(抱歉 Gradle fanboys)，所以只要适应你选择的构建系统就行了。

首先，为 JUnit 和 Surefire 添加一些属性:

```
<!-- Need at least 1.1.x for compatibility with Surefire -->
<junit-platform.version>1.1.0-SNAPSHOT</junit-platform.version>

<!-- Need at least 5.1.x for compatibility with Surefire -->
<junit-jupiter.version>5.1.0-SNAPSHOT</junit-jupiter.version>

<!-- *TODO remove once a newer version of surefire (2.21.1+) is compatible with JUnit 5 and used by Spring Boot* -->
<!-- Reference: https://github.com/junit-team/junit5/issues/809 -->
<maven-surefire-plugin.version>2.19.1</maven-surefire-plugin.version>
```

正如您现在所看到的，我们需要使用不稳定的版本，但是这不应该持续太久；-)

对于 Maven Surefire 插件，请关注发布的[。](https://github.com/junit-team/junit5/issues/809)

接下来，添加 JUnit 依赖项:

```
<!-- JUnit 5 -->
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <scope>test</scope>
    <version>${junit-platform.version}</version>
</dependency>
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-engine</artifactId>
    <scope>test</scope>
    <version>${junit-platform.version}</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <!-- *TODO put back scope test once we can remove the MockitoExtension.kt class* -->
    <scope>compile</scope>
    <version>${junit-jupiter.version}</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <scope>test</scope>
    <version>${junit-jupiter.version}</version>
</dependency>
```

如果您想要支持[参数化测试](http://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)，您还可以添加以下依赖项:

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>${junit-jupiter.version}</version>
</dependency>
```

如果您已经用 JUnit 4 编写了许多测试，那么您可以使用 JUnit 5+的 Vintage 模块来简化转换:

```
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>${junit-jupiter.version}</version>
</dependency>
```

如果您使用 Mockito，那么您还需要添加一个直接依赖项(即，不使用范围测试)；我们将在下一节中了解原因:

```
<!-- *TODO: Remove once we don't need MockitoExtension anymore* -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
</dependency>
```

为了能够使用 Maven 执行测试，您还需要正确配置 Surefire:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${maven-surefire-plugin.version}</version>
    <configuration>
        <failIfNoTests>true</failIfNoTests>
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Test.kt</include>
            <include>**/*Tests.java</include>
            <include>**/*Tests.kt</include>
        </includes>
        <properties>
            <excludeTags>integration-test</excludeTags>
        </properties>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-surefire-provider</artifactId>
            <version>${junit-platform.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit-jupiter.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit-jupiter.version}</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</plugin>
```

关于上面代码片段的一些备注:

*   包含的内容应该符合你的个人喜好
*   excludeTags(您也可以使用 includeTags)允许您过滤您不感兴趣的测试；我通常这样做是为了区分快速和慢速运行的测试，您宁愿只在某个 Maven 概要文件处于活动状态时才执行这些测试。轻松标记和过滤测试的能力是 JUnit 5 的一个很好的改进
*   插件的依赖关系确保 Surefire 的 JUnit 提供程序是活动的；这使得 Surefire 能够找到 JUnit 5+测试

添加以下存储库:

```
<repositories>
    ...
    <repository>
        <id>sonatype-snaphosts</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <snapshots>
            <!-- Always update snapshot JARs -->
            <updatePolicy>always</updatePolicy>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>sonatype-snaphosts</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <!-- Always update snapshot JARs -->
            <updatePolicy>always</updatePolicy>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
    <pluginRepository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/snapshot</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
    ...
</pluginRepositories>
```

最后，在 src/test/resources 中创建一个名为 junit-platform.properties 的文件:

```
# JUnit configuration
junit.jupiter.testinstance.lifecycle.default = per_class
junit.jupiter.conditions.deactivate = *
junit.jupier.extensions.autodetection.enabled = true
```

参考:[http://JUnit . org/JUnit 5/docs/current/user-guide/# running-tests-config-params](http://junit.org/junit5/docs/current/user-guide/#running-tests-config-params)

# 莫奇托扩展

在撰写本文时，Mockito 还不支持 JUnit 5:[https://github.com/mockito/mockito/issues/445](https://github.com/mockito/mockito/issues/445)

因此，目前需要一些手动管道。

JUnit 5+使用了扩展模型，而不是我们习惯的旧的“测试运行器”(@ run with):【http://junit.org/junit5/docs/current/user-guide/#extensions

JUnit 背后的家伙为 Mockito 编写并发布了这样一个扩展:[*https://github . com/JUnit-team/JUnit 5-samples/blob/master/JUnit 5-mock ITO-extension/src/main/Java/com/example/mock ITO/mock ITO extension . Java*](https://github.com/junit-team/junit5-samples/blob/master/junit5-mockito-extension/src/main/java/com/example/mockito/MockitoExtension.java)

这是代码的一个基本的 Kotlin 转换；现在，您的代码库中需要它:

```
package who.cares.mockito

import org.junit.jupiter.api.extension.ExtensionContext
import org.junit.jupiter.api.extension.ParameterContext
import org.junit.jupiter.api.extension.ParameterResolver
import org.junit.jupiter.api.extension.TestInstancePostProcessor
import org.mockito.Mock
import org.mockito.Mockito.mock
import org.mockito.MockitoAnnotations
import java.lang.reflect.Parameter

// *TODO remove once Mockito officially supports Junit 5+* // See: https://github.com/mockito/mockito/issues/445

*/**
 * JUnit 5+ extension for Mockito
 * Reference: https://github.com/junit-team/junit5-samples/blob/master/junit5-mockito-extension/src/main/java/com/example/mockito/MockitoExtension.java
 */* class MockitoExtension : TestInstancePostProcessor, ParameterResolver {

    override fun postProcessTestInstance(testInstance: Any,
                                         context: ExtensionContext) {
        MockitoAnnotations.initMocks(testInstance)
    }

    override fun supportsParameter(parameterContext: ParameterContext,
                                   extensionContext: ExtensionContext): Boolean {
        return parameterContext.*parameter*.isAnnotationPresent(Mock::class.*java*)
    }

    override fun resolveParameter(parameterContext: ParameterContext,
                                  extensionContext: ExtensionContext): Any {
        return getMock(parameterContext.*parameter*, extensionContext)
    }

    private fun getMock(
            parameter: Parameter, extensionContext: ExtensionContext): Any {

        val mockType = parameter.*type* val mocks = extensionContext.getStore(ExtensionContext.Namespace.create(
                MockitoExtension::class.*java*, mockType))
        val mockName = getMockName(parameter)

        return if (mockName != null) {
            mocks.getOrComputeIfAbsent(
                    mockName, **{** _ **->** mock(mockType, mockName) **}**)
        } else {
            mocks.getOrComputeIfAbsent(
                    mockType.*canonicalName*, **{** _ **->** mock(mockType) **}**)
        }
    }

    private fun getMockName(parameter: Parameter): String? {
        val explicitMockName = parameter.getAnnotation(Mock::class.*java*)
                .name.*trim*()
        if (!explicitMockName.*isEmpty*()) {
            return explicitMockName
        } else if (parameter.*isNamePresent*) {
            return parameter.*name* }
        return null
    }
}
```

有了类路径上的上述扩展，现在可以将它添加到基于 Mockito 的测试中:

```
@ExtendWith(MockitoExtension::class)
```

# 调整现有测试

很抱歉，但我们不要在这里多此一举，[官方迁移指南](http://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4)应该为您提供了足够多的信息:)

就这样，好好享受吧！
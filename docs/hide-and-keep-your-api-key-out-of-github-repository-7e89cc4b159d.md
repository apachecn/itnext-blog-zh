# 将您的 API 密钥隐藏在 GitHub 存储库之外

> 原文：<https://itnext.io/hide-and-keep-your-api-key-out-of-github-repository-7e89cc4b159d?source=collection_archive---------0----------------------->

![](img/eaf596e393f82d029cb1d3e7da586ec2.png)

保护您的 API 密钥

当你在 GitHub 上上传你的 Android 应用程序时，你需要隐藏它，因为除了你没有人能访问它。这被认为是一个安全漏洞，所以这就是为什么隐藏你的 API 密匙是重要的。我将向您展示如何轻松做到这一点。

一些开发人员将他们的 API 密匙存储在这样的字符串变量中。

```
private static final String APK_KEY = "asjsdakf4d3ggs2ytm4x";
```

将你的秘密信息发布到公共库中是不好的，因为其他人可能会用完你有限的 API 调用。这可能是最不值得担心的情况。如果 API 密钥对某人访问数据子集进行身份验证，那么 API 密钥的共享就变得更加令人担忧。

那么，让我们来看看我们如何做到这一点。

1.  **将 API 密匙添加到您的** `**local.properties**` **文件**:

```
apiKey="Your Key"
```

2.**将这两行添加到您的 app 级** `**build.gradle**` **文件的根级:**

```
def localProperties = new Properties()
localProperties.load(new FileInputStream(rootProject.file("local.properties")))
```

3.**将下面这一行添加到您的应用程序级** `**build.gradle**` **文件:**

```
android {

  defaultConfig {
     // ...
     buildConfigField "String", "API_KEY",localProperties['apiKey']
  }

}
```

4.**同步 Gradle 并构建项目。您现在可以参考密钥:**

```
String apiKey = BuildConfig.API_KEY;
```

# 另一种方法是:

1.  **在中创建一个名为 gradle.properties 的文件。格拉德文件夹。**

```
- Drive C
 - Users folder
  - your user folder
   - .gradle folder
   **Create it here**
   - gradle.propertiesThen, write your APPNAME_API_KEY = "asjsdakf4d3ggs2ytm4x" inside it.
Save it as PROPERTIES File by enclosing with double quotes like this "gradle.properties"
```

2.下一步，**转到项目中的模块级 build.gradle 文件**

然后，将用于调试和发布目的的 API 键放在 buildTypes 树下。

```
buildConfigField ‘String’, “ApiKey”, APPNAME_API_KEY
```

会是那样的

```
buildTypes {
        debug{
            buildConfigField 'String', "ApiKey", APPNAME_API_KEY
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            buildConfigField 'String', "ApiKey", APPNAME_API_KEY
        }
    }
```

同步它

3.最后一步，**像这样访问你的 Java 文件中的 APK 密钥**

```
private static final String API_KEY = BuildConfig.ApiKey;
```

如果 ApiKey 变红，按下制作项目按钮或使用 **Ctrl+F9**

听起来很简单，是吗？。每当你在 GitHub 上上传你的 Android 项目时，使用你的库的人将无法知道你的 API 密匙是什么。因此，你现在是安全的。

更多文章:点击 [**此处**](https://marwa-eltayeb.medium.com/)

**上找我:**[GitHub](https://github.com/Marwa-Eltayeb)|[LinkedIn](https://www.linkedin.com/in/marwa-eltayeb/)|[Twitter](https://twitter.com/Marwa_Eltayeb1)
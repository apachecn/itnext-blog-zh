# 由 golang 管理的 I18n 消息

> 原文：<https://itnext.io/i18n-messages-manage-by-golang-5c6b60532a0?source=collection_archive---------6----------------------->

[Gookit/i18n](https://github.com/gookit/i18n) 是一个简单的 i18n 消息管理器，使用`INI`文件。

![](img/61d551070c8af2ead106cc3b7db724bd.png)

# 特征

*   易于使用，支持加载多种语言，多种文件
*   两种数据加载方式:单个文件`FileMode`，文件夹 `DirMode`；默认为文件夹模式
*   支持设置默认语言和后备语言；当找不到默认语言数据时，它会自动尝试查找备用语言
*   支持参数替换，有两种方式:`SprintfMode`通过 `fmt.Sprintf`替换参数，`ReplaceMode`使用 `strings.Replacer`

# 安装

```
go get github.com/gookit/i18n
```

# 使用

```
lang/
    en/
        default.ini
        ...
    zh-CN/
        default.ini
        ...
```

# 初始化 i18n

```
import "github/gookit/i18n" languages := map[string]string{
        "en": "English",
        "zh-CN": "简体中文",
        // "zh-TW": "繁体中文",
    } // The default instance initialized directly here
    i18n.Init("conf/lang", "en", languages)

    // Create a custom new instance
    // i18n.New(langDir string, defLang string, languages)
    // i18n.NewEmpty()
```

# 翻译

```
 // translate from special language
    msg := i18n.Tr("en", "key") // translate from default language
    msg = i18n.DefTr("key")
    // with arguments. 
    msg = i18n.DefTr("key1", "arg1", "arg2")
```

# 参数替换模式

**使用** `**SprintfMode**` **(默认):**

```
# en.ini
desc = I am %s, age is %d
```

带参数的用法:

```
msg := i18n.Tr("en", "desc", "name", "tom", "age", 22)
// Output: "I am tom, age is 22"
```

**使用** `**ReplaceMode**` **:**

```
# en.ini
desc = I am {name}, age is {age}
```

使用`map[string]interface{}`参数:

```
i18n.TransMode = i18n.ReplaceModemsg := i18n.Tr("en", "desc", "desc", map[string]interface{}{
    "name": "tom",
    "age": 22,
})
// Output: "I am tom, age is 22"
```

# 试验

```
go test -cover
```

# Dep 包

*   Google kit/ini 是一个 INI 配置文件/数据管理器

# 许可证

[麻省理工](https://github.com/gookit/i18n/blob/master/LICENSE)

[](https://github.com/gookit/i18n) [## gookit/i18n

### Use INI file, simple i18n manager implement. 使用 INI 文件，实现的简单方便的语言加载与管理 - gookit/i18n

github.com](https://github.com/gookit/i18n)
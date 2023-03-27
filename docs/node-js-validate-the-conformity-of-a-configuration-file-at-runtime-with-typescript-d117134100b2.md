# Node.js:在运行时验证配置文件与 TypeScript 的一致性

> 原文：<https://itnext.io/node-js-validate-the-conformity-of-a-configuration-file-at-runtime-with-typescript-d117134100b2?source=collection_archive---------6----------------------->

![](img/b8046e1d7271f99400b3902ea83d10d2.png)

感谢[好的免费照片](https://www.goodfreephotos.com/united-states/washington/northern-cascades-national-park/sahale-at-north-cascades-np-in-wa-landscape.jpg.php)。

在本教程中，我将展示一种在运行时验证任何现有项目中 JSON 配置文件一致性的方法。在 [TypeOnly](https://www.npmjs.com/package/typeonly) 的帮助下，我们将能够使用我们自己的配置类型的类型脚本定义。

# 让我们从一个 JSON 配置文件开始

假设我们有一个 Node.js 项目。在主目录下，TypeScript 源文件在`src/`目录下，生成的 JavaScript 文件在`dist/`目录下。我们也有一个类似这样的`config.json`配置文件:

```
{
  "tmpDir": "/tmp/data",
  "httpServer": {
    "port": 8212,
    "hostName": null
  },
  "log": {
    "level": "trace",
    "prettyPrint": true
  }
}
```

# 配置文件的 TypeScript 定义

TypeScript 可以帮助描述这种数据结构。我们需要在一个定义文件中对与配置相关的类型进行分组。在我们的`src/`目录下，创建一个文件`configuration-types.d.ts`:

```
export interface AppConf {
  tmpDir: string
  httpServer: HttpServerConf
  log: LogConf
}export interface HttpServerConf {
  port: number
  hostName: string | null
}export interface LogConf {
  level: "silent" | "error" | "warn" | "info" | "debug" | "trace"
  */**
   * Omit for stdout.
   */*
  file?: string
  prettyPrint: boolean
}
```

# 安装`typeonly`解析器及其验证器

现在，安装`typeonly`作为开发依赖项，安装`@typeonly/validator`作为普通依赖项:

```
*# Needed at compile time.*
npm install typeonly --save-dev*# Needed at run time.*
npm install @typeonly/validator
```

# 编写要在编译时执行的命令

编辑项目的`package.json`文件，以便在`scripts`部分添加命令:

```
 "typeonly": "typeonly --bundle dist/conf-types.to.json src/configuration-types.d.ts"
```

现在，运行新命令:

```
npm run typeonly
```

这个命令执行`typeonly`解析器:解析`src/configuration-types.d.ts`文件，然后提取类型并存储以供运行时使用。因此，创建了一个新的`dist/conf-types.to.json`文件。

# 在运行时使用我们的 TypeScript 类型验证数据

现在，`@typeonly/validator`包可以使用我们在`dist/conf-types.to.json`中的类型在运行时验证数据。在`src/`目录下，创建一个新的源文件`validate-configuration.ts`:

```
import { createValidator } from "@typeonly/validator"
import { join } from "path"
import { AppConf } from "./configuration-types"export function validateConfiguration(
  packageDir: string, 
  data: unknown
): asserts data is AppConf {
  *// Create the* validator *using a JSON file with types*
  const validator = createValidator({
    bundle: require(join(packageDir, "dist", "conf-types.to.json"))
  })
  *// Use the* validator *to validate 'data' using the interface*
  *// 'AppConf'*
  const result = validator.validate("AppConf", data)
  if (!result.valid)
    throw new Error(`Invalid config file: ${result.error}`)
}
```

在主源文件中，我们现在可以使用它:

```
import { readFileSync } from "fs"
import { dirname } from "path"
import { validateConfiguration } from "./validate-configuration";const packageDir = dirname(__dirname)
const conf = JSON.parse(readFileSync("config.json", "utf8"))
validateConfiguration(packageDir, conf)*// Here, 'conf' really is of type 'AppConf'*console.log(conf)
```

完成了。我们的配置文件在运行时得到验证。你可以检查一下:编辑`config.json`文件；设置日志级别为`"dummy"`，你会得到一个非常清晰的错误信息；

> 在属性“level”中，值“dummy”不符合联合:在“silent”|“error”|“warn”|“info”|“debug”|“trace”中没有匹配的类型。

# 关于 TypeOnly 的一句话

TypeOnly 使得在运行时使用 TypeScript 类型成为可能。

以下是关于 TypeOnly 的一些附加信息:

*   TypeOnly 解析器是使用 ANTLR 从头开始编写的。它根本不使用 TypeScript。它产生 JSON 数据。
*   在运行时，不需要解析器。所有的输入数据都存储在一个简单的 JSON 文件中。所以它真的很有表演性。
*   前端也可以使用类型作为运行时。事实上，生成的包含输入数据的 JSON 文件可以被打包:只需`require`它们！
*   TypeOnly 源文件必须是纯定义文件(扩展名为`.d.ts`)。它们可以是几个:通过在命令行中列出它们，或者通过使用`--source-dir`选项([参见文档](https://www.npmjs.com/package/typeonly#using-the-cli))。一个定义文件可以`import`别人。
*   一个很大的限制:泛型还没有实现。
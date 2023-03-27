# gradle & kot Lin——创建测试工件

> 原文：<https://itnext.io/gradle-kotlin-create-tests-artifact-1e22bb00b1e1?source=collection_archive---------3----------------------->

![](img/dc2e2fada389b4e3d57336677cd90b5d.png)

有时您需要创建一个“实用程序”jar——公共类、实用程序，它们在几个项目中使用。

有时，您也需要对测试类做同样的事情——测试基础设施和实用程序类。您可能希望将这些类放在“commons”项目的“tests”部分。

因此，您需要打包并发布您的库类和测试类，两者都有源代码。

我们将为 gradle 使用 Kotlin DSL:

然后，我们可以在其他项目中使用这个库:
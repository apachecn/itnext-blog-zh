# C++20 模块—完整指南

> 原文：<https://itnext.io/c-20-modules-complete-guide-ae741ddbae3d?source=collection_archive---------1----------------------->

编译器和构建系统慢慢开始支持 C++20 模块。这是您阅读本指南并从大规模编译加速中获益的绝佳时机。本文反映的是截至 2021 年 9 月的状态。

![](img/ba0063d910f7ca2622b553fd3e7c9695.png)

## 你好世界

首先，稍微提醒一下头文件在 C++中是如何工作的。当您编写`#include "header.h"`时，预处理器实际上将复制粘贴`header.h`的已处理内容来代替 include。这确实涉及到扩展任何递归包含，因此您可以很容易地从一个简单的包含中获得数兆字节的文本。例如，对于一个简单的 hello world，我们最终得到了几乎 2MB 的文本。

```
**> cat hello_world.cc**
#include <iostream>int main() {
    std::cout << "Hello World!\n";
}**> clang++ -std=c++20 -stdlib=libc++ -E hello_world.cc | wc -c**
1956614
```

这已经是个问题了，但是考虑到编译器会对项目中的每个实现文件都这样做。处理普通包括多次。各种供应商已经认识到这种浪费，并提供了使用预编译头文件的非标准解决方案，其中一组公共头文件被处理一次，然后被包括在任何地方。

模块的目标是解决同样的问题，但是以标准的方式。让我们再来看看 hello world，但现在是模块:

```
**> cat hello_modular_world.cc**
import <iostream>;int main() {
    std::cout << "Hello Modular World!\n";
}**> clang++ -std=c++20 -stdlib=libc++ -fmodules -fbuiltin-module-map \
          -E hello_modular_world.cc | wc -c**
239
```

这也是通过转换到模块节省时间的一个极端例子:

```
**> time clang++ -std=c++20 -stdlib=libc++ hello_world.cc**real 0m0.639s
user 0m0.584s
sys 0m0.058s**> time clang++ -std=c++20 -stdlib=libc++ -fmodules \
               -fbuiltin-module-map hello_modular_world.cc**real 0m0.087s
user 0m0.052s
sys 0m0.037s
```

而 Clang 附带了标准头文件到模块的映射。使用 GCC，我们需要手动编译`iostream`。

```
**> g++ -std=c++20 -fmodules-ts -xc++-system-header iostream
> g++ -std=c++20 -fmodules-ts hello_modular_world.cc**
```

## 编写模块

所以我们已经可以使用标准的头作为模块。现在让我们来看看如何编写自己的模块。和 C++一样，它的语言特性非常广泛，并且是非规定性的。

每个模块必须有一个导出该模块的文件。该文件被命名为*模块接口单元*。模块与名称空间完全正交，所以没有什么可以阻止您将符号导出到全局名称空间(就像这里的`plus`函数)或者作为名称空间的一部分导出(就像这里的`minus`函数)。

```
**> cat advanced_mathematics.cc**
export module AdvancedMathematics;export auto plus(auto x, auto y) -> decltype(x+y) {
    return x + y;
}export namespace AdvancedMathematics {
    auto minus(auto x, auto y) -> decltype(x-y) {
        return x - y;  
    }   
}void this_function_will_not_be_exported() {}**> cat main.cc**
import <iostream>;
import AdvancedMathematics;int main() {
    std::cout << "1+2 = " <<  plus(1,2) << "\n";
    std::cout << "3-2 = " << AdvancedMathematics::minus(3,2) 
              << "\n";
}
```

未导出的符号具有私有可见性，这与正常的 C++行为相反，在正常的 c++行为中，只有匿名命名空间中的符号是未导出的。

用 GCC 编译这个例子相当简单。GCC 在构建模块接口单元时会自动发出一个预编译的模块。Clangs 实现仍然缺乏，`clang++`接口没有公开用于发出预编译模块的标志，所以我们需要使用`-Xclang`将`-emit-module-interface`直接传递给 clang 编译器模块。对于更多的例子，我将坚持使用 GCC。

```
# Compiling with GCC
**> g++ -std=c++20 -fmodules-ts -xc++-system-header iostream
> g++ -std=c++20 -fmodules-ts -c advanced_mathematics.cc
> g++ -std=c++20 -fmodules-ts -c main.cc
> g++ -std=c++20 main.o advanced_mathematics.o -o main**# Compiling with Clang
**> FLAGS="-std=c++20 -stdlib=libc++ -fmodules -fbuiltin-module-map"
> clang++ $FLAGS -fmodules-ts -Xclang -emit-module-interface -c advanced_mathematics.cc -o AdvancedMathematics.pcm
> clang++ $FLAGS -c advanced_mathematics.cc
> clang++ $FLAGS -fprebuilt-module-path=. -c main.cc
> clang++ $FLAGS main.o advanced_mathematics.o -o main****> ./main**
1+2 = 3
3-2 = 1
```

如果需要，一个模块可以分布在多个文件中——多个*模块单元。*但是，这些文件中只有一个可以导出模块(并且是*模块接口单元*)。

```
**> cat spreadables.cc**
export module Spreadables;import <string>;export {
    std::string butter();
    std::string jam();
}**> cat butter.cc**
module Spreadables;import <string>;std::string butter() {
    return "Spreading some butter.";
}**> cat jam.cc**
module Spreadables;import <string>;std::string jam() {
    return "Spreading some jam.";
}**> cat bread.cc**
import <iostream>;import Spreadables;int main() {
    std::cout << "Taking some bread.\n";
    std::cout << butter() << "\n";
    std::cout << jam() << "\n";
}
```

以这种方式使用模块反映了如何使用头文件和实现文件，除了需要编译模块接口单元来生成预编译模块。

```
**> g++ -std=c++20 -fmodules-ts -xc++-system-header iostream
> g++ -std=c++20 -fmodules-ts -xc++-system-header string****> g++ -std=c++20 -fmodules-ts -c spreadables.cc
> g++ -std=c++20 -fmodules-ts -c jam.cc
> g++ -std=c++20 -fmodules-ts -c butter.cc
> g++ -std=c++20 -fmodules-ts -c bread.cc
> g++ -std=c++20 bread.o spreadables.o jam.o butter.o -o bread****> ./bread** Taking some bread.
Spreading some butter.
Spreading some jam.
```

## 模块分区

将一个模块分布到多个文件的另一种方法是模块分区(目前 Clang 不支持)。分区只能由主模块访问，必须是单个单元(单个文件)，但可以是接口单元(导出符号)或只是一个单元。但是，如果分区是一个模块接口单元，它必须在主模块`export import :partition;`中重新导出。主模块将可以访问所有分区符号，即使它们没有被标记为导出。

```
**> cat main_internal.cc**
// This file is module unit for a partition internal of module main
module main:internal;void internal_function() {} // will be visible in main module**> cat main_base.cc** // This file is module interface unit for a partition base
// of module main
export module main:base; // this must be re-exportedexport void visible() {} // will be visible to users of main
void internal_base() {} // will be visible to main module only**> cat main.cc**
// This file is module interface unit for module main
export module main;
// Import partition internal, cannot be re-exported
import :internal;
// Import partition base and re-export its exported symbols
export import :base;
```

## 编译时结构与模块

您可以在模块接口或模块分区内的任何地方使用编译时构造(templates、auto、decltype、constexpr、consteval)。

```
**> cat main_internal.cc** module main:internal;template <typename T>
T identity(T x) { return x; }**> cat main.cc** export module main;
import :internal;void function() {
    if (identity(3) != 3) abort();
}
```

然而，当使用多文件模块时，只有模块接口单元对预编译模块有贡献。因此，所有编译时构造都需要放在接口文件中。

## 处理遗留标头

如果您不能简单地导入您的遗留头文件(特别是在使用特性宏时)，您仍然可以在全局模块片段部分使用它们。

```
module; // start of global module fragment#define _POSIX_C_SOURCE 200809L
#include <stdlib.h>export module MyModule;// etc...
```

## 风格推荐

现在是这篇文章中更主观的部分。以下是我的两条建议，我认为作为一种风格来遵循是有意义的。

首先，关于名称空间。模块名可以包含没有任何语义意义的`.`符号。这允许我们构造命名来匹配嵌套的名称空间。因此，我的建议是按如下方式构造模块的命名和导出:

```
export module my.nested.named.module;export namespace my::nested::named::module {
    // exported content of the module
}
```

其次，关于多文件模块。多文件模块有两种支持方式。首先，简单地通过具有多个模块单元，其次通过具有模块分区。我的建议如下:

1.  如果可能的话，每个模块一个文件。
2.  如果多个文件有益于代码可读性，明智地利用模块分区(分离逻辑上相关的功能)。

## 链接和技术说明

所有的例子都使用 GCC 和 Clang(2021 年 9 月)的主干版本进行了演示。

所有代码示例和脚本都可以在 https://github.com/HappyCerberus/article-cpp20-modules 的[获得。](https://github.com/HappyCerberus/article-cpp20-modules)

# 感谢您的阅读

感谢您阅读这篇文章。你喜欢吗？

我还在 YouTube[youtube.com/c/simontoth](https://youtube.com/c/simontoth)上发布视频，如果你想聊天，可以在 Twitter [@SimonToth83](https://twitter.com/SimonToth83) 或 LinkedIn[linkedin.com/in/simontoth](https://www.linkedin.com/in/simontoth)上联系我。
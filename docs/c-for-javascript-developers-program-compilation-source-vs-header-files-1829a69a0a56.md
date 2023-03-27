# 面向 JavaScript 开发人员的 c。程序编译。源文件和头文件

> 原文：<https://itnext.io/c-for-javascript-developers-program-compilation-source-vs-header-files-1829a69a0a56?source=collection_archive---------2----------------------->

![](img/a93b4d35352b87d69a83b26777caa2ac.png)

让我们看一个 hello-world 应用程序。

`main.c`:

```
#include <stdio.h>int main(void) {
  puts("Hello World!");
  return 0;
}
```

这是一个普通的 c 语言工作程序。如果你来自一个更高级的语言背景(比如说 JavaScript)，这段代码似乎很有意义。我们定义一个函数，这个函数又调用另一个函数并返回值。

但是 C 和 JavaScript 是非常不同的语言。前者是编译的，相对低级，它的原语很好地映射到计算机在硬件层面上的工作方式。后者处理更高级的结构，需要一个复杂的引擎来运行代码，并抽象出大量机器级的细节(如何管理内存，如何存储某些数据结构，等等)。).

接下来的一系列文章意在解释 C 语言的一些领域，如果你只有高级解释语言的经验，这些领域可能并不明显。

下面是第一个问题:让这段代码运行需要什么(它是如何编译的)？而`puts`函数到底从何而来？

注意:我在 Ubuntu Linux 上运行下面所有的命令。在 Mac 上，您应该能够不加修改地运行这些命令。在 Windows 上，查看你的编译器的文档，或者试试 [Cygwin](https://www.cygwin.com/) 。这里提出的想法是独立于操作系统的。

## 编译过程

为了理解文章开头的例子，我们来看看 C 程序是如何编译的(以及编译成什么样子)。

首先，解释语言和编译语言的主要区别是什么？前者依赖于语言引擎——一个独立的软件，它将 JavaScript(或 Python，或其他什么)代码作为输入并运行它。后者使用编译器生成机器指令，处理器稍后直接执行这些指令，无需额外软件层的帮助。所以在这两种情况下，都有一个将人类可读代码翻译成机器代码的过程，区别在于它是发生在编译时还是运行时。

为了编译 C 代码，你需要什么？初学者可能认为用 C 做任何事情都需要一个大而复杂的 IDE，比如 MS Visual Studio。事实上，你唯一需要的就是一个编译器。诚然，现代编译器也往往很大很复杂，但它们可以在命令行上运行，它们唯一需要的是一个输入文件，可以在任何文本编辑器中编写。流行的编译器有 Mac 和 Linux 版的 gcc 和 clang，或者 Windows 版的 Visual C++(没错，命令行上也有)。

因此，如果我们从本文开始就有了`main.c`源文件，编译它就像调用如下代码一样简单:

```
> gcc main.c -o out
```

`-o out`代表“将输出写入文件`out`”(将成为可执行文件)。当您运行可执行文件时，您会看到预期的输出:

```
> ./out
Hello World!
```

实际上,“编译”这个词用得太多了。它可以是指将一个源文件(或几个源文件)转换成可执行文件的整个过程，也可以只是这个过程的一部分，即为每个源文件创建一个对应的[汇编](https://en.wikipedia.org/wiki/Assembly_language)文件(这也称为“适当编译”)。整个过程是这样的:

1.  预处理。编译开始前，执行预处理指令(`#include`、`#define`、`#if`等)。在这一步的最后，我们得到了没有预处理指令的文件。这些文件(编译器的输入)被称为 [*翻译单元*](https://en.wikipedia.org/wiki/Translation_unit_(programming)) 。
2.  编译(正确)和汇编。每个翻译单元被单独转换成一个目标文件，其中包含机器代码和链接器指令。这些文件还不能执行。
3.  链接。所有的目标文件最终被组合起来产生一个可执行文件。

## 依赖性解析

`puts`功能从何而来？如果你认为它是通过包含`stdio.h`以某种方式引入的，你只对了一部分。

让我们去掉`#include`看看会发生什么。

`main.c`:

```
int main(void) {
  puts("Hello World!");
  return 0;
}> gcc main.c -o out
main.c: In function ‘main’:
main.c:2:3: warning: implicit declaration of function ‘puts’ [-Wimplicit-function-declaration]
   puts(“Hello World!”);
   ^
```

我们得到一个警告，但没有错误。让我们试着运行`out`:

```
> ./out
Hello World!
```

成功！这是怎么回事？我们没有在`main.c`中导入或包含任何包含函数名`puts`的东西，所以它怎么知道要调用什么函数呢？

为了理解它是如何工作的，让我们来看看`main.c`被编译成的汇编代码。使用`gcc`，您可以传递一个`-S`选项，使其在预处理和适当编译后停止。

```
> gcc -S main.c -o main.s -masm=intel -fno-asynchronous-unwind-tables
main.c: In function ‘main’:
main.c:2:3: warning: implicit declaration of function ‘puts’ [-Wimplicit-function-declaration]
   puts(“Hello World!”);
   ^
```

我们在这里得到了同样的警告(这是意料之中的)，但是让我们来看看结果汇编文件。

`main.s`:

```
 .file "main.c"
  .intel_syntax noprefix
  .section .rodata
.LC0:
  .string "Hello World!"
  .text
  .globl main
  .type main, @function
main:
  push rbp
  mov rbp, rsp
  mov edi, OFFSET FLAT:.LC0
  call puts
  mov eax, 0
  pop rbp
  ret
  .size main, .-main
  .ident “GCC: (Ubuntu 5.4.0–6ubuntu1~16.04.10) 5.4.0 20160609”
  .section .note.GNU-stack,””,@progbits
```

确切的输出取决于编译器、[处理器架构](https://en.wikipedia.org/wiki/Instruction_set_architecture)和您正在编译的选项(在上面的例子中省略`-masm=intel -fno-asynchronous-unwind-tables`会稍微改变输出，但是相关的位保持相似)。我们应该寻找某种`call`或`jump`命令。

`call puts` —即将[指令指针](https://en.wikipedia.org/wiki/Program_counter)设置为标签`puts`的值。或者，更简单地说，“让处理器执行的下一个命令是由`puts`标记的命令”。之前的命令(`mov edi, OFFSET FLAT:.LC0`)将`"Hello World!"`字符串的地址放在`edi`寄存器中 x64 Linux 中的任何函数都会在那里寻找它。这两个命令是我们从`puts`函数调用中得到的全部内容。

公平地说，C 标准的现代版本(从 C99 开始)要求，如果要调用一个函数，必须事先声明它(因此出现了警告)。如果我们想解决这个问题，我们实际上不需要包含`stdio.h`。我们需要做的就是为`puts`添加一个声明。

`main.c`:

```
**int puts (const char *);**int main(void) {
  puts("Hello World!");
  return 0;
}
```

如果我们尝试编译上面的代码，就会很顺利。

```
> gcc main.c -o out
> ./out
Hello World!
```

现在我们知道了函数的签名(编译器通过不发出警告来确认)，但是仍然没有指示在哪里可以找到函数体。

事实上，编译器和汇编器并不关心函数的位置。当编译器看到函数调用时，它所做的就是添加汇编指令，用于将参数放置在它们各自的位置(由相关的[应用二进制接口](https://en.wikipedia.org/wiki/Application_binary_interface)指定)并用于调用函数。

能够调用一个函数而不声明它是以前类型不太严格的 c 语言版本的残余。gcc 允许出于兼容性原因编译这样的代码——它只是通过传递给它的参数来推断函数参数的数量和类型，并假设它的类型是`int`。我们没有使用`puts`的返回值(即使我们使用了，它的实际类型恰好是`int`，就像编译器的默认一样)，我们传递给它一个字符串，实际函数期望得到这个字符串，这就是为什么即使没有声明，一切都可以工作。

那么在什么时候我们最终解决了函数调用和它的定义之间的联系呢？定义在哪里？

这是链接器负责的事情之一。正如我们前面讨论的，链接器处理不可执行的目标文件——它接受一个或多个目标文件作为输入，并生成一个可执行文件。在这个过程中，它确保如果某个地方调用了某个东西，那么在其他地方就存在相应的标签。如果它确实存在，标签被转换成一个具体的内存地址。否则—链接器会产生错误。

让我们看看*如何调用*链接器(下面的`-v`标志用于详细输出)。

```
> gcc -v main.c -o out
... (output omitted)
/usr/lib/gcc/x86_64-linux-gnu/5/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/5/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper -plugin-opt=-fresolution=/tmp/ccz0r2qe.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s — sysroot=/ — build-id — eh-frame-hdr -m elf_x86_64 — hash-style=gnu — as-needed -dynamic-linker /lib64/ld-linux-x86–64.so.2 -z relro -o out /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/5/crtbegin.o -L/usr/lib/gcc/x86_64-linux-gnu/5 -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/5/../../.. /tmp/ccAjWoyV.o -lgcc — as-needed -lgcc_s — no-as-needed -lc -lgcc — as-needed -lgcc_s — no-as-needed /usr/lib/gcc/x86_64-linux-gnu/5/crtend.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crtn.o
```

这里的最后一个命令——我们感兴趣的一个——是完整呈现的。它是自动生成的，并不打算特别可读，但我们仍然可以看到发生了什么。用几个选项调用`collect2`实用程序(它是 gcc 链接器)，一堆目标文件(`*.o`)被传递给它(包括`/tmp/ccAjWoyV.o`，它是从`main.c`生成的临时目标文件)。我们感兴趣的选项是`-lc`，它告诉链接器与[共享库](https://en.wikipedia.org/wiki/Library_(computing)#Shared_libraries) `libc.so`链接，共享库可以在预定义的系统位置找到。这实际上是您系统中的一个文件，其中存放了包括`puts`在内的 [C 标准库](https://en.wikipedia.org/wiki/C_standard_library)函数的实现。

如果你想知道这个函数在 C 中是如何实现的，你可以在网上[找到](https://chromium.googlesource.com/native_client/nacl-newlib/+/master/newlib/libc/stdio/puts.c) [例子](https://opensource.apple.com/source/Libc/Libc-825.26/stdio/FreeBSD/puts.c.auto.html)。但是源代码不一定要安装在您的机器上——编译的库版本就足够了。

## 头文件

让我们回到最初的例子。

`main.c`:

```
#include <stdio.h>int main(void) {
  puts("Hello World!");
  return 0;
}
```

我们已经展示了上面的`#include`指令对于编译来说是不必要的，可以用一个声明来代替。但是为什么我们通常包含头文件，当我们这样做时会发生什么呢？头文件和源文件在本质上有什么不同？

让我们在同一个目录中创建另一个文件。

`misc.c`:

```
#include <stdio.h>void say_hi(void) {
  puts("Hi!");
}void say_thanks(void) {
  puts("Thanks!");
}void say_bye(void) {
  puts("Bye!");
}
```

现在我们想在`main.c`中使用源文件中的函数:

```
int main(void) {
  say_hi();
  say_thanks();
  say_bye();
  return 0;
}
```

我们试着编译一下。多个源文件可以被编译成一个可执行文件，成功编译的条件之一是，在所有这些源文件中，有且只有一个名为`main`的函数，它是应用程序的入口点(我将在后面的文章中详细阐述`main`的语义及其在不同环境中的用法)。

```
> gcc main.c misc.c -o out
main.c: In function ‘main’:
main.c:2:3: warning: implicit declaration of function ‘say_hi’ [-Wimplicit-function-declaration]
   say_hi();
   ^
main.c:3:3: warning: implicit declaration of function ‘say_thanks’ [-Wimplicit-function-declaration]
   say_thanks();
   ^
main.c:4:3: warning: implicit declaration of function ‘say_bye’ [-Wimplicit-function-declaration]
   say_bye();
   ^
> ./out
Hi!
Thanks!
Bye!
```

我们得到了预期的警告，但是程序确实按计划编译并运行了。我们可以像前面的例子一样，通过在使用之前声明`main.c`中的函数来消除警告。

`main.c`:

```
**void say_hi(void);
void say_thanks(void);
void say_bye(void);**int main(void) {
  say_hi();
  say_thanks();
  say_bye();
  return 0;
}
```

如果我们尝试运行相同的编译命令，将不会有警告。

事实上，在定义之前，通常的做法是在`misc.c`的顶部声明所有的`misc.c`函数。

`misc.c`:

```
#include <stdio.h>**void say_hi(void);
void say_thanks(void);
void say_bye(void);**void say_hi(void) {
  puts("Hi!");
}void say_thanks(void) {
  puts("Thanks!");
}void say_bye(void) {
  puts("Bye!");
}
```

头文件是避免这种重复的一种方法。

`misc.h`:

```
void say_hi(void);
void say_thanks(void);
void say_bye(void);
```

`misc.c`:

```
#include <stdio.h>**#include "misc.h"**void say_hi(void) {
  puts("Hi!");
}void say_thanks(void) {
  puts("Thanks!");
}void say_bye(void) {
  puts("Bye!");
}
```

`main.c`:

```
**#include "misc.h"**int main(void) {
  say_hi();
  say_thanks();
  say_bye();
  return 0;
}
```

我们只需将重复的代码提取到一个专用文件中，并用一个`#include`指令替换它。

然后在预处理过程中发生相反的情况——该指令被替换为头文件的全部内容。我们可以这样试一下(`-E`选项是为了在预处理后停止编译，`*.i`文件扩展名是 gcc 约定的不需要预处理的源代码)。

```
> gcc -E main.c -o main.i
```

`main.i`:

```
# 1 "main.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 1 "<command-line>" 2
# 1 "main.c"
# 1 "misc.h" 1
void say_hi(void);
void say_thanks(void);
void say_bye(void);
# 2 "main.c" 2int main(void) {
  say_hi();
  say_thanks();
  say_bye();
  return 0;
}
```

你可以忽略[行标](https://gcc.gnu.org/onlinedocs/cpp/Preprocessor-Output.html)(以`#`开头的行)——剩下的就是原来`main.c`的内容加上`misc.h`的全部内容。

就是这样。头文件并不代表模块系统——预处理程序只是进行文本替换。头文件可以包含任何东西，不仅仅是函数声明(只要它在编译时有意义)。使它成为头文件的不是它的扩展名或它的内容。事实上，我们没有将它作为源文件列表的一部分传递给编译器，而是通过`#include`指令将其内容粘贴到其他源文件中。它最终会被编译，而且会被多次编译——每次一次`#include`。

顺便说一下，头文件也可能有`#include`指令，所以预处理程序必须递归地执行替换。两个不同的头文件可能独立地包含`stdio.h`，并且它们都包含在一个源文件中。我们最终得到的是一个包含了两次的头文件，因为没有什么可以阻止我们这样做。这并不一定会破坏任何东西，因为允许双重声明，但是它会降低编译速度。一种常见的解决方法是手动检查是否包含了某个标头。

`misc.h`:

```
**#ifndef _MISC_H_
#define _MISC_H_**void say_hi(void);
void say_thanks(void);
void say_bye(void);**#endif**
```

上面的代码定义了一个名为`_MISC_H_`的预处理宏，但是如果预处理程序再次遇到相同的块，它会删除它，因为`_MISC_H_`已经被定义了。通过这种方式，我们可以确保预处理后的任何给定文件中只有一个上述头文件的副本，不管它被直接或间接包含了多少次。

另外，`#include`指令中的引号(`""`)对尖括号(`<>`)符号指定了将在哪些目录中搜索该文件。该标准没有规定这一点，但所有主要编译器的行为方式都是一样的:尖括号版本只是在一组预定义的系统目录中进行搜索，对于引号，搜索从源文件所在的目录开始，如果没有找到，就退回到系统目录。简单点说，尖括号是系统头文件，引号是自定义头文件。

现在，您可以在本文开头的例子中使用`-E`选项运行 gcc 输出应该有几百行长，但是应该有`puts`声明。

## 结论

c 不是一种可以取代 JavaScript 或 Python 的语言——用它写高级软件就没那么愉快了。但学习它仍然很有意义，因为它让你更好地理解计算机是如何工作的。理解程序是如何编译和运行的，对于编写比 hello-world 应用程序更严肃的东西是必不可少的。

# 全系列

1.  (本文)程序编译。源文件和头文件
2.  [内存分配。指针、数组和字符串](https://medium.com/@pavelpomerantsev/c-for-javascript-developers-memory-allocation-pointers-arrays-and-strings-bcf387ad7f44)
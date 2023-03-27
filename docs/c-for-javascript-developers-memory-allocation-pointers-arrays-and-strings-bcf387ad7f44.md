# 面向 JavaScript 开发人员的 c。内存分配。指针、数组和字符串

> 原文：<https://itnext.io/c-for-javascript-developers-memory-allocation-pointers-arrays-and-strings-bcf387ad7f44?source=collection_archive---------2----------------------->

![](img/5b8bef91c471b435df4a1e9949ad260f.png)

如果你有一些 C 语言的经验，那么你可能知道一个字符串只是一个由`char`组成的数组，或者它可以用一个指向`char`的指针来表示——因为指针和数组几乎是一回事，对吗？这在某种程度上是正确的，但现实稍微复杂一些。

让我们看几个例子(假设源文件总是叫 main.c)。注意:我在 Ubuntu Linux 上使用 gcc 编译器，在不同的环境中可能会得到不同的结果。关于设置的更详细的解释，请参考我之前关于 C 的文章。

```
#include <stdio.h>int main() {
  char str[3] = "cat";
  puts(str);
  return 0;
}> gcc main.c -o out
> ./out
cat�
```

这里刚刚发生了什么？单词末尾的那个字符是哪里来的？

再比如。

```
int main(void) {
  char *str = "cat";
  str[0] = 'b';
  return 0;
}> gcc main.c -o out
> ./out
Segmentation fault (core dumped)
```

对前一个例子的修改。

```
int main(void) {
  char str[] = "cat";
  str[0] = 'b';
  return 0;
}> gcc main.c -o out
> ./out
```

我们简单地用数组替换了一个指针，它修复了分段错误。但是为什么呢？

第三个例子。

```
#include <string.h>int main() {
  char *str1 = "cat";
  char *str2;
  strcpy(str2, str1);
  return 0;
}> gcc main.c -o out
> ./out
Segmentation fault (core dumped)
```

对于一个有经验的开发人员来说，不难看出我在上面的代码片段中犯了什么错误。但是坦率地说:如果您从更高级的语言结构(打印一个字符串、修改一个字符串、复制一个字符串)的角度来考虑这些代码，它们都是有意义的，但是输出与您所期望的相差甚远。

几个相关的问题:

*   为什么我们不能在函数内部声明一个未知大小的数组而不初始化它，就像这样:`int arr[];`？
*   为什么*可以*我们用未知大小的数组作为函数参数？
*   为什么数组不能作为函数返回值？
*   如果我们比较两个字符串会发生什么:`"cat" == "cat"`？这是真的还是假的？

我们使用的三个概念(字符串、数组和指针)是 C 语言中最基本的东西，但是如果你试图用它们做任何事情而不了解它们在内存中是如何表示的，你很可能会得到我们刚刚看到的错误(或类似的错误)。当然，你可以学习一些该做的和不该做的事情来避免某些类型的错误，但是如果你学习基本原则，你会有更好的理解。

# 自动变量和堆栈

如果任何类型的变量在函数中声明，并且没有关键字`extern`或`static`，那么它就是一个*自动*变量。这意味着当执行进入声明该变量的块时，该变量的存储被自动*分配*，当该块的执行结束时，它同样被自动*释放。*

变量作用域(源代码中变量可见的部分)和存储持续时间(内存中相应区域保持其值的时间)是相关但不同的概念，将在以后的文章中探讨。现在，让我们只关注自动存储持续时间。

在 JavaScript 中，我们不处理内存，所以如果我们声明一个变量，我们永远不用担心它是如何存储的或者存储在哪里。在 C 语言中并非如此——理解变量存储是如何组织的非常有用(也不太难)。

自动分配通常使用一个*堆栈*来实现——一个保存局部变量和返回地址的特殊内存区域，也可能包含函数参数和返回值。

这里有一个详细的例子。这个程序不做任何有用的事情——我们只是通过它来理解堆栈上的内存分配是如何工作的。

```
void one(void);void two(void);int main(void) {
  one();
  return 0;
}void one(void) {
  long local1 = 1;
  two();
}void two(void) {
  long local2 = 2;
}
```

当程序在我的 Linux 机器上编译和链接时，它看起来是这样的:

```
00000000004004d6 <main>:
  4004d6: push rbp
  4004d7: mov rbp,rsp
  4004da: call 4004e6 <one>
  4004df: mov eax,0x0
  4004e4: pop rbp
  4004e5: ret00000000004004e6 <one>:
  4004e6: push rbp
  4004e7: mov rbp,rsp
  4004ea: sub rsp,0x10
  4004ee: mov QWORD PTR [rbp-0x8],0x1
  4004f6: call 4004fe <two>
  4004fb: nop
  4004fc: leave
  4004fd: ret00000000004004fe <two>:
  4004fe: push rbp
  4004ff: mov rbp,rsp
  400502: mov QWORD PTR [rbp-0x8],0x2
  40050a: nop
  40050b: pop rbp
  40050c: ret
  40050d: nop DWORD PTR [rax]
```

以上是 objdump 的删节输出:

```
> gcc main.c -o out
> objdump -d -M intel — no-show-raw-insn out
```

你可以在手册页上阅读更多关于 objdump 及其选项的内容。

左边的地址是指令在执行时驻留的实际内存地址。你不必理解所有的命令，我们只看几行。

这里的处理器( [x86_64 架构](https://en.wikipedia.org/wiki/X86-64))有许多寄存器，它们只是特殊的 8 或 4 字节存储单元，具有比主存储器快得多的访问时间，并使用助记符(`rbp`、`rsp`、`eax`等)进行寻址。)而不是数字地址。

`rbp`和`rsp`是两个有特殊含义的 8 字节寄存器(分别称为基址指针和堆栈指针)。在程序执行过程中，这些寄存器的值以这样一种方式改变，即它们总是定义当前函数的[堆栈帧](https://en.wikipedia.org/wiki/Call_stack#Structure)。

另外，`long`被选为源代码中的变量类型，因为它在 64 位机器上的大小是 8 字节，就像内存地址和寄存器`rbp`和`rsp`一样。我们可以在这里使用任何其他整数类型(如`int`、`short`或`char`)，但是使用`long`堆栈被很好地分成 8 字节的段，每个段保存一个值。

`4004e7: mov rbp,rsp`:将`rsp`的值复制到`rbp`。

下面是执行上面一行代码后堆栈的样子(堆栈没有完整显示，只显示了相关部分):

```
**rbp == 0x7fffffffdc70** rsp == 0x7fffffffdc70Address        | Value          |
0x7fffffffdc78 | 0x400fdf       | <- “main” function return address
0x7fffffffdc70 | 0x7fffffffdc80 | <- rbp and rsp both point here; the value stored at this memory location is the previous value of rbp (0x7fffffffdc80)
```

我们刚刚使`rbp`和`rsp`相等。`rsp`总是指向栈顶，也就是它最小的地址(目前是 0x7fffffffdc70)，因为[x86 _ 64 上的栈向下增长](https://eli.thegreenplace.net/2011/02/04/where-the-top-of-the-stack-is-on-x86/)。

所有例子中的运行时堆栈地址都是使用 GNU 调试器 [gdb](https://www.gnu.org/software/gdb/documentation/) 获得的。对其用法的解释超出了本文的范围。

`4004ea: sub rsp,0x10`:在更高级别的伪代码中，它读为`rsp = rsp — 16`——为了在堆栈上为局部变量分配空间，`rsp`减少了 16。`rbp`保持不变。

```
rbp == 0x7fffffffdc70
**rsp == 0x7fffffffdc60**Address        | Value          |
0x7fffffffdc78 | 0x400fdf       |
0x7fffffffdc70 | 0x7fffffffdc80 | <- rbp still points here
**0x7fffffffdc68 | cruft          |
0x7fffffffdc60 | cruft          | <- rsp points here**
```

“Cruft”意味着单元格可以保存任何值，但我们对它不感兴趣，因为我们不打算在写之前读取它。

`4004ee: mov QWORD PTR [rbp-0x8],0x1` — `rbp — 8`在函数`one`的上下文中是变量`local1`的位置。这一行是实际的`long local1 = 1`初始化发生的地方。我们可以稍后改变变量的值——将会发生的是不同的值将被写入相同的内存位置，并且它将总是被寻址为`rbp — 8`,因为当执行在`one`内部时，`rbp`的值不会改变。

```
rbp == 0x7fffffffdc70
rsp == 0x7fffffffdc60Address        | Value          |
0x7fffffffdc78 | 0x400fdf       |
0x7fffffffdc70 | 0x7fffffffdc80 | <- rbp still points here
**0x7fffffffdc68 | 1              | <- local1 value** 0x7fffffffdc60 | cruft          | <- rsp points here
```

编译器在堆栈上分配了比我们的变量所需更多的空间，但这是由于[堆栈对齐](https://wr.informatik.uni-hamburg.de/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf)的要求，我们不必担心这个问题。

`4004f6: call 4004fe <two>` —我们正在进入功能`two`。这里发生两件事:执行跳转到`0x4004fe`(`two`第一条指令的地址)，返回地址被压入堆栈。

```
rbp == 0x7fffffffdc70
**rsp == 0x7fffffffdc58**Address        | Value          |
0x7fffffffdc78 | 0x400fdf       |
0x7fffffffdc70 | 0x7fffffffdc80 | <- rbp still points here
0x7fffffffdc68 | 1              | <- local1 value
0x7fffffffdc60 | cruft          |
**0x7fffffffdc58 | 0x400fb        | <- "one" function return address; rsp points here**
```

`4004fe: push rbp` —将`rbp`的当前值推送到堆栈上

```
rbp == 0x7fffffffdc70
**rsp == 0x7fffffffdc50**Address        | Value          |
0x7fffffffdc78 | 0x400fdf       |
0x7fffffffdc70 | 0x7fffffffdc80 | <- rbp still points here
0x7fffffffdc68 | 1              | <- local1 value
0x7fffffffdc60 | cruft          |
0x7fffffffdc58 | 0x400fb        | <- "one" function return address
**0x7fffffffdc50 | 0x7fffffffdc70 | <- rsp points here**
```

`4004ff: mov rbp,rsp` —将`rsp`的值复制到`rbp`。同样的事情发生在我们进入`one`的时候。事实上，任何函数开始执行时都会执行该操作。

```
**rbp == 0x7fffffffdc50** rsp == 0x7fffffffdc50Address        | Value          |
0x7fffffffdc78 | 0x400fdf       |
0x7fffffffdc70 | 0x7fffffffdc80 |
0x7fffffffdc68 | 1              | <- local1 value
0x7fffffffdc60 | cruft          |
0x7fffffffdc58 | 0x400fb        | <- "one" function return address
**0x7fffffffdc50 | 0x7fffffffdc70 | <- both rsp and rbp point here**
```

此时，我们不能再将`local1`作为`rbp — 8`来访问，但是没关系，因为变量超出了范围。记住源代码— `local1`对于`one`来说是本地的，所以当我们在`two`内部时我们不能访问它。

我们刚刚看到的是*自动*为 *local1* 分配内存。在 C 代码中，我们只需声明它，由编译器为它留出内存(`sub rsp, 0x10`)。

当`one`的执行结束时，`local1`的内存将被*以类似的方式自动*释放:递增堆栈指针是`leave`指令要做的事情之一。“解除分配”并不意味着该内存单元的值会立即改变，但从现在起它被视为 cruft，可以随时更新。

# 两颗北极指极星

配备了自动变量的知识，再来说指针。

指针是一个固定大小为八个字节的变量(在 64 位机器上)。我们经常使用指针来访问向量数据(数组)，但本质上它们是标量值。我们可以将一个整型常量转换为指针类型——这是一个合理的操作。

```
void dummy_func(void) {
  int *p = (int *) 123;
}
```

事实上，如果我们将指针类型替换为“long ”,编译后的代码可能会完全相同:

```
void dummy_func(void) {
  long p = (long) 123;
}
```

在第二个例子中，cast 操作符是不必要的，我只是想指出这里的`int *`可以直接用`long`替换，因为这两种类型都占用了 8 个字节的内存。这 8 个字节由*自动*分配，类似于上一节中的`long`变量。

一般来说，我们不会像那样使用指针，而是直接给它们赋值，因为如果我们试图用`*`操作符[去引用](https://stackoverflow.com/questions/4955198/what-does-dereferencing-a-pointer-mean)这样的指针，我们很可能会导致[分段错误](https://en.wikipedia.org/wiki/Segmentation_fault)错误(因为地址 123 可能在[受保护的内存区域](https://en.wikipedia.org/wiki/Memory_protection))。

有几种方法可以让指针变得有用。例如，我们可以间接引用另一个变量。

```
#include <stdio.h>void dummy_func(void) {
  long i = 123;
  long *p = &i; // Initialize p with the address of i
  printf("%ld\n", *p); // "123" will be printed
}
```

创建了两个*自动*变量，第二个变量(`p`)保存第一个变量的地址，因此堆栈可能如下所示:

```
Address        | Value          |
0x7fffffffdc70 | 0x7fffffffdc68 | <- the "p" variable, holding the address of "i"
0x7fffffffdc68 | 123            | <- the "i" variable, holding the value 123
```

由编译器决定自动变量在堆栈中的顺序——`i`在源代码中排在第一位并不重要。

如前所述，指针也可以用作数组。

```
#include <stdio.h>
#include <stdlib.h>void dummy_func(void) {
  long *p = malloc(10 * sizeof(long)); // Allocate memory for a 10-element array
  p[0] = 123;
  printf("%ld\n", p[0]); // "123" will be printed
  free(p);
}
```

同样，当声明`long *p`时，堆栈上唯一自动分配的内存*是指针本身的内存。10 元素数组的内存必须通过调用`malloc`显式分配，然后通过调用`free`释放。这两个函数由标准库提供，它们在被称为[堆](https://medium.com/fhinkel/confused-about-stack-and-heap-2cf3e6adb771)的内存区域上操作。操作系统通常提供从堆中请求内存和回收内存的低级原语，这些原语分别由`malloc`和`free`使用。*

*使用`malloc`在堆上分配也被称为*动态*分配，与*自动*(如上所述)和*静态*(将在字符串部分讨论)相对。*

*注意，我们使用数组语法(`p[0]`)和指针类型变量，而不是解引用操作符(`*`)。我们将在下一节探讨为什么这是可能的。*

*我们也可以使用指针来访问字符串:*

```
*#include <stdio.h>void dummy_func(void) {
  int *p = "cat";
  printf("%c\n", p[0]); // "c" will be printed
}*
```

# *数组*

*按照标准的说法，C 中的数组是“一组连续分配的非空对象，具有特定的成员对象类型”。*

*又分配到哪里？如果任何变量，包括一个数组，在一个没有关键字`static`或`extern`的函数中被声明，它会被自动分配*。**

**所以，如果我们有一个函数:**

```
**void dummy_func(void) {
  long arr[3] = {1, 2, 3};
  // Use the array
}**
```

**那么在执行过程中，堆栈将如下所示:**

```
**Address        | Value          |
0x7fffffffdc70 | 3              | <- arr[2]
0x7fffffffdc68 | 2              | <- arr[1]
0x7fffffffdc60 | 1              | <- arr[0]**
```

**还记得我们之前说过，下标`arr[1]`相当于指针解引用(`*(arr + 1)`)？原因是，在大多数情况下，当编译器看到一个数组变量时，它会假设它正在处理一个指针变量。指针没有专用的存储，但是编译器知道它的值(它总是指向数组的第一个元素)。**

**事实上，数组下标(使用方括号语法获取数组元素的操作)根据定义*等同于将元素索引添加到数组变量中，然后取消对它的引用(类似于`*(arr + 1)`)。***

**为了说明这一点，让我们给这个例子添加一个指针:**

```
**void dummy_func(void) {
  long arr[] = {1, 2, 3}; // We don’t need to specify the size of the array since we’re initializing it with three elements
  long *ptr = arr; // We can do it since arr is implicitly substituted with a pointer that holds the address of array’s first element. We’re assigning a pointer value to a pointer variable here, so no error or warning will be generated.
}**
```

**在这种情况下，`arr[1]`、`*(arr + 1)`、`ptr[1]`和`*(ptr + 1)`都产生相同的结果。**

**但是`arr[1]`和`ptr[1]`等价吗？答案是**否**。为这两个操作生成的汇编代码是不同的。在第一种情况下，编译器知道数组第一个元素的地址(从技术上讲，不是绝对地址，而是它相对于基指针的偏移量)，所以在运行时我们只需获取所需的元素。**

**指针就不是这样了——我们必须在运行时获取它包含的值*(恰好是第一个数组元素的地址)，只有这样我们才能获得必要的数组元素。因此，使用指针变量比使用数组变量需要多一条指令来获得一个元素。***

***下面是示例的堆栈:***

```
***Address        | Value          |
0x7fffffffdc70 | 3              | <- arr[2] == *(arr + 2) == ptr[2] == *(ptr + 2) == 3
0x7fffffffdc68 | 2              | <- arr[1] == *(arr + 1) == ptr[1] == *(ptr + 1) == 2
0x7fffffffdc60 | 1              | <- arr[0] == *arr == ptr[0] == *ptr == 1
0x7fffffffdc58 | 0x7fffffffdc60 | <- ptr, holds the address of the first array element***
```

***在编译时，数组不仅产生与指针不同的汇编代码，而且还有语法差异。您可以从数组创建指针，但不能从数组创建指针。也不能赋值给数组(初始化不算)。***

```
***#include <stdlib.h>void dummy_func(void) {
  long *ptr = malloc(3 * sizeof(long));
  long arr1[] = ptr; // Wrong, the initializer should contain a number of long values in braces, but not a pointer; will cause a compile-time error long arr2[3];
  arr2 = {1, 2, 3}; // Wrong, an array cannot be the left side of an assignment operation (except at initialization); will also cause a compile-time error
}***
```

***你不能用一个指针的值来初始化一个数组，因为每当一个数组被声明的时候，就必须为一定数量的元素分配存储空间，就像我们之前讨论的那样。所以`long arr1[] = ptr;`没有意义，因为不清楚分配多少内存。而且，就算写成`long arr1[3] = ptr;`，还是会模棱两可。我们应该将`ptr`转换为`long`并存储在第一个数组元素中，还是应该将`ptr`指向的一个连续内存块复制到由`arr1`表示的内存块中？***

***不能给数组赋值的原因是数组变量本身不占用任何内存(尽管数组的元素占用)。C #中的赋值操作符总是将一个值存储在内存位置中，赋值操作的左侧将计算该值。但是`arr2`没有相关联的存储位置。所以，如果左边有一个数组，要么我们必须改变赋值操作符的语义，要么不允许数组赋值。C 的创造者选择了后者，不难看出他们的推理。***

***还有几件事需要注意:使用数组作为函数参数，从函数中返回数组。***

***下面是一个函数声明:`void dummy_func(int a[]);`。该函数接受一个参数，这是一个数组`int`。我们如何通过它？在关于堆栈的部分，我们讨论了如何分配局部变量的存储。事实上，参数以类似的方式传递。C 语言中的所有参数都是通过值传递的(不像 JavaScript 中的对象是通过引用传递的)，所以每个参数在堆栈上都有一个具体的表示(在某些编译器和操作系统中，寄存器可以用于参数传递，但是为了讨论方便，我们假设总是使用堆栈)。因此，如果函数看起来像`void dummy_func(long l);`，那么在函数执行期间，从基址指针固定偏移的堆栈上的 8 个字节将被参数值占用。***

***未知大小的数组(`int a[]`)是不同的，因为不出所料，它的大小在编译时是未知的。为了解决这个问题，C 在编译时将数组参数转换为指针参数，因此`void dummy_func(int a[]);`被完全视为`void dummy_func(int *a);`。由于在编译时必须知道参数的大小，所以传递指针是很自然的事情——在 64 位机器上，指针的大小固定为 8 个字节。***

***C #中不允许从函数中返回数组，这也不是一个奇怪的异常。相反，很可能是不愿意做出怪异的例外。返回值被放在堆栈上，就像参数一样，它们必须有固定的大小，所以我们不能只传递数组，我们必须把数组转换成指针。但是如果我们在函数中创建数组，然后试图返回它，我们将返回一个指向自动数组的指针，当创建数组的函数返回时，自动数组的存储时间结束。***

***长话短说，从函数中返回数组在 C 中会很笨拙，所以语言不允许这样做。***

****更新:*事实上，让函数拥有数组返回类型并不是不现实的，但是数组的大小总是必须固定的(类似于`int dummy(void)[3]`)。但是这只会使事情变得复杂——返回值的行为将不同于参数的行为(将整个数组放在堆栈上，而不是传递一个指针)。***

# **用线串**

**C 中有指针类型和数组类型，但是字符串和 JavaScript 不同，它不是一个独立的数据类型。根据该标准，字符串“是一个连续的字符序列，以第一个空字符结束，并包含第一个空字符。”**

**换句话说，字符串是简单的`char`数组，它的最后一个元素是它的第一个`'\0'`字符。“第一个”这个词在这里很重要——字符串中间不能有`'\0'`。(较新版本的 C 也支持多字节字符串，但是为了便于讨论，我们可以假设每个字符都用一个`char`来表示)。**

**所以如果我们说的是代表单词“cat”的字符串，那就是四个字符:`'c'`、`'a'`、`'t'`、`'\0'`。它们位于内存中的连续地址，最后一个字符就是数字 0。**

**它看起来像这样:**

```
**Address        | Value          |
0x7fffffffdc73 | '\0' (0)       |
0x7fffffffdc72 | 't'  (116)      |
0x7fffffffdc71 | 'a'  (97)       |
0x7fffffffdc70 | 'c'  (99)       |**
```

**看看我们之前是如何以 8 个字节的步长显示地址的，而这里的步长是一个字节。与字符相关的数字来自 [ASCII 表](https://en.wikipedia.org/wiki/ASCII)。**

***字符串文字*与*字符串*相关，但这是一个不同的概念。字符串是源文件(不在内存中)中用双引号括起来的字符序列，就像`"cat"`。**

**字符串文字不一定代表字符串。例如，`"cat\0dog"`是一个有效的字符串文字，但是它在内存中创建的不是一个字符串，因为字符串中间不能包含空字符。**

**另外，`char str[] = {'c', 'a', 't', '\0'};`创建一个字符串，但不使用字符串文字。**

**当编译器遇到长度为 N 的字符串文字时，它会确保在程序运行时，在程序开始执行之前，分配(N+1)字节的静态存储，并用必要的字符填充。*静态*存储，与*自动*和*动态*相反，在执行开始时分配，在程序终止后释放。需要(N+1)个字节来包含文字的所有字符以及结尾的空字符。**

**然后编译器将文字转换为指向其第一个字符的指针，因此`char *ptr = "cat";`意味着将在内存中的某个地方创建一个字符序列，并且`"cat"`将被隐式转换为“指向字符的指针”类型的值，然后该值将用于初始化变量`ptr`。`ptr`将包含存储字符`'c'`的存储单元的地址。**

**“将字符串文字转换为指针”行为有一个例外，即数组初始化。函数内部的数组是自动变量，所以如果我们使用 char `arr[] = "cat";`，那么在栈上为数组*分配四个字节，并用文字的字符填充(加上空字符)。`"cat"`在这个实例中没有被转换成指针。这个初始化的结果将和我们写`char str[] = {'c', 'a', 't', '\0'};`一样。***

**我们还需要理解为什么需要一个空字符来结束一个字符串。不是很浪费吗？其实不是。字符串只是内存中的字节序列，没有元信息存储在任何地方。每当我们希望在控制台中显示一个字符串时，我们需要知道要显示多少个字符。这就是空字符的用途—它是一种在没有显式存储长度的情况下分隔字符串的方法。**

# **问题的答案**

**我们现在已经学到了足够多的知识，能够理解本文开头的例子中发生了什么。**

****问题 1:** 为什么我们在输出中得到一个奇怪的字符？**

```
**#include <stdio.h>int main() {
  char str[3] = "cat";
  puts(str);
  return 0;
}> gcc main.c -o out
> ./out
cat�**
```

**答:我们使用了一个长度不足的数组来存储字符串(应该是四个，而不是三个)。打印字符串时，遍历内存位置，直到遇到一个所有位都设置为零的字节。在这种情况下，打印的字符数取决于编译器和执行环境。**

**你不应该做上面的事情，简单地使用一个未知大小的数组(`char str[] = "cat";`):它将被初始化为正确的长度。**

****问题 2:** 如果我们用一个字符串初始化一个数组和一个指针，然后尝试重新分配结果数组的一个元素，为什么行为会不同？`char *str = "cat"; str[0] = 'b';`和`char str[] = "cat"; str[0] = 'b';`。**

**答:请记住，当代码中有一个字符串文字时，编译器会为它所代表的字符数组分配静态存储空间，在编译期间，该文字会被替换为指向该内存的指针。所以当一个指针变量被初始化时(`char *str = "cat";`)，值*可能是*一个只读内存地址(由编译器决定将字符串放在哪里)。当我们试图写入那个地址(`str[0] = 'b';`)时，进程会立即终止并出现错误。但是当你初始化一个数组时，`"cat"`的存储被分配在堆栈上，在程序执行期间是可写的，所以它工作得很好。**

****问题 3:** 为什么字符串复制会失败？**

```
**#include <string.h>int main() {
  char *str1 = "cat";
  char *str2;
  strcpy(str2, str1);
  return 0;
}**
```

**答:`strcpy`函数将第二个参数指向的字符串复制到第一个参数指向的数组中。`str2`未用值初始化，因此包含 cruft。`strcpy`仍然愉快地把那个 cruft 当作一个地址，并试图写入它，但是那个地址很可能不在可写的内存区域中。**

****问题 4:** 为什么不初始化就不能在函数内部声明一个大小未知的数组，像这样:`int arr[];`？**

**答:因为自动变量*在声明时必须有一个已知的大小。编译代码时，变量没有符号名，所有值都是通过从堆栈帧的开始或结束的偏移量来存储和检索的。如果我们不知道一个变量的大小，我们就不知道它后面的变量的偏移量。***

***更新:* C99 在语言中引入了*可变长度数组*，它们的大小在编译时实际上是未知的，它既不是堆栈指针，也不是用于计算数组元素地址的基指针。然而，可变长度仅仅意味着长度是在运行时定义的，而不是一旦定义就可能改变。在未知大小数组的情况下，如果允许使用`int arr[];`,该语言将不得不允许数组赋值(实际上并不允许),并且不允许进一步的赋值(以防止数组大小的改变)或者使数组可调整大小，这在堆栈上是相当低效的。**

****问题 5:** 为什么*可以*使用大小未知的数组作为函数参数？**

**答:作为函数参数的数组只是简单地转换为指针，所以这样一个参数的大小总是指针变量的大小(在 64 位处理器架构的情况下是 8 个字节)。**

****问题 6:** 为什么数组不能作为函数返回值？**

**答:参见“数组”部分。**

****问题 7:** 如果我们比较两个字符串会怎么样:`"cat" == "cat"`？这是真的还是假的？**

**回答:我们首先需要理解我们在这里比较什么。一个字符串被一个指针代替，所以两个指针被比较。这与字符串的词典比较无关(就像 JavaScript 中的等号运算符所做的那样)。那么指针有相同的值吗？答案是，可能。编译器可以选择为代码中的每个字符串单独分配内存，或者为相同的字符串重用内存。这种行为是不确定的，所以你不应该依赖它的实现方式。基本上，永远不要编写比较运算符将字符串文字作为其操作数之一的代码，因为您无法控制它将被分配到的地址。如果你需要按字典顺序比较两个字符串，使用`strcmp`函数。**

# **结论**

**希望现在能更清楚为什么数组和指针有这样的功能，以及字符串到底是什么。语言创造者的某些决定也可能开始变得有意义了——你以前可能认为是怪癖的特性实际上是基于合理的原则。**

# **资源**

*   **C 标准:[http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)**
*   **AMD64 Linux ABI:[https://software . Intel . com/sites/default/files/article/402129/mpx-Linux 64-ABI . pdf](https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf)**
*   **C 语言的发展丹尼斯·里奇:[https://www.bell-labs.com/usr/dmr/www/chist.html](https://www.bell-labs.com/usr/dmr/www/chist.html)**
*   **C 常见问题解答(特别是它的“数组和指针”部分):【http://c-faq.com/aryptr/index.html **
*   **伊莱·本德斯基的优秀文章:[https://Eli . the green place . net/2009/10/21/are-pointers-and-arrays-equivalent-in-c](https://eli.thegreenplace.net/2009/10/21/are-pointers-and-arrays-equivalent-in-c)和[https://Eli . the green place . net/2011/09/06/stack-frame-layout-on-x86-64/](https://eli.thegreenplace.net/2011/09/06/stack-frame-layout-on-x86-64/)**

# **全系列**

1.  **[程序编译。源文件与头文件](https://medium.com/@pavelpomerantsev/c-for-javascript-developers-program-compilation-source-vs-header-files-1829a69a0a56)**
2.  **(本文)内存分配。指针、数组和字符串**
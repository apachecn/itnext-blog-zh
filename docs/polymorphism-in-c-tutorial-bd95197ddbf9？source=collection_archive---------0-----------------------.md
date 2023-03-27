# 教程:C 语言中的多态性

> 原文：<https://itnext.io/polymorphism-in-c-tutorial-bd95197ddbf9?source=collection_archive---------0----------------------->

![](img/8f3795a9c3765b69e4298f21faa06daa.png)

# 介绍

我在我的观点文章中引用了一些子类型多态性的例子，' [C 对 C++:战斗！本教程将这些例子发展成一个完整的程序，展示了这些技术为什么有用以及如何实现它们。](https://medium.com/p/201b3f07a94f)

多态这个词来自古希腊语，简单的意思是“许多形状”。这个词的字典定义通常是“以几种不同的形式存在”。

维基百科为程序员提供了一个更实用的定义:

> 多态性是向不同类型的实体提供单一接口

希望本教程对学习 C 编程的人有用，或者对那些不确定这样一个理论概念如何应用于 C 之类的低级语言的人有用。

# 问题陈述

假设我们要编写一个命令行工具，将未知长度的文件中每个字节的值相加。结果可以用作校验和来检测文件中的错误，尽管所提出的算法不能检测字节的重新排序。

这里有一个简单的 C 程序实现了这个想法:

```
#include <stdio.h>

unsigned char sum_bytes(FILE *f)
{
    unsigned char sum = 0;

    for (int c = fgetc(f); c != EOF; c = fgetc(f)) {
        sum += c;
    }

    return sum;
}

int main(int argc, char const *argv[])
{
    return sum_bytes(stdin);
}
```

现在，我们需要测试我们的程序，以确保它不包含编码错误。为此，我们可以创建一个包含已知字节序列的文件，通过管道将输入输入到程序中，然后检查程序的返回值是否是我们所期望的。

例如，这样的测试可以实现为以下 Bash 脚本:

```
echo -en '\x1\x2\x3\x4' | checksum

if [ $? -eq 10 ]; then
    echo "Test passed"
else
    echo "Test failed"
fi
```

这个解决方案并不理想，因为测试脚本的功能有限，并且不能移植到其他平台。相反，我们可能希望单独测试`sum_bytes()`函数，以帮助定位任何错误的原因。

我们可以编写一个测试程序，在一个文件上调用`sum_bytes()`而不是标准的输入流，但是它需要首先创建那个文件:

```
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char const *argv[])
{
    FILE *f = fopen("testdata", "w+b");

    if (!f) {
        fputs("Can't create file\n", stderr);
        return EXIT_FAILURE;
    }

    static unsigned char const testdata[] = {1, 2, 3, 4};
    int err = EXIT_SUCCESS;

    if (fwrite(testdata, sizeof(testdata), 1, f) != 1) {
        fputs("Can't write to file\n", stderr);
        err = EXIT_FAILURE;
    } else {
        rewind(f);

        if (sum_bytes(f) == 10) {
            puts("Test passed");
        } else {
            fputs("Test failed\n", stderr);
            err = EXIT_FAILURE;
        }
    }

    fclose(f);
    return err;
}
```

上述解决方案要求测试对存储临时文件的文件系统具有写访问权。我们可以通过将测试与读取测试向量的`fgetc()`的模拟定义联系起来，或者通过使用宏将`fgetc()`重新定义为不同名称的模拟函数的别名来解决这个问题。

两者都是单元测试的有效方法，但是都属于链接时间多态性和通用编程的范畴。本教程关注的是运行时多态，它允许一个`sum_bytes()`实例从一个静态链接的程序中读取不同类型的数据源。

`sum_bytes()`函数的基本问题是它只对来自`<stdio.h>`的类型`FILE *`进行操作。相反，我们可能希望为存储在只读存储器中的数据、从网络中读取的数据或者由诸如 ZLib 之类的解压缩库输出的数据计算校验和。C 标准库的流是一个有用的抽象，但是太有限了，不能支持所有的用例。

# 天真的解决方案

允许`sum_bytes()`函数从不同数据源读取数据的一个简单解决方案是使用条件逻辑。例如，我们可以传递一个`FILE`的地址和一个数组的地址，并使用一个布尔参数来决定使用哪个数据源。

下面是一个经过适当修改的测试程序:

```
#include <stdbool.h>
#include <stdlib.h>
#include <stdio.h>

unsigned char sum_bytes(bool read_file, FILE *f,
                        int const *buffer)
{
    unsigned char sum = 0;
    size_t i = 0;

    for (int c = read_file ? fgetc(f) : buffer[i++];
         c != EOF;
         c = read_file ? fgetc(f) : buffer[i++])
    {
        sum += c;
    }

    return sum;
}

int main(int argc, char const *argv[])
{
    static int const testdata[] = {1, 2, 3, 4, EOF};

    if (sum_bytes(false, NULL, testdata) == 10) {
      puts("Test passed");
      return EXIT_SUCCESS;
    } else {
      fputs("Test failed\n", stderr);
      return EXIT_FAILURE;
    }
}
```

因为调用者必须传递三个参数而不是一个，所以`sum_bytes()`函数变得难以使用并且容易出错。很容易错过`true`而不是`false`，或者反过来。这种解决方案的可扩展性也很差，因为`sum_bytes()`的复杂性随着支持的输入数据类型的数量而线性增加。

# 多态性的拯救

一个更具可伸缩性的解决方案是让`sum_bytes()`的调用者传递要调用的函数的地址来代替`fgetc()`，并且在调用该函数时传递一个`void *`指针:

```
unsigned char sum_bytes(int (*getc_fn)(void *), void *getc_arg)
{
    unsigned char sum = 0;

    for (int c = getc_fn(getc_arg);
         c != EOF;
         c = getc_fn(getc_arg))
    {
        sum += c;
    }

    return sum;
}
```

我刚刚描述了不同类型实体的单个接口，因此这个解决方案可以称为多态。

这个接口仍然有些笨拙，因为`sum_bytes()`的调用者必须传递两个参数，而不是一个。它的类型安全性也不如前面的接口:调用者必须注意传递一个与指定的`getc_fn`兼容的`getc_arg`值。编译器无法强制执行，因为任何(非限定的)指针类型都会隐式转换为`void *`。

最后，所需的类型`getc_fn`与标准的`fgetc()`函数不兼容，该函数需要类型`FILE *`而不是`void *`的参数，因此`sum_bytes()`的调用者必须使用适配器函数:

```
static int stream_reader_getc(void *cb_arg)
{
    return fgetc(cb_arg); // implicit conversion to FILE *
}

int main(int argc, char const *argv[])
{
    return sum_bytes(stream_reader_getc, stdin);
}
```

我们真正想要的是`sum_bytes()`呈现一个简单且类型安全的接口，消除出错的可能性。

让我们首先将回调函数地址和相关的数据指针封装到一个`struct`中，以避免它们与其他变量分离或混淆:

```
struct reader {
  int (*getc_fn)(void *);
  void *getc_arg;
};
```

上面的定义创建了一个独特的类型，`struct reader`，它不能与任何其他类型互换使用(甚至不能是碰巧有相同成员的另一个`struct`)。在面向对象编程中，`struct reader`被称为“抽象基类”,它的`getc_fn`指针被称为“虚方法”。

先前版本的`sum_bytes()`可以接受任何具有兼容签名(参数和返回类型)的*回调函数的地址，而新版本只接受一个`struct reader`对象的地址:*

```
unsigned char sum_bytes(struct reader *r)
{
    unsigned char sum = 0;

    for (int c = r->getc_fn(r->getc_arg);
         c != EOF;
         c = r->getc_fn(r->getc_arg))
    {
        sum += c;
    }

    return sum;
}

int main(int argc, char const *argv[])
{
    struct reader r = {stream_reader_getc, stdin};
    return sum_bytes(&r);
}
```

为了避免在实例化类型为`struct reader`的对象时出错，让我们创建一些函数来为不同类型的数据源初始化一个对象。在面向对象编程中，这些被称为“子类构造函数”:

```
void stream_reader_init(struct reader *r, FILE *stream)
{
    *r = (struct reader){stream_reader_getc, stream};
}

void buf_reader_init(struct reader *r, int *buffer)
{
    *r = (struct reader){buf_reader_getc, buffer};
}
```

这看起来已经安全多了，但是我还没有提供读取测试向量的`buf_reader_getc()`函数的定义。编写这个函数是有问题的:

```
static int buf_reader_getc(void *cb_arg)
{
    static size_t bytes_read;
    int const *const buffer = cb_arg; // implicit conv. to int *
    return buffer[bytes_read++];
}
```

虽然这对于一次性测试来说是可以接受的，但是对于真正的程序来说却是不可接受的！`bytes_read`计数器的`static`存储类别意味着使用该功能只能读取一个缓冲区，并且该缓冲区只能读取一次。

我们真正需要的是面向对象编程中的“实例变量”:每个由`buf_reader_init`初始化的`struct reader`实例都必须有自己的`bytes_read`副本，从 0 开始计数。这只能通过为从缓冲区而不是从文件读取的`struct reader`实例分配额外的内存来实现。

现在可能是考虑子类型的好时机。

# 亚型多态性

C 不支持子类型，因为子类型可以和它们的父类型互换使用，因为类型系统没有层次结构。

然而，类似于子类型的东西可以通过将表示超类型的`struct`嵌套在表示其子类型的`struct`中来实现。这是应用于`struct reader`的方法，它被一个名为`struct buf_reader`的新子类型所扩展:

```
struct buf_reader {
  struct reader super;
  int const *buffer;
  size_t bytes_read;
};

static int buf_reader_getc(void *cb_arg)
{
    struct buf_reader *br = cb_arg;
    return br->buffer[br->bytes_read++];
}

void buf_reader_init(struct buf_reader *br, int const *buffer)
{
    *br = (struct buf_reader){
      .super = {buf_reader_getc, br},
      .buffer = buffer,
      .bytes_read = 0
    };
}
```

测试程序现在声明了一个新子类型的对象，它包含了实例变量`buffer`和`bytes_read`的额外空间。和以前一样，它调用`buf_reader_init()`来初始化对象:

```
int main(int argc, char const *argv[])
{
    struct buf_reader br;
    static int const testdata[] = {1, 2, 3, 4, EOF};
    buf_reader_init(&br, testdata);

    if (sum_bytes(&br.super) == 10) {
      puts("Test passed");
      return EXIT_SUCCESS;
    } else {
      fputs("Test failed\n", stderr);
      return EXIT_FAILURE;
    }
}
```

测试向量现在可以被限定为`const`(也许存储在只读存储器中)，因为这个数组的地址不再被转换为`buf_reader_init`构造函数中的非限定`void *`指针。

注意，在调用`sum_bytes()`时，测试必须显式传递嵌入式`struct reader`的地址，因为从编译器的角度来看`struct reader`和`struct buf_reader`是截然不同的。在面向对象编程中，这被称为“向上转换”，通常是隐式的。

# 丰富

这个设计仍然有一些问题:

*   要求测试提供一个由宏值`EOF`终止的`int`类型的数组，而不是一个无符号字节的数组，这是很笨拙的。
*   在`struct reader`中存储`struct buf_reader`的地址似乎是多余的；两个对象有相同的地址。在任何情况下，都可以使用`offsetof()`从另一个对象计算出另一个对象的地址。

第一点可以通过添加一个新的实例变量`buf_size`来解决，以存储`buf_reader_getc()`返回的最大字节数:

```
struct buf_reader {
  struct reader super;
  unsigned char const *buffer;
  size_t bytes_read, buf_size;
};

static int buf_reader_getc(void *cb_arg)
{
    struct buf_reader *br = cb_arg;

    if (br->bytes_read >= br->buf_size) {
      return EOF;
    }

    return br->buffer[br->bytes_read++];
}

void buf_reader_init(struct buf_reader *br,
                     unsigned char const *buffer,
                     size_t buf_size)
{
    *br = (struct buf_reader){
      .super = {buf_reader_getc, br},
      .buffer = buffer,
      .bytes_read = 0,
      .buf_size = buf_size,
    };
}

int main(int argc, char const *argv[])
{
    struct buf_reader br;
    static unsigned char const testdata[] = {1, 2, 3, 4};
    buf_reader_init(&br, testdata, sizeof(testdata));

    if (sum_bytes(&br.super) == 10) {
      puts("Test passed");
      return EXIT_SUCCESS;
    } else {
      fputs("Test failed\n", stderr);
      return EXIT_FAILURE;
    }
}
```

寻址第二个点需要一个接口变化:不是在调用虚拟方法获取一个字节时传递一个`void *`指针，`sum_bytes()`可以传递`struct reader`实例的地址。这个指向当前对象的指针在面向对象的编程中被称为“自我”或“这个”。

以下是修改后的`struct reader`和`sum_bytes()`定义:

```
struct reader {
  int (*getc_fn)(struct reader *);
};

unsigned char sum_bytes(struct reader *r)
{
    unsigned char sum = 0;

    for (int c = r->getc_fn(r);
         c != EOF;
         c = r->getc_fn(r))
    {
        sum += c;
    }

    return sum;
}
```

删除`struct reader`的`cb_arg`成员意味着`fgetc()`从中读取数据的`FILE *`必须存储在一个实例变量中。这需要定义第二个子类型，我已经命名为`stream_reader`:

```
#include <stddef.h>

#define container_of(addr, type, member) \
  ((type *)(((char *)(addr)) - offsetof(type, member)))

struct stream_reader {
  struct reader super;
  FILE *stream;
};

static int stream_reader_getc(struct reader *r)
{
    struct stream_reader *sr =
      container_of(r, struct stream_reader, super);

    return fgetc(sr->stream);
}

void stream_reader_init(struct stream_reader *sr, FILE *stream)
{
    *sr = (struct stream_reader){
      .super = {stream_reader_getc},
      .stream = stream
    };
}
```

更新与第一个子类型(`buf_reader_init()`和`buf_reader_getc()`)相关的函数以符合这个新的接口是留给读者的练习。

# 遗传还是作文？

我使用了一个事实上的标准宏`container_of()`，从`struct reader`成员的地址计算出`struct stream_reader`的地址。这在面向对象编程中被称为“向下转换”。它本质上是不安全的，因为编译器不能确定给定的对象实际上是指定子类型的实例。

我之前写过“两个对象有相同的地址”，所以你可能会奇怪为什么我使用了`container_of()`，而不是简单地转换成所需的子类型`struct stream_reader *`。

在某种程度上，这种选择归结于你是否认同继承的概念。如果你相信`stream_reader` *是*一个`reader`(继承)，那么一个简单的造型就很有意义；如果你反而相信`stream_reader` *有*有`reader`(构图)，那么`container_of()`更有意义。

使用`container_of()`确实有一些实际优势:

*   编译器检查`struct stream_reader`是否有一个名为`super`的成员；`container_of()`更聪明的定义也迫使编译器检查`super`的类型是否为`struct reader`。
*   如果`super`成员意外地从其在`struct stream_reader`定义开始时的位置移开，那么程序仍然可能工作。
*   您以后可能会决定让类型为`struct stream_reader`的对象实现多个接口(“多重继承”)。

然而，`container_of()`也有一个很大的缺点，如果你关心编写严格符合标准的代码:

> 当具有整数类型的表达式被添加到指针或从指针中减去时…如果指针操作数和结果都指向同一个数组对象的元素，或者都超过数组对象的最后一个元素，则求值不应产生溢出；否则，行为是未定义的。

(6.5.6“加法运算符”，ISO/IEC 9899:1999“编程语言— C”)

宏定义中的`char *`强制转换允许将指定成员重新解释为数组，但它不是数组对象，减去`offsetof(type, member)`的结果可能指向它之外。实际上，给定的定义适用于所有已知的编译器和目标体系结构。

大多数程序员只是忽略了`container_of()`可能有未定义的行为这一事实。你要不要跟着他们，你自己看着办。相反，您可以继续在超类型的每个实例中存储一个`void *`指针，或者(如果您不需要多重继承)确保`super`总是子类型`struct`的第一个成员。在后一种情况下，`container_of()`定义了等同于简单造型的行为。

# 封装向何处去？

在大多数情况下，我会对我上面提出的解决方案感到满意。它简单、高效、类型安全(除了我在虚方法定义中使用的`container_of()`)。

当处理较大的项目时，或者当设计需要保持二进制兼容性的接口时，您可能希望隐藏一些实现细节。在 C 程序中，这样做的方法是使用不完整的`struct`类型。这些可以用在任何不需要一个`struct`的大小并且它的成员不能被直接访问的情况下。

以下是一些需要考虑补救的封装失败:

*   `sum_bytes()`函数要求`struct reader`是一个完整的类型，以便调用它用来获取数据的虚拟方法。这将公开属于超类型的任何实例变量。
*   `checksum`程序要求`struct stream_reader`是一个完整的类型，以便声明该类型的对象，并允许它获得嵌入的`struct reader`的地址。这将暴露属于父类型和子类型的任何实例变量。
*   测试要求`struct buf_reader`是一个完整的类型，以便声明该类型的对象，并允许它获得嵌入的`struct reader`的地址。

第一点可以通过创建一个头文件来解决，该头文件声明一个不完整的`struct reader`类型和包装函数来调用它提供的任何虚拟方法(在本例中，只有一个)。

实现子类型所需的`struct reader`的完整定义必须放在一个单独的私有头文件中。如果私有头文件只包含在需要完整类型的源文件中，那么`struct`成员将被隐藏。

## reader.h —公共接口

```
struct reader;
int reader_getc(struct reader *);
```

## reader_struct.h —私有接口

```
struct reader {
  int (*getc_fn)(struct reader *);
};
```

## reader.c —私有实现

```
#include "reader.h"
#include "reader_struct.h"

int reader_getc(struct reader *r)
{
  return r->getc_fn(r);
}
```

第二点可以通过为`struct stream_reader`编写一个可选的构造函数来解决，该构造函数为新的对象实例动态分配内存，并编写一个函数来获取嵌入在`struct stream_reader`实例中的`struct reader`的地址。

同样，实现子类型或分配其实例所需的`struct stream_reader`的完整定义必须放在私有头文件中。

## stream_reader.h —公共接口

```
#include <stdio.h>
#include "reader.h"

struct stream_reader;
void stream_reader_init(struct stream_reader *sr, FILE *stream);
struct stream_reader *stream_reader_new(FILE *stream);
struct reader *stream_reader_get_reader(struct stream_reader *);
```

## stream_reader_struct.h —私有接口

```
#include <stdio.h>
#include "reader_struct.h"

struct stream_reader {
  struct reader super;
  FILE *stream;
};
```

## stream_reader.c —私有实现

```
#include "stream_reader.h"
#include "stream_reader_struct.h"

static int stream_reader_getc(struct reader *r)
{
    struct stream_reader *sr =
      container_of(r, struct stream_reader, super);

    return fgetc(sr->stream);
}

void stream_reader_init(struct stream_reader *sr, FILE *stream)
{
    *sr = (struct stream_reader){
      .super = {stream_reader_getc},
      .stream = stream
    };
}

struct stream_reader *stream_reader_new(FILE *stream)
{
  struct stream_reader *sr = malloc(sizeof(*sr));

  if (sr) {
    stream_reader_init(sr, stream);
  }

  return sr;
}

struct reader *stream_reader_get_reader(struct stream_reader *sr)
{
  return &sr->super;
}
```

为另一个子类型定义单独的公共和私有接口留给读者做练习。

# 引人深思的事

*   如果子类型的实例被分配在堆上，并且不提供额外的(即非虚拟的)方法，那么这些类型需要成为公共接口的一部分吗？
*   如果子类型不是公共接口的一部分，那么程序如何销毁子类型的实例呢？
*   你能按照提供一个稳定的接口对每一个头文件的重要性来排列推荐的头文件吗？
*   如果一个子类型的构造函数没有初始化一个函数指针，那么调用相应的方法就会有不确定的行为。对于“getc”来说，什么是有用的默认行为，你将如何实现它？

# 天色已晚

我希望你觉得这个教程有用；我写得很开心。如果没有，为什么不发表评论，让我知道你还想阅读哪些 C 编程主题？

感谢阅读。
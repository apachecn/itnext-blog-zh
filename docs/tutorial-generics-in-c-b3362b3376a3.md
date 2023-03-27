# 教程:C 语言中的泛型

> 原文：<https://itnext.io/tutorial-generics-in-c-b3362b3376a3?source=collection_archive---------0----------------------->

![](img/91f6aec0c9bf07cf79341b4b111629ec.png)

# 介绍

在我看来，' [C 对 C++:战斗！](https://medium.com/p/201b3f07a94f)’，我提到过使用 C 预处理器进行泛型编程是可能的。本教程提供了一些例子来说明这些技术为什么有用以及如何实现它们。

generic 这个词来自拉丁语词根‘genus’，简单的意思是‘种类’或‘排序’。这个词的字典定义通常是“与一组事物相关”或“一类事物的特征”。

维基百科将泛型编程定义为:

> 计算机程序设计的一种风格，其中算法是根据以后要指定的类型编写的，然后在需要时对作为参数提供的特定类型进行实例化。

例如，实现[抽象数据类型](https://en.wikipedia.org/wiki/Abstract_data_type)如堆栈的通用化代码可以针对整数类型、浮点类型或用户定义的类型进行不同的实例化。这种编程风格也被称为参数多态性。

希望本教程对学习 C 编程的人有用，或者对那些不确定这样一个理论概念如何应用于 C 之类的低级语言的人有用。

# 参数化什么？

在编程理论中，多态性意味着不同类型实体的单一接口。参数化仅仅意味着某些东西是用参数来定义的。

不同种类的多态性适用于不同类型的问题。重要的是要考虑哪种多态性适合您的用例(以及您是否需要多态性)。

参数多态性可以与特定多态性和[子类型多态性](/polymorphism-in-c-tutorial-bd95197ddbf9)进行对比:

*   子类型多态性允许使用接口的代码*以泛型风格编写(通过使用超类型而不是它的任何子类型)；特别多态性和参数多态性则不然。*
*   参数多态允许代码*实现*一个接口以通用风格编写(通过使用参数化类型)；相反，特殊和子类型多态性需要为每种类型编写单独的代码。
*   特设和参数多态都允许基本类型(如`int`)与一个接口一起使用；子类型多态性需要用户定义的类型。

一般来说，参数多态性用于特定类型的专门化数据结构，而子类型多态性用于使业务逻辑更加可重用和可组合。除了噱头和方便之外，我还没有发现特别多态性的用途。(为给定的数据类型调用适当的函数*真的有多难*？)

在 C 编程语言中:

*   使用`_Generic()`表达式实现了特定的多态性。
*   子类型多态性(我的[上一篇教程](/polymorphism-in-c-tutorial-bd95197ddbf9)的主题)是使用函数指针实现的。
*   参数多态性(本教程的主题)是使用宏实现的。

# 宏不被认为是有害的吗？

我以前的一个同事(我非常尊敬他)曾经说过:

> 每当程序员说‘我不相信宏’，就有一个 bug 死掉。

当然，这只是 J.M .巴里的经典故事《彼得潘》中一个更著名的关于儿童和仙女的陈述的变体。

的确，宏可能对代码的正确性、可读性和可调试性不利，但是如果没有预处理器，C 语言就不是一门完整的语言。您可以选择只使用`#include`并放弃其他预处理命令，如`#define`，但是这样做，您将失去许多有用的语言工具。别忘了`bool`、`NULL`、`INT_MAX`、`assert()`都是宏，不可或缺的伪函数如`ARRAY_SIZE()`、`container_of()`也是。

比起细粒度地使用`#if`或`#ifdef`，我更喜欢函数的外部链接；比起`#define`常量，我更喜欢枚举常量；比起类似函数的宏，我更喜欢内联函数。然而，上述信条并不意味着 C 预处理器没有合法用途。

Stroustrup 在《c++(1994)的设计和进化》中驳斥了“Cpp 的丑陋和低级语义”，他还写道:

> 确定我们需要参数化类型工具的关键活动是使用宏编写程序来伪造参数化类型。

我经常从那些认为 C 不适合某个特定目的的人那里看到这种态度。当面对一个解决方案时，他们往往认为它是“假的”，因为它不符合他们的先入为主的观念。我会让你判断我即将呈现的参数化是否真实。

# 离题进入递归

假设我们想为一个图像编辑器实现一个洪水填充算法。首先，填充起始位置的像素。然后，如果与先前填充的像素相邻的每个像素具有与初始像素最初具有的颜色相同的颜色，则该像素同样被填充。

这个问题的一个简单解决方案可能是使用递归。递归函数调用自身(直接或间接)，例如，为了探索树状数据结构的分支而不丢失其先前位置的轨迹。洪水填充算法将图像解释为这样的树。

下面是一个 C 程序，它递归地实现了洪水填充算法:

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

enum {
    IMG_SIZE = 8
};

struct image {
    char pixels[IMG_SIZE][IMG_SIZE];
};

static void flood_fill(struct image *const img,
                       int x, int y,
                       char find, char replace)
{
    if (x < 0 || y < 0 ||
        x >= IMG_SIZE || y >= IMG_SIZE ||
        img->pixels[y][x] != find)
    {
        return;
    }

    img->pixels[y][x] = replace;
    flood_fill(img, x+1, y, find, replace);
    flood_fill(img, x, y+1, find, replace);
    flood_fill(img, x-1, y, find, replace);
    flood_fill(img, x, y-1, find, replace);
}

int main(int argc, char const *argv[])
{
    struct image img = {{
        {' ', ' ', ' ', '#', '#', ' ', ' ', ' '},
        {' ', '#', ' ', ' ', ' ', ' ', '#', ' '},
        {' ', ' ', '#', ' ', '#', '#', ' ', ' '},
        {'#', '#', '#', ' ', ' ', '#', '#', '#'},
        {'#', ' ', '#', '#', ' ', '#', ' ', '#'},
        {' ', ' ', '#', ' ', ' ', '#', ' ', '#'},
        {'#', ' ', '#', ' ', '#', '#', ' ', ' '},
        {'#', '#', ' ', ' ', '#', ' ', '#', '#'},
    }};

    for (int y = 0; y < IMG_SIZE; ++y) {
        printf("%.*s\n", IMG_SIZE, img.pixels[y]);
    }

    puts("Enter coordinates (x,y):");
    char input[16];
    int start_x, start_y;

    if (fgets(input, sizeof(input), stdin) == NULL ||
        strchr(input, '\n') == NULL ||
        sscanf(input, "%d,%d", &start_x, &start_y) != 2 ||
        start_x < 0 || start_y < 0 ||
        start_x >= IMG_SIZE || start_y >= IMG_SIZE) 
    {
        fputs("Bad input\n", stderr);
        return EXIT_FAILURE;
    }

    char const find = img.pixels[start_y][start_x];
    flood_fill(&img, start_x, start_y, find, '.');
    puts("New image:");

    for (int y = 0; y < IMG_SIZE; ++y) {
        printf("%.*s\n", IMG_SIZE, img.pixels[y]);
    }

    return EXIT_SUCCESS;
}
```

递归有一个主要缺点:在函数中使用的自动变量的存储直到它退出时才能被释放。过多的嵌套函数调用会导致堆栈溢出。如果发生溢出，程序会被突然终止，而没有先给予恢复的机会(例如，不像对`malloc()`的调用失败)。

鉴于 C 运行时系统使用堆栈来存储活动函数调用的变量值，显而易见的解决方案是在程序本身中实现堆栈数据类型。这个显式堆栈可用于跟踪泛洪填充操作的进度，而不是通过递归隐式使用调用堆栈。

# 堆栈数据类型

下面的代码实现了一个堆栈数据类型，包括推送和弹出项目的函数，以及另一个检查堆栈是否为空的函数:

```
#include <stdbool.h>
#include <stdlib.h>
#include <assert.h>

struct coord {
    int x, y;
};

struct coord_stack {
    size_t nitems, limit;
    struct coord *items;
};

bool coord_stack_init(struct coord_stack *const stack,
                      size_t const limit)
{
    *stack = (struct coord_stack){
        .nitems = 0,
        .limit = limit,
        .items = malloc(sizeof(struct coord) * limit)
    };

    return stack->items != NULL;
}

void coord_stack_term(struct coord_stack *const stack)
{
    free(stack->items);
}

bool coord_stack_push(struct coord_stack *const stack,
                      struct coord item)
{
    if (stack->nitems >= stack->limit) {
        return false;
    }

    stack->items[stack->nitems++] = item;
    return true;
}

struct coord coord_stack_pop(struct coord_stack *const stack)
{
    assert(stack->nitems > 0);
    return stack->items[--stack->nitems];
}

bool coord_stack_is_empty(struct coord_stack *const stack)
{
    return stack->nitems == 0;
}
```

在这个堆栈实现中，没有任何东西关心项目类型`struct coord`实际上代表什么，因此这段代码是将来通用化的一个很好的候选。

下面是使用新数据类型重写的不递归的`flood_fill()`函数:

```
enum { 
    STACK_LIMIT = 128
};

static void flood_fill(struct image *const img,
                       int x, int y, char find, char replace)
{
    struct coord_stack stack;

    if (!coord_stack_init(&stack, STACK_LIMIT)) {
        fputs("Out of memory\n", stderr);
        return;
    }

    coord_stack_push(&stack, (struct coord){x,y});

    while (!coord_stack_is_empty(&stack)) {
        struct coord p = coord_stack_pop(&stack);

        if (p.x < 0 || p.y < 0 ||
            p.x >= IMG_SIZE ||
            p.y >= IMG_SIZE ||
            img->pixels[p.y][p.x] != find) 
        {
            continue;
        }

        img->pixels[p.y][p.x] = replace;

        if (!coord_stack_push(&stack, (struct coord){p.x+1,p.y}) ||
            !coord_stack_push(&stack, (struct coord){p.x,p.y+1}) ||
            !coord_stack_push(&stack, (struct coord){p.x-1,p.y}) ||
            !coord_stack_push(&stack, (struct coord){p.x,p.y-1})) 
        {
            fputs("Stack overflow\n", stderr);
            break;
        }
    }

    coord_stack_term(&stack);
}
```

# 更多堆栈

既然我们的图像编辑器有了一个洪水填充算法，我们可以考虑让编辑器更加用户友好。填充错误的像素很容易意外破坏图像。

支持撤消和重做需要编辑器存储一个可以撤消的先前编辑的列表(以相反的顺序)，以及一个可以重做的已撤消编辑的列表(以它们的原始顺序)。

一种可能的实现使用两个堆栈:

*   当编辑“完成”时，它被推到“撤消”堆栈上(并且“重做”堆栈被清除)。
*   当一个编辑被撤销时，它从“撤销”堆栈中弹出，并被推送到“重做”堆栈中。
*   当重做一个编辑时，它从“重做”堆栈中弹出，并被推送到“撤消”堆栈中。

“撤销”堆栈还需要在每次编辑之前存储图像状态的(全部或部分)快照，以便它可以恢复该状态。

下面是图像编辑器的`main()`功能，修改后支持撤销和重做:

```
#include <ctype.h>

struct undo_record {
    struct coord start;
    struct image before;
};

struct redo_record {
    struct coord start;
};

static bool process_user_input(struct undo_stack *const undos,
                               struct redo_stack *const redos,
                               struct image *const img)
{
    puts("'U'ndo, 'R'edo, 'Q'uit, or coords (x,y):");
    char input[16];

    if (fgets(input, sizeof(input), stdin) == NULL ||
        strchr(input, '\n') == NULL)
    {
        fputs("Bad input\n", stderr);
        return true; // ready for next input
    }

    switch (tolower(input[0])) {
    case 'u':
        if (!undo_stack_is_empty(undos)) {
            struct undo_record undo =
                undo_stack_pop(undos);

            struct redo_record redo = {
                .start = undo.start,
            };

            if (!redo_stack_push(redos, redo)) {
                fputs("Out of memory\n", stderr);
                undo_stack_push(undos, undo);
                break;
            }
            *img = undo.before;
        }
        break;

    case 'r':
        if (!redo_stack_is_empty(redos)) {
            struct redo_record redo =
                redo_stack_pop(redos);

            struct undo_record undo = {
                .start = redo.start,
                .before = *img,
            };

            if (!undo_stack_push(undos, undo)) {
                fputs("Out of memory\n", stderr);
                redo_stack_push(redos, redo);
                break;
            }

            char const find =
                img->pixels[undo.start.y][undo.start.x];

            flood_fill(img, undo.start.x, undo.start.y, find, '.');
        }
        break;

    case 'q':
        return false; // no more user input

    default:
        {
            int start_x, start_y;

            if (sscanf(input, "%d,%d",
                        &start_x, &start_y) != 2 ||
                start_x < 0 || start_y < 0 ||
                start_x >= IMG_SIZE || start_y >= IMG_SIZE)
            {
                fputs("Bad input\n", stderr);
                break;
            }

            struct undo_record undo = {
                .start = {start_x, start_y},
                .before = *img,
            };

            if (!undo_stack_push(undos, undo)) {
                fputs("Out of memory\n", stderr);
                break;
            }

            while (!redo_stack_is_empty(redos)) {
                redo_stack_pop(redos);
            }

            char const find = img->pixels[start_y][start_x];
            flood_fill(img, start_x, start_y, find, '.');
        }
        break;
    }

    return true; // ready for next input
}

int main(int argc, char const *argv[])
{
    struct image img = {{
        /* ... As before ... */
    }};

    struct undo_stack undos;

    if (!undo_stack_init(&undos, STACK_LIMIT)) {
        fputs("Out of memory\n", stderr);
        return EXIT_FAILURE;
    }

    int exit_code = EXIT_SUCCESS;
    struct redo_stack redos;

    if (!redo_stack_init(&redos, STACK_LIMIT)) {
        fputs("Out of memory\n", stderr);
        exit_code = EXIT_FAILURE;
    } else {
        do {
            for (int y = 0; y < IMG_SIZE; ++y) {
                printf("%.*s\n", IMG_SIZE, img.pixels[y]);
            }
        } while (process_user_input(&undos, &redos, &img));

        redo_stack_term(&redos);
    }

    undo_stack_term(&undos);
    return exit_code;
}
```

编译这段代码时，您会看到由于缺少函数声明(如`undo_stack_init()`和类型声明(如`struct undo_stack`)而导致的错误。假设这些名称遵循与已经为`flood_fill()`函数实现的堆栈数据类型相同的模式。

下一个问题是如何添加缺失的定义。

# 不那么通用的解决方案

最简单的方法是复制两个现有的堆栈代码，并修改它们以用作撤销和重做堆栈。生成的代码将是类型安全的，易于调试和阅读，但任何未来的更改都需要在三个地方复制。这可能需要很高的维护成本。

为了避免这种代码重复，我们可以定义一个`union`来表示所有三种堆栈项目类型(坐标、撤销和重做)。每个堆栈项目都将使用最大项目类型所需的内存:

```
union stack_item {
  struct coord coord;
  struct undo_record undo;
  struct redo_record redo;
};
```

实际上，这是子类型多态性，因此在从堆栈中弹出一个项后，需要(概念上)向下转换到适当的子类型。这不是类型安全的，尽管可以将`union`包装在一个`struct`中，以指示哪个成员是有效的。

另一种选择是在每个堆栈项目中存储一个`void *`指针，而不是一个`struct`类型。这也不是类型安全的，并且需要为每个堆栈项目分配和释放单独的存储空间，这似乎容易出错并且效率低下。

# 类型通用宏

我之前写道，泛型代码是使用“以后指定的类型”编写的。有时，一个可行的替代方法是根本不指定类型！

Kernighan & Ritchie 在《C 编程语言》(1988)中提供了一个类似函数的宏的例子，它“适用于任何数据类型”:

```
#define max(A, B) ((A) > (B) ? (A) : (B))
```

这种宏之所以有效，是因为 C 语言提供的操作符(`[]`、`->`、`*`、`++`等)就像内置的通用函数，可以在任何兼容的类型上工作(在合理的范围内)。

我们的 stack 数据类型比选择两个值中的最大值的简单宏要复杂得多，但是如果我们可以对 stack 方法使用相同的技术，那么所有三种 stack 类型都可以调用它们。

以下是重新定义为类似函数的宏的堆栈方法:

```
#define STACK_INIT(stack, new_limit) \
    ((stack)->nitems = 0, \
     (stack)->limit = (new_limit), \
     ((stack)->items = malloc(sizeof((stack)->items[0]) * \
     (new_limit))) != NULL)

#define STACK_TERM(stack) \
    free((stack)->items)

#define STACK_PUSH(stack, item) \
    (((stack)->nitems >= (stack)->limit) ? \
     false : \
     ((stack)->items[(stack)->nitems++] = (item), true))

#define STACK_POP(stack) \
    ((stack)->items[--(stack)->nitems])

#define STACK_IS_EMPTY(stack) \
    ((stack)->nitems == 0)
```

下面是为了使用类型通用宏而重写的`flood_fill()`函数:

```
static void flood_fill(struct image *const img,
                       int x, int y, char find, char replace)
{
    struct coord_stack stack;

    if (!STACK_INIT(&stack, STACK_LIMIT)) {
        fputs("Out of memory\n", stderr);
        return;
    }

    STACK_PUSH(&stack, ((struct coord){x,y}));

    while (!STACK_IS_EMPTY(&stack)) {
        struct coord p = STACK_POP(&stack);

        if (p.x < 0 || p.y < 0 ||
            p.x >= IMG_SIZE ||
            p.y >= IMG_SIZE ||
            img->pixels[p.y][p.x] != find) 
        {
            continue;
        }

        img->pixels[p.y][p.x] = replace;

        if (!STACK_PUSH(&stack, ((struct coord){p.x+1,p.y})) ||
            !STACK_PUSH(&stack, ((struct coord){p.x,p.y+1})) ||
            !STACK_PUSH(&stack, ((struct coord){p.x,p.y+1})) ||
            !STACK_PUSH(&stack, ((struct coord){p.x,p.y-1}))) 
        {
            fputs("Stack overflow\n", stderr);
            break;
        }
    }
    STACK_TERM(&stack);
}
```

将这些宏描述为非类型安全可能有点误导，但是编译器只能检查底层`struct`类型(在别处定义)的使用，所以很难解释任何错误:

```
stacks.c:106:43: error: incompatible types when assigning to type
'struct redo_record' from type 'struct coord'
  106 |      ((stack)->items[(stack)->nitems++] = (item), true))
      |                                           ^
stacks.c:123:5: note: in expansion of macro 'STACK_PUSH'
  123 |     STACK_PUSH(&stack, ((struct coord){x,y}));
      |     ^~~~~~~~~~
```

使用类似函数的宏还有其他缺点:

*   返回值的宏必须写成复杂的表达式(使用`,`)，而不是将代码分割成逻辑块。
*   参数在宏定义中出现多少次，就计算多少次。例如，`STACK_INIT()`存储的`new_limit`的值可能与用于为项目分配内存的值不同(如果调用者给出的表达式有副作用，例如`limit++`)。
*   参数名不能用于宏定义中的其他目的(而`limit`既是`struct coord_stack`的成员，也是原始`coord_stack_init()`的参数)。
*   调用宏时，如果没有额外的括号，就不能在参数列表中使用逗号。这使得诸如`STACK_PUSH(&stack, (struct coord){x,y})`这样的调用无效。

然而，如果你对这个解决方案满意，那么祝你好运！

# 参数化堆栈数据类型

我们真正想要的是能够将堆栈项目类型指定为参数，在实例化原始`struct`和函数定义的变体时使用。

在编译时参数化任何东西的显而易见的方法是使用宏。以下类似函数的宏扩展为实例化给定`item_type`的堆栈数据类型所需的*定义*，而不是直接扩展为内联方法代码(如前一示例所示):

```
#define STACK_STRUCT(tag, item_type) \
struct tag { \
    size_t nitems, limit; \
    item_type *items; \
};

#define STACK_PUSH(tag, item_type) \
bool tag ## _push(struct tag *const stack, \
                  item_type item) \
{ \
    if (stack->nitems >= stack->limit) { \
        return false; \
    } \
    stack->items[stack->nitems++] = item; \
    return true; \
}

#define STACK_POP(tag, item_type) \
item_type tag ## _pop(struct tag *const stack) \
{ \
    assert(stack->nitems > 0); \
    return stack->items[--stack->nitems]; \
}

// etc.

#define STACK_DECLARE(tag, item_type) \
    STACK_STRUCT(tag, item_type) \
    STACK_INIT(tag, item_type) \
    STACK_TERM(tag, item_type) \
    STACK_PUSH(tag, item_type) \
    STACK_POP(tag, item_type) \
    STACK_IS_EMPTY(tag, item_type)
```

预处理器的`##`操作符用于将每个堆栈方法名与实例化类型的`struct`标记连接起来，以便创建一个具有预期前缀的唯一函数名。这允许在同一个源文件中实例化多个堆栈类型。

以与“push”和“pop”相同的方式参数化其他堆栈函数定义，这是留给读者的练习。

图像编辑器所需的三种堆栈类型现在可以实例化如下:

```
STACK_DECLARE(coord_stack, struct coord)
STACK_DECLARE(undo_stack, struct undo_record)
STACK_DECLARE(redo_stack, struct redo_record)
```

如果意外地用错误类型的参数调用了堆栈方法，那么我们会得到一个比使用类型通用宏更有用的错误:

```
stacks.c:129:22: warning: passing argument 1 of 'coord_stack_push'
from incompatible pointer type [-Wincompatible-pointer-types]
  129 |     coord_stack_push(&stack, (struct coord){x,y});
      |                      ^~~~~~
      |                      |
      |                      struct redo_stack *
stacks.c:80:37: note: expected 'struct coord_stack * const' but
argument is of type 'struct redo_stack *'
   80 | bool tag ## _push(struct tag *const stack, \
      |                   ~~~~~~~~~~~~~~~~~~^~~~~
stacks.c:107:5: note: in expansion of macro 'STACK_PUSH'
  107 |     STACK_PUSH(tag, item_type) \
      |     ^~~~~~~~~~~
stacks.c:111:1: note: in expansion of macro 'STACK_DECLARE'
  111 | STACK_DECLARE(coord_stack, struct coord)
      | ^~~~~~~~~~~~~~
```

新的堆栈方法定义可以像普通函数一样读写，但是我仍然认为可以改进这个解决方案:

*   我喜欢将方法定义拆分成多行，但我不希望它们被延续字符`\`弄得乱七八糟。
*   `STACK_DECLARE()`的调用看起来像函数调用，在函数之外是无意义的。
*   在每次`STACK_DECLARE()`调用后放一个分号很诱人，但是标准 C 不允许在函数之外多加一个`;`。

# 但是我只想写正常的代码

c 实际上提供了两种将参数化代码替换到程序中的机制。最明显的是宏，我们已经看过了；另一个是谦逊的`#include`指令。

一个`#include`行通常用于包含一个头文件，其中声明了具有外部链接的函数，但是它同样适用于将任意代码片段粘贴到程序中。

> 这是通往许多被认为是不自然的能力的途径。

——议长帕尔帕庭，《西斯的复仇》

与宏不同，`#include`行不能有参数(除了文件名)。这没关系，因为在处理包含的文件时，在到达`#include`行之前定义的任何宏也会被扩展。

下面是一些原始的堆栈代码，使用两个必须在包含该文件之前定义的宏进行了参数化:

```
// generic_stack.h

#if !defined(STACK_TAG) || !defined(STACK_ITEM_TYPE)
#error Missing type or tag definition
#endif

#define STACK_CONCAT(tag, method) tag ## _ ## method
#define STACK_METHOD2(tag, method) STACK_CONCAT(tag, method)
#define STACK_METHOD(method) STACK_METHOD2(STACK_TAG, method)

struct STACK_TAG {
    size_t nitems, limit;
    STACK_ITEM_TYPE *items;
};

bool STACK_METHOD(push)(struct STACK_TAG *const stack,
                        STACK_ITEM_TYPE item)
{
    if (stack->nitems >= stack->limit) {
        return false;
    }

    stack->items[stack->nitems++] = item;
    return true;
}

STACK_ITEM_TYPE STACK_METHOD(pop)(struct STACK_TAG *const stack)
{
    assert(stack->nitems > 0);
    return stack->items[--stack->nitems];
}

// etc.

#undef STACK_TAG
#undef STACK_ITEM_TYPE
#undef STACK_CONCAT
#undef STACK_METHOD2
#undef STACK_METHOD
```

参数化缺失的函数定义再次成为读者的练习。

与普通头文件不同的是，“generic _ stack . h”*不能*包含习惯的`#ifndef`、`#define`和`#endif`行，以防止被多次包含。这是为了允许在同一源文件中为不同的类型实例化一个堆栈。

宏`STACK_METHOD2()`的必要性有些模糊，但 K & R (1988)的附录 A 对此进行了解释:

> 除非替换序列中的参数前面有`*#*`，或者前面或后面有`*##*`，否则在插入之前，将检查宏调用的参数标记，并根据需要进行扩展。

换句话说，如果`STACK_TAG`被直接用作`STACK_CONCAT()`的参数，那么得到的函数名应该类似于`STACK_TAG_push()`！

图像编辑器所需的三种堆栈类型现在可以实例化如下:

```
#define STACK_TAG coord_stack
#define STACK_ITEM_TYPE struct coord
#include "generic_stack.h"

#define STACK_TAG undo_stack
#define STACK_ITEM_TYPE struct undo_record
#include "generic_stack.h"

#define STACK_TAG redo_stack
#define STACK_ITEM_TYPE struct redo_record
#include "generic_stack.h"
```

# 引人深思的事

*   为什么上个世纪的程序员更喜欢类似函数的宏而不是函数？
*   为什么认为“generic_stack.h”取消定义了它没有定义的宏`#define`？
*   除了使用宏(如`STACK_METHOD()`)生成函数名，还有其他选择吗？
*   为什么`STACK_TAG`宏仅仅指定了一个`struct`标签，而`STACK_ITEM_TYPE`指定了一个完整的类型？
*   你如何扩展图像编辑器以允许用户指定填充颜色？

# 这是所有的乡亲

我希望你觉得这个教程有用；我写得很开心。如果没有，为什么不发表评论，让我知道你还想阅读哪些 C 编程主题？

感谢阅读。
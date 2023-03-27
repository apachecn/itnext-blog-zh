# 是的，RISC-V 实际上是一个很好的设计

> 原文：<https://itnext.io/yeah-risc-v-is-actually-a-good-design-1982d577c0eb?source=collection_archive---------0----------------------->

## 业内知名人士如 Dave Jaggar、Jim B. Keller 和 Dave Ditzel 都对 RISC-V 竖起了大拇指。

![](img/bc19a35b103c0d93804b2a61641cbaf7.png)

我写的关于 RISC-V 的文章越多，我就越意识到 RISC-V 在许多圈子里是多么有争议。20 世纪 80 年代，RISC 狂热者声称 RISC-V 是一个糟糕的设计，这并不罕见。问题是:你应该对互联网上发表这种言论的人投入多少股票？

理想情况下，你应该根据它的优点来判断一个设计，但是如果这对你来说还不够好，那么也许你可以尊重备受尊敬的微处理器设计师的意见。这是他们中的一些人对 RISC-V 的评论。

# 戴夫·贾格尔——手臂设计师

最近，Dave Jaggar 做了一个关于 Arm 公司及其微处理器历史的演讲。你问戴夫·贾格尔是谁？维基百科是这样说的:

> David Jaggar(生于 1967 年 2 月 4 日)是一名计算机科学家，他在 1992 年至 2000 年期间负责 ARM 架构的开发，将其从低成本工作站处理器重新定义为占主导地位的嵌入式系统处理器。

贾格尔做了许多重要的设计，这些设计有助于 Arm 的成功。一个重要的选择是，他负责发明压缩 16 位指令(Thumb ),这使得 Arm 在嵌入式领域取得了巨大成功。如果没有拇指臂，它永远也不会进入智能手机市场。Arm 被诺基亚选中，因为他们将密集代码、高性能和低功耗集于一身。这对于压缩指令来说是不可能的。有趣的是，现代 64 位 Arm 没有压缩指令，但它是 RISC-V 的重要组成部分。

在关于 Arm 历史的演讲中，Dave Jaggar 被问到，如果你对微处理器感兴趣，应该研究什么架构。[贾格尔回应](https://youtu.be/%5C_6sh097Dk5k?t=3071) (51 分钟):

> 有武装狙击手吗？我会在谷歌上搜索 RISC-V，找到关于它的所有信息。**他们做了一个很好的指令集**，一个很好的工作，他们正在解释它。伯克利和斯坦福是幕后黑手。显然有 SiFive 这样的商业公司在做事情，但它是 32 位通用指令集的最先进产品，它有 16 位压缩的东西。

戴夫·贾格尔讲述他是如何设计 Arm 芯片的，以及他对 RISC-V 的评论

## 吉姆·凯勒——AMD、苹果、特斯拉和英特尔设计师

吉姆·b·凯勒可能是当今最著名的微处理器设计师之一，因为他在 AMD 和苹果的成功中发挥了关键作用。让我们再次引用[维基百科](https://en.wikipedia.org/wiki/Jim_Keller_(engineer)):

> 詹姆斯·b·凯勒(生于 1958 年/1959 年)是一名微处理器工程师，因其在 AMD 和苹果公司的工作而闻名。他是 AMD K8 微架构(包括最初的 Athlon 64)的首席架构师，并参与了 Athlon (K7)和苹果 A4/A5 处理器的设计。他还是 x86–64 指令集和 HyperTransport 互连规范的合著者。从 2012 年到 2015 年，他回到 AMD，从事 AMD K12 和 Zen 微体系结构的工作。

用于苹果重要产品如 iPhone 4、iPad 和 iPad 2 的芯片是由凯勒设计的。在 TechTechPotato 关于 Arm vs x86 vs RISC-V 的采访中，他这样说道:

> 现在，RISC-V 出现了。它是闪亮的新表亲，因为没有遗产，它实际上是一个开放的指令集架构，人们在大学里构建，他们没有时间或兴趣像一些架构那样添加太多垃圾，所以相对而言，因为它的血统和年龄，它处于复杂性生命周期的早期，并且**它是一个非常好的指令集。他们做得很好**，所以如果我只是想说我想建立一个计算机真的很快，我希望它去快，RISC-V 是最简单的一个。它拥有所有合适的特征。它有所有正确的前八条指令。那些你真正需要优化的。它没有太多的垃圾。

Kelly 提到的八条指令是在之前的采访中提到的，当时他谈到微处理器在大多数时间里只执行八条常用指令中的一条，如 load、store、branch、add 等。

吉姆·凯勒对 RISC-V 进行了评论，并解释了为什么他会选择它来制造快速 CPU

## 戴夫·迪泽尔— Transmeta，RISC，世界语

戴夫·迪泽尔以打破常规思维著称。让我引用 [EE 期刊](https://www.eejournal.com/article/machine-learning-esperanto-coaxes-1092-risc-v-processors-to-dance-on-the-head-of-a-pin-er-chip/)来介绍他:

> 在贝尔实验室，迪特泽尔还与加州大学伯克利分校的大卫·帕特森教授合著了基础 RISC 文档“精简指令集计算机的案例”。然后，Ditzel 加入 Sun Microsystems，担任 SPARC 技术业务的首席技术官，领导 SPARC RISC 处理器体系结构和 64 位 SPARC ISA 的开发。甲骨文在 2010 年收购了太阳微系统公司，后来在 2016 年停止了 SPARC 公司的开发。但是由于 SPARC 国际，SPARC ISA 继续作为完全开放的，非专有的，免版税的知识产权。

你们中的一些人可能还记得 Transmeta，它是由迪策尔于 1995 年创立的。这是 21 世纪初 Crusoe 处理器的热门话题，许多人认为它会取代 x86。它基于超长指令字(VLIW)的思想，编译器捆绑了可以并行执行的指令以提高性能。这个想法也是 Transmeta 芯片可以模拟任何 ISA，包括 x86。

阅读更多关于 VLIW: [超长指令字微处理器](https://erik-engheim.medium.com/very-long-instruction-word-microprocessors-17262def3037)

无论如何，我的观点是迪泽尔有跳出框框思考的历史。在创办世界语科技公司时，他和他的处理器设计师们着眼于创造这些定制芯片来加速人工智能。他们为这些芯片开发了自己的指令集，但意识到在工具和其他基础设施方面需要大量投资，因此开始关注 RISC-V。通过一系列内部测试，他们得出结论，RISC-V 实际上是一种设计非常好的 ISA，可以用来构建他们的 ET-SOC-1 推理引擎。

## 自己做比较

多亏了 [Godbolt](https://godbolt.org/z/bhhjscr3W) ，很容易快速地对各种指令集架构进行比较。许多人会制作这些小的汇编代码片段来证明 RISC-V 是一个糟糕的设计，它需要比同类架构多很多倍的指令。然而，如果你不仅仅看 4-5 个汇编操作码的玩具例子，这种情况就不会出现。

```
// C - Most basic sorting algorithm

void bubble_sort(int xs[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {    
            // swap values if not in order
            if (xs[j] > xs[j + 1]) {
                int temp = xs[j];
                xs[j] = xs[j + 1];
                xs[j + 1] = temp;
            }
        }
    }
}
```

我们可以在 [Godbolt](https://godbolt.org/z/bhhjscr3W) 中比较一下这个基础代码的 RISC-V、Arm 和 x86 汇编代码。我尝试使用 gcc 10.2.0，其中的选择有些随意，我使用针对小代码优化的`-Os`开关得到了以下结果。

*   RV32gc (RISC-V 32 位)— 24 行代码
*   ARM 32 位— 25 行代码
*   x86–64–26 行代码
*   POWER (IBM RISC ISA) — 32 行代码

有人可能会说，我们应该与 64 位 Arm 指令集等较新的架构进行比较。我们可以做到。然而，这对代码数量没有影响。奇怪的是，它在 RISC-V 代码中添加了另一行，但这一行完全没有意义。RV32gc 开头的以下 RISC-V 汇编被重写:

```
# RISC-V 32-bit code

li      a4,0            # Load 0 into register a4
addi    a1,a1,-1        # Add -1 to register a1 and store in a1
```

对于 RV64gc，我们得到了一个不需要的 move ( `MV`)指令:

```
# RISC-V 64-bit code

addiw   t1,a1,-1   # could have written ADDIW a1, a1, -1 instead 
li      a4,0
mv      a1,t1
```

因此，即使 RISC-V 具有避免复杂指令的最小指令集，这个汇编代码对于冒泡排序也是最短的:

```
bubble_sort:
        li      a4,0           # load 0 into register a4
        addi    a1,a1,-1       # add -1 to a1 and store in a1
.L2:
        ble     a1,a4,.L1      # if a1 <= a4 goto L1
        mv      a5,a0          # copy a0 into register a0
        li      a3,0
        sub     a7,a1,a4
        j       .L6            # jump to .L6
.L4:
        lw      a2,0(a5)       # load word at address 0 + a5 into a2
        lw      a6,4(a5)
        ble     a2,a6,.L3
        sw      a6,0(a5)       # store word in a6 at address 0 + a5
        sw      a2,4(a5)
.L3:
        addi    a3,a3,1
        addi    a5,a5,4
.L6:
        blt     a3,a7,.L4     # if a3 < a7 goto .L4
        addi    a4,a4,1
        j       .L2
.L1:
        ret
```

也许只是侥幸，那么用斐波那契递归调用怎么样？

```
int fibonacci(int n) {
   if (n == 0)
      return 0;
   else if (n == 1)
      return 1;
   else
      return fibonacci(n-1) + fibonacci(n-2);
}
```

在这种情况下，RISC-V 稍微长一点:

*   RISC-V 32 和 64 位 25 行代码
*   ARM 64 位— 20 行代码
*   x86–64–22 行代码
*   POWER64 (IBM RISC ISA) — 30 行代码

RISC-V 变得更长的主要原因是递归调用需要在堆栈上加载和存储变量:

```
# RISC-V 32-bit - recursive fibonacci 

fibonacci:
        addi    sp,sp,-16
        sw      s0,8(sp)
        sw      s1,4(sp)
        sw      s2,0(sp)
        sw      ra,12(sp)
        mv      s0,a0
        li      s1,0
        li      s2,1
.L3:
        beq     s0,zero,.L2
        beq     s0,s2,.L2
        addi    a0,s0,-1
        call    fibonacci
        addi    s0,s0,-2
        add     s1,s1,a0
        j       .L3
.L2:
        add     a0,s0,s1
        lw      ra,12(sp)
        lw      s0,8(sp)
        lw      s1,4(sp)
        lw      s2,0(sp)
        addi    sp,sp,16
        jr      ra
```

如果我们写一个斐波那契的非递归变体，那么它最终会占用与 64 位 Arm 完全相同的 19 行代码。[看这里 Godbolt](https://godbolt.org/z/K91jTWsrq) 。

```
# RISC-V 64-bit - non-recursive fibonacci 

fib:
        li      a4,1
        mv      a5,a0
        bleu    a0,a4,.L5     # Branch Less or Equal Unsigned
        li      a4,2
        li      a0,0
        li      a3,1
        li      a2,0
.L3:
        ble     a4,a5,.L4
        ret
.L4:
        addw    a0,a2,a3     # Add word (full 64-bits)
        addiw   a4,a4,1
        mv      a2,a3        # MoVe a3 to a2 (copy a3 to a2)
        mv      a3,a0
        j       .L3          # Jump to .L3
.L5:
        ret
```

x86 版本看起来更短，只有 18 行，但这只是因为它使用了更少的标签。如果你计算一下指令的数量，你会得到 15 条指令，这与 64 位 Arm 和 RISC-V 完全相同。

```
; x86-64 - non-recursive fibonacci

fib:
        mov     eax, edi
        cmp     edi, 1
        jbe     .L1
        mov     edx, 2
        xor     eax, eax     # x86 trick to zero out eax register
        mov     ecx, 1
        xor     esi, esi     # set esi register to zero
.L3:
        cmp     edx, edi
        jg      .L1
        lea     eax, [rsi+rcx]
        inc     edx          # increment edx (add 1)
        mov     esi, ecx
        mov     ecx, eax
        jmp     .L3
.L1:
        ret
```

你可以做很多这样的测试，看看你得到了什么。我尝试了合并排序，得到了 102 条 RISC-V 指令、96 条 Arm 指令和 112 条 x86 指令(去掉了标签)。我尝试了二叉树搜索，矩阵乘法和许多其他例子。一般来说，RISC-V 做得很好。当有大量的保存寄存器要堆栈时，它做得不好，因为 RISC-V 没有像 Arm 那样存储和加载对或寄存器的指令。

阅读 RISC-V 如何产生更紧凑的代码:精简指令集的新案例 [Set Computer:通过 RISC-V 的宏操作融合避免 ISA 膨胀](https://arxiv.org/abs/1607.02318)

视频讲座论文覆盖:[ISA Shootout—RISC V、ARM 和 x86 的比较加州大学伯克利分校 Chris Celio](https://www.youtube.com/watch?v=HNjcQcjINNY)

我也会谈到这个话题:[RISC-V 微处理器的天才](https://erik-engheim.medium.com/the-genius-of-risc-v-microprocessors-b19d735abaa6)

## 结论

RISC-V 完美吗？绝对不是，但 RISC-V 是一个平庸的设计，没有从处理器设计的最新创新中学到任何东西的想法是没有根据的。如果 RISC-V 是一个糟糕的设计，它就不会迅速发展，也不会出现在众多新的芯片设计中。RISC-V 之所以成功，只是因为你不必支付许可费，这种想法是垃圾。如今有大量的开放架构，如 [MIPS Open](https://www.mips.com/mipsopen/) 、 [OpenPOWER](https://openpowerfoundation.org) 和 [OpenSPARC](https://www.oracle.com/servers/technologies/opensparc-t2-page.html) ，然而这些都没有引起太多关注，尽管它们都是公认的行业标准。
Title: 编写可综合的 Verilog HDL 代码
Date: 2014-04-04 01:35
Category: FPGA
Tags: FPGA,Verilog
Slug: writing_synthesizable_code
Author: Chien Gu
Summary: 总结可综合 Verilog HDL 代码，如何编写可综合的代码

对电路建模方法有很多种，可以用绘制原理图，也可以用 *硬件描述语言（Hardware Description Language）* 建模 。硬件描述语言中最常用的就是 Verlilog 和 VHDL 。

Verilog HDL和VHDL相比有很多优点，有C语言基础的话很容易上手。搜集了一些网上大神的经验总结和书上的例子，所以对于和我一样的初学者，这篇博客应该还是很有提高作用的，至于具体语法，任何一本书都讲的很详细。

## HDL
* * *
从网上找到一篇文章，把 HDL 的历史说的非常清楚。

作者 董培良 

题目： [浅谈VHDL/Verilog的可综合性以及对初学者的一些建议][blog1]

首先要明确的是 VHDL 和 Verilog 并非是针对硬件设计而开发的语言，只不过目前被我们用来设计硬件。HDL 是 Hardware **Description** Language 的缩写，正式中文名称是 “硬件描述语言”。也就是说，HDL 并不是 “硬件设计语言（Hardware **Design** Language）”。别看只差这一个单词，正是这一个单词才决定了绝大部分电路设计必须遵循RTL的模式来编写代码，而不能随心所欲得写仅仅符合语法的 HDL 代码。

VHDL 于 1980 年开始在美国国防部的指导下开发，完成于 1983 年，并于 1987 年成为 IEEE 的标准。当初开发这种语言，是出于美国国防部采购电子设备的需要。美军的装备采购自私人企业，时常要面对这样一种风险：如果某种武器大量装备部队，而其中某个零件的供应商却在几年后倒闭了，那这种武器的再生产、维修和保养都会出现大问题。而电子设备、尤其是集成电路的内部结构较为复杂，若出现前面所说的情况要找其他公司生产代用品非常困难。于是美国防部希望供应商能以某种形式留下其产品的信息，以保证一旦其破产后能由其他厂商迅速生产出代用品。

显然，当初的设计文档显然是不能交出来的，这在美国会涉及商业机密和知识产权问题。于是美国防部就想出了一种折衷的方法——描述硬件的语言，也就是 VHDL 。通过 VHDL，供应商要把自己生产的集成电路芯片的行为描述出来：比如说，加了什么样的信号后过多少时间它能输出什么等等。这样，如果有必要让其他厂商生产代用品，他们只需照着 VHDL 文档，设计出行为与其相同的芯片即可。这样的代用品相当于是新厂商在不了解原产品结构的情况下独立设计的，所以不太会涉及知识侵权。

Verilog HDL 也形成于差不多的年代，是由 Gateway Design Automation 公司大约在 1983 年左右开发的。其架构同 VHDL 相似，但主要被用来进行硬件仿真。或许私人公司更注重实用，Verilog 要比 VHDL 简洁得多。

由此可见，这两种最流行的用于电路设计的语言，没有一种是为了设计硬件而开发的（更何况 80 年代还没有现在的那些功能强大的EDA软件呢）。因此，当初制订 HDL 语言标准的时候，并没有考虑这些代码如何用硬件来实现。换句话说，有些代码写起来简单，实现起来却可能非常复杂，或者几乎不可能实现。

[blog1]: http://www.dzkf.cn/html/EDAjishu/2006/0720/9.html

<br>

## 可综合代码 
* * *

任何符合 HDL 语法标准的代码都是对硬件行为的一种描述，但不一定是可直接对应成电路的设计信息。行为描述可以基于不同的层次，如系统级，算法级，寄存器传输级(RTL)、门级等等。以目前大部分EDA软件的综合能力来说，**只有RTL或更低层次的行为描述才能保证是可综合的**。而众多初学者试图做的，却是想让软件去综合 *算法级或者更加抽象的硬件行为描述*。

### 所有综合工具都支持的语法：

```Verilog
    always, assign, begin, end, 
    wire, tri, inout, aupply0, supply1, input, reg, integer, default, 
    and, nand, or, nor, xor, xnor, buf, not, bufif0,bufif1, notif0, notif1, 
    if, case, for, function, instantitation, module, negedge, posedge, operators, output, parameter  
```

### 有些工具支持，有些工具不支持的语法：

```Verilog
    asex, casez, wand, triand, wor, trior, real, disable, forever, arrays, memories, repreat, task,while
```

### 建立可综合模块的原则

+ 不要用 initial （FPGA 上电时初始状态不定，一般需要上电复位信号，在复位信号有效的时候进行初始化，上电复位信号可以由外部手动输入，也可以系统自己产生 —— 写一个实现上电产生自动复位信号的模块）。P.S. 现在的综合软件功能已经足够强大，即使写了 initial 语句，在 ISE 13.3 中仍然是可综合的，而且没有 warning 和 info 的提示）

+ 不使用 `#10`（在仿真中有用，实际在硬件上不会实现）

+ 不使用循环次数不定的循环语句，如 `forever`、`while` 等

+ 不使用用户自定义原语（UDP 原件）

+ 除非是关键路径设计，一般不采用调用门级原件描述的设计的方法，建议采用行为语句完成设计

+ 尽量使用同步方式设计电路

+ 用 `always` 语句描述组合逻辑时，在敏感信号列表中要列出所有输入信号

+ 所有的内部寄存器都应该可以被复位，在 FPGA 设计时应尽量使用器件的全局复位端信号作为系统的总复位

+ 时序逻辑使用非阻塞赋值，组合逻辑使用阻塞赋值，同一过程块中不要同时使用阻塞和非阻塞两种方式

+ 不要在不同的 `always` 过程块中对同一变量赋值（否则综合时会提示有多驱动源错误，*multiple source
*），对同一赋值对象，不能既使用阻塞赋值，又使用非阻塞赋值

+ 如果不打算把变量综合成锁存器，在 `if` 语句或 `case` 语句的所有分支中都要对变量明确赋值（不能省去 `else` 或 `default`，原理：在省去的情况下，变量的值会保持原来的值不变，所以系统会综合出一个锁存器 Latch）

+ 避免混合使用上升沿和下降沿触发器

+ 同一变量的赋值不能受多个时钟控制，也不能受两种不同时钟条件（或不同时钟沿）控制

+ 避免在 `case` 语句中使用 `x` 或 `z` 值

<br>

## 不可综合代码
* * *

### 不可综合语法：

```Verilog
    time, defparam, $finish, fork, join, initial, delays, UDP, wait
```

+ `initial` 只能在 Testbench 中使用，不能综合

+ `events` 在 Testbench 中更有用，不能综合

+ 不支持 `real` 类型的综合

+ 不支持 `time` 类型的综合

+ `force` 和 `release`

+ `assign` 和 `deassign` 不支持 `reg` 类型的综合，支持 `wire` 类型的综合

+ `fork...join` 不可综合，可以用非块语句达到同样的效果

+ `primitives` 支持门级原语综合，不支持非门级原语综合

+ 不支持 `table` 和 `UDP` 的综合

+ 同一个 `reg` 被多个 `always` 块驱动

+ 延时，不可综合为硬件电路延时，综合工具会忽略延时，但是不会报错

+ 与 `x`、`z` 比较，综合工具会忽略，所以要保证信号只有两个状态，`0` 或 `1`

<br>

## 判断是否可综合
* * *

继续引用 **董培良** 的文章：

用一句简单的话概括：电脑永远没有你聪明 。具体来说，通常 EDA 软件对 HDL 代码的综合能力总是比人差 。对于一段代码，如果你不能想象出一个较直观的硬件实现方法，那 EDA 软件肯定也不行。比如说，加法器、多路选择器是大家都很熟悉的电路，所以类似

```Verilog
        A+B-C
        (A>B)?C:D
```

这样的运算一定可以综合。而除法、开根、对数等等较复杂的运算，必须通过一定的算法实现，没有直观简单的实现方法，则可以判断那些计算式是不能综合的，必须按它们的算法写出更具体的代码才能实现 。此外，硬件无法支持的行为描述，当然也不能被综合（比如想在 FPGA 上实现 DDR 内存那样的双延触发逻辑，代码很容易写，但却不能实现）。

不过，这样的判断标准非常主观模糊，遇到具体情况还得按设计人员自己的经验来判断 。如果要一个相对客观的标准，一般来说：在 RTL 级的描述中，所有逻辑运算和加减法运算、以及他们的有限次组合，基本上是可综合的，否则就有无法综合的可能性 。当然，这样的标准仍然有缺陷，更况且 EDA 的技术也在不断发展，过去无法综合的代码或许将来行，某些软件不支持的代码换个软件或许行 。比如固定次数的循环，含一个常数参数的乘法运算等等，有些 EDA 软件支持对它们的综合，而有些软件不行。

所以，正确的判断仍然要靠实践来积累经验。当你可以较准确判断代码的可综合性的时候，你对 HDL 的掌握就算完全入门了。

<br>

## 参考

[浅谈VHDL/Verilog的可综合性以及对初学者的一些建议][blog1]
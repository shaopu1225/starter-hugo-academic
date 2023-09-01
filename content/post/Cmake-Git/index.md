---
title:		CMake-Git
subtitle:	CMake-Git
summary:	notes of how to use CMake and Git
date:		2021-07-31
lastmod:	2023-08-31
author:		shaopu
draft: 		false
image:		  
  focal_point: ''
  placement: 2
  preview_only: true

tags:
    - tools
    - CMake
    - Git

categories:
    - Tools
---

# Lecture 1

## Interpretation & Translation

根据维基百科上的定义：

> A **translator** or **programming language processor** is a generic term that can refer to anything that [converts](https://en.wikipedia.org/wiki/Data_conversion) code from one computer language into another.[[1\]](https://en.wikipedia.org/wiki/Translator_(computing)#cite_note-MCT-1)[[2\]](https://en.wikipedia.org/wiki/Translator_(computing)#cite_note-Intel_1983_SH-2) A program written in high-level language is called source program. These include translations between [high-level](https://en.wikipedia.org/wiki/High-level_language) and [human-readable computer languages](https://en.wikipedia.org/wiki/Source_code) such as [C++](https://en.wikipedia.org/wiki/C%2B%2B) and [Java](https://en.wikipedia.org/wiki/Java_(programming_language)), intermediate-level languages such as [Java bytecode](https://en.wikipedia.org/wiki/Java_bytecode), [low-level languages](https://en.wikipedia.org/wiki/Low-level_language) such as the [assembly language](https://en.wikipedia.org/wiki/Assembly_language) and [machine code](https://en.wikipedia.org/wiki/Machine_code), and between similar levels of language on different [computing platforms](https://en.wikipedia.org/wiki/Computing_platform), as well as from any of the above to another.

这是一种较为广义的`Translator`定义：就是说翻译器是将一种语言转化为另一种语言的处理器。在这种定义下，翻译器包含三个类型：

- **Compiler**:

> A [compiler](https://en.wikipedia.org/wiki/Compiler) is a translator used to convert [high-level programming language](https://en.wikipedia.org/wiki/High-level_programming_language) to [low-level programming language](https://en.wikipedia.org/wiki/Low-level_programming_language). It converts the whole [program](https://en.wikipedia.org/wiki/Computer_program) in one session and reports [errors](https://en.wikipedia.org/wiki/Software_bug) detected after the conversion. The compiler takes time to do its work as it translates high-level code to lower-level code all at once and then saves it to memory.

- **Interpreter**:

> The [interpreter](https://en.wikipedia.org/wiki/Interpreter_(computing)) is similar to a compiler, as it is a translator used to convert [high-level programming language](https://en.wikipedia.org/wiki/High-level_programming_language) to [low-level programming language](https://en.wikipedia.org/wiki/Low-level_programming_language). The difference is that it converts the program one line of code at a time and reports errors when detected, while also doing the conversion. An interpreter is faster than a compiler as it immediately executes the code upon reading the code. It is often used as a [debugging tool](https://en.wikipedia.org/wiki/Debugging_tool) for [software development](https://en.wikipedia.org/wiki/Software_development) as it can execute a single line of [code](https://en.wikipedia.org/wiki/Computer_code) at a time. An interpreter is also more portable than a compiler as it is [processor](https://en.wikipedia.org/wiki/Central_processing_unit)-independent, you can work between different [hardware](https://en.wikipedia.org/wiki/Computer_hardware) [architectures](https://en.wikipedia.org/wiki/Computer_architecture).

- **Assembler**:

> An [assembler](https://en.wikipedia.org/wiki/Assembler_(computing)) is a translator used to translate [assembly language](https://en.wikipedia.org/wiki/Assembly_language) into [machine language](https://en.wikipedia.org/wiki/Machine_language). It has the same function as a compiler for the assembly language but works like an interpreter.

**CS 61C**课上给出的`Translator`的定义更类似于对`Compiler`与`Assembler`这类的统称。

常见的**解释器**的例子有：

1. Python Interpreter
2. Simulator(e.g. Venus Simulator)
3. Emulator

关于第二种与第三种，实际上他们是在**软件中**`interpret machine code`，仿真器（`Simulator`）使用软件解释机器语言，在软件中模拟硬件行为。

而模拟器（`Emulator`）则类似于苹果公司在改变指令集架构(**ISA**)后对旧软件的处理方法–这也是**backward-compatible**策略的体现，它允许我们让一个系统(`host`)像另一个系统(`guest`)一样运行，这其实也是要先通过在软件层面**解释**旧的指令，因为我们无法在硬件上直接运行旧程序：

> Could require all programs to be re-translated from high level language. Instead, let executables contain old and/or new machine code, interpret old code in software if necessary (emulation)

**模拟器**的另一些例子比如我们使用的**DOSBox**或者街机游戏模拟器等等。

## Basic Compiling Process

> 以下内容来自于CS 61C：
>
> C/C++是一种**编译**型语言（`Compiler`），而非**解释**型语言（如`Python`）："C compilers map C programs directly into architecture-specific machine code (string of 1s and 0s".
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206214747874.png" alt="image-20211206214747874" style="zoom:50%;" />
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206215609119.png" alt="image-20211206215609119" style="zoom:50%;" />
>
> 而也正是基于编译型语言的原因，在编译阶段可以做许多优化：比如修改文件后编译器会指定(`Makefiles`)只编译更改后的文件，而不是做修改后要把项目里所有文件都编译一遍。这是编译型语言的一大优点之一。
>
> 但同时，编译型语言意味着编译生成的文件是`architecture-specific`的："Executable must be rebuilt on each new system". 另一方面，由于我们每次更改文件后都需要经过编译->运行的过程，这可能会拖慢我们的速度，这一点可使用`parallel compile`的方法抑制：`make -j`.

我们有许多编译器可以选择，比如`clang, g++`等等，如果我们想要编译的是`C`文件，那么可以使用`gcc`。编译过程包括：

1. 首先预处理，转换为`.i`文件，这一步需要展开所有的**宏定义**（**做简单的文本替换）、处理`#id/#endif`命令、预编译`#include`命令(insert xxx.h into output directly**)、**删除注释**(**replace with a single space**)等等，我们可以使用`-save-temps`来检查预编译文件。
2. 之后把文件内容进行**词法分析、语法分析**，转化为**汇编语言**，即`.s`文件；
3. 第三步为了让机器读取代码，还需要转化为**机器语言**，即二进制代码（也是所说的库）-`.o`文件；
4. 最后链接目标代码，生成**可执行文件**.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731111340.png" alt="image-20210731111340507" style="zoom:50%;" />

1. ```bash
   clang++ -E main.cpp
   ```

2. ```bash
   clang++ -S main.i
   ```

3. ```bash
   clang++ -c main.s
   ```

4. ```bash
   clang++ main.o -o main
   ```

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731112408.png" alt="image-20210731112408724" style="zoom:50%;" />

> 优化主要包含进行跳转和延迟退栈两种优化，`O~`选项将使编译速度减慢，但是通常产生的代码执行速度会更快

> `-Wall`表示打印出警告信息，如果要关闭所有警告信息，使用`-w`

### Concrete Compiling Process (CS 61C)

本章节记录了CS 61C课程中的编译流程部分。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223222358876.png" alt="image-20211223222358876" style="zoom: 50%;" />

#### Compiler

`Compiler`的输入是一种高级语言，将其翻译为汇编语言，需要注意的是，在转换过程中，会产生`Pseudo-instructions`.

一般来说，编译器的处理包括如下几步：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220102173849162.png" alt="image-20220102173849162" style="zoom:50%;" />

#### Assembler

`Assembler`以汇编语言为输入，最终形成机器码（其中包含了`information tables`）.

1. 首先，`Assembler`要阅读**Directives**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223224434360.png" alt="image-20211223224434360" style="zoom:50%;" />

需要注意的是，这些**Directive**并不会产生对应的机器码，它们只是相当于通知`Assembler`该如何做接下来发生的事情。

2. 其次，`Assembler`会替换先前在`Compiler`处理时产生的`Pseudo-instruction`.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223224705872.png" alt="image-20211223224705872" style="zoom:50%;" />

3. 现在，`Assembler`可以尝试将汇编代码转化为机器码了：

- Simple Case

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223224933860.png" alt="image-20211223224933860" style="zoom: 33%;" />

- Branches & Jumps?

对于分支和跳转，我们需要思考，他们能否直接生成完整的机器码？

首先，所有的**PC-Relative**寻址的指令都可以在这一步骤中生成完整的对应机器码。但是这里有一个`Forward Reference`的问题，就是跳转的地址位于我们当前位置的前方，也即代码还没有走到的位置，所以我们需要对汇编代码增加一次遍历：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223225437130.png" alt="image-20211223225437130" style="zoom:50%;" />

接下来，我们要面对**jal**，**jalr**这些指令，以及**lui**,**auipc**等。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223225956732.png" alt="image-20211223225956732" style="zoom:50%;" />

当**jal**指令想要跳转到<u>其他文件</u>中的位置时，我们无法给出完全对应的机器码，因为此时我们还没有进行**link**，不知道其余文件中的情况。

同时，我们也无法处理使用`absolute address`的指令，因为我们不知道这部分指令将被安排在内存中的哪些位置。

为了记录这些**当前无法处理**的部分，我们使用两个表：

| Symbol Table                                                 | Relocation Table                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存储了**当前文件**中可能被其他文件使用的东西，包括：**Label**: function calling; **Data**: .data section + variables which may be accessed across files | 存储了当前文件中需要提供**地址**的东西，包括：Any absolute label jumped to **jal**, **jalr** (e.g. External, Internal); Any piece of data in **static** section (此时我们也无法确定他们的地址) |

借由这两个表，我们可以在之后的**linker**步骤中根据他们的对应关系，补全真正对应的机器码表示。

> | Label | Address    |
> | ----- | ---------- |
> | sum   | 0x00061c00 |

3. 经过`Assembler`最终的表示:

按顺序：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223232510900.png" alt="image-20211223232510900" style="zoom:50%;" />

一种标准的表示方法（格式）是**ELF**，我们可以自己在Ubuntu下写一个C文件试一下，如果我们利用VS Code中的*hexdump*插件查看生成的机器码，就是按照这种格式制定的。

> 需要注意的是，这一步产生的机器码常被称为`object code`，在经过`linking`的步骤之后，变成`machine code`.

#### Linker

在拥有了许多个二进制.o文件后，我们想要把他们**链接**起来形成.out文件（`executable`）。而也正是这一步骤，允许我们在修改某个文件之后，不需要把所有的文件从头编译一遍。

根据在`Assembler`阶段我们构造的`symbol table`与`relocation table`，链接器将文件链接起来：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224000242605.png" alt="image-20211224000242605" style="zoom:50%;" />

- Take **text** segment from each .o file and put them together
- Take **data** segment from each .o file, put them together, and concatenate this onto end of text segments
- Resolve **references**：
  - Go through Relocation Table; handle each entry (fill in all absolute addresses)

##### Addresses

共存在四种类型的地址：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224000530667.png" alt="image-20211224000530667" style="zoom:50%;" />

那么哪些种类的地址需要在这一步被处理呢？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224000636125.png" alt="image-20211224000636125" style="zoom:50%;" />

**jal**指令被放在这里较为容易理解，虽然这种方法使用的是`PC-Relative`的寻址方法，但是只有我们将几个文件+所有的数据合并到一起，并安放到内存中时我们才知道所需要的`offset`是多少。而对于`static area`的`load & store`操作，我们需要注意的是，此时并不是我们先要给出汇编文件直接进行编译，而是我们提供了高级语言的源文件，机器负责将其翻译为机器码，那么如此一来，我们在源文件中使用的诸如`static x = 10`的语句，机器就必须要先定位`static area`，之后才能确定变量的地址。但是现在不知道他们的具体地址，所以在后边我们会看到，是采用了`placeholder`的方式生成汇编码与机器码，之后在**linker**这一步把**机器码**替换掉。

> 这里的`gp`是全局指针寄存器，他可以用来优化$$\pm 2kB$$的全局变量的访问：
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224001651003.png" alt="image-20211224001651003" style="zoom:50%;" />

而对于条件跳转类指令，由于它们采用的计算方法全部为`PC-Relative`，所以不需要额外处理地址问题。

##### How to resolve references?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224001846144.png" alt="image-20211224001846144" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224001929170.png" alt="image-20211224001929170" style="zoom:50%;" />

##### Static vs. Dynamic Linked Libraries

关于**静态链接库**与**动态链接库**的问题，我们在下边的C++部分也有提到，静态链接库将库作为可执行文件的一部分，而动态链接库则是通过在可执行文件中包含一条指向它的链接（`linking at the machine code level`，这是一种流行的做法，但并不唯一）实现的，我们会在程序运行时加载这些库，这也是**loader**需要完成的事情。动态链接的方法方便我们对库进行更新，而又不需要重新把原本的可执行文件的部分重新编译。但是动态链接库也有一些缺点，比如如果库文件损坏，我们的可执行文件即使没有发生任何改变也会导致程序无法正常运行，同时库文件的加载需要时间等等。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224002445046.png" alt="image-20211224002445046" style="zoom:50%;" />

#### Loader

`Loader`其实就是**操作系统**(`Loading is one of the OS tasks`)！此时我们已经开始要**运行文件**了！它以一个可执行文件作为输入，输出是一个正在运行的程序，具体的步骤如下:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224002837086.png" style="zoom:50%;" />

> 在一开始，需要把参数先拷贝到栈上，再拷贝到寄存器中，因为不知道参数的大小。

这里我产生了一点疑问，因为在之前`linker`的步骤已经把所有有关地址的问题处理好了，可是在装载(`loader`)的过程中需要将指令与数据从可执行文件中复制到新的地址空间里，这不会出现问题吗？这涉及到**地址重定位**问题，包括静态地址重定位与动态地址重定位方法，牵扯到**逻辑地址空间**与**物理地址空间**等，不做详述。但实质上，操作系统将**disk**中的文件复制到了**memory**中(DRAM)，这一点在之后的`cache`介绍部分会提到。

#### 关于变量的内存分配与初始化

通过上述章节我们了解到，在**编译阶段**(compile, assemble, linker)，程序需要确定下来**如何内存分配整个程序**，并在接下来的**加载阶段**给程序分配内存（当然之后还会有一些内存分配的时机，比如`malloc`之类的）。

这意味着某些时刻我们必须明确的在编译阶段告知程序内存分配的方法，一个例子就是**数组**：我们必须在编译阶段确定下来数组的维度，否则编译器不知道如何给这块数组分配内存。

另一方面，**变量初始化**也是一个很重要的主题。初始化可能发生在不同的阶段：

- **编译阶段初始化**-e.g. 全局变量为内置类型，且大小确定；或使用`constexpr`等

```c++
int a = 10;

int main() {
    
}
```

- **加载阶段初始化**-在`main()`函数执行前，完成包括**全局变量，静态变量**的初始化。e.g. 在加载阶段会调用类的构造函数初始化类的静态成员等(显而易见，编译阶段无法调用类的构造函数)
- **运行阶段初始化**-指代实际程序运行期间对象（变量）的创建，包含那些动态创建的对象。

```c++
int main() {
    int a = 10;
}
```

关于变量初始化，`static`关键字引发了多种不同的情况，这一点放在*CS 106L*的`static`关键字部分探讨。

#### An Example

```C
#include <stdio.h>
int main() {
    printf("Hello, %s\n", "world");
    return 0;
}
```

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224003618043.png" alt="image-20211224003618043" style="zoom:50%;" />

> 在`.align`中，给入的是`logX`的结果，这与`.balign`不同。同时我们要注意给出变量低位地址(`%lo`)与高位地址(`%hi`)的方法。

现在，我们来看一下在**linker**发生之前，汇编代码是怎么样处理那些需要链接的值的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224003821038.png" alt="image-20211224003821038" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224003842615.png" style="zoom:50%;" />

我们注意到，在经过**linker**的步骤后，所有原本由`00000`等`placeholder`占据的位置变成了实际的地址填充。

> 需要注意的是，指令**LUI/ADDI**会有立即数`sign-extend`的问题：
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211224003947682.png" alt="image-20211224003947682" style="zoom:50%;" />

至此，程序编译的全部流程便结束了.

### Libraries

- library是什么？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731112926.png" alt="image-20210731112926554" style="zoom:50%;" />

在编译时，如果将函数的定义也放在头文件内，将会导致运行变慢，因为没有使用linker，每次调用都会直接使用定义在其中的函数。

### link library

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731125102.png" alt="image-20210731125102493" style="zoom:50%;" />

出现该错误的原因是我们在主函数文件`program.cpp`中包含的是`tools.hpp`，也就是说程序完全不知道到哪里去寻找函数`SomeFunc()`的实现，他只知道存在这个函数（该函数的实现被我们放在`tools.cpp`中，但是我们没有告诉编译器这一点）。

所以我们要做的是：先让编译器生成一个`tools`库；之后要把`tools`和`program`链接起来，这样当`program.cpp`中找到`tools.h`时，也就知道其中函数`SomeFunc()`的实现被放在哪里了。

对于这种较为简单的情形，我们可以做如下操作：

```bash
# 生成tools.o库文件（第三步：编译）
g++ --std=c++17 -Wall -O0 tools.cpp -c

# 生成main.o库文件（第三步：编译）
g++ --std=c++17 -Wall -O0 main.cpp -c

#链接两者！
g++ --std=c++17 -Wall -O0 main.o tools.o -o main
```

在做链接时，需要注意**越是底层的库，被依赖的项，越往后边写**。

**当然，我们也可以在主程序直接包含`tools.cpp`，可以直接一步用`-o`搞定，不需要单独链接库了，但这不是一种好的方法**。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731131645.png" alt="image-20210731131645773" style="zoom:50%;" />

也就是说，在`linking`时，会**将函数声明定位到它的已经经过编译的实现上**。而为了完成这一点，我们需要的`library`需要包括两个部分：

- 一个包含声明的头文件
- 经过编译的二进制代码库函数实现

### How to build libraries

现在的情况是我们**自己定义了库文件**，并将库文件中的函数声明在头文件中：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731135146.png" alt="image-20210731135146844" style="zoom:50%;" />

> `-L`参数用来指定程序要链接的库，`-l`参数紧接着就是库名

首先将库文件编译为一个`object file`，这一步对应着C++文件编译过程中的**第三步**，**将文件转化为二进制代码**；之后将其**打包组合建立一个静态库**(`ar rcs`)，后边的`.`表示库所在位置，库里边是`binary code`，接下来把库**链接**到`main`文件里，即告知程序`function/symbol`的位置。

我们也可以用这种方法对上边的`program&tools`案例进行测试，结果是一样的，只不过我们先把两个`.o`文件打包成了静态链接库而已：

```bash
# 打包成静态链接库
ar rcs libtools.a main.o tools.o

# linking
g++ --std=c++17 main.cpp -L . -ltools -o main_1
```



### Build systems

发展历程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731140111.png" alt="image-20210731140111783" style="zoom:50%;" />

> Cmake是一个*MetaBuild System*，通过我们写好的CMakeLists.txt生成makefile，系统会根据makefile当中的指令去编译和链接程序

## cmake使用

cmake的教程可以参考[这里](https://github.com/ttroy50/cmake-examples).

cmake的官方文档写的有些过于晦涩，这个教程相对清晰明了，先掌握一些cmake的基本用法即可。

本节课作业中的内容对应着该GitHub教程的`01-Basic`和`02-sub-project`的部分内容。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731140139.png" alt="image-20210731140139832" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731140345.png" alt="image-20210731140345122" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/20210731140947.png" alt="image-20210731140947572" style="zoom:50%;" />

## Git的使用

**Git**的命令介绍可以参考[这里](https://www.runoob.com/git/git-basic-operations.html)。

在一个机器上安装好**Git**之后我们需要使用如下命令初始化这台机器：

```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

### 本地仓库

在学习**Git**过程中遇到的几个简易命令：

- 工作区、暂存区、版本库(`.git`)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111061825173.jpeg" alt="git-repo" style="zoom: 80%;" />

```bash
# 创建版本库(.git folder)
git init
```



1. `checkout`被替换

在**创建或者切换分支**时，使用`switch`命令代替：

```bash
# 切换分支
git switch newBranch

# 创建分支
git switch -c newBranch
```

在**恢复文件内容**时，使用`restore`命令代替,不论使用`–worktree`还是`stage`，我们指的都是**需要恢复的区域**：

```bash
# 撤销工作区(working tree)修改：如果暂存区有文件则恢复到暂存区的样子，如果没有，则恢复到上一次commit的样子，也即版本库master中的样子
git restore readme.txt

# 与上一个一样的效果，因为可省略worktree关键字
git restore --worktree readme.txt

# 撤销暂存区修改，从版本库master恢复暂存区(stage/index)
git restore --staged readme.txt

# 可以指定source关键字，从版本库master当前HEAD同时恢复工作区与暂存区内容
git restore --source=HEAD --staged --worktree readme.txt
```

2. `reset`用于**版本回退**+**撤销修改**

```bash
# 版本回退(已经提交到了本地的版本库master内)
# HEAD/HEAD^^/HEAD~100/commit-id(e.g. bavdc8743)
# 该命令把工作区（worktree）与暂存区（index/stage）的内容替换成指定的版本库master的内容，也即把他们回退到版本库的某个版本
git reset --hard HEAD^

# 使用reflog查看commit-id
git reflog

# unstage(撤销暂存区修改)，该命令撤销git add的在暂存区的内容
git reset HEAD readme.txt
```

根据上述代码，我们不难发现`--hard`主要用来撤销工作区的内容，当我们使用`--hard`后，工作区+暂存区的修改都会回退掉。

3. `diff`的使用

```bash
# 比较working tree & index/stage
git diff readme.txt

# 比较暂存区与版本库
git diff --cached readme.txt

# 比较工作区与版本库
# HEAD如果指向master分支，也可以换成master
# HEAD也可以写作某个commit-id
git diff HEAD readme.txt

# 同理，我们可以比较暂存区与某个指定的commit-id
git diff --cached [<commit-id>] readme.txt

# 或者比较两个commit-id的差异
git diff [<commit-id>] [<commit-id>]
```

此外，`git diff`还可以用来打补丁（`patch`），这里就先不列出具体方法了。

4. 其余一些基本命令

- `git init`
- `git add`
- `git commit -m`
- `git rm`(从**版本库**中删除文件)
- `git tag <tagname>`创建标签

### 远程仓库

#### push

1. 注意一开始要在自己**本地生成**`SSH Key`，这样`Github`才允许**把远程仓库和本地仓库关联起来并进行后续操作**
2. 关联本地与远程命令：`git remote add origin git@github.com:michaelliao/learngit.git`（本地有一个仓库，我们现在GitHub上创建了一个远程仓库，之后用此命令将其关联起来）

> 为什么是`origin`？这是**远程库的默认名字**，而非仓库本身的名字，对于不同的远程服务器上的同样的仓库，名字应当不同（`server-1`,`server-2`,`server-3`...），一般来说就关联一个远程库，习惯命名为`origin`。

3. 第一次推送：`git push -u origin master`

> ```bash
> git push <远程主机名> <本地分支名>:<远程分支名>
> ```
>
> 在这里，远程主机名为`origin`，遵循**来源地:目的地**，所以`git push`命令的顺序是**本地分支:远程分支**
>
> - 如果远程分支被省略，则表示将本地分支推送到与之存在**追踪关系**的远程分支:
>
> ```bash
> git push origin master
> ```
>
> - 如果本地分支被省略，则表示推送一个空分支，等同于`git push origin --delete master`，会将远程主机上的`master`分支删除：
>
> ```bash
> git push origin :master
> ```
>
> - 如果当前只有一个追踪分支，则可以简略为`git push`
> - 如果当前分支与多个主机存在追踪关系，则可以使用`-u`选项指定一个默认主机，这样后面就可以不加任何参数使用git push

4. 查看远程库信息：

```bash
git remote -v
```

5. 解除本地与远程的**绑定关系**：

```bash
git remote rm origin
```

#### clone

当我们试图从远程库克隆到本地时，可以使用`git clone`命令，该[命令](https://www.worldhello.net/gotgit/02-git-solo/100-git-clone.html)的用法有以下几种：

```bash
用法1: git clone <repository> <directory>
用法2: git clone --bare   <repository> <directory.git>
用法3: git clone --mirror <repository> <directory.git>
```

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202261702954.png" alt="image-20220226170228817" style="zoom:50%;" />

### 分支管理

参看廖雪峰老师的[教程](https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424)，这部分写的很清楚。

比如创建了一个分支并在此分支上提交了一次之后，分支结构就会变成下图所示的样子：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211205165634501.png" alt="image-20211205165634501" style="zoom:50%;" />

1. 注意我们可能需要处理**分支冲突**，解决方法也很简单，我们只要让某个分支领先于另一个分支即可，这样一来便会直接使用`fast-forward`模式进行合并
2. 同时，我们不应该做回退分支合并，即不能让一个最新的提交往回合并（这其实也是没有意义的）我们只能把以前的分支合并到我们新的分支上：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211205165850740.png" alt="image-20211205165850740" style="zoom: 33%;" />

在这之后，我们可以将`dev`分支删除，于是便只剩下`master`主分支了。

需要注意的是，在切换分支时，要把当前分支的内容`stash`或者`commit`，如果直接切换分支，那么仓库并不知道文件发生了什么改动，所以在切换到新分支时，`Git`会把在旧分支工作区中做的改动复制到新分支上。

1. 分支相关操作：

```bash
# 将dev分支合并到当前分支中
# fast-forward合并模式，将master指向dev分支的当前提交位置
git merge dev

# 查看分支结构
git branch

# 删除分支
git branch -d dev

# 强制删除分支（该分支可能还未被merge）
git branch -D dev

# 创建分支
git switch -c dev

# 进入分支, 此时会先查找本地是否存在此分支，如果不存在，则会查看远程仓库中是否有对应的分支，如果找到了，则会将远程仓库的对应分支拷贝到本地中 (MIT 6.S081)的多个分支结构就是这样设计和使用的
git switch dev

# 查看分支合并图
git log --graph --pretty=oneline --abbrev-commit

# 保留分支结构(普通模式)，合并
git merge --no-ff -m "merge with no-ff" dev
```

什么时候无法使用`fast-forward`模式合并呢？当我们在两个分支上分别有提交的时候：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211205170638223.png" alt="image-20211205170638223" style="zoom: 33%;" />

此时我们应当手动修改文件（文件中会显示提示内容）处理冲突，之后再进行分支合并。

> 通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。
>
> 如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

```bash
$ git log --graph --pretty=oneline --abbrev-commit
*   e1e9c68 (HEAD -> master) merge with no-ff
|\  
| * f52c633 (dev) add merge
|/  
*   cf810e4 conflict fixed
...
```

当我们使用**普通模式**进行合并时，代码结构如下：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111062303336.png" alt="git-no-ff-mode" style="zoom:80%;" />

4. **Bug**分支

引入一个新命令：`stash`，作用如下：

> Using the git stash command, developers can temporarily shelve changes made in the working directory. It allows them to quickly switch contexts when they are not quite ready to commit changes. And it allows them to more easily switch between branches.

```bash
# 在当前分支下，储存现场，使得git status为空，工作区是干净的
git stash

# 查看工作现场储存情况
git stash list

# 恢复工作现场，但不删除stash内容
git stash apply

# 删除stash内容
git stash drop

# 同时完成以上两项操作
git stash pop
```

需要注意的是，考虑到原本在`master`主分支中存在的bug也在`dev`分支中存在，所以我们需要同样修复`dev`中相同的问题，可以考虑在`dev`分支中**复制修复bug的对应操作的commit-id**，从而避免重复劳动。

```bash
git cherry-pick [<commit-id>]
```

> 在使用`cherry-pick`之前，我们需要先将之前`dev`分支中使用`stash`暂存起来的、还没有`commit`的内容提交，否则会提示错误：`error: Your local changes to the following files would be overwritten by merge`

#### 多人协作

[教程](https://www.liaoxuefeng.com/wiki/896043488029600/900375748016320)

- 创建本地对应的远程分支：

```bash
git switch -c dev origin/dev
```

因为当我们把远程仓库下载下来时，我们只能查看到`master`分支，要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，用这个命令创建本地`dev`分支。

- 多人协作推送流程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111070125878.png" alt="image-20211107012558802" style="zoom:50%;" />

这里，`git pull`的作用相当于`git fetch + git merge`，先把远程库的内容下载到版本库中，之后在本地将分支合并。

一张大佬制作的**Git**命令图：

[原图作者地址](https://www.processon.com/view/link/5c6e2755e4b03334b523ffc3)

![git思维导图](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111072200435.png)

### 其他一些教程

[100秒git介绍](https://www.youtube.com/watch?v=hwP7WQkmECE)

[Git官方文档-分支的新建与合并](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)

[一个更加完整的git视频教程](https://www.youtube.com/watch?v=gQSd2lFkZHk)


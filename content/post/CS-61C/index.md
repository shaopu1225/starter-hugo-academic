---
title:		CS 61C
subtitle:	CS 61C
summary:	notes of 61C: Great Ideas in Computer Architecture (Machine Structures)
date:		2021-12-05
lastmod:	2023-08-31
author:		shaopu
draft: 		false
image:		  
  focal_point: ''
  placement: 2
  preview_only: true

tags:
    - course
    - Computer Architecture

categories:
    - CS course notes
---

![image-20220212160706270](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212160706270.png)

![image-20220212160806760](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212160806760.png)

![image-20220212160821975](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212160821975.png)

![image-20220212160837018](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212160837018.png)

![image-20220212160949250](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212160949250.png)

![image-20220212161002000](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212161002000.png)

![image-20220212161033218](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212161033218.png)

## Number representation

1. **进制转换表**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206182043678.png" alt="image-20211206182043678" style="zoom:33%;" />

2. **Number representations**

共有五种表示方法，其中较为常用的，比较适用于计算机使用的方法有三种。

- `Unsigned`

这种方法即不带有符号的表示方法，是最简单的一种表示，当我们有五位可以用来表示数字的时候，无符号表示方法可以记录`0~31`的数字，但是接下来的有符号表示只能记录`-16~15`的区间了（因为最高位用以记录正负–没有免费午餐定理）。

- `Sign and Magnitude`

这种方法根据老师的描述，就是我们数电课程中学过的**原码**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206182536990.png" alt="image-20211206182536990" style="zoom:50%;" />

这种方法主要有两大缺点：

1. 零位重合
2. 正数与负数的计算方向是相反的（依上图）

- `One's Complement`

这种方法就是**反码**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206182709493.png" alt="image-20211206182709493" style="zoom:50%;" />

反码相比源码，虽然从负数到正数的计算方向保持了一致，但是仍然存在**零位重合**的问题。

- `Two's Complement`

这就是我们所说的**补码**了：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206182913504.png" alt="image-20211206182913504" style="zoom:50%;" />

根据上图我们可以看出，补码完美的解决了先前方法存在的问题，`11111`这里不再是`-0`，而是`-1`了。公式：**补码=反码+1.（正数的反码与补码均是其本身）**

- `Bias Encoding`

这种编码方式常用于将一个`unsigned representation`转化成`signed`的形式（比如要把一段在$$x$$轴上方的正弦波抬到中轴处），我们通过给入程序一个`bias`为$$-(2^{N-1}-1)$$的数值来辅助实现这一变换，在进行了$$unsigned+bias$$的操作之后，他最终形成的表示范围（以$$N=5$$为例）是`-15~16`(与补码不同)！也就是说，原本的`00000`现在变成了最小的那一个**负数**了：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206184425110.png" alt="image-20211206184425110" style="zoom:50%;" />



在上述的众多方法中，我们常使用的是`unsigned`,`two's complement`,`bias encoding`.

3. **overflow**

在计算的过程中，可能会发生**溢出**的情况，那到底什么时候才算做溢出呢？当我们**舍弃掉最高位进位后，如果发现运算结果是错的，那就说明确实是溢出**了，一般来讲，我们使用**双高位判定法**来判断是否发生溢出：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211206190532715.png" alt="image-20211206190532715" style="zoom:50%;" />

**当最高数据位向符号位进位不等于符号位进位时，即发生溢出！**在硬件电路的实现中，我们可以通过**异或**来实现这一点。

> 这种判定方法是针对**有符号**数字而言，如果我们面对的是**无符号**数字，那么只要最高位进位1，就发生溢出。

## C Programming

### Pointing to Different Size Objects

#### 32？64？

首先我们要有**存储单元**的概念，在现代的微型存储器中，一个**存储单元**是8个`bit`，也就是一个`byte`，换句话说：这些机器是`byte-addressable`的 – 微型存储器的容量是以`byte`为最小单位计算的。

需要注意的是，当我们谈论*32位机器*或者*64位机器*（`cpu`）时，我们所指的是它**一次能够处理的数据的长度**，也就是**寄存器的位数**，也被叫做**字长(`word length`)**，它和我们所说的**数据总线**，**地址总线**等没有*直接*关系。

**数据总线**的长度一般要**等于**`cpu`的**字长**，这是为了保证`cpu`的数据处理能力得到充分利用，所以我们可以说字长由[微处理器](https://baike.baidu.com/item/微处理器)对外数据通路的[数据总线](https://baike.baidu.com/item/数据总线)条数决定。同时，由于**指针**也是数据，所以地址在进行数据传送时会被匹配到同样的位数，但这不代表**地址总线**长度要和**字长**或者**数据总线**的长度一样，**地址总线**的长度由`cpu`自身设计决定，与`cpu`的位数没有直接关系，但由于**地址总线**决定了`cpu`的寻址能力，所以**地址总线**所能够支持的寻址一定要大于等于**数据总线**的位数，不论通过什么方法来实现这一要求。

总得来说，内存的大小是由**硬件**以及**操作系统**共同决定的，硬件对应着地址总线，而操作系统对应了我们在后边将会提到的虚拟内存。

#### word alignment

在实际使用时，我们常常会遇到**内存对齐**的说法，什么是内存对齐？为什么我们需要这样做？

比如对于一个32位的处理器，它一次能够处理数据的能力为`32 bits`，换算成内存单元就是`4 bytes`。`cpu`正是按照这个`4 bytes`的**块(`chunk`)**来读写内存的，块的大小被我们定义为**内存访问粒度**。

我们可以把内存想成一个无穷大的`array`，寻址从`0`位开始，我们的数据存储也正是从这个地址开始，如果**内存访问粒度**是`4 bytes`，那么`0`,`4`,`8`等等都是`aligned address`。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211207141611819.png" alt="image-20211207141611819" style="zoom:50%;" />

那如果我们的数据没有做到`word alignment`会发生什么后果呢？对于某些处理器而言，这会让它们进行多次的内存访存（**速度变慢**），分别访问每一个**内存单元**，读取其中需要的数据，最后再把多个单元读取到的数据进行`merge`，放到**寄存器**中。而对于某些`cpu`，则不会支持**非内存对齐**的数据存储形式，会直接产生报错。

根据[这里的文章](https://zhuanlan.zhihu.com/p/83449008)，在每一个**chip**内，都有多个**bank**，每一个**bank**就是一个二维平面上的矩阵，矩阵中的每一个元素就是一个字节。实际上，**bank**是按照如下的排列方式存在的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202071413361.png" alt="image-20220207141302223" style="zoom:50%;" />

> 这种设计允许在读写数据时几个地址并行工作，提升电路效率。所以如果我们不按照内存对其的方式存储，每次读取数据时，电路操作的次数就要更多，

#### some notice on pointers

这里还是记录一下关于**指针**的使用事项。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211207200933346.png" alt="image-20211207200933346" style="zoom:50%;" />

上面的这幅图很好的包含了我们需要注意的关键点。首先我们可以看到：指针与整型均占4个`bytes`，之后我们声明了一个包含4个元素的整型数组，所以占据了28、32、36、40总共4个**内存单元**。

而在此之后，我们调用`malloc`函数，分配了一段内存，并让先前定义的指针`p`指向了这块我们刚刚分配的内存，但此时，这**段新分配的内存里边的值还是垃圾值**。相应的，指针`p`的值就变成了指向的那个地址，也就是40了！

> 我们一定要**先分配内存再使用内存**，如果在定义指针后没有使用`malloc`分配内存就直接`*p=1`，是不可以的。

最后，我们注意到对于数组`a`，`a`与`&a`的值是一样的，**不同于先前我们所定义的`变量`的表现！**这一点我们在*CS 106L*的课程笔记中也有提过，该课程的汇编部分会再次提到这一话题。

> 需要注意的是，数组名称**不可以被重新赋值**（`reassigned`），所以我们也不能对其进行`++a`的操作。这一点也是与指针有很大差异的一点.

- 此外，我们还要注意，在实际编程中，我们可能经常需要**向一个函数传递数组**，如果我们想在函数内部利用`sizeof`操作符获取数组大小，那么由于传入函数的是一个指针，我们只会得到指针的大小而非实际的数组大小–此时我们必须要向函数传递一个额外的`size`参数；当然，如果我们在处理一个**字符数组**，我们完全可以利用其`null terminator`的特征来定位整个数组。
- 在**C**中使用字符串（字符数组）时，末尾的`\0`在**ASCII**码中对应了**NUL**，即**ASCII**码中的**0**！所以我们在判定字符串是否到达末尾时，可以直接将`*str++*`作为判定条件，而无需和`\0`比较。我们需要注意，当使用字符数组来储存字符形成字符串时，我们需要手动添加一个`\0`，但是在使用`string literals`会自动添加，不需要我们手动操作.

### Memory locations

首先关于**声明**:

> - **Structure** declaration **does not** allocate memory.
> - **Variable** declaration **does** allocate memory.

在先前的**C++**课程中，其实也简短的[提到过](https://shaopu.tech/2021/11/07/CS%E5%9F%BA%E7%A1%80%E8%AF%BE%E7%A8%8B4/#globals--statics)，变量的生命周期（`storage duration`）有三种，我们形容为`static`,`automatic`,`dynamic`.分别对应着静态、全局变量；局部变量；动态分配内存的变量.

相应的，这三种不同的变量也被存储在不同类型的**内存池**中：

- **Static storage**:储存全局变量，生命周期为整个程序运行的时间
- **The Stack**: 储存**局部变量**、**函数参数**、**返回地址**
- **The heap**: 使用`malloc`动态分配的数据，直到`free`生命周期结束（这里的生命周期结束其实并非变量消失，后便会提到）

#### Memory management

在一个程序中，共有四种类型的**地址空间**，除了在上一小节中提到的三种内存池之外，还包括`code`段，他们的分布形式如下图所示：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211208000712193.png" alt="image-20211208000712193" style="zoom: 33%;" />

**从上至下**，**地址空间从最高一直到0**；与之相对应的，分别用以存储`stack`,`heap`,`static data`以及`code`:

- 当我们向`stack`中存入数据时，它会向下分配新的内存，而`heap`则是向上分配内存（我们不需要在这里考虑两个内存池发生重叠的问题，*CS 162*会介绍处理方法）.
- `static data`与`code`均是在程序运行过程中**不会发生改变**的内存区域，`code`主要用以存储运行需要的代码，在程序一开始即生成。

##### Stack

在上边我们提到，`stack`用于存储以下三种类型的数据：

1. `Return "instruction" address`
2. `Parameters`
3. `Space for other local variables`

`stack`区的内存分配是**连续**的。他有一个**栈指针**(`stack pointer`)，指向当前**栈顶**的位置（随着新开辟的栈区域不断地向下改变位置）：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211208001902974.png" alt="image-20211208001902974" style="zoom:50%;" />

当我们结束了一个`procedure`时，栈内存被自动"释放"，之所以这里加了引号，是因为栈内存释放的方式与我们常理解的释放不同，**它在释放的过程中将栈指针不断上移到上一个还没有被释放的栈块的栈顶，而先前存放在已经被"释放"额区域内的数据，其实仍然保留在其中！**

栈内存的分配与释放都是**很快**的！

##### Heap

与栈不同，堆的内存分配**不是连续**的。两段看起来应当被分配在前后未知的动态内存，很可能彼此离得很远。相应地，**堆的内存分配和释放过程相比起栈是很慢的！**在使用堆时，我们想尽量避免碎片(`fragmentation`)化的出现：

> In this case, we might have many free bytes but not be able to satisfy a large request since the free bytes are not contiguous in memory.

那么堆内存的分配和释放等等过程究竟是如何实现的呢？

每一块堆内存的头部都有这样两组数据：

1. 该块的大小(`size of the block`)
2. 指向下一个块的指针(`a pointer to the next block`)–有点像指针

所有的**空内存块**被保存在一个**循环链表**中！（在一块已经被分配走的内存中，上述的`pointer field`是处于不被使用的状态）。

而函数`malloc`便会从这个**循环链表**中替我们寻找可用的内存，寻找的方法有这样几种：

1. `best-fit`:在循环链表中**选择满足大小要求的最小的内存块**
2. `first-fit`:**选择第一个满足大小要求的内存块**
3. `next-fit`:进行`first-fit`，但是记忆上一次搜索完的位置，下一次从此处继续搜索可用内存

而相对应地，`free`函数在将指针指向的内存块释放后会**检查前后的内存区域是否也是空内存块**，

- 如果是，则将几段内存`merge`;
- 如果不是，则将刚**被释放的内存加入循环链表中**

#### When memory goes bad

在课程中，提到了几种应当避免出现的内存分配和释放问题，这里说几个：

- <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211208004619281.png" alt="image-20211208004619281" style="zoom:50%;" />

这个情况其实很`tricky`，观察结果我们发现，第一次使用`content`和第二次的结果不一样，why？

注意到这里**我们把指向栈内数据的指针作为了函数返回值**，当栈内存被释放后，根据前边的描述，其中的数据其实并没有被抹掉，所以我们仍能够通过返回的指针`stackAddr`获取原先的数据，所以第一个`printf`没有问题。

但是`printf`也是一个函数，也被分配到了**栈**上，由于栈内存是连续分配的，他恰好占用了原本数据`y`的位置，导致`y`被毁掉了–这就导致第二次我们再次尝试获取`y`的数据时，指针把我们带到了垃圾值上！

- `Realloc`

`Realloc`函数也是可能诱发内存分配问题的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211208005401436.png" alt="image-20211208005401436" style="zoom:50%;" />

在使用`realloc`时，如果程序发现接下来的空**堆内存块不够扩展到指定大小，他会将原来的数据一起搬运到新的能够存放整体大小的内存块上**！这就导致我们原本指向`foo`的指针`g`现在不可用了。

- `calloc`

除了`malloc`和`realloc`外，我们还有一个`calloc`函数，可以避免我们忘记了初始化刚分配的内存空间的问题：

```c
// define a n*m matrix of int
int **mat = (int **)calloc(n, sizeof(int *));
for (int i = 0; i < n; ++i) {
    mat[i] = (int *)calloc(m, sizeof(int));
}
```

需要注意的是，该函数可以直接给出初始化的**空间+初始化值**–**0**（真正意义上的**0**，比如*NULL*或者数值的0之类的…）



几种会出现`segfault`的情况：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211213235209723.png" alt="image-20211213235209723" style="zoom: 33%;" />

#### some notice on memory menagement

- 对于程序中定义的**常数(`constants`)**,它可能存放的位置有：

1. Code段：`x=x+1`,其中`1`在编译阶段直接被存储到`machine instruction`中，以及`#define y 5`时
2. static段：全局变量：`const int x = 1`中的`1`
3. stack段：将常量定义在函数中：`int total = 1`

- 此外，`string literals`被存放在`static`段中，具体可见[这里](https://shaopu.tech/2021/11/07/CS%E5%9F%BA%E7%A1%80%E8%AF%BE%E7%A8%8B4/#storage-duration).

- 与**C++**不同的是，**C**中没有*引用*的说法，所以当我们在对一些数据结构比如链表进行内存的分配和释放时，我们不能够传递指针的引用，需要传递指针的指针.

### Stream

这里主要说一下关于**C**中的**流**这一概念，以及之前我一直没懂的`FILE`这个对象。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211210194228562.png" alt="image-20211210194228562" style="zoom: 33%;" />

其中，`stdin`是输入**流**，因为在`linux`中一切皆文件，所以他也是标准**输入文件**。同理，`stdout`是输出**流**–标准**输出文件**。

我们也可以这样理解：**linux**通过**文件**的形式实现了输入/输出流，文件们分别对应着不同的**文件描述符**，每当我们打开`文件`（也是**广义**的！）时，就有一个代表着该打开文件的**文件描述符**，程序启动时默认打开三个`I/O`设备文件：标准输入文件`stdin`，标准输出文件`stdout`，标准错误输出文件`stderr`，分别得到文件描述符 `0`, `1`, `2`:

> **[文件描述符](https://segmentfault.com/a/1190000009724931)**在形式上是一个非负整数。实际上，它是一个**索引值**，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开.

在*Stanford CS 106L*最开始的笔记章节中，我们对`stream`对象做了很多的分析，有了初步的了解。`stdin`默认从键盘接收输入，`stdout`默认将内容输出到屏幕上，但这不是它们唯一可行的路径和流入（出）方式，比如读入/写入文件当中，我们可以使用`redirection`（[重定向](http://c.biancheng.net/view/942.html)）的方法改变其输入/输出的对象.

> `</>`重定向在使用`gdb`调试时也可使用

需要注意的是，我们经常使用的`FILE`和我们日常所说的狭义的*文件*没有任何关系！根据**C++**官方文档：

> **Object containing information to control a stream**
>
> Object type that **identifies a stream and contains the information needed to control it**, including a pointer to its buffer, its position indicator and all its state indicators.

换句话说，他可以用来表示一个`stream`对象。这也是为什么我们在函数`fgetc(FILE *stream)`中，可以使用`fgetc(stdin)`的原因。

### Bit operation

在*Fa 2021 lab 02*中，我们需要解决三个位操作的函数，查看[Github仓库](https://github.com/SongShaopu1998/Berkeley-CS-61C/blob/main/labs/fa21-lab-starter-main/lab02/bit_ops.c)。分别为：

- 获取某位
- 改变某位
- 翻转(`flip`)某位

在后两种操作中，我们需要注意的是使用一个所有位均为`1`的参考值辅助操作，同时将原本的二进制数分为前后两个部分加以处理（`flip`）可以将需要`flip`的位单独拿出来，最后使用`|`合并.

## Floating Point

这一节来处理浮点数.

### Basic Representation

首先，我们如何把一个二进制浮点数计算成我们习惯的十进制的样子？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211175738895.png" alt="image-20211211175738895" style="zoom:50%;" />

为什么叫`floating point`?这是因为在一开始，我们思考的储存方式是**固定小数点的位置**，这种方法被称为`fixed point`，而与之相对应地`floating point`则是允许小数点在数字之间移动–这种表示模式允许我们记录**更大范围的数字**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211182809936.png" alt="image-20211211182809936" style="zoom:50%;" />

为了表示一个`floating point`，我们的基本思路是用一段`bits`表示**需要记录的数字部分**，再用另一段`bits`表示**小数点所在的位置**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211182739063.png" alt="image-20211211182739063" style="zoom:50%;" />

按照如上的思想，我们需要将浮点数在内存中用如下方式表示：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211182857518.png" alt="image-20211211182857518" style="zoom:50%;" />

`float`共使用`32`bits记录，其中，浮点数本身的最前边的`1`我们**不在这里单独记录**–因为除了`0`，其他的任何一个浮点数按照**科学计数法（`Normalization`）**的模式，都需要有一个`1`在前边。（后边我们会看到，即使是`0`，我们也有办法在不使用这个开头的数字的前提下表示出来！）

`float`(*Single Precision*)的储存形式整体上分为三个部分：

1. `Sign bit`-1 bit
2. `Exponent bits`-8 bits(对于`double`为`11 bits`)
3. `Signtificand bits`-23 bits(对于`double`(*Double Precision*),为`52 bits`)

我们可以先对这种表示模式进行一下思考：其中的`Exponent`部分应当使用一种类似于`Two's complement`的表示方法，因为我们需要最终表示形式的指数部分**可正可负**。`Significand`部分则是直接存储了二进制浮点数小数点后的二进制数字序列。

但是使用`Two's Complement`的一个问题在于，当机器上不存在**用于浮点数比较的硬件（如早期的计算机）时，我们无法直接通过整数比较的方法来比较浮点数**，并且，即使存在用于浮点数比较的硬件，浮点数比较的过程也是远远慢于整数比较的！

当我们使用`Two's Complement`来表示`Exponent`时，当`Ex`**所记录的整数**越大，我们并不能够得出对应的`float`越大的结论（在其他部分相同的前提下）– 当`Ex`从`00...0`到`11...1`，实际的`Exponent`值，在此时也可以用于表示实际的浮点数值会从`0~+MAX`，再到`-MAX~0`变化！

于是我们想到了`Bias Encoding/Notation`!在这种表示模式下，`Ex`所代表的整数的变化趋势与实际值的变化趋势是完全一致的，所以对于`float`，我们使用`bias=-127`，相对应地，`double`中`bias=-1023`. 

这样一来，我们的比较过程可以设计为：

1. `sort the sign field by just +/-`
2. `sort by more significant exponent`
3. `If Exponent the same, using mantissa sorting`

于是，以`float32`为例，最终我们的计算公式为：



$$(-1)^S\times (1+significand)\times 2^{(Exponent - 127)}$$



同时，显然，这种表示方法对于数字范围是存在限制的，这意味着我们会遇到`overflow`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211184944361.png" alt="image-20211211184944361" style="zoom:50%;" />

### Special Numbers

#### ∞

在浮点数的表示方法中，我们可以表示一些特殊数字，比如首先我们会想到表示正/负无穷。那么**IEEE 754**是如何表示∞的呢？

- **Most positive exponent** reserved for ∞
- **Significands all zeros**

也就是说，当我们从`1111110|11...1`再加一个1，就变成`∞`了！如果`sign bit`为1，那么就是`-∞`.

#### 0

其实严格意义上来讲，0并不能算作`special number`，因为他的表示与一般的方法是一样的，需要注意的是，在浮点数中，有一个`+0`，也有一个`-0`，从数学意义上来说，这两个0并非真正的0，而是指的从两个方向无限趋近于真正的**0**。他们都是合理的：



**+0: 0 00000000 00000000000000000000000**
**-0: 1 00000000 00000000000000000000000**

如此一来，我们用`significand`和`Exponent`均为`0`的方法来表示数字`0`（或者说数字0和这种表示方法是等价的）–>这允许我们不使用整数部分的数字，也能够表示0了.

所以我们在偌大的浮点数数字范围中，还有哪些区域的数字表示没有被使用？我们还需要表示那些特殊数字？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211202701559.png" alt="image-20211211202701559" style="zoom:50%;" />

根据上图，我们还有两个数字区域没有使用，分别是`Exponent`为0以及255，且`significand`(小数部分)不为0的情况.

> 为什么我们把正常数字的`Exponent`范围限制到`1-254`?这是因为我们刻意为`special numbers`保留了范围.

#### NaN

我们使用`Exponent`为**255**，`Significand`**不为0**的情况，来表示所谓的**NaN**(**Not-a-Number**).也即是说，当我们从`inf`再往前走一步，就回到了`Nan sea`.

那么NaN有什么用呢？

- 可以提供`debug`讯息
- op(NaN, X)=NaN

需要注意的是，NaN与任何东西比较的结果都是`false`，并且`NaN != NaN`.

#### Denorms

首先，什么是`normalization`? 通俗地讲，`normalization`就是将一个数字表示成科学计数法的`1`打头的表示，那么如果我们想把开头的数字搞成`0`而非`1`，这就叫`denorm`.

对于浮点数，我们当然希望它的**尺度**是均衡的，换句话说，我们希望在一段范围内，浮点数之间的距离是恒定的.

我们来计算一下现在的情况是否满足这一条款：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211215921254.png" alt="image-20211211215921254" style="zoom:50%;" />

显然，从0到a的距离…实在是相比起来有点太大了…

所以我们怎么解决这个巨大的`gap`的问题？奔着制造一个尺度均衡分布的数字范围的目标，我们利用还没有被分配任务的`Exponent`为0，且`significand`不为0的范围，提出`DEnormalizaed number`:

**No leading 1, implicit exponent = -126 (rather than -127)**

如此一来：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211221234402.png" alt="image-20211211221234402" style="zoom:50%;" />

需要注意的是，随着`Exponent`部分的变化，我们在`significand`的每一段的尺度也会发生变化，在最开始时每一段的间隔为$$2^{-149}$$，当`Exponent`加1之后，`significand`的间隔会变为$$2^{-149}\times 2$$. 换句话说：

> Exponent tells Significand how much (2^i) to count by (..., 1/4, 1/2, 1, 2, ...).

当存储内部的`Exponent`为150时，`significand`的间隔会变为1（$$2^{(150-127)}\times 2^{-23}=1$$）



最终，我们得到了完整的浮点数表示：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211221437771.png" alt="image-20211211221437771" style="zoom:50%;" />

> 当我们试图从`00...0`一直加到`11...1`时，浮点数的大小变化范围和我们先前提到的`Sign and magnitude`（**原码**）一致！

#### Attributes

首先，浮点数运算不具备**关联性**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211230807221.png" alt="image-20211211230807221" style="zoom:50%;" />

其次，正如我们在写程序中经常遇到的，浮点数存在`rounding`这一机制，通常，浮点数硬件带有额外的两个`bit`，之后使用`rounding`机制将浮点数表示为合适的值。`rounding`发生在将`double`转为`float`时，也发生在将`float`转为`int`时：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211232700680.png" alt="image-20211211232700680" style="zoom:50%;" />

需要注意的是，当浮点数恰好位于`rounding`的分界线上时，我们将浮点数`rounding`到**偶数**上。

再者，浮点数的**加法运算**并不像整数的加法运算那样简单，我们可能先需要将浮点数`denormalize`，以匹配其指数部分，在完成`significands`的加法运算后，再将结果`normalize`.

我们可以使用**强制转换**机制，这一机制在内部通过`rounding`来实现。需要注意的是，由于浮点数尺度的问题，它并不能表示某些整数！所以：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211233056007.png" alt="image-20211211233056007" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211211233254194.png" alt="image-20211211233254194" style="zoom:50%;" />



除了上述的`float`以及`double`之外，还有其他一些表示方式：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211212000057633.png" alt="image-20211212000057633" style="zoom:50%;" />

特别的是，由于机器学习并不需要太高的精度要求，`bfloat16`类型被使用在ML中，它牺牲了一部分`significand`空间以换取更大的范围（`Exponent`）.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211212000317259.png" alt="image-20211212000317259" style="zoom:50%;" />

特别的是，有一种观点提出，我们可以对`float point`做进一步优化，使得`Exponent`和`Significant`的长度都可以变化，以适应不同的场景。同时，携带一个`u-bit`来表示这个浮点数是否经过了`rounding`.换而言之，他是一个准确的值还是一个估计值。

## RISC-V Instructions

汇编语言的操作数(`operand`)是**寄存器(`register`)**!

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211214232809988.png" alt="image-20211214232809988" style="zoom:50%;" />

在**RISC-V**中，共有**32**个寄存器，其中`x0`是最特殊的，他始终保有`0`值！这意味着我们只允许使用其他31个寄存器来存储变量！

关于**RISC-V**的几个基本的操作，如`add`,`addi`,`sub`这里不详述.

### storing data in memory

我们首先要知道CPU与内存交互的两种基本模式：

1. **Write** Data – Store **To Memory**
2. **Read** Data – Load **from Memory**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216122249838.png" alt="image-20211216122249838" style="zoom:50%;" />

其次，在`word alignment`的一节我们已经提到过，一个`8bit`的块被称为`byte`，是一个微型存储单元（`byte-addressable`）。字(`word`)在`32`位CPU中，是`4 bytes`，内存读取按照`word`读取（**内存访问粒度**）……(参见原章节)

那么每一个`word`在内存中是以什么样的格式存储的呢？在这里，我们的存储方式为**Little-endian convention**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216123319753.png" alt="image-20211216123319753" style="zoom: 33%;" />

> Word address is same as address of rightmost byte – least-significant byte.

也就是说，`word`中最低的那一个`byte`被安排在了最小的地址上。与之相对的是**Big-endian convention**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216124545397.png" alt="image-20211216124545397" style="zoom:50%;" />

需要注意的是，无论是哪一种`bytes`的存储方式，其中`bits`的存储方式是不变的.

### Data Transfer Instructions

在**RISC-V**中，一个寄存器为4bytes，共有32个寄存器，也就是128bytes。而相对应的`Memory`有2GB-64GB，这也意味着寄存器一定要比内存（`DRAM`）更快，大概是它的50~500倍。所以我们更想让指令(`Instructions`)在寄存器中被操作，而不是直接在内存中操作–这就需要内存与寄存器的双向数据交互。

- <u>Load word from</u> Memory to Register

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216161325926.png" alt="image-20211216161325926" style="zoom: 33%;" />

**lw**这个语句的含义是将储存在`x15`中的`base pointer`的值加上`12bytes`的地址里储存的内容，即`A[3]`复制到x10中. 与之相对应地，是从寄存器向内存中储存内容的**sw**：

- <u>Store word from</u> Register to Memory

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216161736950.png" alt="image-20211216161736950" style="zoom:33%;" />

除了对于`word`存取的操作指令外，我们还可以直接对`Bytes`进行操作：

- Loading and Storing Bytes

分别于**lw**,**sw**相对应地：

- **lb**
- **sb**

E.g.

```assembly
lb x10,3(x11)
```

该指令将指针指向的内容**复制**到寄存器`x10`的`low byte position`.

#### Sign-extend

**sign-extend**的意义在于保证一个有符号数字(可能不到32bits)在被储存进寄存器(32bits)时仍然保证是一个有符号数字：

> 如果在前边所有的高位里都存储**0**，那么对于这个寄存器内的32位数字来说，就始终是一个`positive number`了.

所以，在**lb**中采用了如下方法：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216164345305.png" alt="image-20211216164345305" style="zoom:50%;" />

需要注意的是，在**sb**中没有`sign-extend`的说法，我们只需要将寄存器中的内容取出来就好，`sign-extend`在**sb**中没有意义。

> RISC-V也有一种储存`unsigned byte`的方法：**lbu**，顾名思义，是使用`zero extend`的方法来填充寄存器.

#### Why need addi?

我们发现，这两种写法具有相同的效果：

```assembly
// first
lw x10,12(x5)
add x12,x12,x10

// second
addi x12, value
```

那为什么我们需要`addi`这个指令呢？（RISC-V提出了最小指令集的要求）这是因为第一种写法要求从`memory`加载数据，这一操作相对较慢，而直接使用`immediate value`可以利用寄存器序列中的`temporary`(专门用于存储临时值的寄存器)来实现运算.

### Decision Making

什么是`Decision Making`?就是我们在高级语言中常用的`if`判断等.

#### Conditional Branch

1. `beq reg1, reg2, L1`

> L1 branch if reg1 == reg2

2. `bne reg1, reg2, L1`

> L1 branch if reg1 != reg2

3. `bge reg1, reg2, L1`

> L1 branch if reg1 >= reg2

4. `blt reg1, reg2, L1`

> L1 branch if reg1 < reg2

5. **unsigned version**–`bltu`, `bgeu`

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216173548887.png" alt="image-20211216173548887" style="zoom:50%;" />

#### Unconditional branch

- `j Label`(`jump Label`)

#### Loop

书写循环的关键在于使用**Conditional branch**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216173706372.png" alt="image-20211216173706372" style="zoom:50%;" />

### Logical instructions

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216182826110.png" alt="image-20211216182826110" style="zoom:50%;" />

- and-andi(`andi x5, x6, 3`)
- sll-slli(`slli x11, x12, 2`)
- srl-srli(`Right shifting`)
- …

- 不存在**NOT**这一指令的，因为我们可以使用`xor 1111111`。
- **Arithmetic Shifting**

又名**Shift right arithmetic**: `sra-srai`，应用于**signed numbers**。这两条指令使用**sign bit**填充右移需要的位。这一操作看起来像是将原本的数除以2，但事实并非如此。这是因为**C语言要求`round towards zero`**.

### A bit about Machine Program

我们的高级语言程序在经过处理之后成为汇编指令(Instructions)，储存在`memory`中：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216190734553.png" alt="image-20211216190734553" style="zoom:50%;" />

那么程序是如何被执行的呢？

从宏观上讲，当然是通过CPU与内存的交互–CPU从内存中读取指令+数据，之后将数据写入内存。

进一步分析，是CPU的**Control Unit**（控制器）读取指令，控制CPU中**Datapath**（运算器）里的寄存器们，并由CPU内部的**PC(Program Counter)**寄存器记录**需要执行的指令的`byte address`**（update PC）实现的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216191436104.png" alt="image-20211216191436104" style="zoom:50%;" />

### Helpful Assembler Features

1. Symbolic register names

- **a0-a7 (x10-x17)**: eight *<u>argument</u>* registers to pass parameters & two *<u>return values</u>* (**a0-a1**)
- **ra (x1)**: one *<u>return address</u>* register to return to the point of origin
- **s0-s1(x8-x9), s2-s11(x18-x27)**: saved registers
- …(More)

2. Pseudo-instructions

- **mv rd, rs(addi rd, rs, 0)**(actually copy)
- **li rd, 13(addi rd, x0, 13)**
- **nop(addi x0, x0, 0)**

### Function calls (single/nested)

在RISC-V中的函数调用需要经过哪些步骤？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216202401580.png" alt="image-20211216202401580" style="zoom:50%;" />

需要注意的是，对于RISC-V，所有的指令都是`4 bytes`，被存储在`memory`中，就如同**data**一样.

我们以一段程序为例，展示这一过程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216202608172.png" alt="image-20211216202608172" style="zoom:50%;" />

在函数调用之前，我们先要将被储存在`saved register`中的两个变量a, b放入用于储存函数参数的寄存器**a0**与**a1**中。（如果此时这几个寄存器的容量不足以存储函数参数，那么需要借助`stack`，这个后边会提到）。在函数执行时，我们把结果（`return value`）放到寄存器**a0**中。

之后我们需要把函数调用结束后需要返回的位置存储在寄存器**ra**中(**link**)，再跳转到函数的位置执行函数代码（相当于把控制权交给函数）(**jump**)。

当函数调用结束后，我们通过`jr ra`的指令返回储存的需要跳转回的位置。之所以不在这里使用`j label`的指令，是因为**函数可能在程序中被多次在不同位置调用，所以我们需要把跳转回的位置储存在一个变量中**（该语句也是函数代码的一部分）

#### jump in Function calls

在上述函数调用的过程中，我们首先将`return address`储存在寄存器**ra**中，之后使用**j**指令跳转，这两条指令可以合并为一句（**jump and link**）:

```assembly
jal sum # ra=1012, goto sum
```

> 这也是一个pseudo-instruction，真正的指令为**jump-and-link**：
>
> ```assembly
> jal x1, sum
> ```

与之类似的，**jr**也是一个`pseudo-instruction`，对应的`base-instruction`为**jump-and-link register**：

```assembly
jalr rd, rs, imm
```

指令**jr ra**还可以缩写为**ret**。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216214513552.png" alt="image-20211216214513552" style="zoom:50%;" />

#### Nested Calls & Register Conventions

在写出具体的解决方案之前，我们先来思考如下的案例：

```C
int leaf(int g, int h, int i, int j) {
    int f;
    f = (g + h) - (i + j);
    return f;
}
```

我们需要用到寄存器**a0, a1, a2, a3**来储存函数参数与函数返回值，同时**s0, s1**这两个`saved register`被用来存储我们的`local variables`.

那么问题来了，既然我们需要用到这么多寄存器？那原本存储在寄存器中的值怎么办？直接毁掉（`clobber`）吗？当然不是，我们需要保存原本储存在寄存器中的值–借助`stack`来实现：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216220928653.png" alt="image-20211216220928653" style="zoom:50%;" />

在有的汇编语言指令集中，是有**push**和**pop**这两个指令的，但是RISC-V没有，所以需要我们自己去实现它。

复习一下先前关于**stack**的知识：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216221114317.png" alt="image-20211216221114317" style="zoom:50%;" />

举个例子，对于函数`leaf`，编译后是这样:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216221340465.png" alt="image-20211216221340465" style="zoom:50%;" />

可以看到，在函数最开始我们调整栈顶指针的位置，开辟新的栈空间，同时将寄存器**s0**,**s1**存入刚开辟的栈空间内。在程序结束之后，恢复寄存器的内容，并释放栈空间。这前后两段操作分别被称为**prologue**与**epilogue**。

那么问题来了，如果我们面对的是**嵌套函数调用**，怎么办？哪些寄存器里的内容应当暂时被放到栈内保存？（因为要考虑用于存储函数参数与函数返回地址的寄存器的内容了）

```c
int sumSquare(int x, int y) {
    return mult(x,x)+ y;
}
```

一个与嵌套函数调用相关的是在RISC-V中使用递归，我写了一个递归版本的斐波那契，在我的[Github仓库](https://github.com/SongShaopu1998/RISC-V-Recursive-Fibonacci/blob/main/fib.s)中。在课程提供的**lab3**中，也有几个递归调用的例子。

#### Register Conventions

区分**caller**与**callee**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216222800860.png" alt="image-20211216222800860" style="zoom:50%;" />

**caller**表示调用函数的那个“人”，**callee**表示被调用的函数。根据`register convention`，我们将寄存器分为这两大类：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216223552640.png" alt="image-20211216223552640" style="zoom:50%;" />

对于第一类，**caller**假定这几个寄存器中的内容在函数调用的过程中**不会发生改变**–所以如果内容有改变，那这是**callee**应该负责的事情；

而与之相对应地，**caller**不能够认为第二类寄存器里的内容在函数调用的过程中不发生改变–所以如果内容需要改变，**caller**应当首先把这些个寄存器中的内容**push**到栈中。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216224149786.png" alt="image-20211216224149786" style="zoom:50%;" />

### Memory Allocation

一个问题，如果寄存器的空间**不够**储存一个**局部变量**咋办？



使用**栈**解决问题！

> **Procedure frame** or **activation record**: segment of stack with saved registers and local variables.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216224612879.png" alt="image-20211216224612879" style="zoom:50%;" />

当我们想在栈内分配新空间时，就将栈指针**减去**需要的内存大小！

所以，对于之前的嵌套函数调用，RISC-V代码如下（包含了**push**与**pop**的过程）：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216224850731.png" alt="image-20211216224850731" style="zoom:50%;" />

代码也符合上图我们`Procedure frame`中分配的顺序：首先储存**ra**，之后储存**a1**，即函数参数。在函数调用结束后，**pop**栈内存，恢复寄存器内容。

在调用`SumSquare`之前，我们要先把它需要的两个参数储存到正确的寄存器-**a0 & a1**中，之后我们使用上边的代码调用该函数–因为需要嵌套函数调用，我们要开辟一个栈空间来存储可能改变的寄存器。接下来，我们开始对内部函数的调用做些准备–比如把参数存放到指定的寄存器里。

在调用函数结束之后，恢复栈内存与寄存器内容，并把控制权转移给上一级函数（`jr ra`）.

> 我们不要把**jal**指令默认成函数调用的唯一方式，我们之所以使用**jal**来调用函数，是因为函数的调用通常通过给入一个**label**的方式来实现（在内部其实是通过程序自动计算`PC-relative offset`来完成指定函数调用）。
>
> 但是，**jalr**其实也是可以调用函数的，他调用函数的方式一般是通过`absolute address`而不是`PC-relative address`实现。我们把一个地址存放到某个寄存器中，然后把它交给**jalr**指令。

#### Stack/Heap/Static/Code

这里列出他们在**RISC-V32**中的储存位置：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211216225700500.png" alt="image-20211216225700500" style="zoom:50%;" />

> - **PC**段存储了系统运行的关键信息
> - Stack must be aligned on 16-byte boundary **here**
> - RISC-V convention global pointer (**gp**) points to static

## Instruction formats

现在需要解决的问题是，机器是如何理解我们写出的汇编代码的？或者说，前边所述的这些汇编代码对应的二进制表示是什么？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221181146759.png" alt="image-20211221181146759" style="zoom:50%;" />

-  **Everything Has a Memory Address**

在介绍他们的二进制表示之前，我们首先要清楚，这些指令(`Instruction`)是如何被机器执行的。我们知道，无论是**指令**还是**数据**，都是存放在内存(`memory`)区域的，并对应着不同的地址(`address`)。而前边我们提到过，需要执行的指令的地址是保存在**PC(`Program Counter`)**中的：

> - Basically a pointer to memory
> - Intel calls it Instruction Pointer (IP)

我们正是通过不断更新**PC**的值，来告诉机器我们将要执行的指令在哪里。

- **Binary Compatibility**

另一方面，由于产品需要不断的更新迭代，当时我们不想在每一次更新迭代产品之后都需要重新为原本的软件和应用准备一套新的二进制代码表示，或者说我们想要在新的机器上运行旧的程序，这就需要指令集具备**backward-compatible**的特征。



在正式开始介绍之前，我们需要知道：无论是**RV32**,**RV64**还是**RV128**，他们使用的都是`32-bit`的指令集。

### Different Instruction formats

为了让机器明白我们的汇编代码包含哪些信息，需要执行哪些功能，我们将这个`32-bit`的指令分成了很多个`field`.

根据`field`被划分的不同的格式，指令被分为如下六种：

- **R-format** for **register-register** arithmetic operations
- **I-format** for **register-immediate** arithmetic operations and loads
- **S-format** for **stores**
- **B-format** for **branches** (minor variant of S-format)
- **U-format** for **20-bit upper immediate** instructions
- **J-format** for **jumps** (minor variant of U-format)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221191847346.png" alt="image-20211221191847346" style="zoom:50%;" />

观察上图我们可以发现，所有需要的**寄存器**或者是**立即数**，基本都被存放在同一段位置，这是为了方便电脑进行查找与运算。正是为了这一特性，可以看到，可能一个**立即数**被分成了几段来存放。

- 其中，`opcode`指定了这几个不同的指令类型–在同一类指令中，它们将永远一样。
- 由于我们总共有32个寄存器，所以我们需要使用5 bits来表示使用的寄存器。
- `funct`帮助我们确定同一类指令中的具体指令。

#### R-Format

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221195051110.png" alt="image-20211221195051110" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221192903460.png" alt="image-20211221192903460" style="zoom:50%;" />

> **slt**: set-less-than (`rd = (rs1 < rs2) ? 1 : 0`)

同时我们注意到，在**R-Format**中的`funct7`段，指令`sub`与`sra`的次高位是1–这表示，**它们的操作需要进行`sign extension`**.

#### I-Format

之所以要单独出现一个立即数格式，是因为汇编指令在将立即数指令转化为二进制代码时，是需要**将立即数编入二进制代码的**。而如果我们直接将**R-Format**中的`rd`段作为立即数使用，很明显，我们只能表示一个极为有限的立即数范围。所以我们需要一种新的指令格式。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221195115381.png" alt="image-20211221195115381" style="zoom:50%;" />

需要注意的是，立即数被在算术运算中使用时，需要**sign-extend**到`32 bits`.

另一方面，由于这里用于表示具体指令的段只有`funct3`这一个，而我们总共需要表示9种**I-Format**指令，我们需要使用一种特殊技巧：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221203437919.png" alt="image-20211221203437919" style="zoom:50%;" />

因为对于32位寄存器而言，我们再进行**移位操作**时最多移动$$2^5$$位，所以我们不需要使用12位来表示这里的立即数，从而，如同**R-Format**所述，我们可以利用次高位表示是否进行**sign-extend**.

#### Loads(also I-type)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221210750463.png" alt="image-20211221210750463" style="zoom:50%;" />

12位的立即数（`offset`）被加到`base register`，`rs1`中存储的地址中，得到新的地址，并放入`dest register`内。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221212354952.png" alt="image-20211221212354952" style="zoom:50%;" />

需要注意的是，**load**指令的`opcode`与`i-type`中的不同。在这里，我们引入了一个新的指令`lh`与`lhu`，传入一个`2 bytes`的数据，之后将其`(un)sign extend`到`32 bits`.

#### S-Format

这种指令格式是给存储指令设计的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221214234345.png" alt="image-20211221214234345" style="zoom:50%;" />

在格式上类似于**R-Format**，但是存储指令不向寄存器内部写数据，而是像内存写数据。这种将立即数分段设计的原因我们在先前提到过，是为了保证机器查找寄存器未知的方便。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221214834695.png" alt="image-20211221214834695" style="zoom:50%;" />

#### B-Format

这种格式应用于**条件跳转**指令。`branches`常被用于**循环**中（while, for, if-else）.这一类循环通常间隔的指令较少，如果我们需要跳转到一个很远的地方，那么此时我们可能需要`jump instructions(J-Format)`.

而由于指令的地址被存放在`Program Counter`中，我们可以使用**PC-Relative Addressing**的方法来实现指令的跳转，我们使用立即数实现这一操作，由于立即数共有12位可供使用，所以范围为$$\pm 2^{11}$$-**unit**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221223600322.png" alt="image-20211221223600322" style="zoom:50%;" />

需要特别注意的是，我们在这里使用的跳转单位并不是**byte**，因为一条指令的大小是**4 bytes**，这样做我们会跳转到指令的中间位置，无任何意义。

但同时我们要考虑的一点是，存在一种**Compressed Instructions**，他的每条指令的大小是`16 bits`，一般使用在廉价微型电子产品中。为了与这种压缩指令集相兼容，我们将跳转单位设定为**2 bytes**，而非**4 bytes**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221224320365.png" alt="image-20211221224320365" style="zoom:50%;" />

所以我们可以跳转的总的相对地址范围是$$\pm 2^{11}\times 16$$-bit也即$$\pm 2^{10}\times 32$$-bit.

所以，对于`Program Counter`来说：

- Don't take the branch:

$$PC = PC + 4$$

- Take the branch

$$PC = PC + imm*2$$



<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221224929494.png" alt="image-20211221224929494" style="zoom:50%;" />

在这种表示方法中，我们虽然只有**12**位立即数空间表示，但是却可以表示**13**位的`signed bit offsets`，原因是每次跳转的最小单位是**2 bytes**，所以地址一定是偶数(*even*)！也就是说，最低一位一定是0.也正是基于这个原因，我们无须储存这个最低位。

##### Immediate Encoding

前边提到过，所有的立即数(`Immediate`)在进行算术运算时，都会被**sign extend**到**32 bits**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221230002210.png" alt="image-20211221230002210" style="zoom:50%;" />

可以看到，这些格式中一致的立即数存储位置，这对提高存储器的处理速度很有帮助。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221230355329.png" alt="image-20211221230355329" style="zoom:50%;" />

#### U-Format(long immediates)

现在的问题是，如果需要跳转的指令距离我们大于$$2^{10}$$条指令咋办？于是我们有**U-Format**，用于处理高20位的立即数数据:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221230849453.png" alt="image-20211221230849453" style="zoom:50%;" />

我们当然可以把高位立即数处理与低位立即数处理结合起来使用：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221231608300.png" alt="image-20211221231608300" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221231632631.png" alt="image-20211221231632631" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221231650686.png" alt="image-20211221231650686" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221231723528.png" alt="image-20211221231723528" style="zoom:50%;" />

#### J-Format

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221233258631.png" alt="image-20211221233258631" style="zoom:50%;" />

我们所使用的**`jal rd offset`**指令，就是遵循了**J-Format**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221234129694.png" alt="image-20211221234129694" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223164359146.png" alt="image-20211223164359146" style="zoom:50%;" />

我们注意到，指令**jal/j**只能最多只能允许我们使用一个**20**位的立即数来进行指令跳转，如果我们需要更远的跳转距离，比如我们就是需要32位的立即数，那么我们需要分为两个部分写入跳转地址需要借助指令**`jalr rd, rs, imm`**来实现：

指令**jalr**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221235407372.png" alt="image-20211221235407372" style="zoom:50%;" />

很明显，指令**jalr**是**jal**的“对立”指令，因为它允许我们使用`32-20=12`位的立即数偏移跳转–如此一来，如果我们切实需要上述情况，即跳转到一个更远的地址（需要一个更大的偏移量），那么我们可以考虑将其分为`upper imm`与`lower imm`两部分两条指令处理：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211221235812925.png" alt="image-20211221235812925" style="zoom:50%;" />

而**RISC-V**也提供了相关的`pseudo-instruction`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211223170702921.png" alt="image-20211223170702921" style="zoom:50%;" />

有趣的是，在上述的使用方法中我们可以看到，虽然**jalr**的使用方法是利用`absolute address`操作，即直接设置**地址**值来决定下一步去哪里；我们也可以通过结合**PC**并给入`offset`的方式，进行类似`PC-Relative`的跳转操作（但还是借助于绝对地址的值）。

##### Tail Call Optimization？

在CS 61A中，我们学习过`Tail Call Optimization`的相关知识，比如如下的语句就是一个**尾调用(`Tail Call`)**:

```c
{
	// lots of code
	return foo(y);
}
```

而尾调用优化，一般对于递归调用会有比较明显的提升效果：比如对于如上的代码，按照一般的递归调用流程，**当前层**函数的函数栈会一直存在，程序会在内存中不断建立新的函数栈，而直到这些递归调用的函数一层一层返回后，才会将原本开辟的函数栈一个一个的**释放**掉。

而`Tail Call Optimization`允许我们在进入**下一层**函数调用的时候，就把**当前层**的函数栈释放掉，具体从指令集角度而言：

- Evaluate the arguments for `foo()` and place them in `a0-a7`
- Restore `ra`, all **callee** saved registers, and `sp`
- Then call `foo()` with **j** or **tail**

如此一来，当我们最终完成了所有的递归调用之后，程序就会直接返回到最开始的位置，或者说**最终应当返回以继续程序运行**的位置。

所以由此我们也知道，这种形式的递归调用是不能够做优化的！因为我们还需要在**当前层**函数栈中的变量：

```c
{
    // lots of code
    return n * foo(n - 1);
}
```

## CALL (Running a Program)

这部分写入先前的有关C/C++文件编译执行过程的博文中（`CS基础课程2`）。



## Introduction to Synchronous Digital Systems

- Switches
- Transistors
- Signals & Waveforms

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211230232112769.png" alt="image-20211230232112769" style="zoom:50%;" />

处理器的硬件部分就是一个**同步数字系统**。所谓同步，即所有的操作都是由一个内部的中心时钟协同完成的。而“数字”表示所有的值都是**离散**的。电信号被表示为**1**与**0**.

- Switches

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211231002234322.png" alt="image-20211231002234322" style="zoom:50%;" />

- Transistors

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211231002413822.png" alt="image-20211231002413822" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211231002619211.png" alt="image-20211231002619211" style="zoom:50%;" />

- Clocks

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211231002657875.png" alt="image-20211231002657875" style="zoom:50%;" />

### Type of Circuits

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211231002801568.png" alt="image-20211231002801568" style="zoom:50%;" />



## State (machines) & Combinational Logic

本章节内容在课程主页提供的几个**notes**([sds](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/sds.pdf),[state](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/state.pdf),[boolean](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/boolean.pdf),[block](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/blocks.pdf))中有详尽的描述，我将其上传到自己的图床中，这里不再赘述。

## RISC-V Processor Design

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104110853370.png" alt="image-20220104110853370" style="zoom:50%;" />

我们再来温习一下之前使用过的这张图：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104111302389.png" alt="image-20220104111302389" style="zoom:50%;" />

在这张图中，我们对CPU里各部分的功能做一个定义：

- **Processor(CPU)**: the active part of the computer that does all the work (data manipulation and decision-making)
- **Datapath**: portion of the processor that contains hardware necessary to perform operations required by the processor (the brawn)
- **Control**: portion of the processor (also in hardware) that tells the datapath what needs to be done (the brain)

我们采用如下的设计模式来设计CPU，需要注意的是，在当前阶段，我们还采用在一个时钟周期内完成包括取出指令、执行指令等的一系列过程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104114839859.png" alt="image-20220104114839859" style="zoom:50%;" />

Datapath中对一个指令的操作过程可以分为五个阶段：

1. Instruction Fetch (IF)
2. Instruction Decode (ID)
3. Execute (EX) - ALU
4. Memory Access (MEM)–`lw`,`sw`
5. Write Back to Register (WB)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104115241112.png" alt="image-20220104115241112" style="zoom:50%;" />

上图中包含的模块可以作如下定义：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104130837821.png" alt="image-20220104130837821" style="zoom:50%;" />

> 细节见PPT.

需要注意的是，这里我们将存放指令的内存–**IMEM**与存放数据的内存–**DMEM**放在不同的位置（使用了两块内存）.

接下来，我们需要根据先前介绍的不同的指令种类来决定我们所使用的Datapath构造。这里只对一些重要的结果和流程进行记录，详细参见授课PPT.

### R-Type Add Datapath

首先假定我们只需要处理**ADD**这一个指令，那么我们的构造应当如下图所示：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104132256542.png" alt="image-20220104132256542" style="zoom:50%;" />

需要注意，上图中有两个操作是在第二个时钟周期的上升沿给出的：

1. Program Counter中的地址+4–被存入Program counter内部
2. 经过ALU计算得到的结果被写入寄存器中

而如果我们想利用上边的电路来实现**SUB**指令的话，我们需要对他作如下修改：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104133409333.png" alt="image-20220104133409333" style="zoom:50%;" />

在控制单元内的0/1控制字正是根据指令中开头部分的**次高位**得到的。至于在ALU内部如何进行加减计算，在先前的**notes**(block)中已经有过详细论述。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104133457365.png" alt="image-20220104133457365" style="zoom: 33%;" />

### Datapath With Immediates

与**R**类指令所不同的是，立即数指令将第二个源寄存器改成了一个立即数，所以：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104133936131.png" alt="image-20220104133936131" style="zoom:50%;" />

在控制单元内，我们设置了一个`ImmSel`，他在这里被设置为**I type**. 而由于在**I**指令里立即数只有12位，所以我们需要进行`sign-extend`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104134242484.png" alt="image-20220104134242484" style="zoom: 33%;" />

与**R**指令系列中的其他指令的处理方法一样，我们可以通过更改控制单元内的**ALUSel**来决定到底使用哪一个具体的**I**指令计算。

对于**R/I type**的指令，均不需要包含第**4**阶段的指令操作过程，因为它们无需从**DMEM**中获取数据。

### Datapath For Loads

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104165636388.png" alt="image-20220104165636388" style="zoom: 33%;" />

 需要注意的是，在这里控制单元通过将`ALUSel`设置为**ADD**来表示进行立即数+起始地址的计算，最终得到目的地址。

### Datapath For Store

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104211604272.png" alt="image-20220104211604272" style="zoom: 33%;" />

这里的`DataB`输出的是寄存器内的数据，我们通过将控制单元的`MemRW`设置为`Write`，表示当下一个时钟上升沿到来时，将该数据写入内存中。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104211933861.png" alt="image-20220104211933861" style="zoom: 33%;" />

为了在控制单元中同时支持**I**与**S**两种立即数表示形式，我们可以使用一个MUX，决定到底使用`inst`中的[11:7]还是[24:20].

### Implementing Branches

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104225253833.png" alt="image-20220104225253833" style="zoom: 33%;" />

在`Branch Comp`的部分，我们需要输入`signed/unsigned`，并输出`BrEq/BrLT`.此时的**ALU**单元被用来对`Program counter`中的地址加上进行了`sign-extend`之后的立即数。最后，我们把得到的新的跳转地址放入`Program counter`内。

得益于**RISC-V**指令集设计的便利，在`Imm Gen`模块内部，我们不需要做太多变化——因为按照一般的习惯，我们需要作如下的操作以获取**B format**的立即数：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104230026049.png" alt="image-20220104230026049" style="zoom: 33%;" />

由于**B format**需要在最后补一个`0`, 这样的形式意味着我们在11位立即数的每一个`bit`以及需要增补`0`的位置（共12个）都需要一个`mux input`来决定到底使用原输入(`inst`)的哪一位。而在**RISC-V**中：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104230424697.png" alt="image-20220104230424697" style="zoom: 33%;" />

对于**I, S, B**这三种拥有不同的立即数格式的指令，我们可以使用两个`2-way mux`来解决问题：

1. 处理**I**: 如前一节所述，这帮助我们决定到底使用`inst`的哪一部分.
2. 处理**S, B**: 如果是**B format**，那么我们需要将`inst[7]`换下位置，并用0填补空缺.

### Adding JALR To Datapath

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220104235611894.png" alt="image-20220104235611894" style="zoom: 33%;" />

### Adding JAL To Datapath

这种模式我们只需要在上图的基础上将`ImmSel`改为**J**即可.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220126154140466.png" alt="image-20220126154140466" style="zoom:50%;" />

### Adding U-Type To Datapath

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220126151046208.png" alt="image-20220126151046208" style="zoom:50%;" />

这种模式我们只需要在上图的基础上将`ImmSel`改为**U**即可.同时可能需要切换**ALUSel**.因为对于**LUI**指令来说，我们只需要计算单元将B端口的内容输出即可：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220126151147224.png" alt="image-20220126151147224" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220126151204487.png" alt="image-20220126151204487" style="zoom:50%;" />

## Control and Status Registers

这一类寄存器如果简单理解，可以认为是用于保存和检查当前某个“系统”状态的，他们独立于我们前边所介绍的32个整数寄存器。可以总计有4096个该类寄存器($$2^{12}$$)。

它们不存在于基本的指令集中，但是在实际实现中却是必须的。我们有如下几种操作**CSR**的指令：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107223327757.png" alt="image-20220107223327757" style="zoom:33%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107223352662.png" alt="image-20220107223352662" style="zoom:33%;" />

```assembly
# read sstatus value into a0
csrr a0, sstatus

# pseudoinstruction: only perform writing rs1 to csr
csrw csr, rs1
# equal to
csrrw x0, csr, rs1

# pseudoinstruction: only perform writing uimm to csr
csrwi csr, uimm
# equal to
csrrwi x0, csr, uimm
```

## Datapath Control

对于**Datapath**的运行过程，这里补充几点：

1. 在课程一开始时提到过，从内存获取数据的时间要远大于从寄存器获取数据的时间，但是在**Datapath**中我们使用的**IMEM**实质上是**cache**，所以大致是与**Program counter**一个速度的。
2. 在**Datapath**中存在两条路线，一条是走寄存器内部并获取数据，另一条是立即数的判定与运算。这两条路线相比较起来，立即数的路线要耗费相对多一些时间，因为**我们要确定它是一个什么类型的立即数**，从而将其补全到32位。（但我们还是可以将这两条路线视为同时进行的）
3. 当Inst[31:0]被输入`Control Unit`时，后续一系列**可以被设置**的`flag`将被设置。
4. 对于**B format**的指令，当我们`fetch instruction`并将指令传入`control unit`之后，是不可以立即设置**PCSel**位的。因为我们需要根据具体指令的比较结果设置的**BrEq**和**BrLT**位决定是否进入由**label**指定的位置中–到底是**执行**`pc+4`还是`pc+imm`。
5. 在对`Program Counter`操作(`PC+4`)的同时进行`fetch instruction`。

### Instruction timing

我们以指令**add**为例：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107231334201.png" alt="image-20220107231334201" style="zoom:50%;" />

在上图中，我们可以观察到以下几点：

1. 观察`inst[31:0]`与`PC+4`的稳定时间可以发现，`PC+4`要稍微滞后一些，这是因为对于`Program Counter`来说，它的所有**bits**都会在同一时刻被设置(same type of **flip-flops**)，**但是**根据我们先前介绍的**adder**的设计方法，它的最低位会最先被设置好，一直到最高位。而我们在途中考察的应当以最后整体被设置好的时间为准。故`PC+4`的时间应当比`fetch instruction`稍稍滞后。
2. 对于新的**PC**以及从内存读取的数据会在下一个**rising edge**经过`clk-to-q`后被设置（或写入寄存器中），但是我们务必确保**在寄存器或者PC的`setup time`之前，信号就已经稳定了吗**，这意味着**时钟的周期不能太短，否则当在`setup time`中时，信号还没稳定下来**。
3. 根据**ISA**的设计，**ALUSel**需要4bits，根据**control unit**的设计，**WBSel**需要2bits。

如果我们将上述详尽的时间表划分出之前提到过的**5个指令执行的阶段**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107233446791.png" alt="image-20220107233446791" style="zoom: 33%;" />

接下来，我们可以考虑一些关于**delay**的问题。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107233323289.png" alt="image-20220107233323289" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107233339844.png" alt="image-20220107233339844" style="zoom:50%;" />

同时我们可以尝试计算出系统允许的最大时钟频率：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220107233542498.png" alt="image-20220107233542498" style="zoom:50%;" />

同时，在**disc 6**中我总结了以下几个关于**delay**的问题：

> Regs can hold until the earliest possible time its input might change:
>
> $$MAX\_HOLD = clk-to-q(from\ prev\ reg) + CL_{shortest}$$

这个等式之所以成立，是因为我们是从一个新的`rising edge`开始考虑的。

> Clock cycle must be lage enough to account for clk-to-q times, setup times and CL times:
>
> $$MIN\_CLOCK = clk-to-q(starting\ reg) + CL_{longest} + setup(ending\ reg)$$

这个等式之所以成立，可以从如下两个角度描述：

1. 如果一个周期内无法包含这三项，那么意味着某一个时刻的`setup time`会与`clk-to-q`发生交汇，会导致**亚稳态**。
2. 我们从先前的简单模型：adder+reg的角度来看，如果不满足，则在**一个周期**内，$$S_i$$的值无法被成功更新。

### Control Logic Design

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220108003957821.png" alt="image-20220108003957821" style="zoom:50%;" />

一般来说，有两种实现控制逻辑的方法，即**ROM**与**CL**：

其中，ROM的一般做法是直接将不同的`control words`存储起来，当我们需要增加指令或者构建复杂指令集时，就向其中增加新的控制字。

对于**RISC-V**来说，我们也将它称为`Nine-Bit ISA`，因为只有9个bits为我们提供了必要的信息:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220108004244841.png" alt="image-20220108004244841" style="zoom:50%;" />

此外，根据先前的控制电路设计，我们知道还提供了**BrEq**与**BrLT**这两个输入，所以总计11个：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220108004602014.png" alt="image-20220108004602014" style="zoom: 33%;" />

而如果我们将上图抽象一下，则会发现这种结构类似于一个**and**加上一个**or**的结构：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220108004822800.png" alt="image-20220108004822800" style="zoom: 33%;" />

故而我们可以尝试使用**CL**来设计控制单元，这里提供一个指令**add**的**Address Decoder**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220108004903402.png" alt="image-20220108004903402" style="zoom: 33%;" />

如果我们需要增添指令，那么就可以对`opcode`的后两位下手。它们代表着**RV32I**

## Pipelining

**Pipelining**主题的意义在于提升处理器的**Performance Measurement & Improvement**。我们想通过某种方式使得处理器能够一次(即一个时钟周期内)执行多次任务。

在解决这个问题之前，我们首先要知道如何评价一个处理器的性能，如果我们使用轿车与公交车的例子与电脑处理器进行对比，我们可以有如下的结论：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111201103971.png" alt="image-20220111201103971" style="zoom:50%;" />

其中，**energy**对应着“能量”；**power**对应着“功率”。需要注意的是，**power**在这里并不是一个好的衡量指标，因为功率的单位是`J/s`，低功率的机器执行任务的时间会比大功率的机器多，这意味着消耗的总能量未必小于大功率的机器。

### Iron Law

如何量化评价一个处理器的性能？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114162040450.png" alt="image-20220114162040450" style="zoom:33%;" />

> **CPI**指的是处理一条指令需要的时钟周期数.

对于上述三个因子，我们逐个论述：

#### Instructions per Program

取决于：

- Task
- Algorithm
- Programming language
- Compiler
- Instruction Set Architecture (ISA)

#### CPI

取决于：

- ISA
- Processor implementation (`microarchitecture`)
- Complex instructions (`CPI >> 1`)
- Superscalar processors (`CPI < 1`)

#### Frequency

取决于：

- Processor microarchitecture (determines critical path through logic gates)
- Technology (5nm vs. 28nm)
- Power budget (lower voltages reduce transistor speed)

### Energy efficiency

我们一般使用**CMOS**来制作各种门电路：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111204519037.png" alt="image-20220111204519037" style="zoom: 33%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111204639758.png" alt="image-20220111204639758" style="zoom:33%;" />

从上图中我们可以看出，显然电压**V**相对于当前电路的总电容**C**是一个更加“有力”的因子。但需要注意，如果我们将电压下降的太厉害，那么晶体管的运行速度会更慢，这是一个`trade-off`。同时，电压的下降会导致`leakage`的比率变高。另一方面，为了控制电路总电容**C**，我们往往会将电路在`three dimension`的方向扩展（因为相当于**并联**电容）。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111212329679.png" alt="image-20220111212329679" style="zoom:50%;" />

最终，我们总结出了如下的规律：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111212303850.png" alt="image-20220111212303850" style="zoom: 33%;" />

#### Energy "Iron Law"

能量效率为什么如此重要？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111212429331.png" alt="image-20220111212429331" style="zoom:50%;" />

### Pipelining RISC-V

对于**Pipeline**的介绍这里略过，详见课件内容。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111215029548.png" alt="image-20220111215029548" style="zoom:50%;" />

> 需要注意的是，如果每一段的耗时不同，那么我们必须给每一段分配对应的**最长**的时间段，以确保在该阶段内的指令能够执行完成.

于是我们可以借助**寄存器**保存当前的状态直到下一个时钟时刻到来的时候的性质，将**5**个指令的子执行过程在我们先前设计的`single-cycle CPU`中划分出来，这种设计的理由参见先前在**Introduction to SDS**部分上传的`pipeline`文档中。于是在这里，我们使用**寄存器**保存下每一个阶段结束后的状态：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111215430545.png" alt="image-20220111215430545" style="zoom:50%;" />

> 如果没有寄存器，会怎么样？
>
> 如果我们不添加寄存器，也就是说不适用寄存器来保存当前状态，那么自遇到`rising edge`起，系统就会开始处理指令，由于当前我们假定的系统并非**single-cycle**，所以在一个时钟周期内，一条指令无法执行完成。这就带来一个问题——在导线上的信号传递速度是无限快的，由于我们没有保存好前方已经被传入的指令的信息，如**PC**和**control bits**，那么在指令需要用到他们的时候，这些值可能已经被后来的指令的信息替代了.

在这里，我们需要注意以下几点：

- 用于保存`Program Counter`的寄存器始终储存着`PC`，而非`PC + 4`，在需要使用到`PC + 4`的值的地方使用一个**adder**来重新计算，这种设计避免了我们将两者都传递下去（因为我们可以看到，在通路中间有需要用到`PC`的地方）
- 在上图的下侧，在每个阶段都有寄存器保存了指令的二进制代码：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111222147606.png" alt="image-20220111222147606" style="zoom: 33%;" />

同时在这些储存着指令的寄存器(`move the instruction along the pipeline`)中，我们还需要储存`control bits`:随着阶段的后移，我们需要储存的`control bits`应当是越来越少的，我们只需要在其中保留余下阶段的控制字即可：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111222520023.png" alt="image-20220111222520023" style="zoom:33%;" />

- 我们可以将上图中这些隔离开的小寄存器理解为几个大寄存器，每个寄存器保存了在当前状态下需要的所有信息，这也意味着更多的bits，比如在**IF/ID**阶段，就需要`64`bits的寄存器。同时根据前边对于**control bits**的描述我们知道，在**ID/EX**阶段的大寄存器应当包含`7`bits的控制字：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220113000720221.png" alt="image-20220113000720221" style="zoom:50%;" />

#### Pipeline Hazards

> There are situations in pipelining when the next instruction cannot
> execute in the following clock cycle. These events are called hazards.

共有三种类型的`hazard`:

1. **Structual hazard**

> It means that the hardware cannot support the combination of instructions that we want to execute in the same clock cycle.

> A required **resource is** **busy** (e.g. 如果`PC+4`的过程也需要用到**ALU**，那么在`pipeline`进行的过程中，就会形成`resource compete`)

2. **Data hazard**

> Data hazards occur when the pipeline must be stalled because one
> step must **wait for another to complete**.

- Data dependency between instructions
- Need to wait for previous instruction to complete its data read/write

关于**Data hazard**，我们在之后会详细论述.

3. **Control hazard**

> Also called **branch hazard**. When the proper instruction cannot
> execute in the proper pipeline clock cycle because t**he instruction**
> **that was fetched is not the one that is needed**; that is, the flow of
> instruction addresses is not what the pipeline expected.

#### How to solve pipeline hazards?

##### Structural Hazard

共有两种解决方案：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111232009686.png" alt="image-20220111232009686" style="zoom:50%;" />

那么在实际的设计过程中我们是如何解决**Structual Hazards**的呢？

首先，我们要确保**reg file**具备**3个端口**–`2 for read, 1 for write`.

其次，为了能够在同一个`clock cycle`中(可能需要)完成对内存的读和写，我们需要一个**IMEM**和一个**DMEM**.

> 真正的内存位于**DRAM**中，但是在处理器上的**cache**块保证了我们这一功能，也就是说**cache**可以被分为**I cache**与**D cache**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111233728283.png" alt="image-20220111233728283" style="zoom:50%;" />

最后，我们的**ISA**设计也在规避**structural hazards**的发生，这由保证一个指令中仅进行一次内存操作实现。

##### Data Hazard

在**data hazards**的主题下，我们也分成两个小问题来讨论。

###### Register Access

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111234327399.png" alt="image-20220111234327399" style="zoom:50%;" />

在上图中我们可以看到，在一个时钟周期内，我们对同一个寄存器进行了读和写。为了解决这个问题，有**double pumping**的解决方案：

> Exploit high speed of register file (100 ps)
>
> 1) **WB** updates value
> 2) **ID** reads new value

在`200ps`的时间里，我们使用前`100ps`完成写入，利用后`100ps`完成读取。需要注意的是，在面对一些高频率的处理器设计时，这种快速的**先写再读的方法不一定能够实现**。

###### ALU Result

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111234750902.png" alt="image-20220111234750902" style="zoom:50%;" />

为了解决上图所示的问题，我们有两种方法：

1. 既然我们在后续指令中要用到的结果还没有被成功更新，那我们不妨等一等，这就是我们的第一种思路——**stalling**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111235330818.png" alt="image-20220111235330818" style="zoom:50%;" />

需要注意的是，在实际的实现过程中，编译器会把一些不与当前指令存在`data hazards`的指令先执行，当不存在此种指令时，编译器会使用我们上图中表示出来的**nop**方法，对于**RISC-V**来说是：

```assembly
addi x0, x0, 0
```

但这种方法显然存在一个弊端，就是会减慢运行速度，削弱性能表现。

2. 我们来思考，是否能够借助第一条指令**EX**之后的结果，利用逻辑电路把已经计算好但没有储存在寄存器或者内存中的结果传递给需要他们的指令，换而言之，我们从**Pipeline**中抓取结果，而非寄存器？这就是我们的第二种思路–**Forwarding**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111235950615.png" alt="image-20220111235950615" style="zoom:50%;" />

为了判定究竟什么时候才需要将**Pipeline**中的结果传递，我们可以借助先前在**Datapath**中为了划分（储存）阶段信息的寄存器们，通过比较前一阶段与后一阶段指令寄存器中的信息，我们可以判定上述情况，即后一个指令是否在使用前一个指令还未将结果写入的寄存器：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220112000438742.png" alt="image-20220112000438742" style="zoom:50%;" />

> 为什么需要无视向`x0`写入的情况？
>
> 因为当我们向`x0`中写入时，**ALU**计算得到的结果并非是最终的结果（最终的结果始终为0），如果这时我们还是把前一指令的**ALU**计算结果写入下一个运算单元的输入，那么显然是不对的！为了实现这个“无视”的操作，我们需要判断上图两个寄存器中对应位置是否为0.

最后借助**mux**将电路串联起来吗，如此一来我们就可以选择到底是否做一个**等价的操作数替换(将分配给下一次指令计算的寄存器中的内容在`ALU`计算之前替换成已经在上一条指令中计算好的结果)**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220112000804485.png" alt="image-20220112000804485" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220113003439895.png" alt="image-20220113003439895" style="zoom:50%;" />

##### Load Data Hazard

当我们的指令序列中含有**load**并且遇到了`hazards`时，这需要我们单独处理。原因在于，我们无法实现先前所述的`forwarding`操作：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114184726124.png" alt="image-20220114184726124" style="zoom:50%;" />



在上图中，出现了一个`backward`，这是不可能实现的。

所以我们最简单直接的处理方法就是**stall pipeline**，在**stall**之后，再执行第二条指令。

对于上图所示的情况，我们也有具体的行为定义：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114203843023.png" alt="image-20220114203843023" style="zoom:50%;" />

上述的处理方式等效于添加一个第二条指令，即**nop**，但这种处理方式会毫无疑问的拉低性能。

> 如何将指令转化为**nop**？
>
> 首先我们需要检测到**lw**指令，之后在这一时钟周期内，将所有写操作的控制字都设置成**不写入**，比如寄存器的写、**DMEM**的写、**PC**的写等.

于是我们想到：

**可以将无关的指令放入`load delay slot`**来解决这个问题：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114205026660.png" alt="image-20220114205026660" style="zoom:50%;" />

#### Control Hazards

这种`hazards`来源于分支指令：只有在分支指令执行完毕**EX**这一阶段后，我们才能够确定此时的**PC**是多少，这意味着在分支指令的紧随其后的两条指令，都有可能是错误的！(`Executed regardless of branch outcome`). 接下来的第三条指令则是正确的，因为此时我们可以根据**已经更新的`PC`**来确定到底执行哪一条指令：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114211835019.png" alt="image-20220114211835019" style="zoom:50%;" />

一开始想到的处理方法如下图所示：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114212029784.png" alt="image-20220114212029784" style="zoom:33%;" />

也就是说，仅在**taken branch**时，将**pipeline**中的相关指令使用在上一节中介绍的相同方法转化为**nop**。

在当前的情景设置下，我们有50%的几率判定失误，因为此时我们的策略是先去执行紧随其后的两条指令，如果后来发现不对，再将他们改成**nop**，于是为了提升性能，我们想做一个`branch prediction`——只有当`branch prediction`错误的时候，才`flush pipeline`.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114214418230.png" alt="image-20220114214418230" style="zoom:50%;" />

需要注意的是，使用分支指令的大多数时候都是想做一个**loop**，于是最简单的`branch prediction`正是依托这种想法：如果我们上一次执行了分支，则继续执行分支，这大大提高了我们的判定准确率（当然还有更复杂的机构我们可以选择）。关于`branch predictor`，我们在后续的**cache**部分会做更详细的介绍。但总而言之，这是一个研究热点问题，并且对于处理器的性能表现起到了至关重要的作用，在*CS 152*这门课程中将会介绍一些应用的相关技术。

在上图中，我们首先经过预测之后决定执行分支，并在两条指令之后通过比较**PC**的预测值与真实值（将**ALU**结果传入）来确定是否要将前两条正执行了一半的指令设置成**nop**.

### Superscalar Processors

总得来说，提升处理器性能的方法有三种：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114215743724.png" alt="image-20220114215743724" style="zoom:50%;" />

需要注意在使用`pipeline`时，越多的`pipeline`将带来更大的`hazards`可能性！这可能会导致最终需要的**CPI**大于1.

> 对于`pipeline`操作，当指令极多时，假如不存在任何`hazards`，那么**CPI**将无限趋近于1，

什么是**superscalar Processor**？就是在处理器中安放多个不同的处理单元，比如**add**,**lw**,**sub**等等，在每个处理单元中`pipeline`执行指令(`multiple pipeline`)，这一操作允许我们在同一时刻执行更多指令，故可以有**CPI<1**的结果。这时，我们可以用**IPC**来描述结果。

为了将指令放入不同的处理单元执行，我们需要`Out-of-Order execution`：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114221956196.png" alt="image-20220114221956196" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114222013649.png" alt="image-20220114222013649" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114222115697.png" alt="image-20220114222115697" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220114222208392.png" alt="image-20220114222208392" style="zoom:50%;" />

### disc08

- **latency**(**延迟**): the time for one instruction for finish
- **throughput**(**吞吐量**): the number of instructions processed per unit time

当我们考虑**pipelined Datapath**时，关于**EX**段的最短时间计算——在**ALU**计算单元完成之后，传出的数据会立即被传递到**PC**之前的**mux**处，从而我们可以得到**正确的下一条指令的地址**。（这也是**B format**的运作流程之一）详见`Performance Analysis`讨论.

- Why is the speedup of pipelined datapth is lees than what is expected?

1. The necessity of adding pipeline registers, which have *clk-to-q* and *setup* times;
2. The need to set the clock to the **maxmium** of the five stages, which takes different amount of time;

> Because of hazards, which require addtional logic to resolve, the actual speedup would be even less.

## Caches

### Binary Prefix

本部分内容见课件。

### Memory Hierarchy

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115145919149.png" alt="image-20220115145919149" style="zoom:50%;" />

上图中空缺的部分就是**cache**，对于绝大多数的处理器来说，他们具备独立的`instruction cache`与`data cache`.一般来讲，**cache**都会与**cpu**集成在同一块主板上。而距离**cpu**越远，则速度越慢，但相应地可以具备更大的储存量。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115151438456.png" alt="image-20220115151438456" style="zoom:50%;" />

在这幅图中，我们留意到不同的处理器内存层级，需要注意它们`inclusive`的特性，即**上层是下层的`subset`**.具体的讲，**memory**即**DRAM**是**disk**的子集复制品，**cache**又是**memory**的子集，寄存器中的内容又是把**cache**的内容复制进去（这一点在前几个章节的**Datapath**中我们已经看到）

> 在**CS基础课程2**的博客中，我们介绍了**loader**的作用，正是将可执行文件从**disk**复制到**memory**里。

### Locality, Design, Management

**cache**的工作原理基于如下两大特性：

- **Temporal locality**
- **Spatial locality**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115153325585.png" alt="image-20220115153325585" style="zoom: 33%;" />

如上图所示，对于`temporal locality`，我们保存了最近使用的数据在距离处理器很近的位置；而`spacial locality`则基于我们很有可能使用刚获取的内存为止附近的地址中的资源的想法，将邻近的内存块储存到**cache**中。

#### How is the Hierarchy Managed?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115153632969.png" alt="image-20220115153632969" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122141301512.png" alt="image-20220122141301512" style="zoom:50%;" />

### Direct Mapped Caches

在这种**cache**的表示模式中，每一个内存地址都与**cache**中的一个`block`相关联。`block`是类似于寄存器与内存之间的最小传递单位`word`一样的存在：

> Block is the unit of transfer between cache and memory.

在进行**cache**的相关计算时我们应当把**cache**画成和**memory**一样宽度来方便后续计算。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115214331039.png" alt="image-20220115214331039" style="zoom: 50%;" />

我们将**memory**按照**cache**的大小划分为0，1，2，3……个区域，每个**cache**中又包含了几个**block**。这样做的目的是尝试在**cache**与**memory**之间建立一种映射，使得我们可以通过内存地址判定出他应当存放的**cache**位置；反过来也可以通过**cache**位置判定出数据来自于哪个内存地址中。

而我们发现，由于对于一个固定的**cache**，其行和列也是一定的，所以我们想到为了完成上述这种映射，并不需要把内存地址完整的记录下来，比如对于想储存`8,2,14,1E`的这种情况，我们可以省略后几个`bits`，**因为内存地址中后边的几个`bits`在当前的模式下会表示数据将会位于cache的哪一行和哪一列**。而这些行列对应的储存位置将保持固定，如图中不同颜色所示。

所以我们需要记录前边的一些`bits`，从而在**cache**中保存内存地址的所有信息，完成完整的映射。这些`bits`即上图开始的`1,0,2,3`，称为**tags**。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115221109834.png" alt="image-20220115221109834" style="zoom:50%;" />

于是我们可以将内存地址划分为三块：

- **t**ag

> the remaining bits after offset and index are determined; these are used to **distinguish between all the memory addresses that map to the same location**

- **i**ndex

> specifies the cache index (which “row”/block of the cache we should look in)

- **o**ffset

> once we’ve found correct block, specifies which byte within the block we want

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115221414122.png" alt="image-20220115221414122" style="zoom:50%;" />

#### Memory access with/without cache

无论内存的获取是否经由**cache**，我们先前设计的**Datapath**都可以适用，从内存或者**cache**中获取数据的过程对于**processor**来说是一个`generated process`.当不使用**cache**时：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115223002340.png" alt="image-20220115223002340" style="zoom:50%;" />

如果使用**cache**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115223415019.png" alt="image-20220115223415019" style="zoom:50%;" />

处理器会把地址交给**cache**，**cache**会首先检查它是否存储了指定地址对应的`block`，如果没有，再去内存中读取数据并存入**cache**中，最后由**cache**发送数据给处理器。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115224104952.png" alt="image-20220115224104952" style="zoom:50%;" />

### Cache Terminology

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115224148919.png" alt="image-20220115224148919" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115224232912.png" alt="image-20220115224232912" style="zoom:50%;" />

> `access`可以被理解为`find & retrieve`.

需要注意的是，这里的`Miss penalty`本意并非`replace`，而是`get`:

> **Miss Penalty** refers to the extra time required to bring the data into cache from the Main memory whenever there is a “miss” in the cache.

需要注意的是，如果发生**cache miss**，我们需要首先从内存中取出数据放入**cache**，之后还需要一次**Hit**才行，所以**miss penalty**要从**total delay time**中减去一个**Hit time**.

而`Hit time`的计算则需要加上`tag comparison`的时间。

最后再来解决一个问题，即当一个新程序开始运行时，即`cold cache`，我们如何获知此时的`tag entry`是否合理？即使在执行了一些指令之后，许多`block`可能仍然是**空着的**。为了解决这个问题，我们想要给**cache**增加一个额外的**Valid Bit**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220115224853607.png" alt="image-20220115224853607" style="zoom:50%;" />

> The most common method is to add a **valid bit** to indicate whether an entry contains a valid address. If the bit is not set, there cannot be a match for this block.

该标志位为`invalid`的情况不仅仅有初始情况这一种，当我们面对**coherence miss**时，如果其他处理器内核更新了其中的**cache**内容，那么我们也要将其他的内核中的**cache**设置为`invalid`。这表明**valid bit**存在的意义可以被理解为“我们是否需要从内存中获取新的数据”。

### Directed-map Read Example

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117163614241.png" alt="image-20220117163614241" style="zoom:50%;" />

根据上图，从**cache**中获取数据分为如下几步：

首先根据**index**，找到可以存储的行，在该行上，判定`Valid Bit`，如果为0，则意味着这是一次**compulsory miss**，接下来需要从**内存**中取出数据放入**cache**中并根据指定的`offset`返回对应的数据(并把`Valid Bit`设为1)。如果`Valid Bit`为1，则我们需要判定`tag`是否产生了**conflict miss**(并重复上述的操作过程)。

### Writes, Block, Sizes, Misses

我们使用如下的逻辑电路设计`Multiword-Block Direct-Mapped Cache`的写操作:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117173302111.png" alt="image-20220117173302111" style="zoom:50%;" />

可以看到，判定是否**cache hit**的方法是将`tags`与`Valid Bit`做**and**操作。在右侧，通过一个**mux**选择出需要的数据(在哪个`offset`中)。

#### What to do on a write hit?

有两种方法实现写操作：

1. **Write-through**

我们仍然把**memory**当做目标点(终点)，所以不仅要更新**cache**也要更新**memory**.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122160403497.png" alt="image-20220122160403497" style="zoom:50%;" />

使用**write-buffer**的[原因](https://www.quora.com/What-is-the-role-of-the-write-buffer-in-a-write-back-cache)如下：

> TLDR The write buffer makes the machine faster, because the program doesn’t have to wait until data is written back to RAM.
>
> When a program reads from memory, and the data is already in the cache, it is called a cache hit and the program moves on.
>
> When the data is not in the cache, the data is read from RAM and put into the cache. This is called a cache miss.
>
> The hardware chooses a cache location for the new data. The previous stuff in that cache location (the cache is always full) is called the victim.
>
> If the victim data has not been written, it will be the same as the data in RAM for that address. This is called a clean victim or a clean miss.
>
> If the victim data has been written since it was originally put into cache, it may be different than the data in RAM for that address. In order to not lose those changes, the victim must be written back to RAM. This is called a dirty victim or dirty miss.
>
> Obviously, you have to write the dirty victim data back to RAM before you overwrite it with the new data coming from RAM (for a different address), but if you do the memory write first (for the victim data) and the memory read (for the new data) second, the program will have to wait twice as long.
>
> Smart people realized that you don’t have to do the write first. The victim data was chosen to be the victim *because it wasn’t being used by the program*. This idea led to the write-buffer. The victim data is not immediately written back to RAM, it is stashed away in the write-buffer and then written back to RAM later as time permits.
>
> This lets the hardware do the memory read for the new data first, which speeds up the program because it doesn’t have to wait longer for a dirty miss than it does for a clean miss.

简单来说，`write-through`下的`write-buffer`的存在允许系统不需要先等待数据被完全写入内存再将数据读入对应的**cache**位置。

2. **Write-back**

这种方法，我们将**cache**作为一般情况下的最终目标区域，这意味着我们允许**memory**与**cache**的数据可以不同（`allow memory word to be "stale"`）。而正是为了辅助我们判定两者的数据到底是否相同（可以存在向**cache**中存储了恰好与**memory**相同数据的特殊情况），以此决定是否需要将**cache**的数据复制回**memory**中，于是我们增设一个`dirty bit`:

- indicate memory & Cache inconsistent
- needs to be updated when block is replaced(*cache only writes data up the hierarchy when a cache line is evicted*)——…OS flushes cache before I/O…(then the `dirty bit` will be set to 0)

> **Write-back** means that on a write hit, data is exclusively written to the cache. In addition, after the write, the dirty bit for the block that was just written becomes 1 (mental check: what does having a dirty bit of 1 mean?). Writing to the cache is quite fast, so the write latency in write-back caches is usually quite small. However, when a block is evicted from a write-back cache, if its dirty bit is 1, then the memory must be updated with the contents of that block, as it currently contains changes that are not reflected in memory. Thanks to this, write-back caches tend to be more difficult to implement in hardware.

当然，这两种方法存在`Performance trade-off`：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120234022116.png" alt="image-20220120234022116" style="zoom:50%;" />

使用**write-back**策略时，如果`write-hit`，则不需要再访问内存(**0**)，如果`write-miss`，则需要从内存中读取新`tag`的`block`更新**cache**，此时看`dirty-bit`，若为0，则无需将**cache**内容写入内存(**1**)，否则需要更新内存中的内容(**2**).

除此之外，在**lab 7**中还提到了一个模式：

> - **Write-around** means that in every situation, data is written to main memory only, kind of like an opposite version of the write-back policy. An important thing to keep in mind is if the block we just updated in the main memory is also in our cache, then we need to change the cache block's valid bit to invalid. This is because our cache no longer has the most up-to-date memory. In addition, under the Write-Around policy, there is no such thing as a write-hit, as it effectively does the exact same thing as a write-miss.

**When write miss:**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220121002618180.png" alt="image-20220121002618180" style="zoom:50%;" />

#### Block Size Tradeoff

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117190249100.png" alt="image-20220117190249100" style="zoom:50%;" />

关于过大的`Block size`带来的负面影响，首先是因为`Block`中的元素太多，所以必需耗费更长的**miss penalty**.另一方面，当`Block size`接近`Cache size`时，比如可以假想一种极端情况，即**One Big Block**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117190940309.png" alt="image-20220117190940309" style="zoom:50%;" />

这意味着基本没一次取数据都会发生**conflict miss**，这被叫做**Ping Pong Effect**.

随着`Block size`的增大，`miss rate`首先会减少，这是因为更加充分地利用了`Spatial Locality`，在这之后它便会开始增加，而这正是由于上述原因，即趋近于`Cache size`，这导致`temporal locality`几乎失效。

如果我们综合考虑`Miss Penalty`与`Miss rate`，以及两者的乘积`Average Access Time`，则会得到如下的曲线图：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117191358522.png" alt="image-20220117191358522" style="zoom:50%;" />

### Fully-Associative Caches & Type of Cache misses

**Fully-Associative Cache**的思路很简单，就是想要避免**conflict miss**，故而在`Memory address fields`中取消了**Index**的部分，但同时这也意味着当试图读取数据时，我们必须在整个**cache**中比较所有`tags`（一般是**并行进行**），以寻找数据位置。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117203352796.png" alt="image-20220117203352796" style="zoom:50%;" />

当然这种设计也是有优劣的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117203440040.png" alt="image-20220117203440040" style="zoom: 33%;" />

#### Types of Cache misses

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120230241106.png" alt="image-20220120230241106" style="zoom:50%;" />

需要注意的是，我们使用`valid bit`的目的是应对`empty cache`的情况，也就是说当`valid bit`没有被设置时(程序刚开始运行)，我们一定会遇到**compulsory miss**。但这并不意味着**complusory miss**仅有这一种情况。它的定义为**第一次获取某块数据时**，那么我们如何得知获取数据的历史呢？历史记录由硬件实现，比如我们使用的**LRU**替换策略就会记录历史所使用的`block`.

与之相对应的**conflict miss**则定义为：

> A conflict miss occurs when a block is needed which existed in the cache before, but was **evicted** in favor of another block that had to be mapped to the same slot.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117204550516.png" alt="image-20220117204550516" style="zoom:50%;" />

在这第二种解决**conflict miss**的方法里，就是我们先前描述的`Fully-Associative Caches`。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220117204755697.png" alt="image-20220117204755697" style="zoom:50%;" />

用简短的语言描述第三种，就是因为**cache**的容量不够导致的，如果我们将**cache**扩展到与**memory**相同的容量，那么就不会出现这种情况了。当使用**fully-associative model**时，由于**cache**的容量不够，便可能会产生该问题.

但实际上，上述的定义是十分模糊的，对于**conflict miss**与**capacity miss**，还是难以分辨，我们可以使用如下方法。

#### Cache Miss judgement

我们可以使用如下的算法来判定**miss**的类型：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220121162007905.png" alt="image-20220121162007905" style="zoom:50%;" />

需要注意的是，当我们遇到需要判定**cache miss**类型的题目时，一般需要同时画出`original cache`与**相同大小的**`fully-associative cache(LRU)`.因为我们需要依据准则“**If you hypothetically ran the ENTIRE string of memory accesses with a fully**
**associative cache (with an LRU replacement policy) of the same size as your cache, and it was a miss for that specific access, then this miss is a capacity miss.**”

换句话说，在`fully-associative cache`里，无论**cache**的大小是多少，始终**不会发生`conflict miss`**.

> **compulsory miss**的判定方法很简单，当且仅当我们检查了一块先前从未见过的**block**，那么就会遇到**compulsory miss**. 如果不是，那么我们就要从剩下两种**miss**中选一种。如果我们发现对应大小的`fullu-associative cache(LRU)`中不会出现**miss**，那就是**conflict miss**；反之则为**capacity miss**.

当产生**write miss**时，我们有两种策略：

- **Write-allocate** means that on a write miss, **you also pull the block you missed on into the cache**. For example, for write-back, write-allocate caches, memory is never written to directly. This is because we are only ever writing to the cache (thanks to the write-back policy), and we only end up updating the memory upon eviction of the (potentially) updated block.
- **No write-allocate** means that on a write miss, you do not pull the block you missed on into the cache. **Only memory is updated**.

### Set-Associative Caches

这种**cache**的表示方法将地址的`Index`域作为一个**set**进行看待：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118175144546.png" alt="image-20220118175144546" style="zoom: 33%;" />

在每一个**set**内含有多个`block`，当我们根据`Index`找到对应的**set**后，必须使用与`fully-associative caches`中相同的方法，即逐个比较**set**中每一个元素的`tag`，来确定数据的具体位置。（每一个**set**都是一个具有**N**个元素的`fully-associative block`）

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118175515469.png" alt="image-20220118175515469" style="zoom: 33%;" />

对于这种设计方法来说，即使当使用**2-way set assoc cache**时，我们会避免很多**conflict miss**。三种**cache**的关系可以形容为：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118175844689.png" alt="image-20220118175844689" style="zoom:33%;" />

#### 4-Way Set Associative Cache Circuit

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118175933127.png" alt="image-20220118175933127" style="zoom:50%;" />

- 判定**cache hit**的方法是将四个结果用**or**组合起来，在每一个**set**内都需要进行**N**次比较，当然一般他们也是`parallel comparator`.
- 选取**data**时使用了一个`4-1 mux`，需要注意的是他是**4**脚判定，而非先前我们见到过的**2**脚判定。这种方法叫做`One Hot Encoding`.

##### cost

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122111927670.png" alt="image-20220122111927670" style="zoom:50%;" />

### Block Replacement

现在需要解决一个问题，当我们发现需要进行`Block repalcement`时，应当使用什么策略呢？

#### LRU(Least Recently Used)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118180949727.png" alt="image-20220118180949727" style="zoom: 50%;" />

这种方法借助`temporal locality`，硬件系统需要记住不同`block`的访问顺序，并在需要时将最先前访问的也就是最老的那一个排除掉，但这种方法在`set-associative`的**N**稍大一些时，需要保存的情况太多，为了能够记录下所有的可能情况，即$$N!$$，需要辅之以大量硬件。

#### FIFO

这种方法忽视**access**，直接将最先加入的那一个`block`踢出去。

#### Random

当`temporal locality`较小时，这种方法可能真的是一种不错的方法！

### Average Memory Access Time (AMAT)

我们将它作为一种性能指标。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118191708742.png" alt="image-20220118191708742" style="zoom:50%;" />

需要注意的是，$$Hit\ Time$$之后不需要乘上$$Hit\ Rate$$，原因是在**miss**并将数据从内存中取出来放入**cache**后还需要进行一次**hit**才可以把数据取出来。

于是我们思考，如何通过改变参数或者硬件的方式提升性能表现？

#### Increasing Associativity?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122113748913.png" alt="image-20220122113748913" style="zoom:50%;" />

注意这里**miss penalty**的变化，这里的逻辑在于：替换策略与从内存或者上级高速缓存中取出数据的过程是**并行**的：如果在发现**miss**后，先要去寻找哪个位置是我们要替换的，再从内存中取出需要的数据，那么很显然，当**associativity**很大时（比如`fully-associativity`），我们自然需要更长的时间在对应的**set**中寻找替换的位置。但**实际上**，寻找替换位置（也可能根本不需要所谓的寻找，比如**LRU**硬件就会记录需要替换的位置，并在替换之后立即指向下一个**LRU**）的过程与从内存中取出数据的过程是同时进行的。

#### Increasing # Entries?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122113913666.png" alt="image-20220122113913666" style="zoom:50%;" />

越大的内存速度越慢！使大存储器运行的更快总是要难一些的。

虽然内存越大，向其中写入的速度也可能会相应变慢，但对于`miss penalty`，根据课上教授的说法，向其中写入数据的同时我们可以向处理器发送数据（两者并行），故`miss penalty`不会被影响。

#### Increasing Block Size?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122115115152.png" alt="image-20220122115115152" style="zoom:50%;" />

`Hit time`之所以基本不变，首先`block size`变大意味着我们在选择数据时需要一个更大的**mux**，不过这无伤大雅。同时更大的`block size`会带来更少的`tag`位数，这会减少我们比较的时间。

此时的`miss penalty`会变大，因为我们需要传输更多的数据。

#### Improving Miss Penalty

针对上边的这个公式，我们尝试降低**miss penalty**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118192309750.png" alt="image-20220118192309750" style="zoom:50%;" />

这种设计基于`recursion`，由于直接去一个很远的地方(**memory**)取数据需要很长的时间，所以我们想把过程分解，即增加几级新的**cache**。需要注意的是，这种设计会使得距离**CPU**越**远**的**cache**中存储的数据越**好**，即有更低的**miss rate**；我们可以用递归的思路来理解这个结论，比如在只有一层，即`base case`时，所谓的**L2 cache**其实就是**memory**了，当然，设计始终遵循越小的存储空间（越靠近CPU的存储是它的上一层级的子集这一原则）.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118192946866.png" alt="image-20220118192946866" style="zoom:50%;" />

##### Analyzing Multi-level cache hierarchy

递归的计算原则：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118193044632.png" alt="image-20220118193044632" style="zoom:50%;" />

具体的案例在课件中给出了，我们会发现，通过加上一级**cache**，性能表现(**AMAT**)提升了将近四倍。

实际应用的**cache**常呈以下数据：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118193306629.png" alt="image-20220118193306629" style="zoom:50%;" />

> 还有**L3 cache**，它是由多核**CPU**所共用的，并非如同前两级**cache**一样在每一个**CPU**核内都有.

#### Reduce Miss rate

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118201813727.png" alt="image-20220118201813727" style="zoom:50%;" />

我们使用如下公式计算`global/local miss rate`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122233740454.png" alt="image-20220122233740454" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220122233749609.png" alt="image-20220122233749609" style="zoom:50%;" />

可以看到，在下方的公式中，整体的`miss rate`等于级联的`local miss rate`的乘积。

### Cache Analyze

在软件层面，对于**cache**的了解可以帮我们更好的规划代码结构，比如对于如下的例子：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124003424571.png" alt="image-20220124003424571" style="zoom:50%;" />

这种`stride access`在代码中经常出现。我们在**lab 07**中以及**CSAPP**中也看到过关于这部分优化的介绍。在谈到优化策略之前，先说以下可能出现所谓`miss/failure`的原因，并根据在树莓派上的实验数据得到一些相关结论。

#### Failure? WHY?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124005538383.png" alt="image-20220124005538383" style="zoom:50%;" />

`capacity miss`会出现在数据容量大于**cache**容量时，这是一件很显然的事情。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124005635899.png" alt="image-20220124005635899" style="zoom:50%;" />

除了`capacity miss`之外，如果我们在每一次将一个`block`的数据从内存搬运到**cache**后，只对它们进行一次原本需要的数据的访问，那么显然会造成`spacial locality`的浪费！这种情况的临界条件出现在：

$$stride*sizeof(int)==block\ size$$

试想如下情况：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124010021540.png" alt="image-20220124010021540" style="zoom:50%;" />

虽然我们并没有向**cache**中输入足够将所有`block`塞满的数据量，但还是出现了**miss**. 因为这是**conflict miss**.我们的数据位置发生了重叠，需要进行`eviction`.

#### SHOW ME THE DATA!

在展示数据之前，先要明晰一些事项：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124010932928.png" alt="image-20220124010932928" style="zoom:50%;" />

从一个抽象的认识上，我们知道**L2 cache**的容量或者类似于`associavity`之类的东西都要比**L1 cache**更大，但这里我们拎出两个重要的特性`line size`与`associativity`来看下为什么.

首先，从测试的角度来说，我们必须保证能够出现*L1 miss, L2 hit*这种情况。而无论是**L1 cache**还是**L2 cache**，数据都是从内存中取出或者是被处理器写入的。在这个前提下，加入我们遇到了**L1 miss**，那么就想要到下一级缓存中寻找数据，而如果下一级的`line size`或者`associativity`反而更小，那么在绝大多数情况下，也会发生**L2 miss**，这样一来，下一级缓存就失去了意义。

现在，来分析一下课程中给出的实验数据。

##### L1

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124013302131.png" style="zoom:33%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124013326154.png" alt="image-20220124013326154" style="zoom: 33%;" />

对于**L1 cache**，当`array_size`为`32kb`时，所有的**AMAT**均被控制在**10ns**以内，说明没有出现任何**miss**，说明此时还未达到`cache size`. 而在`64kb`时出现了多次**miss**(**10ns**及以上)，故`cache size`为`32kb`.（一般来讲，`cache size`都是2的指数）

而在`64kb`的测试内部我们观察到，当`stride`在`128bytes`之前**AMAT**均被控制在**10ns**以下，不过这里的数据不是特别有说服力，因为测试时间过短，即使出现多次**miss**可能最终结果也很贴近。如果根据判定依据：当`array_size`为`cache_size`的两倍时，如果$$stride*sizeof(int)==block\ size$$，那么我们每一行的数据只被使用了一次，没有利用`spacial locality`，那么这张图会告诉我们`block size`大致为`128bytes`. 但基于**AMAT**过短的原因（**8ns**与**10ns**也很近，**L2**的存在一定程度上缓解了**AMAT**的变化），`block size`为`64bytes`也是有可能的。

而在这张测试数据图的最后，我们发现当元素个数为1，2，4时，均没有出现**miss**，但是在8元素处出现**miss**，说明这是一个`4-way set associative cache`.

> 从**AMAT**的变化趋势可以看出，一开始`stride`很小，所以每次我们从内存或者下一级`cache`取回一个`block`的数据时，可以**miss hit**很多次（但是因为`srray_size`为`cache size`的两倍，所以之后会出现**capacity miss**）
>
> 随着`stride`的不断增加，超过了一个`block`的大小，我们每一次数据读写的**间隔**越来越大，这也意味着`cache`不会全部用完，如先前的分析所述，此时会产生很多`conflict miss`，读写间隔越大，`conflict miss`的可能性越高。**直到**，我们仅仅需要读写几次数据，这些数据可以被`N-way set associative cache`所容纳。

##### L2

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124131635518.png" alt="image-20220124131635518" style="zoom:50%;" />

这组数据使用了`1024kb`的`array_size`，意味着在`array_size`为`512kb`时并没有出现**L2 miss**的情况，故**L2**的`cache size`为`512kb`.

我们可以更加清晰的看到当`stride`为`64bytes`时出现了一个明显的**Fall Off A Cliff**的情况，说明我们达到了**每行只使用一次数据的最坏情况(`no spacial locality`)**。而**L1 cache**与**L2 cache**的`line size`应相同，故两者均为`64bytes`.

随着`stride`的变大，根据我们对**L1 cache**的分析，会产生更多的`conflict miss`，于是处理器就会想要前往**L2 cache**中寻找数据(每一次内存传输的数据实质上并不止一个`block`的大小)，一开始还是可以找到的，后来随着`stride`的继续增大，**L2 cache**中也找不到了，于是**AMAT**进一步增长。

> Performance drops by an **order of magnitude** when you exceed the capabilities of the cache even by not that much!

最后的3行数据为**L1 hit**，4-7行为**L1 miss, L2 hit**. 于是我们可以猜测**L2**大概是一个`64-way set associative cache`.

但是事实果真如此吗？

##### Some complication technology

实际上，**L2 cache**并非`64-way set associative cache`。因为涉及到一些比较复杂的细节：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124145830409.png" alt="image-20220124145830409" style="zoom:50%;" />

首先，对于内存的抓取并非真的完全按照我们所说的“每次抓取一个`cache block`”.而是每次抓取更多的内容，或者进行`prefetching`以更好地预备`cache`中的数据读写，其包括数据预取和指令预取，可以通过处理器硬件或者编译器软件实现，是一个较为复杂的主题。

其次，除了主缓存之外，我们还有`victim cache`. 

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124151359145.png" alt="image-20220124151359145" style="zoom: 67%;" />

根据维基百科：

> 所谓**受害者缓存**（`Victim Cache`），是一个与直接匹配或低相联缓存并用的、容量很小的全相联缓存。当一个数据块被逐出缓存时，并不直接丢弃，而是暂先进入受害者缓存。如果受害者缓存已满，就替换掉其中一项。当进行缓存标签匹配时，在与索引指向标签匹配的同时，并行查看受害者缓存，如果在受害者缓存发现匹配，就将其此数据块与缓存中的不匹配数据块做交换，同时返回给处理器。
>
> 受害者缓存的意图是弥补因为低相联度造成的频繁替换所损失的时间局部性。

所以，实质上在树莓派中的**L2 cache**为`16-way set associative cache`，但是`victim cache`的存在让它看起来像是一个`64-way set associative cache`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124151319608.png" style="zoom:50%;" />

#### Loop! Improve Performance?

##### change innerloop stride

对于如下这样一个例子：

```c
int j, k, array[256*256];
for (k = 0; k < 255; k++){
    for (j = 0; j < 256; j++){
         array[256*j] += array[256*j + k + 1]
}}
```

我们并没有在内层循环中很好地利用`cache`的`spacial locality`.而如果我们将它改为这样：

```c
int j, k, array[256*256];
for (j = 0; j < 256; j++){
    for (k = 0; k < 255; k++){
     array[256*j] += array[256*j + k + 1]
}}
```

在内层循环中，`stride`从两个**256**变为一个**0**和一个**1**——很好的利用了`termporal locality`和`spacial locality`.

##### Blocking-out Data

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124160357829.png" alt="image-20220124160357829" style="zoom:50%;" />

在上边的这个二层循环中，如果数组**B**过大，那么会产生多次`capacity miss`.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124160603425.png" alt="image-20220124160603425" style="zoom: 50%;" />

我们可以将矩阵变成多个`block`，至于为什么这样做能够优化？**CMU 15-213**设置了更详细的**CacheLab**，从数据的角度出发详细分析这个过程。[这里](https://blog.csdn.net/aufefgavo/article/details/120672276?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.queryctrv2&spm=1001.2101.3001.4242.1&utm_relevant_index=3)有一片较为详细的分析文章。但是大致可以总结为如下原因：

当我们面对一个很大的矩阵时，矩阵本身的维度是**cache**的`block size`的好几倍，假如`cache size`为**8\*8**，`array_size`为**16\*16**，如果我们使用朴素的矩阵转置算法，那么在在每一行转置位置超过了矩阵维度的一半时，矩阵**B**将会由于`index`部分在`[0-7]`与`[8-15]`的重合，在**cache**中发生多次`eviction`。

但是如果此时我们将矩阵分为四块，那么对角矩阵的转置将不会出现反复`eviction`的情况。

需要注意的是，如果`array_size`过小，则根本没有分块处理的必要，因为**cache**足够容纳整个矩阵，分块处理反而会增加处理时间。

而如果`block size`过小，则也不会起到很好的优化效果，原因是没有充分利用`spacial locality`. 

> 我们也可以从[数学的角度](<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060138089.pdf"/>)粗浅分析为什么**blocking**更好.

#### Some other caches

##### Branch Predictor

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124180148197.png" alt="image-20220124180148197" style="zoom:50%;" />

在先前**control hazards**的部分，简单来说，我们曾提到过`branch predictor`. `branch predictor`也是一种**cache**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124190837775.png" alt="image-20220124190837775" style="zoom:50%;" />

如果我们在该缓存区中没有找到对应的跳转地址，说明这是第一次遇到该跳转，针对这种情况，如果是`forward branch`则预测为**not taken**，如果是`backward branch`则预测为**taken**.

> `forward branch`为一般的`if`跳转指令；`backward branch`为`loop`结构的跳转.

一个简单的`branch predictor`设计如下：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124191549741.png" alt="image-20220124191549741" style="zoom:50%;" />

这里我们选择存储**10**个地址低位，由于在我们当前使用的**RISC-V**中，每个指令都是`4byte`，所以我们可以选择不存储最低的两位地址数据。我们在`history cache`中存储曾经跳转到的地址的低位，并设置一个`bit`表示先前是否跳转进入该地址。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124192119756.png" alt="image-20220124192119756" style="zoom:50%;" />

我们可以在**IF**阶段进行`branch predict`，也可以在之后的**ID**阶段进行，显然如果在**ID**阶段预测，那么紧随其后的一条指令是无论如何也会被抓取的。

关于这里的`stall`数量的问题，需要注意这里的`taken/not-taken`指的是最后的结果，而并非我们的预测。

##### Branch Target Buffer

除了**B format**的跳转指令之外，我们还有**J format**的跳转指令，显然在这里，我们也会在`pipeline`中加载不必要的指令，所以最好也进行预测，以在一定程度上减少`stall`的几率：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124204039948.png" alt="image-20220124204039948" style="zoom:50%;" />

首先，只要**jal/jalr**向寄存器`ra`中写入，我们就把下一条指令的地址`PC+4`写入一个维护的`stack`中。当我们意识到函数即将返回–`jalr`从`ra`读取并将`x0`作为目的寄存器时，就从维护的`stack`中读出顶层数据–**一定会成功的预测返回地址，因为是非条件跳转指令，只要我们不超过`stack`的存储深度**。

除了上述这种在读取`ra`内的返回地址时使用预测，在某些情况下，我们也需要在“**jalr**跳转到函数”–`jalr ra t0`的步骤，即上图中向`ra`写入数据时做预测：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124211246340.png" alt="image-20220124211246340" style="zoom:50%;" />

在上图中，如果我们使用`foo.bar()`的调用方式（可以留意以下这里对`virtual function table`的使用方式），我们可以保存一个**cache**，里边记录先前跳转的函数地址。这种想法是基于“之前跳转过的函数地址，下一次可能还要使用”。我们通过将**PC**设置为这个**cache**中的函数地址来实现预测。

#### Cache and Multiprocessors

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124213447690.png" alt="image-20220124213447690" style="zoom:50%;" />

为了避免`pipeline`中出现`structual hazards`，在每一个处理器上都有独立的**cache**。现在问题来了，如果多个处理器想要访问同一块内存要怎么办？它们各自含有的**cache**应当做出什么样的反应？

读取内存相对简单，因为虽然不同的处理器内有独立的**cache**，但是它们都应当具有相同的内容（如果此时允许被读取的话）。如果我们想要向内存中写入数据，那么需要一些额外的操作：

##### Multiple Processors writing

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124214815702.png" style="zoom:50%;" />

我们需要一个`coherency`的理念，即当我们通过一个处理器向内存中写入数据时，在一小段时间后，其他的处理器应当能够获知：数据更新了。

为了达到这个目的，我们使用一种**Broadcasting**的技巧。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124215406459.png" alt="image-20220124215406459" style="zoom:50%;" />

为了让数据可以在不同的处理器之间通信，我们可以让他们具备`shared bus`或者建立一种类似于计算机网络的传输数据结构。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124215625775.png" alt="image-20220124215625775" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124220910804.png" alt="image-20220124220910804" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124221618898.png" alt="image-20220124221618898" style="zoom:50%;" />

由此引出一种新的**cache miss**类型，即`coherence`.

##### Coherence miss

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124230957329.png" alt="image-20220124230957329" style="zoom:50%;" />

**coherence miss**出现在当一个或多个处理器想要写入某块内存，但是正有其他处理器正尝试读取该块内存内容时，因为只要存在写入，那个其他**cache**上的该块数据就应当被标记为`invalid`.

这也是为什么在现代CPU的设计中会存在一块很大的**shared cache**的原因！这避免了出现大量**coherence miss**的情况。

##### Incoherence miss

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124231848151.png" alt="image-20220124231848151" style="zoom:50%;" />

所谓`working sets`，就是在程序运行时“真正需要的(`actually really using`)”内存大小，而不是系统分配给的内存大小(或者说是程序请求的)。

### Actual CPU

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220118201902329.png" alt="image-20220118201902329" style="zoom:50%;" />

## Operating Systems & Virtual memory & I/O

在本章节中，我们将该`Memory Hierarchy`补充完整：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119161732287.png" alt="image-20220119161732287" style="zoom:50%;" />

同时，对**I/O**交互有基本的理解：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119161857195.png" alt="image-20220119161857195" style="zoom:50%;" />

### Operating System Basics

首先熟悉一些操作系统的基本知识，由于本课程并非**OS**课，详细的内容在**CS 162**中.

操作系统主要干什么？它的工作流程是什么？

1. 操作系统是计算机打开时第一个运行的“程序”
2. 在开启操作系统后，操作系统负责找到并控制所有`devices`，使用`hardware-specific device drivers`.
3. 接下来，它启动所谓的`services`，包括`file system`, `network stack(Ethernet, WiFi, Bluetooth...)`, `TTY(keyboard)`等
4. 最后，它会加载、运行和管理`Programs`.在这一步中，操作系统保证以下三个特性：

- Multiple programs at the same time (**time-sharing**)
- Isolate programs from each other (**isolation**)
- Multiplex resources between applications(所有的程序共同分享相同的系统资源)

总得来说，OS内核(在`supervisor mode`下运行的操作系统核心)完成了如下使命：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119164543469.png" alt="image-20220119164543469" style="zoom:50%;" />

为了实现**进程间通信**，**UNIX**系统提供了两种机制：`pipe`与`shared memory`，`pipe`可以允许我们以`stream`的形式发送和接收数据，而`shared memory`顾名思义是共享内存的意思。但无论如何，在使用他们之前，**OS**必须首先`set it up`.

#### What does OS need from hardware

我们先前构建的`dataPath`其实已经可以运行一部分操作系统的基本功能了，但是如果添加上用以满足以下功能的硬件，那么就可以运行一个简单的操作系统了：

1. **Memory translation**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119165640220.png" alt="image-20220119165640220" style="zoom:50%;" />

显然，许多程序，比如我们先前所使用的**Venus**都尝试使用所谓的“同一片内存区域”，为了解决这个问题，操作系统会进行**virtual address**与**physical address**的转换。这样即使对于程序本身来说，看起来是使用了同一块内存，实际上那只是`virtual address`而已，操作系统为他们分配到了**DRAM**上不同的`physical address`.

2. **Protection and Privilege**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119170110613.png" alt="image-20220119170110613" style="zoom:50%;" />

如果我们放任程序运行，他们可能会对系统本身的正常工作和运行造成干扰甚至损坏，所以OS设置了两种模式（**RISC-V**有三种）.`Supervisor mode`可以改变`memory mapping`，在这种模式下，操作系统可以将不同的内存块以不同的`memory mapping`的方式分配给不同的用户. 这是通过设置**CSR**实现的。

3. **Traps & Interrupts**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119171326883.png" alt="image-20220119171326883" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201151045243.png" alt="image-20220201151045243" style="zoom:80%;" />

这种情况我们之后会详细介绍，根据《深入理解计算机系统》，处理器会检测“状态”是否有变化，如果检测到有**事件**发生，则会通过**异常表(`exception table`)**进行一个间接过程调用(异常)，到一个专门设计用来处理这类事件的**操作系统子程序**，即**异常处理程序(`exception handler`)**。**在这门课中**，我们称之为执行`Trap handler`(被操作系统以`supervisor mode`调用)，如上图所示。

当异常处理程序完成处理后，根据引起异常的事件类型，会发生以下三种情况：

- 处理程序将控制返回给当前指令$$I_{curr}$$，即当事件发生时正在执行的指令
- 处理程序将控制返回给$$I_{next}$$，如果没有发生异常将会执行的下一条指令
- 处理程序**终止**被中断的那个程序运行

#### Control and Status Registers

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120115407503.png" alt="image-20220120115407503" style="zoom:50%;" />

需要注意的是，并非只有在`supervisor mode`下可以控制`CSR`，但是在`user level`下我们不能控制`supervisor-level CSR`.

#### What Happens at Boot

当计算机被打开时，**CPU**会从某个**预设好的**起始地址开始执行指令，这个起始地址指令一开始是被存放在**Flash ROM**中的，一般来说，我们可以直接读取**Flash ROM**中的指令内容，但是习惯上我们将它复制到**memory**(`memory-mapped I/O`)里的那个预设起始地址再开始读取并执行。当我们在启动电脑时可以进入的**BIOS**界面就是它了。

> **Flash ROM**在极少情况下需要“写”操作，比如只有在我们进行**BIOS**更新的时候才需要用到，对于他本身来说，写操作是较慢的，读操作是很快的.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119171630345.png" alt="image-20220119171630345" style="zoom:50%;" />

那么在**Boot**的时候发生了什么呢？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119171818162.png" alt="image-20220119171818162" style="zoom:50%;" />

1. 首先，计算机根据我们上述的描述，开始执行**BIOS**(`EFI firmware`)，它会找到一个存储设备(`disk`)，同时**加载找到的存储设备中的第一块数据**并开始执行其中的内容。

2. 在这**第一块数据**里存储了许多重要信息，包括**Bootloader**程序，他帮助我们执行一些最基本的操作系统函数，**将操作系统内核加载入内存并跳转到操作系统内核处**。需要注意的是，**BootLoader**可以支持**多个不同版本的操作系统的启动**（如同我们会在**BIOS**界面中见到的那样）。
3. 接下来，操作系统就开始执行它应该做的工作，正如前边所介绍的那样——找到`device drivers`并启动`services`.
4. 最后，当操作系统完成上述工作后，它会进入循环，等待用户输入，或者等待程序被启动…

### Operating System Functions

#### Launching Applications

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119174750302.png" alt="image-20220119174750302" style="zoom:50%;" />

一个进程可以含有多个线程，但无论是对于**进程**还是**线程**，由于他们的数量很多，即使是多核**CPU**也不可能仅仅在一个核内执行一个任务，为了多进程/线程运行，采用的方法正是先前在**CS 106L**课程结尾部分提到的`multithreading`机制，操作系统控制进程在核内执行一段时间后将其移出，让另一个进程占据内核执行，如此循环。当然这个过程是以极快的速度完成的。

为了`launch application`，我们通过`system call`的方式实现：对于**Linux**来说，它使用`fork`创建新的进程，并使用`ececve`加载应用。

通过调用`fork()`，原本的进程会被克隆一份，也就是变成**父进程+子进程**，创建完子进程之后，父进程仍然继续运行原本的代码，子进程用来执行新的任务。子进程把父进程的**Process Control Block(PCB)**克隆了一份，除了其中的`pid`没有复制，子进程获取了一份新的`pid`，这样一来操作系统就可以根据`pid`来区分子进程和父进程了.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120201713356.png" alt="image-20220120201713356" style="zoom:33%;" />

而加载应用的过程，因为也是在调用可执行文件，其实与我们先前对程序运行流程的**loader**部分的介绍如出一辙。

当子进程消亡后，返回.（如果在子进程还没有消亡时，父进程就消亡，则此时父进程会作为一个`zombie process`，直到子进程消亡后消失）具体的过程留到**CS 162**再解决。

> `fork()`函数被调用一次，但返回两次：一次是在父进程中，一次是在新创建的子进程中。在父进程中，`fork`返回子进程的**PID**；在子进程中，`fork`返回0.
>
> 父进程和子进程是并发执行的独立进程。内核能够以任意方式交替执行它们的逻辑控制流中的指令。我们**决不能对不同进程中指令的交替执行做任何假设**。
>
> ![image-20220203112054856](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220203112054856.png)

#### Supervisor Mode

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119182359261.png" alt="image-20220119182359261" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120163952307.png" alt="image-20220120163952307" style="zoom:50%;" />

操作系统将应用对于系统资源的使用施加了一些限制，比如对于内存以及设备等，在`user mode`下我们不能直接访问**内核代码和数据**，只能通过`syscall`**接口**来访问。如此一来即使应用崩溃了也不会对系统整体造成影响。

相比于`user mode`，`supervisor mode`下我们可以执行更多的指令，多出来的指令都是**CSR**相关的。这些**特权指令(`privileged instruction`)**包括停止处理器、改变模式位，或者发起一个**I/O**操作等等。而进入`supervisor mode`时，在硬件层面，其实我们也只是简单的翻转了几个**CSR**的相关`bits`.需要注意的是，对进程本身来说，他可以**脱离**`Supervisor mode`，但是除了使用`interrrupt`来**进入**`Supervisor mode`之外，别无他法.

当我们在`Supervisor mode`下遇到错误时，很可能对操作系统的运行产生灾难性的影响，比如在**Win**下遇到的蓝屏。

#### Syscalls

对于一些基本的底层的任务，我们不想在每一个软件中都单独设置模块来完成和实现它们，操作系统为我们提供了解决这一问题的方法，即**syscalls**.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119211302056.png" alt="image-20220119211302056" style="zoom:50%;" />

调用**syscalls**的方法也很简单，正如先前在**Venus**中使用的**ECALL**指令那样，我们通过给寄存器一些特殊的指定的`bits`来告诉操作系统执行哪一个对应的**syscalls**，而操作系统接下来会通过调用专门生成`software interrrupt`的指令来实现操作（**ECALL**）. 实际上，`software interrupt`并非属于`interrupt`的异常类型，它是由运行的程序本身产生的，并非基于硬件的中断。

借由`software interrupts`，操作系统会接管当前程序运行（进入`Supervisor mode`），在完成操作（比如通过WiFi发送数据啊…）之后会将控制权交还给`user mode`.**也就是说，`系统调用`是从`用户态`切换到`内核态`的一种方法，后边我们会看到，只要发生了中断/异常(系统调用也是一种异常)，系统需要使用`trap handler`来处理它们，便需要进入到`内核态`中去。**

也正是借由这种方式，操作系统保证了可以在同一时间内运行多个此类进程，而不会发生冲突。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119213121732.png" alt="image-20220119213121732" style="zoom:50%;" />

#### Interrupts, Exceptions

这两种不同的“中断”模式的区别如下：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119213815739.png" alt="image-20220119213815739" style="zoom:50%;" />

需要注意的是，`interrupts`并不需要我们在发生中断时立刻解决问题（丢下手头的工作），但是`exceptions`需要我们这么做！这就导致`exception`处理的硬件设计要比`interrupt`**难**。

这里我们给出了`Trap`的定义。而在先前我们提到，我们通过`Trap handler`来控制`interrupts/exceptions`.



> **按照[更为详细的分类标准](https://www.zhihu.com/question/30432536)，`interrupts`又被称为异步中断，是其他硬件依照CPU时钟信号随机产生的，又叫做硬件中断。`interrupts`/异步中断/硬件中断又可分为：**
>
> - **可屏蔽硬件中断(CPU可以延迟)**
> - **不可屏蔽硬件中断(CPU不可延迟)**
>
> **这里的`exception`为异常，也被称为同步中断，是当指令执行时CPU控制单元产生的，`系统调用`就是一种异常。包含`trap(陷阱)+fault(错误)+abort(终止)`，被称为故障指令(`faulting instruction`)。之后介绍的`page fault`就是一种`fault`异常.**
>
> **而`software interrupts`即[软件中断](https://zhuanlan.zhihu.com/p/360683396)，和复位中断、按键中断、串口中断本质上是一类，都是属于真正的中断，只不过软件中断它触发的方式比较奇怪，它是由代码触发的。**
>
> **需要注意的是，还有一种中断名为`软中断(softIRQ)`，它和软件中断不是一个东西，软中断主要用于执行系统调用即\*int 0x80\*以及给调试程序通报一个特定的事件，所以软中断是异常的一种，属于同步中断。**
>
> **这里不去深入探讨，根据百科内容：软中断的一种典型应用就是所谓的"下半部"（bottom half），它的得名来自于将硬件中断处理分离成"上半部"和"下半部"两个阶段的机制：上半部在屏蔽中断的上下文中运行，用于完成关键性的处理动作；而下半部则相对来说并不是非常紧急的，通常还是比较耗时的，因此由系统自行安排运行时机，不在中断服务上下文中执行。bottom half的应用也是激励内核发展出的软中断机制的原因。**

#### Trap Handling

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119214018646.png" alt="image-20220119214018646" style="zoom:50%;" />

> 这张图的视角是`Trap handler`，对于程序本身而言，在执行完**异常处理程序**之后，未必返回到$$I_{curr}$$.要看具体情况.

##### Precise Trap

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119215325335.png" alt="image-20220119215325335" style="zoom:50%;" />

由于`Trap handler`是软件层面的东西，既然`Trap handler`假定先前所有指令执行完成而后续指令均未开始，那么处理的重点就放到了硬件层面上，故而硬件设计要比软件上麻烦得多。

每次需要用到`Trap handler`时，将目前系统的状态（类似于函数调用，但这里我们将所有寄存器中的内容存储起来）保存起来，当`Trap handler`处理完事情之后，将状态恢复到中断还没有发生时的模样。

##### pipeline handling

需要注意的是，对于`pipeline`中的`Trap handling`，我们应当给予特别的注意，因为当在某个指令的某一阶段探测到需要进行`Trap handling`时，由于`pipeline`的存在，后续一些指令可能已经被加载，而显然这些已经被加载的后续指令不应该在`Trap handling`结束之前被继续执行。于是我们可以采用与解决**hazards**问题相同的方法，即：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119225555811.png" alt="image-20220119225555811" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119225638732.png" alt="image-20220119225638732" style="zoom:50%;" />

##### what does hardware & software do?

为了成功处理**异常**，需要硬件和软件分工合作。根据《深入理解计算机系统》，当系统启动时，操作系统分配和初始化一张成为**异常表**的跳转表，每种异常被分配了一个唯一的非负整数的**异常号(`exception number`)**，其中一部分是处理器的设计者分配的，另一部分是**OS**内核的设计者分配的。在**硬件**层面，处理器**触发异常**，通过异常表来形成适当的异常处理程序(`Trap handler`)的地址：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201161908170.png" alt="image-20220201161908170" style="zoom: 67%;" />

一旦硬件触发了异常，剩下的工作就交给`trap handler`，在**软件**中完成。在处理程序处理完异常之后，就通过一条特殊的**“从中断返回”**指令，可选地返回到被中断的程序，该指令将适当的状态弹回到处理器的控制和数据寄存器中。

所以，具体来说当发生**异常/中断**时，**硬件**做什么？

> 下边提到的`Atomic`即原子操作，意味着对于程序本身来说，所有的步骤都是**同时**发生的.

- adjust the privilege level (change some bits of the **CSR**)–>`supervisor mode`
- **Disable** interrputs
- Write the old `program counter` into the `sepc` **CSR** (It is the PC that triggered the exception or the first instruction that hasn't yet executed if an interrupt)
- Write the reason into the `scause` **CSR**
- Set the **PC** to the value in the `stvec` **CSR** (address of the `Trap Handler`, which is the single function that handles ALL exceptions and interrupts)

那么相应的**软件**又要完成什么功能呢？

- Save **all** the registers (Intent is to make the previous program think that **nothing whatsoever actually happened!**)
- Figure out what the exception or interrpt is (Read the appropriate **CSR**s and other pieces ot do what is necessary)
- Restore all the registers
- Return to the right point in execution

###### Save registers?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120145407465.png" alt="image-20220120145407465" style="zoom:50%;" />

###### Figure out what to do!

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120145448957.png" alt="image-20220120145448957" style="zoom:50%;" />

###### Now to Return

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120145542288.png" alt="image-20220120145542288" style="zoom:50%;" />

使用指令**SRET**，我们的任务回到了硬件层面，这是一个返回指令，在处理完硬件层面之后就可以装作无事发生的继续执行程序了：

- **Reenable** interrupts
- Reset back down to user level
- Restore the **PC** to the value in `sepc`
- And now the program continues on like **nothing ever happend**

结合以上描述，我们知道这一段中断的过程对于程序本身其实是不易被察觉的，当使用**ECALL**时，类似于进行了一个`function call`，在其他情况下，就好比无事发生。但是另一方面，**cache**可能会变成垃圾值，因为有别的任务（`Trap handler`）在使用内存。

> In fact, for security reasons the **OS** often needs to **flush all caches** before returning flow to the program

不仅仅是在`Trap handling interrupts/exception`时，上述规则表明，在每一次`context switch`之后我们都会在**cache**中得到大量的`compulsory miss`.

写到这里，我们可以认清一个事实，就是处理**异常**是需要耗费高昂的代价的，我们必须刷新`pipeline`，保存和恢复寄存器，同时**cache**也需要被刷新…

#### Multiprogramming/context switching

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119225837022.png" alt="image-20220119225837022" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120161858521.png" alt="image-20220120161858521" style="zoom:50%;" />

为了能够在用一个**CPU**核内执行多个进程，我们使用**context switch**的方法，简单来说，就是通过在进程间的不断跳转(去+回)的方式保证多个进程可以“同时”运行：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201164645029.png" alt="image-20220201164645029" style="zoom:67%;" />

在上图中，进程**A**和进程**C**的执行在时间上重叠，故为**并行流(`concurrent flow`)**.

在执行一个进程时，**硬件**触发`timer interrupt`，触发中断后`trap handler`执行`context switch`：

![image-20220203094840723](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220203094840723.png)

为了能够在下一次**调度器**(`scheduler`)处理时可以返回这个行程——重建行程并继续运算，在`context switch`之前我们需要先将**行程状态**保存到该行程对应的`bookkepping data structure`中，这个用于保存状态的机构被叫做**Process Control Block(PCB)**。

> 行程状态主要包括通用目的寄存器、浮点寄存器、程序计数器、用户栈、状态寄存器(**CSR**)、内核栈和各种内核数据结构，比如描述地址空间的`page table`，包含当前进程信息的**进程表**，以及包含进程已打开文件的信息的**文件表**.

在储存了上一个进程的状态之后，**调度器**加载下一个要被执行的进程的状态到指定的寄存器中，由于是新的进程，同时出于**安全因素**的考量，**cache**刷新内容，避免旧数据被利用。最后使用**sret**指令返回原先指令的下一条指令。

这种规划和执行进程的过程被称为`scheduling`.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220119231043864.png" alt="image-20220119231043864" style="zoom: 33%;" />

### I/O Basics

这里介绍一部分**I/O**的基本内容，之后会详细介绍。

#### Instruction Set Architecture for I/O

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120203006126.png" alt="image-20220120203006126" style="zoom:50%;" />

需要注意的是，计算机在**CPU**与**外部设备**之间执行**输入输出**操作有两种方法：

1. **独立输入输出**(`isolated I/O`, `port-mapped I.O`)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/Untitled-drawing-1-3-1.png" alt="img" style="zoom:50%;" />

可以看到，内存与I/O使用共同的**地址总线**与**数据总线**，但是有**独立**的**控制总线**，这意味着我们对于内存的读写(如`lw/sw`)和对I/O的读写要使用不同的内部指令+硬件设计。

2. **内存映射输入输出**(`Memory-mapped I/O`)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/Untitled-drawing-2-1.png" alt="img" style="zoom:50%;" />

此时，地址总线、数据总线和控制总线都是一样的，这也是当前比较流行的做法。我们对内存的读取操作和对I/O的操作使用相同的指令(`lw/sw`).这种模式要求**I/O设备上的设备内存和寄存器都已经被映射到内存空间的某个地址**。为了实现**CPU**对MMI/O设备的访问，相应的地址空间必须给这些设备保留，并且不能再分配给系统物理内存。这可以是永久保留，也可以是暂时性的保留。通常来说X86架构都是永久保留的，而在[Commodore 64](https://zh.wikipedia.org/wiki/Commodore_64)中，由于采用了I/O设备和普通内存之间的堆交换技术（[bank switching](https://zh.wikipedia.org/w/index.php?title=Bank_switching&action=edit&redlink=1)）,可以做到暂时性保留。

[存储器映射输入输出](https://zh.wikipedia.org/wiki/%E5%AD%98%E5%82%A8%E5%99%A8%E6%98%A0%E5%B0%84%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA):

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120203854770.png" alt="image-20220120203854770" style="zoom:50%;" />

在虚拟内存的最后部分，我们会介绍`Memory-mapped I/O`的优点。

#### Processor-I/O speed mismatch

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120203954167.png" alt="image-20220120203954167" style="zoom:50%;" />

这里我们可以得到的结论是**I/O**与**CPU**的速度不匹配，那么我们如何检测I/O的读写操作呢？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220207214933447.png" alt="image-20220207214933447" style="zoom:50%;" />

> 无论如何，**I/O**的处理绕不开**OS**本身.

##### polling

第一种方法是**轮询**。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120205441710.png" alt="image-20220120205441710" style="zoom:50%;" />

简单来说，就是循环着一直查询，直到我们发现`control register`告诉我们可以读/写了，就从`Data register`中读取/写入数据。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120210211518.png" alt="image-20220120210211518" style="zoom:50%;" />

**polling**的方法在所需的轮询次数较少时可以胜任，但是当所需的**polling**次数太多，我们会消耗CPU大量资源，这是不可接受的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120211132856.png" alt="image-20220120211132856" style="zoom:50%;" />

我们可以留意一下这里的运算过程：首先我们计算得到在可以在1s内传输16MB数据的前提下，如果想在一块16-byte的块上在1s内完成这一传输过程，那么需要达到多少的`polling`频率？接下来就可以计算出1s内`polling`占了多少个时钟周期，从而可以得出占比。

所以，我们有替代方案吗？

##### interrupts

我们可以使用`interrupts`：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120212717138.png" alt="image-20220120212717138" style="zoom:50%;" />

当然，实现`interrupts`的代价还是很大的，不像`polling`只需要一直不停的循环等待就好了。所以在实际实现中，我们采用如下的解决方案：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220120213018180.png" alt="image-20220120213018180" style="zoom:50%;" />

### Virtual Memory

#### Virtual Memory Concepts

虚拟内存是我们构建的计算机结构金字塔的下一层：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131103255991.png" alt="image-20220131103255991"  />

> **Provides program with illusion of a very large main memory: Working set of “pages” reside in main memory - others are on disk.**

具体来讲，虚拟内存提供一种**Demand paging**(**按需页面调度**)的思路：

> **Demand paging**: Provides the ability to run programs larger than the primary memory (DRAM)

从每个进程的角度出发，它们认为它可以获取全部的内存。但虚拟内存除了作为一个存储层级结构并提供上述功能外，更重要的是虚拟内存可以为不同的进程提供**保护(protection)**。这一点我们在之后会详细介绍。

而正是由于这些原因，我们通过利用虚拟内存对物理内存建立一种映射：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131104839580.png" alt="image-20220131104839580"  />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131111537496.png" alt="image-20220131111537496"  />

为了完成这种内存映射，我们需要建立**page table**.这就好比我们要从图书馆中获取书籍的过程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131113042231.png" alt="image-20220131113042231"  />

一些小细节：

- 在**page table**中存在一个`valid bit`，用于表示存储的**page**是位于**DRAM**中还是位于**disk**里
- 除了`valid bit`之外，页表中还有一些`status bit`，其中包括用于告知`how long we can check out the book`的标志位

#### Physical Memory and Storage

在《深入理解计算机系统》中，详细介绍了几种不同的存储结构。在这里，我仅仅记录一部分在课程中提到的存储方式。

##### Memory

![image-20220131120747071](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131120747071.png)

**RAM**可以分为静态RAM(**SRAM**)和动态RAM(**DRAM**)，**SRAM**被用于**cache**中，由于它的`bistable`特性，只要有电，它就会永远保持它的值。即使有电子噪音等干扰，当干扰消除时，电路就会恢复到稳定值。

我们在主存中使用的是**DRAM**，**DRAM**速度比**SRAM**慢，但是它具有更小的存储单元(`DRAM Cells`)，**排列可以更加紧密**。**DRAM**存储器对干扰十分敏感，所以每隔一段时间，与**DRAM**相连的**memory controller**就需要刷新一遍电荷，将电荷重新排布到他们应该在的位置上。在笔记本和服务器中使用的**DRAM**一般为**DIMM**，即`Dual in-line Memory`.

但不论是**SRAM**还是**DRAM**，都需要电压来保持数据，所以我们称这种存储方式为`Volatile`的。

从上图的数据中可以看出，对**nearby place**的读写耗费的时间远远小于一般情况。**DRAM**一般都支持一种`burst mode`，一次量可以获取8~16倍相对于`64 bit`的数据转移量，有点类似于**cache**中使用的`prefetch`技巧。

##### Disk(HDD-SSD)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131130049014.png" alt="image-20220131130049014"  />

常见的磁盘存储方式有**SSD(`solid state disk`)**和**HDD(`hard-drive disk`)**。**SSD**的数据读写速度远快于**HDD**，但是价格也更贵。由于在断电之后不会丢失数据，故被称为`non-Valotile`.

![image-20220131132011390](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131132011390.png)

**HDD**的结构如图所示。磁盘用读/写头来读写存储在磁性表面的位，而读写头连接到一个传动臂一端，通过沿着半径轴前后移动这个传动臂，驱动器可以将读/写头定位到盘面的任何轨道上。读/写头垂直排列，一致行动。读/写头并不直接接触磁盘，而是在磁盘表面一定距离的气垫上“飞翔”。

每个表面被分为一组**磁道(track)**，每个磁道被分为一组**扇区(sector)**，每个扇区包含相等的数据位。扇区之间的**间隙(gap)**不包含任何数据位。如果读/写头遇到了灰尘，它会停下来撞到盘面，这被称为**head crash**。为此，磁盘总是密封包装的。

![image-20220131133330812](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131133330812.png)

可以看出，**disk**的访问速度是很慢的。

在**Linux**系统中，我们可以通过`fdisk -l`来查看磁盘的扇区大小(`block size`)，为`512bytes`.

与机械硬盘相对应的固态硬盘，使用的基于**闪存**的存储技术。其中没有移动设备。

![image-20220131134114139](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131134114139.png)

一个**SSD**封装由一个或多个闪存芯片+闪存翻译层(`flash translation layer`)组成，闪存翻译层是一个硬件/固件设备，扮演与**磁盘控制器**相同的角色–将对逻辑块的请求翻译成对底层物理设备的访问。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131134625489.png" alt="image-20220131134625489"  />

一个闪存由**B**个`block`的序列组成，每个`block`由**P**个`pages`组成。

对**SSD**的读写过程值得我们注意：**数据是以`page`为单位读写的，但擦除是以`block`为单位进行的。只有在一个`page`所属的`block`被整个擦除后，才可以写这一个`page`**.不过一旦一个`block`被擦除，快中每一个`page`就可以不需要再擦除而直接写。也正是因为这种频繁的擦除过程，**SSD**会因为擦除过多而损坏。

> 根据《深入理解计算机系统》，**SSD**的**随机写**过程很慢，因为需要毫秒级别的擦除，并且有可能需要将已有的数据复制到一个新的块内。

#### Memory Manager

![image-20220131140631891](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131140631891.png)

CPU芯片上的`Memory Management Unit`利用存放在主存中的`page table`来翻译虚拟地址，该表的内容由操作系统管理。在获取一块物理地址之前，**MMU**需要先检查`if process has permission to access a particular part of memory.` 

具体来说，**Memory Manager**的主要功能如下：

- **Map virtual to physical address**
- **Protection**:

1. Isolate memory between processes
2. Each process gets dedicate ”private” memory
3. Errors in one program won’t corrupt memory of other program
4. Prevent user programs from messing with OS’s memory

- **Swap memory to disk**

1. Give illusion of larger memory by storing some content on disk
2. Disk is usually much larger and slower than DRAM

在这里，`virtual memory`也可以被视作一种**cache**，

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124232845666.png" alt="image-20220124232845666" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124233134504.png" alt="image-20220124233134504" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220124233153675.png" alt="image-20220124233153675" style="zoom:50%;" />

如果我们真的超过了可用物理内存的限度，需要借助`disk`来完成工作的话，会产生一种**thrashing**的结果，因为我们需要将数据在主存和磁盘间来回搬运，速度很慢。

#### Paged Memory

也正是基于“虚拟内存将主存视作磁盘上的地址空间的**cache**”这一想法，我们对地址提出`paged memory`的想法，将虚拟内存表示为如下的形式：

![image-20220131143025474](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131143025474.png)

上边地址格式中的`offset`告诉我们在主存/磁盘中的`page memory`中具体的地址位置；而前边的`page number`则帮助我们确定`page table`中的**对应索引位置**，让我们找到对应的**Physical Page Number**。**每一个进程都有一个独立的`page table`**:

![image-20220131143445782](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131143445782.png)

在任意时刻，**虚拟页面的（注意这里说的是`page`，不是`page table`）**集合分为三个不相交的子集：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220307161432843.png" alt="image-20220307161432843" style="zoom:50%;" />

- **未分配的**：**VM**系统还未分配/创建的页。未分配的块没有任何数据和他们相关联，所以不占内存空间
- **缓存的**：当前已缓存在物理内存中的已分配页
- **未缓存的**：未缓存在物理内存中的已分配页

![image-20220131144622431](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131144622431.png)

需要注意的是，虚拟内存的大小未必要和物理地址的大小相同。

`page table`由操作系统在`supervisor mode`下管理，虽然通过这种`page memory`的方式实现了进程之间的**isolation**，我们可以通过设置**sharing bit**来通知`the page is sharable`.

此外，在`page table`中还有一些其他的**status bit**，这是一些控制字，比如告诉我们此时是否能向对应的`page`中写入数据。

##### Where Do Page Tables Reside?

![image-20220131153111022](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131153111022.png)

我们假设`page table`中的一个`entry(PTE)`是**4bytes**. 通过计算之后会发现，我们需要的`page table`的大小只允许我们将它储存在主存里，而不能全部放在**cache**中。

而当我们面对诸如**lw/sw**这种指令时，需要跑两趟**DRAM**——先从主存中搞到`page table`，取得物理地址之后，再去到主存中取出对应地址中存储的数据。

为了减少这个过程的消耗，我们可以将经常使用的`PTE`储存到**cache**中（这可以直接利用**cache**的性质：**LRU**替换策略和每次以`block`传送的性质），而不经常使用的`pages`则被存储在`secondary memory`中—即`swap partition`里。

#### Page Fault

![image-20220131154342990](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131154342990.png)

需要注意的是，`page fault`的对象是由`page table->DRAM`，而非`VA->PA`。

如果**PTE**的`valid bit`为**0**，则表明出现了`page fault`。

- 如果还没有**分配**（**PPN/DPN**为**NULL**）该虚拟页，则在**DRAM**中分配一个新的页(`unused page`)；
- 如果虚拟页存在磁盘中，但没有**缓存**在主存中，按照当前`page table`对应位置中存储的**DPN**索引到磁盘中找到位置并将其内容取出，存储到一个`unused page`中，过程如下所示：

![image-20220131155658701](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131155658701.png)

- 如果**DRAM**中已经没有`unused page`怎么办？我们需要选择一个**牺牲页**(`page to evict`)，并根据牺牲页的`dirty bit`决定是否需要将该页的内容更新到磁盘中的`swap`分区里，如果不需要就直接丢弃。之后再更新对应的`page table`中的**PPN**为**DPN**.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220307162009796.png" alt="image-20220307162009796" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220307162027990.png" alt="image-20220307162027990" style="zoom:50%;" />

需要注意的是，当我们在`page fault`出现后更新`page table`外，还要将**VPN-PPN/DPN**的映射更新到**TLB**中。

那么我们该如何处理**page faults**呢？如同先前在OS部分简要介绍的，这是由操作系统负责的，它把**page faults**当做`exception`，使用`handler`来处理:

![image-20220131155944404](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131155944404.png)

如果我们需要将页从磁盘中加载，那么可能需要进行`context switch`来完成此任务。最终，将我们把`page`处理好后，命令需要重新执行。

> 几种不同的`page fault`类型：
>
> ![image-20220203152709473](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220203152709473.png)
>
> 出现`page fault`的原因：
>
> ![image-20220203152758225](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220203152758225.png)

#### Hierarchical Page Tables

在先前关于`page table`大小的计算中，我们只考虑了一个进程对应的页表大小。但是实际我们面对的情况是几百个进程同时运行，如果我们把所有的`page table`全部存储在主存中，那么将会耗费大量主存容量，这显然是不可接受的。

而对于`64-bit address`，这就更显得不切实际了。所以我们要如何处理`page table`的大小呢？

![image-20220131161420089](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131161420089.png)

1. 增大`page size`，相当于增加`offset`，这样一来**PN**就会减小；同时还带来一个问题，即太大的`page`意味着其中很多内存块都没有被使用。。。但是一直占着坑。
2. 使用`Hierarchical page tables`：

多级页表的好处有两个：

- 如果一级页表的某一个**PTE**是空的，则相应的二级页表根本不会存在。

> 在实际程序中，只有一小部分内存（最上边的一块与最下边的一块被使用）。但是在原本的单级`page table`中间有很大的一部分正是对应着那些基本从来不会被使用的那部分**DRAM**。每个进程都有一个`page table`，如此一来我们对`page table`的储存就会造成极大的空间浪费。
>
> 但是现在有了多级`page table`，原本那些不被使用的内存对应的`page`因为一级`page table`的对应**PTE**为空而不存在，所以大大节省空间。这种存储方式使得我们可以不在**DRAM**中储存大块的连续存储，而是使用离散存储的方式。

- 只有一级页表才需要被一直存储在主存中；虚拟内存系统可以在需要时创建或调出二级页表，只有最经常使用的二级页表才被存储在主存中。

![image-20220131170545327](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131170545327.png)

在**32**位系统中，我们分为两级`page table`，**64**位系统一般对应了四级`page table`.我们将原本**20-bit**的**VPN**，分为两个**10-bit**的**VPN**，分别对应了1级`page table`与**2**级`page table`.

当有新的用户进程开始活跃时，操作系统通过设置**SPTBR**寄存器来建立新的页表。这个寄存器保存了根页表的物理页号(**PPN**).

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131172029616.png" alt="image-20220131172029616" style="zoom:67%;" />

> 在`page table`中存储的都是**PPN**而非**VPN**，**VPN**仅会在分割虚拟地址时被拿来描述

对于**32**位的**RISC-V**来说：

![image-20220131172146951](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131172146951.png)

可以看到，在`Page Table Entry`的存储中设置了很多`status bits`:显然当$$R=0,W=0,X=0$$时我们不能够修改该`page table`的任何内容，这说明它指向下一级`page table`. 否则，这是一个`leaf PTE`(指向我们应当处理的`page`).

![image-20220131183537086](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131183537086.png)

**一个注意的问题**：

> **若采用多级页表机制，则各级页表的大小不超过一个页面.**

从知乎上*copy*一个回答：

> 作者：夏与冬
> 链接：https://www.zhihu.com/question/422716423/answer/1795166483
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
>
> 
>
> 1，为什么要对单级页表进行拆分？----> 在不涉及快表，系统按字节寻址，并且采用分页式存储管理的情况下，一个进程对应的单级页表有可能过大，甚至大到占据了内存中的很多很多连续的页框，不仅会给内存分配造成压力，而且与离散分配的存储管理的思想向违背（降低了内存利用率），**所以需要拆分页表，人们就让一组页表项刚好占据一个页框**，就可以离散存储这些分组，然后建立上层页表来管理底层页表之间的逻辑顺序。
>
> 2，顶级页表这有一张这种树状结构是怎么来的？----> 这样的拆分方式就像一棵树一样，每往上建立更高一层的页表，最高层就只有一张页表，也就是只有一个根节点，即专业术语中所说的顶级页目录表；当每往上建立一层页表后，**该最高层的页表（即根页表）如果大于一个内存页框的容量，就又会往上进行拆分，建立更高层页目录表**。
>
> 3，如果顶级页表设计成多张会怎样？----> **如果顶级页表设计成了两张，甚至多张，由于每张页表的页号都是从 0 开始计数，所以需要区分查询的是哪一张顶级页表**，这就对系统对从逻辑地址到物理地址的转换额外增加了压力和复杂性，如果不区分查询的是哪一张顶级页表，很显然页号会出现重复，这就是BUG了。
>
> 4，综上所述，顶级页表，甚至每一级的页表，都被设计成了一个内存框可以存下的样子，逻辑上是一个树状结构，符合离散存储的思想，具有内存利用率高的优点；但注意随着拆分层次变多，访存次数也会变多，需要权衡系统整体性能。

#### Translation Lookaside Buffers

![image-20220131184616572](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131184616572.png)

通过虚拟地址获取物理地址的过程中，我们首先要进行**Protection Check**，这一点在先前提到过。我们要确保想要获取的物理地址此时没有别的进程在使用（或者说正出于保护状态）、是否允许写入/读取该内存地址。如果发现**不行**，则抛出一个异常(`throw an exception`)，并让操作系统来接管接下来的事情。

我们尝试把整个过程控制在`one cycle`以内。

但是问题是，**Address translation**是十分昂贵的！

> In a single-level page table, each reference becomes two memory accesses.
>
> In a two-level page table, each reference becomes three memory accesses.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131193215668.png" alt="image-20220131193215668" style="zoom: 67%;" />

即使是在借助**cache**的帮助下，可能也要花费1-2个时钟周期；而如果我们真的需要从**DRAM**中取出对应的`page table`信息，那么可能会耗费上百个时钟周期。

于是我们想是否可以借助于**cache**的思想？避免还需要从**cache**中取出**PTE**（其实实际上，**TLB**的提出要早于**cache**）

![image-20220131190049496](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131190049496.png)

上图很清晰的表示了**TLB**的搜索过程。`tag`对应了`VPN`. 那么它是如何设计的呢？

![image-20220131190255295](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131190255295.png)

**TLB**通常具有32-128个`entry`，通常被设计成`fully-associative`。有时在较大的较为复杂的**TLB**中，我们会使用`4-8 way set-associative`来做权衡。

对于**TLB**的替换策略，一般使用的是**FIFO**或者**random**，因为对于我们目标的`one clock cycle`而言，**LRU**策略的时间成本有点高。

> “**TLB Reach**”: Size of largest **virtual** address space that can be simultaneously mapped by **TLB**

但即使是使用**FIFO**的替换策略，**TLB**的`hit rate`仍能够达到**99.9%**以上。

这里的关键点在于**TLB**可以帮助我们直接由**VA**得到**PA**，由于现代操作系统通常使用的是**多级页表**，在读取数据时(即使有`cache`也是一样)，需要多次从主存/`cache`中获取页表数据。但是依靠**TLB**的存在，我们可以直接由最终映射的虚拟地址，获取真实的物理地址，显然这大大提高了我们的效率。

##### Where Are TLBs Located?

显然，**TLB**有充足的理由把自己放在**cache**前边，因为我们想避免的是从**cache**中取出`Page Table Entry`的过程。

![image-20220131193944841](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131193944841.png)

需要注意的是，此时执行`address translation`的不是`page table`而是**TLB**：

![image-20220131194049462](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131194049462.png)

采用与**cache**中相同的策略，我们首先将`virtual address`中的**VPN**分成`TLB tag`和`TLB Index`。

> - `cache` stores the actual contents of the memory.
>
> - `TLB` on the other hand, stores only mapping. `TLB` speeds up the process of locating the operands in the memory.

##### TLBs in Datapath

![image-20220131194346688](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131194346688.png)

处理**TLB miss**的方法通常是硬件处理；在前边的操作系统简介中提到过，`page fault`也是一种异常模式，属于`trap`类型。我们提供`precise trap`，故可以通过使用`software handler`在缓存对应的`page`后恢复状态。

![image-20220131194744831](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131194744831.png)

这里的**memory controller**借由**CPU**负责控制读写**DRAM**以及**cache**。

![image-20220131205812401](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131205812401.png)

![image-20220131231130141](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220131231130141.png)

在进行`context switch`时，除了之前提到的存储寄存器状态之外，我们还需要改变**SPTBR**寄存器中的内容。而此时，由于**TLB**中存储的对应着不同的进程，我们需要将它们全部设置为`invalid`.

#### VM Performance

我们仍可以使用**cache**的评价标准来评判**virual memory**.即可以利用**AMAT**作为评价指标。

![image-20220201014921499](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201014921499.png)

![image-20220201015032526](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201015032526.png)

![image-20220201015045936](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201015045936.png)

通过计算“需要从磁盘中获取数据”这一情况的**AMAT**，我们发现$$HR_{Mem}$$必须极为贴近1，才能够**AMAT**保证达到可以接受的程度。

#### VM Tricks

##### Copy-On-Write Duplication

根据维基百科：

> "**写入时复制**"是一种计算机程序设计领域的优化策略。其核心思想是：如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，**直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者**，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是[透明](https://zh.wikipedia.org/wiki/透明)的。**此作法主要的优点是如果调用者没有修改该资源，就不会有副本（private copy）被创建，因此多个调用者只是读取操作时可以共享同一份资源。**

当我们在`virtual memory`中使用该优化策略时，该过程作如下细化：

![image-20220201193713032](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201193713032.png)

当我们生成一个子进程时，仅复制一份`page table`和对应的**状态寄存器内容**，并将`page table`的**原本**和**副本**都设置为**read-only**，只有当我们尝试向某个进程中写入时，因为此时想要写入**read-only memory**，就会在**protection check**时进入`protection fault handler`，由于知道了具体需要写入的`page`，就可以**复制一份**`page`：

![image-20220201194923638](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201194923638.png)

并将这两个`page table`中对应着`PPN`设置为**可写**.以后，如果我们还要更新**Process P**的`page 3`的内容就可以直接更新了。

如果我们需要处理一整个`page table`对应的内容，由于每一次都要经由`protection fault handler`的OS处理，所以是很慢的。但是全部使用的情况是很少的，所以我们每一次将代价控制在仅复制一个`page table`的处理还是可以接受的。

通过延迟私有对象中的副本直到最后可能的时刻，写时复制最充分的利用了稀有的物理内存。

##### Shared Memory Communication

有三种方法可以实现不同进程间的通信：

1. 文件（要经过磁盘，很慢）
2. **pipe**：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
3. **shared memory**–**相同的物理内存可以对应着不同的虚拟内存**，也即“共享内存”：

###### Linux虚拟内存系统

Linux为每个进程维护了一个单独的虚拟地址空间：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220208103111867.png" alt="image-20220208103111867" style="zoom:50%;" />

一个对象可以被映射到虚拟内存的一个区域，要么作为*共享对象*，要么作为*私有对象*。如果一个进程将一个共享对象映射到它的虚拟地址空间的一个区域内，那么这个进程对这个区域的任何写操作，对于那些也把这个共享对象映射到它们虚拟内存的其他进程而言，也是可见的。而且，这些变化也会反映在**磁盘**的原始对象里：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220307164905766.png" alt="image-20220307164905766" style="zoom:50%;" />

另一方面，对于一个映射到私有对象的区域做的改变，对于其他进程来说是不可见的，并且进程对这个区域所做的任何写操作都不会反映在**磁盘**的对象里；私有对象正是使用`Copy-On-Write`技术被映射到虚拟内存中：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220307165028916.png" alt="image-20220307165028916" style="zoom:50%;" />

此外，信号(`signal`)以及分布式中的套接字(`socket`)也是可以实现进程通信的方式，这里不详细描述。

##### Memory Mapped I/O

上一节提到，一直从磁盘中读写文件是很慢的，因为每次读取一行，都要跑到磁盘中读取一下。所以我们是否可以借助缓存的思想，每一次转移一大块文件内容到内存中，减少前往磁盘的次数？

当然可以的，`virtual memory`就可以用于实现这个目的——因为它一次将一个`page`的内容搞到内存中，这也是我们使用前边介绍的**Memory mapped I/O**带来的一个潜在益处 ：

![image-20220201202514660](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220201202514660.png)

关于`mmap`这个函数，它要求内核创建一个新的虚拟内存区域，并将文件描述符`fd`指定的对象的一个连续的片映射到这个区域：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/Image.png" alt="地图"  />

这里的一个关键点是映射的页面在被引用之前实际上并没有被载入物理内存。因此`mmap()`可用于实现将页面**延迟加载到内存中**（`Demand Paging`），这可以帮助我们节约内存和时间。

此外，`mmap`函数还可以将文件操作的速度大大提高，正如我们先前所说，传统的文件读写方法每次在磁盘和内存之间进行数据交换，涉及大量系统调用，其使用的`read()`，`write()`函数也需要对内容进行错误检查，并且他们的**数据读取**是需要通过Linux中的高速页缓存的等等：

这一小节的内容涉及一些Linux文件系统和内核的知识，我还没有专门学过，所以日后有机会再来补充吧。（[这里](https://www.cnblogs.com/huxiao-tee/p/4660352.html)好像有一篇不错的`mmap`的介绍文章）但之所以这种I/O被称为`Memory-Mapped I/O`，也是因为它把文件当作内存来用。

同时，`page table`也支持`dirty bit`，当我们写入文件时，如果需要该页换出时，`dirty bit`被设置，则将其页内容写入磁盘中；如果没有，则直接忽略这一个`page`的存储内容。

### I/O

这部分大部分的内容存储在[该文件](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/lec20.pdf)中，不再博客中单独列出，仅就一部分关键知识点记录：

#### Bus & Switch

两种不同的`interconnect`结构：

![image-20220202185501146](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202185501146.png)

![image-20220202185517039](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202185517039.png)

需要注意的几个点：

1. 总线式设计需要考虑电线本身的电阻等品质，不能够过长
2. `switch`设计可以避免`bus`设计时`one at a time`的特点，可以做到**同时上下/同层交流**，所以也就有更好的表现。并且，由于这种电线的分离式设计，导线长度不会成为较大的干扰因素
3. `switch`可以做到分离式控制
4. 由于`switch`式设计有更好的性能表现，故当前的**ETHERnet, PCle**等都是用了这种`processor interconnect`结构.

#### PCI & USB

> **PCI-e**一般用于计算机内部，不同模块之间的“高速数据通信”。注重的速度和稳定性，**USB**主要用于外接设备，注重的是兼容性和简洁性。

![image-20220202191237433](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202191237433.png)

> **simplex**意为`one direction`

这种`switch point-to-point`的传输每次仅传输一个`bit`，但是他的速度是很快的！

![image-20220202191349235](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202191349235.png)

**USB**可以被组织为一个`bus/hub`，并不是一个`switched interconnect`，`hub/bus`不可以**终止一个连接**。**USB**使用同一条导线来发送和接收数据，所以不能够设计过长的长度，否则延迟也是一个很大的问题。但是虽然它不比**PCle**般可以并行传输数据，并且不支持过长的设计长度，但是他所建立的连接是**极为稳定和鲁棒的**。

#### DMA

我们的目的是不想让**CPU**把所有的事情都做了，因为数据传输的过程原本也由处理器全部控制，**CPU**要负责把数据在内存和设备之间来回搬运。现在，我们尝试让**CPU**聚焦在`computing`上。数据传送的工作交给**DMA controller**。

![image-20220202192528488](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202192528488.png)

**CPU**需要向`DMA Engine`提供读取/写入地址、数据大小、转移方向等等信息。总得来说，**CPU**负责`initiate the data transfer`.数据传送过程如下图所示：

![image-20220202194001521](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202194001521.png)

> 在后边我们会提到，**disk**中靠近`disk controller`处一般都有一个`cache-like structure`，或者称之为`buffer`，我们将数据储存在其中，以增快数据传送的速度。

当数据传送结束后，`disk controller`发送给`DMA controller`一个**ACK(`Acknowledgement`)**信号，`DMA controller`再给**CPU**一个中断以表明传输结束。

![image-20220202194539948](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202194539948.png)

![image-20220202194607727](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202194607727.png)

> confirm that externl device is ready **By reading its CSR**

##### Where to put DMA?

![image-20220202195532229](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220202195532229.png)

#### Disks

这部分内容参见课件。

#### Networking

本部分参见[此文件](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/lec21.pdf)。

根据维基百科：

> **互联网协议套件**（英语：Internet Protocol Suite，缩写IPS）[[1\]](https://zh.wikipedia.org/wiki/TCP/IP协议族#cite_note-1)是网络通信模型，以及整个[网络传输协议](https://zh.wikipedia.org/wiki/网络传输协议)家族，为[网际网络](https://zh.wikipedia.org/wiki/网际网络)的基础通信架构。它常通称为**TCP/IP协议族**（英语：TCP/IP Protocol Suite，或TCP/IP Protocols），简称**TCP/IP**[[2\]](https://zh.wikipedia.org/wiki/TCP/IP协议族#cite_note-2)。因为该协议家族的两个核心协议：**TCP（[传输控制协议](https://zh.wikipedia.org/wiki/传输控制协议)）**和**IP（[网际协议](https://zh.wikipedia.org/wiki/网际协议)）**，为该家族中最早通过的标准[[3\]](https://zh.wikipedia.org/wiki/TCP/IP协议族#cite_note-3)。由于在网络通讯协议普遍采用分层的结构，当多个层次的协议共同工作时，类似计算机科学中的[堆栈](https://zh.wikipedia.org/wiki/堆栈)，因此又称为**TCP/IP协议栈**（英语：TCP/IP Protocol Stack）[[4\]](https://zh.wikipedia.org/wiki/TCP/IP协议族#cite_note-4)[[5\]](https://zh.wikipedia.org/wiki/TCP/IP协议族#cite_note-5) 。

这里对**TCP**和**UDP**进行了一些介绍，[这里](https://chinese.freecodecamp.org/news/tcp-vs-udp-which-is-faster/)有一些相关的说明。

#### 内存I/O延迟

最后，再来提一嘴内存**I/O**。根据[这里的文章](https://zhuanlan.zhihu.com/p/86513504)，内存延迟有四个关键参数：

- **CL**(Column Address Latency）：发送一个列地址到内存与数据开始响应之间的周期数
- **tRCD**（Row Address to Column Address Delay）：打开一行内存并访问其中的列所需的最小时钟周期数
- **tRP**(Row Precharge Time)：发出预充电命令与打开下一行之间所需的最小时钟周期数。
- **tRAS**(Row Active Time)：行活动命令与发出预充电命令之间所需的最小时钟周期数。也就是对下一次预充电时间进行限制。

> 需要注意的是，在实际计算时，**tRAS**很可能会`cover`掉**CL+tRCD+tRP**，因为**tRAS**往往较大，在一次发出行活动命令->预充电->访问列->取出数据后，我们不能立即做下一次的数据读取和存储，正是因为**tRAS**限制了两次预充电命令的时间间隔。

也正是因为这些个数据的存在，使得内存的**随机I/O**要比**顺序I/O**慢出不少，因为如果行活动指令给出的定位不同，我们必须重新加载（预充电）等等之后才能做新的**I/O**操作。

## Parallelism

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061448998.png" alt="image-20220206144828813" style="zoom:50%;" />

最后一部分内容聚焦并行计算。

### Flynn Taxonomy, SIMD Instructions

本章节内容以**dgemm**(`double-precision floating-point general matrix-multiply`)为切入点，介绍并行计算的基本问题和基本方法。

我们一般的矩阵表示模式如下：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052203828.png" alt="image-20220205220340654" style="zoom:50%;" />

我们可以将矩阵$$a[][]$$表示为`row-major`：

$$a_{ij}:a[i*N+j]$$

或者`column-major`:

$$a_{ij}:a[i+j*N]$$

**Fortran**使用`column-major`的表示方法，所以在示例中我们遵循`column-major`的表示方法。

下边开始对**dgemm**进行逐步优化：

- **Python**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052220773.png" alt="image-20220205222044616" style="zoom:50%;" />

- **C**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052221978.png" alt="image-20220205222143876" style="zoom:50%;" />

但我们还可以做得更好！

#### Why Parallel Processing?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052224427.png" alt="image-20220205222401315" style="zoom:50%;" />

对于“并行”，我们一般有两种基本方法：较为简单的是`Multiprogramming`，即并行运行多个项目。而更难实现的是`Parallel computing`，即如何让**一个程序**运行的更快。

#### Flynn Taxonomy

接下来，我们有几种计算模式：

- **SISD**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052226883.png" alt="image-20220205222631754" style="zoom:50%;" />

- **SIMD(`sim-dee`)**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052227343.png" alt="image-20220205222714227" style="zoom:50%;" />

- **MIMD(`mim-dee`)**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052227436.png" alt="image-20220205222750320" style="zoom:50%;" />

- **MISD**

No examples today.

而所谓的**费林分类法(`Flynn's Taxonomy`)**正是根据`information stream`分为指令和数据两种，又将计算机类型分为了如上四种。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052232441.png" alt="image-20220205223215316" style="zoom:50%;" />

#### SIMD Mode/history

![image-20220205223324250](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052233332.png)

**SIMD**处理器的发展也经历了诸多变革：

![image-20220205223700815](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052237916.png)

其中，**AVX**，即高级向量扩展指令集也经历了如下变革。

![image-20220205223721139](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052237255.png)

[这里](https://www.expreview.com/73093.html)有一篇资料详细介绍了其发展历程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052242335.png" alt="image-20220205224205105" style="zoom:50%;" />

简略来说，我们单独准备了一套**SIMD**寄存器，比如对于`256-bit SIMD register`来说，我们可以输入4个`64-bit`的数据，这样便可以并行计算这4个数据：

![image-20220205225310771](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052253865.png)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052307271.png" alt="image-20220205230659790" style="zoom:50%;" />

![image-20220205225340813](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052253895.png)

#### SIMD on matrix multiplication

> 可以查阅x86 **SIMD** "Intrinsics"

在先前我们对比了C与Python写出的**degmm**代码的**FLOPS**性能，虽然C较Python的提升较大，但是我们可以通过计算`Peak double FLOPS`来判定我们距离目标还有多远：

假设我们使用的是**i5-5557U**的处理器，则其`Clock rate`为**3.1GHz**，对于`double`类型的数据，从上面的图中可以看出，一条指令可以允许我们进行**4**次算术运算。

$$3.1GHz\times inst/cycle\times mults/inst=24.8GFLOPS$$

所以截至目前，我们还离预期很远。所以我们尝试利用**AVX-256**指令做些事情：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052331745.png" alt="image-20220205233155648" style="zoom: 67%;" />

或者也可以写成如下形式：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052341135.png" alt="image-20220205234142030" style="zoom: 67%;" />

我们将上述代码表示成图的形式：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052340351.png" alt="image-20220205234008237" style="zoom:50%;" />

通过上述代码，我们达到了一条指令可以处理**4**个数据的目的，所以预想大概是会快了四倍：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052342928.png" alt="image-20220205234228831" style="zoom:50%;" />

但是这离目标还很远啊！

#### Loop Unrolling

`loop unrolling`可以帮助我们继续提高性能表现！

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052345399.png" alt="image-20220205234541240" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202052348057.png" alt="image-20220205234814954" style="zoom: 67%;" />

我们在循环内部设置一个小循环（从而外部循环的递增模式也要随之改变，每次递增4个）。这样做的好处有：

1. 注意到该实现中，存在许多的**dependencies**，比如`mul`依赖于`load`，`add`又依赖于`mul`。这就会产生**data hazards**，我们必须在**pipeline**中做多个**stall**已解决问题，显然大大降低了性能表现。而当我们设置一个**UNROLL**的内部小循环之后，由于编译器可以提前得知**UNROLL**的值，故编译器会将这个小循环的代码**展开**，并使用先前提到的另一种解决**data hazards**的方法，即**reorder**——(`e.g. load, load, load, load, mul, mul, ..., add, ...`)并利用处理器的**super-scalar**结构提升运算效率！
2. 由于内部**UNROLL**循环的存在，$$i$$每次增加的值就不仅仅是**4**了，而是$$UNROLL\times 4$$.这会降低我们进行**条件跳转判定**的次数，进而减少发生**control hazards**的可能性！

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060121394.png" alt="image-20220206012151293" style="zoom:50%;" />

现在，当$$N$$不太大时，结果已经接近最佳，但是在最后却出现了`fall off a cliff`，这是为什么呢？

#### Memeory access strategy - blocking

首先，假设每个矩阵的每一个元素都只需要从**DRAM**中获取一次：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060129542.png" alt="image-20220206012939391" style="zoom:50%;" />

显然，这是一个不错的假设，因为$$\frac{2}{3}N$$的结果意味着后续大量的数据运算都不需要再从内存中搬运了。但是事实果真如此吗？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060131947.png" alt="image-20220206013120835" style="zoom:50%;" />

事实上，我们需要`ld/sw`的次数绝对不止每个元素一次。这就带来一个问题：如果我们每次都要从**DRAM**中搬运数据，将会产生不可接受的时间代价！这也是为什么**cache**的相关优化十分重要了。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060133945.png" alt="image-20220206013322787" style="zoom:50%;" />

这就回到了先前在**cache**部分介绍过的**Blocking Matrix Multiply**方法（实际上是分治思想的运用）：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060134294.png" alt="image-20220206013441163" style="zoom: 67%;" />

再看下此时的性能指标表现：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202060135870.png" alt="image-20220206013552758" style="zoom:50%;" />

在$$N$$为**960**时的性能断崖不复存在了。

### Thread-Level Parallelism

#### Amdahl's Law

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061314850.png" alt="image-20220206131418662" style="zoom:50%;" />

上图中的最后一个等式就是阿姆达尔定律的数学表示形式。阿姆达尔定律告诉我们，即使我们对系统中的**并行**部分做再多的功夫来提升其性能，最终`bottleneck`还是在于系统中剩下的**串行**部分的表现。如果**串行**部分占比很大，那么即使将**并行**部分优化到极致，对于系统整体性能的提升效果也不会很明显：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061317416.png" alt="image-20220206131722325" style="zoom:80%;" />

所以我们当然想把整个系统都变成并行运行的。

阿姆达尔定律在现实中也有许多应用场景：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061318511.png" alt="image-20220206131831361" style="zoom:50%;" />

#### Parallel Computer Architectures

为了提升计算机的性能表现，我们有如下几种方式：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061447591.png" alt="image-20220206144725462" style="zoom: 67%;" />

首先，关于**时钟频率**，这里多提几句。根据[这里的文章](https://zhuanlan.zhihu.com/p/84194049)，我们有**核心频率**，**数据频率**和**时钟频率**这三个常用频率。其中，**核心频率**是内存电路的振荡频率，是内存工作的基石。时钟频率用于传输数据；核心频率用于颗粒内部对这些数据的处理，比如发送数据前，需要对颗粒bank激活、然后读取数据，然后预充电，这些数据处理相关的工作，都是工作在核心频率下。其余两个频率是在**核心频率**的基础上，通过各种技术手段放大出来的。其中**数据频率**是**时钟频率**的两倍，因为在时钟信号的上沿和下沿均可以传输数据：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202071435562.png" alt="image-20220207143544439" style="zoom:50%;" />

而**核心频率**本身受制于工艺技术问题，已经多年没有实质性进展，因为随着晶体管尺寸越来越小，半导体工艺的提升逐渐逼近极限，单纯提升芯片的工作频率越来越困难。



**multiprocessor(`multicore`)**的结构意味着我们有多个独立的处理器，其中各自具备一套完整的`Datapath+control`，除了下图所示的结构外，我们注意还需要花费一些功夫来处理先前提到的**cache coherency miss**问题：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061339775.png" alt="image-20220206133910642" style="zoom:50%;" />

- Each processor has its own **PC** and executes an independent stream of instructions(**MIMD**)
- Different processors can access the same memory space, each “core” has access to the **entire memory in the processor** (*Processors can communicate via shared memory by storing/loading to/from common locations*)

一般来讲，使用**multicore**的方式有两种，一是每一个核负责**独立**的工作(`job-level parallelism`)；第二种是**对于同一个程序，将其运行在多个核中**(`parallel-processing program`).第二种正是我们目前打算研究的主题，如每个核负责一部分矩阵乘法运算。

另一方面，所谓的**parallelism**的方式也有两种，即我们提到的**SIMD**与**MIMD**。

> - A **SIMD-favorable** problem can map easily to a **MIMD-type** fabric
> - A **MIMD-favorable** problem will not map easily to a **SIMD-type** fabric

并且，**SIMD**的设计相比于**MIMD**要更加简单，且性能更高！

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061537374.png" alt="image-20220206153728269" style="zoom:50%;" />

##### Strong and Weak Scaling

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061540741.png" alt="image-20220206154056612" style="zoom:50%;" />

##### Multiprocessors & You

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061542261.png" alt="image-20220206154204140" style="zoom:50%;" />

这里需要注意的是，如果我们目前处在`scale up`的状态，那么一定可以`scale down`：比如一个运行在**12**核的程序一定可以也可以在**4**核处理器上体现出它的并行优势，但是如果程序本身就是最多**4**核运行，我们想让它在**12**核处理器内取得较高的性能表现并不是那么容易。

#### Threads

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/9faf605ebd46e68d125f5f5ed76495cc_1440w.jpg" alt="img" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220208125241398.png" alt="image-20220208125241398" style="zoom:50%;" />

线程是最小的执行单元，而进程由至少一个线程组成。如何调度进程和线程，完全由操作系统决定，程序自己不能决定什么时候执行，执行多长时间。线程又分为**软件线程**和**硬件线程**；

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061603956.png" alt="image-20220206160348825" style="zoom:50%;" />

进行到这里，我们来解决两个问题：

1. 进程与线程的关系(区别)
2. 硬件线程与软件线程(由操作系统调度的)有什么联系

首先，关于进程与线程，[这里的回答](https://www.zhihu.com/question/25532384)说的比较清楚：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061634278.png" alt="image-20220206163434142" style="zoom:50%;" />

在多线程环境下，每个线程拥有一个**栈**和一个**程序计数器**。栈和程序计数器用来**保存线程的执行历史和线程的执行状态**，是线程私有的资源。其他的资源（比如堆、地址空间、全局变量）是由同一个进程内的多个线程共享。



其次，关于硬件线程和软件线程：

- 软件线程，可以分为内核线程(`klt`)，即**operating system threads**，以及用户线程(`ult`)。这里我们主要讨论`klt`，`klt`会被**映射**到硬件线程上，但由于硬件线程的数量有限，所以某些软件线程会处于**waiting**的状态，那么软件线程的工作机制大致如何呢？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061758927.png" alt="image-20220206175812818" style="zoom: 67%;" />

- 我们会在两种情况下将某一些软件线程从硬件线程上换出：`blocked threads`以及按照固定时序要求的线程置换（这也是我们能够在单核非超线程的硬件机制下执行多线程程序的原因，即**time sharing**）:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061656687.png" alt="image-20220206165625588" style="zoom:50%;" />

>  线程的切换也使用了类似于进程切换的**context switch**的方法，但是需要保存的状态内容相对较少。

- 关于硬件线程，我们不想让**CPU**在遇到**cache miss**后等待数据传送过来的过程中什么也不做，所以想要切换到别的线程上，而硬件线程的设计可以帮助我们更好的解决这个问题——由于硬件线程结构设计的存在（`redundant hardware`），我们无需在每次`thread switch`时保存对应的`context`.

- 正是基于上述原因，每一个处理器内核可以提供一个或者多个线程，这里的“线程”指的是硬件线程。比如目前我们常在Intel的处理器指标里看到`2 threads/core`，这种设计又被Intel称之为**超线程技术(`Hyper-Threading(HT)/Simultaneous Multithreading(SMT)`)**。在一个处理器内核中有多套PC寄存器等：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061745461.png" alt="image-20220206174550312" style="zoom:50%;" />

> 当超线程技术处于激活状态时，**CPU**会在每个物理内核上公开两个执行上下文。这意味着，一个物理内核现在就像两个“逻辑内核”一样，可以处理不同的软件线程。
>
> 如上图所示，超线程技术也共享某些资源，比如**cache**,**execution units**等等.

截至目前，我们知道**multithreading**有两种实现方式：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202061954018.png" alt="image-20220206195454887" style="zoom: 80%;" />

##### ult & klt

前边提到，软件线程又可以分为**内核线程**以及**用户线程**。之后我们使用的**openMP**由于基于**GCC**，使用的就是`klt`。简单来说，**用户线程**通过某种手段映射到**内核线程**上，**内核线程**再被操作系统映射到**硬件线程**上得以实际运行。

`ult`与`klt`之间存在多重映射关系，包括**一对一**、**多对一**、**多对多(N-M)**，以及**两层设计(多对多 & 一对一)**方式。在**Linux**中使用的是**一对一**的映射模式.（下图表示的是多对一的映射模式）

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220208125200403.png" alt="image-20220208125200403" style="zoom:50%;" />

> 根据[这里的回答](https://www.zhihu.com/question/462958378)，并且在**Linux**中比较特殊的一点是，**Liunx**内核不提供任何特殊的调度信息和数据结构来表示线程。线程**本质**上就是一个进程，只不过该进程和其他进程共享某些资源。这一点可以从**Linux**“线程”的创建和普通进程的创建上得到验证：进程在内核由对应的 `task_struct ` 数据结构表示，创建线程调用的 `clone()` 函数不过是指定了创建的新的进程中父子进程共享了更多的资源，如地址空间、文件资源、文件描述符和信号处理函数。

关于进一步的描述这两者之间的联系，可以看这几个参考资料：

- [1](https://www.cnblogs.com/Survivalist/p/11527949.html),[2](https://www.geeksforgeeks.org/relationship-between-user-level-thread-and-kernel-level-thread/),[3](https://www.geeksforgeeks.org/why-must-user-threads-be-mapped-to-a-kernel-thread/)

但简而言之，**用户线程**需要映射到**内核线程**，因为内核将线程调度到**CPU**上执行，因此它必须知道它正在调度的线程。对于一个简单的**进程**，**内核只知道进程的存在**，**而不知道在其中创建的用户线程**，因此内核只会将进程的线程（这是内核线程）调度到**CPU**上，进程中的其他用户线程则必须一一映射到指定给创建进程的内核线程上（具体的映射模式可以是多对一，一对一等等）。



**用户线程**和**内核线程**的优劣各自互补：`ult`由于是在`user mode`下由用户程序直接进行管理，在进行**I/O**操作时需要频繁切换到内核态；也正是由于此原因，线程间通信也需要支付额外的系统调用成本。而`klt`虽然拥有较高的执行权限且**I/O**无需进行系统调用(因为本身就在`supervisor mode`下)，但是它创建的时候需要进行系统调用，且相比`ult`其扩展性要差一些。

#### Latest modern processors: Big/Little design

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202202062003931.png" alt="image-20220206200328832" style="zoom:50%;" />

> - efficiency cores: energy efficienct
> - Why M1 Pro has only **1** thread/core?
>
> 1. Simultaneous multithreading only works well when the twos have same working set.
> 2. The chip size improves about 6%~10%, which will have bad effects on the **cache**.

### OpenMP

关于**OpenMP**的使用，这里不单独列出。

### Data Races & Synchronization

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209132611267.png" alt="image-20220209132611267" style="zoom:50%;" />

一个线程尝试向内存中写入数据，那么此时最终的结果是不确定的！（无论另一个线程是想要向内存中写入数据还是读取数据）这取决于哪个线程先完成它的工作。

为了解决`Data Race`，我们可以使用**lock synchronization**。基本思想是在一个线程已经读取该位置并且尝试改变两个线程的共享变量之前，留下一个`note`，即加一个**锁**。如果另一个线程此时也尝试读取/写入该共享变量，我们就告诉它**等一等**，等前一个到达的线程把**锁**解开之后你再来处理该变量。如此一来，我们的行为就是确定性的了：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209134257501.png" alt="image-20220209134257501" style="zoom:50%;" />



一个简单的**lock synchronization**逻辑如下：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209134844021.png" alt="image-20220209134844021" style="zoom:50%;" />

#### Possible Lock Problem

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209134927646.png" alt="image-20220209134927646" style="zoom:50%;" />

如何解决上图所说的这种问题？

#### Hardware Synchronization

如何处理上述问题？这需要硬件与软件的结合工作。这里，我们主要介绍软件层面的指令方案：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209135259822.png" alt="image-20220209135259822" style="zoom:50%;" />

##### RISC-V option one

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209135811624.png" alt="image-20220209135811624" style="zoom:50%;" />

> 只要在`load reserved`之后有对该内存位置的读/写，`reservation bit`就会变为**invalid**。接下来的`store conditional`就会失败（即不发生）。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209140040331.png" alt="image-20220209140040331" style="zoom:50%;" />

我们可以利用这两个指令来完成*`Test-and-Set`*的过程，即实现**lock operations**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209140156819.png" alt="image-20220209140156819" style="zoom:50%;" />

```assembly
li t2, 1
Try: lr t1, s1          # load state & set a reserve bit
     bne t1, x0, Try    # check if unlocked?
     sc t0, s1, t2      # if unlocked, try to own & lock
     bnez t0, Try       # lock operation successful?
Locked:
    # critical section
Unlock:
    sw x0, 0(s1)
```

##### RISC-V option two

**Atomic Memory Operations(AMOs)**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209142351081.png" alt="image-20220209142351081" style="zoom:50%;" />

> 所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何**context switch**（切换到另一个线程）。 

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209142906053.png" alt="image-20220209142906053" style="zoom:50%;" />

#### Deadlock

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209144239015.png" alt="image-20220209144239015" style="zoom:50%;" />

#### Limiting Parallelism

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209144446940.png" alt="image-20220209144446940" style="zoom:50%;" />

### Shared Memory & Caches

#### Multiprocessor Key Questions

- **Q1** - `How do they share data?`

Single physical address space shared by all processors/cores

- **Q2** - `How do they coordinate?`

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209150159471.png" alt="image-20220209150159471" style="zoom:50%;" />

- **Q3** - `How many processors can be supported?`

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209150611062.png" alt="image-20220209150611062" style="zoom:50%;" />

#### Let's talk about caches again

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209150737191.png" alt="image-20220209150737191" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209151650664.png" alt="image-20220209151650664" style="zoom:50%;" />

##### MOESI

在这里，我们引入了**MOESI**模型：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209154311182.png" alt="image-20220209154311182" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209154324169.png" alt="image-20220209154324169" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209154413631.png" alt="image-20220209154413631" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209154432974.png" alt="image-20220209154432974" style="zoom:50%;" />

需要注意的是，进入**owner**状态的途径除了自身循环外，只有**modified**这一种状态。当**cache**处于**owner**状态时，它与其余的(也包含该数据的)**cache**组成了`dirty sharing of data`：

> This cache is one of several with a valid copy of the cache line, but has the exclusive right to make changes to it. It must broadcast those changes to all other caches sharing the line. The introduction of owned state allows **dirty sharing of data**, i.e., **a modified cache block can be moved around various caches without updating main memory**. The cache line may be changed to the Modified state after invalidating all shared copies, or changed to the Shared state by writing the modifications back to main memory. Owned cache lines must respond to a snoop request with data.

**MOESI**模型帮助我们实现了`lr`与`sc`指令的工作机制：对于`lr`来讲，我们只需要做一次与往常一样的`load`操作即可，之后**cache**处于**E/S/M/O**状态中。

而在`sc`之前（这时要写入操作），由于使用了`write-back`的写机制，我们必须保证对应的内存位置当前被保存在**cache**中。那如果**cache**的状态为**E/M**，我们只需要向**cache**中写入提供的新数据即可；如果**cache**此时位于**S/O**的状态下，那么我们需要将写入操作做`broadcast`. 

自然地，当处于**S/O**状态下，我们需要做设计避免出现两个**cache**在同一个`cycle`内写入数据的情况，这也是为什么要保证多核处理器的正确工作并非易事。

##### Sharing & False Sharing

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209160558775.png" alt="image-20220209160558775" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220209161834415.png" alt="image-20220209161834415" style="zoom:50%;" />

上图所示的情况，就是**伪共享(`False Sharing`)**.此时的**cache**会处于我们先前提到的`Block ping-pongs`状态中，出现**Coherence Miss/Communication Miss**。为了避免这种情形的出现，我们应当尽量避免几个不同的**cache**向同一个**cache line**中写入数据。在并行程序中，除非我们要使用线程共享变量，否则最好将不同的线程写入的位置控制不同的**cache line**中。

### MapReduce & Spark

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212173722596.png" alt="image-20220212173722596" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212173752606.png" alt="image-20220212173752606" style="zoom:50%;" />

### Warehouse Scale Computers

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212173957394.png" alt="image-20220212173957394" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212174116926.png" alt="image-20220212174116926" style="zoom:50%;" />

#### Power Usage Efficiency

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212174358744.png" alt="image-20220212174358744" style="zoom:50%;" />

显然，**PUE**的值会大于**1**，但是超过**1**的部分越小，说明电能使用效率越高。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212175552416.png" alt="image-20220212175552416" style="zoom:50%;" />

## Dependability

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211231325693.png" alt="image-20220211231325693" style="zoom:50%;" />

这里我们主要解决`Runtime Failures`.主要思路是利用**redundancy**带来的**dependency**解决问题。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211231831101.png" alt="image-20220211231831101" style="zoom:50%;" />

在上图中，多个`logic block`就是使用**Redundancy**的实例，简单来说，这种硬件结构提供了一种`voting`机制：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211232233675.png" alt="image-20220211232233675" style="zoom:50%;" />

### Time vs. Space

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211232307030.png" alt="image-20220211232307030" style="zoom:50%;" />

### Dependability Measures

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211232357939.png" alt="image-20220211232357939" style="zoom:50%;" />

根据上图所示的计算公式，我们一般采取两种方式来衡量可靠性：

1. number of 9s of availability per year

   <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211232706545.png" alt="image-20220211232706545" style="zoom: 33%;" />

2. Annualized Failure Rate (AFR, average number of failures per year)

我们将大致的`Failure Rate`随时间的变化绘制成曲线：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220211233142617.png" alt="image-20220211233142617" style="zoom:50%;" />

### Error Detection/Correction Codes (EDC/ECC)

由于我们在计算机中所使用的主存为**DRAM**，这种存储方式会在每一个`bit`对应的位置上依靠少量电荷来存储`bit`.如此一来，就会产生两种**error**：

- “**Soft**” errors occur occasionally when cells are struck by alpha particles or other environmental upsets
- “**Hard**” errors can occur when chips permanently fail

> Problem gets worse as memories get denser and larger.

#### Parity Bit

而如今的内存正是使用**EDC/ECC**的策略来应对上述问题。基本的思路是除了我们用于存储数据的`data bits`之外，增加一些用于判定是否出错以及具体出错位置的`bits`：即所谓的**Parity Bit(s)**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212002621574.png" alt="image-20220212002621574" style="zoom:50%;" />

上图所示的方法是一种很基础的`Error Detection`方法，即**奇偶校验位**。我们通过增加一个`0/1`的奇偶校验位的方法来确保从前到后的**1**的次数是**偶数个**。如果在**检测阶段**，我们发现**1**额次数变成了奇数个，那么就说明出现了错误。

而硬件上实现此操作的方法也很简单，我们需要两个**XOR**，第一个异或门用于确定`Parity Bit`是**0**还是**1**；第二个异或门用于检查有没有发生错误：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212003246432.png" alt="image-20220212003246432" style="zoom:50%;" />

需要注意的是，在这种简单的**奇偶校验位**的错误检测方法里，我们只能够探测到**odd numbers of errors are detected**的情况，如果翻转的`bit`为偶数个，那么无法检测到。另一方面，即使我们检测出发生了错误，那么我们也**无法更改错误内容(无法定位)**。

所以我们有没有更高级的**EDC/ECC**方法呢？这里引入`Hamming Code`.

#### Hamming Code

##### Motivation

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212003653608.png" alt="image-20220212003653608" style="zoom:50%;" />

基本的思路是从$$2^N$$个模式中选出$$2^M$$个作为`valid code words`，除此之外，所有的**中间状态**均为`invalid code words`。那么如果在检查时我们发现面对的是一个`invalid code word`，那么可以根据一定的准则来找到原本的正确的数据，这也就是我们所说的**ECC**。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212011527805.png" alt="image-20220212011527805" style="zoom:50%;" />

我们可以用如下的形式表述`Hamming Distance`以及**ECC**的过程：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212012645021.png" alt="image-20220212012645021" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212012625156.png" alt="image-20220212012625156" style="zoom:50%;" />

上图表明，为了能够探测出**1-bit error**，我们必须仅使用**000**与**111**这两个`Hamming Distance`为**3**(**Minimum Hamming Distance is 3**)的数字作为`valid code word`。能够探测到的`bit errors`共计两位。（从一个`valid code word`走到下一个`valid code word`总共需要**3**次）

> - **2-bit Error Detection**
> - **1-bit Error Correction**

这种情况也代表着**Hamming code**的`Minimum Hamming Distance`为**3**,因为数据位最小只有一个，在`Hamming code`中我们需要两个`Parity Bit`来检测错误，恰好为上图所示情况。

需要注意的是，虽然我们可以检测出仅仅发生`1-bit error`的数字所对应的正确值是多少，但是如果允许出现`2-bit error`，我们仅能够探测到，但是并不能够确定它对应的正确值是哪一个。

##### Rules

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212122403327.png" alt="image-20220212122403327" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212122457424.png" alt="image-20220212122457424" style="zoom:50%;" />

我们将`Parity Bits`排列在`1,2,4,8,16...`的位置，穿插在原本的`Data Bits`中。这些`Parity Bits`将整个的数据分成了不同的组，我们要保证没一个组内满足**even parity**；分组规则上图所示，需要注意的是，**Hamming code**中的数字标识是**自左向右**的。和先前一样，在硬件结构上，我们只需要在这不同的组别中使用**异或门**即可。

一个栗子：

- 计算`Parity Bits`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212123959661.png" alt="image-20220212123959661" style="zoom:50%;" />

- 检查错误(`check`):

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212124055952.png" alt="image-20220212124055952" style="zoom:50%;" />

接下来的问题是，在计算出如上图所示的**检查**结果后，我们如何确定需要更改的(发生错误的)位置。

根据线性代数：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212124248122.png" alt="image-20220212124248122" style="zoom:50%;" />

当然，我们也可以从另一个角度尝试解释这个运算方法，我们可以将上图倒转过来便会发现，从`Parity bit 8 Group`到`Parity bit 1 Group`，组成的`1010`的数组序列恰好对应了横轴上`Bit Position`为`10`的位置：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212124545185.png" alt="image-20220212124545185" style="zoom:50%;" />

##### Hamming ECC Cost

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212124855653.png" alt="image-20220212124855653" style="zoom:50%;" />

我们正是通过$$2^r\ge m+r+1$$来计算**冗余**位数。

##### Bonus Slides

目前介绍的**Hamming code**是：

- **Single Error Detection**
- **Single Error Correction**

> 如前所述，在立方体的例子中提到的`2-bit Error Detection`并不等同于`Double Error Detection`。原因我们先前也描述过，当允许同时出现两个位置的错误时，我们无法得知其对应的正确值是哪一个。

我们可以通过如下方法来进行**Double Error Detection**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212131736701.png" alt="image-20220212131736701" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212131752293.png" alt="image-20220212131752293" style="zoom:50%;" />

在现代的**DRAM**设计中，我们正是采用了这种**DED**的错误探测机制：对于`64-bit`的`Data Block`来说，根据公式$$2^r\ge m+r+1$$我们需要**7**个`Parity Bit`来处理`Hamming code`；那么对于**DED**，就要使用**8**个`Parity Bit`。所以`code word`共计`72-bit`。

##### other codes

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212131841231.png" alt="image-20220212131841231" style="zoom:50%;" />

### RAID

**EDC/ECC**可以帮助我们解决内存中的**错误**，即通过一些`Redundancy Bits`建立一套规则来提升`Dependability`。

同样地，在**disk**中我们也可以通过**Redundancy**来形成`High Data Availability`:

> Service still provided to user, even if some components (disks) fail.

**RAID**全名为**Redundant Array of Independent Disks**，简称磁盘阵列，常见的方案有：`RAID0`、`RAID1`、`RAID5`、`RAID6`、`RAID10`等。

**RAID**的基本思路是将同一份文件`stripe`到不同个磁盘中，这样即使某一个磁盘发生了数据损坏，我么也可以根据其他磁盘中的内容保证文件不发生丢失。

#### RAID0

这种存储方式其实并不算做真正具备`Redundancy`，它只是将数据分散到了不同的磁盘中：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212140632321.png" alt="image-20220212140632321" style="zoom:50%;" />

不过由于我们可以并行处理多个磁盘内容，这种数据分布方式要比将文件全部存储在同一个磁盘中更快。

#### RAID1

这种方式又被称为**Disk Mirroring**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212140834203.png" alt="image-20220212140834203" style="zoom:50%;" />

#### RAID 2-4

这三种方式主要基于`Data Striping + Parity`的模式，分别使用了不同的数据大小`stripe`在磁盘中：

- **2：bit**
- **3：byte**
- **4：sector(block)**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212144616798.png" alt="image-20220212144616798" style="zoom:50%;" />

**RAID2**使用多个`check disks`，这种设计完全基于**ECC/EDC**的思路，通过`check disks`确定错误位置。而在**RAID3**中，考虑到当前的磁盘本身就带有错误检测机制，所以在已知出错磁盘的前提下，我们可以仅使用一个`Parity Disk`，如果某个磁盘出错，将其余`Data disks`和`Parity Disk`进行**异或**即可得到正确数据：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212145217434.png" alt="image-20220212145217434" style="zoom:50%;" />

于是在**RAID4**中我们更进一步，将`sector/block`作为一个`disk`的存储单位，这允许我们进行更快速的数据并行和存取操作：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212145456216.png" alt="image-20220212145456216" style="zoom:50%;" />

如果尝试向磁盘阵列中写入数据，则需要更新**data disk**以及**parity disk**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212145613310.png" alt="image-20220212145613310" style="zoom:50%;" />

#### RAID5

所以我们的**RAID4**存在什么问题？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212145735751.png" alt="image-20220212145735751" style="zoom:50%;" />

为了能最大程度利用数据并行处理的优势，并且避免同一块磁盘因为过于频繁的数据读写出现损坏，我们最好不要把对于`Parity Bit`的读写集中在一个磁盘中。正是基于这种想法，**RAID5**使用了`Interleaved Parity`的办法，减少更新同一个`Parity Disk`的可能性：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212152231149.png" alt="image-20220212152231149" style="zoom:50%;" />

#### Newer RAID

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220212152309784.png" alt="image-20220212152309784" style="zoom:50%;" />


本文总结了当前常见的LLM与LRM并行策略模式，从LLM和LRM的区别出发，基于经典的并行模式结合业务模型梳理几种不同并行策略在搜广推场景下的应用。

一些todo：

- zero
- 并行
- LLM与LRM的区别
- transformer与MOE，以及字节内部对其做的改造（为什么MOE可以节省推理成本）

## LRM与LLM的区别

我们可以从特征以及模型结构两个方面来看待这个问题：

### 特征

对于LLM模型来讲，其基本是全dense模型（不像LRM模型那样拥有较多的sparse参数），这是因为：

- 从传统的判别式推荐来看，搜广推模型在做 f(user_id, item, context) -> score；但是LLM做的是P(next_token | previous_tokens)
- LRM的记忆实体为user_id -> embedding, LLM为knowledge -> weights

当然，现在LRM也在尝试采用生成式推荐的范式，比如将用户特征编码成token，让模型自回归的预测下一个token（物品）；或者像快手onerec一样，采用e2e的模型结构替代传统的级联架构（召回-粗排-精排）。

正是因为LRM要建立这种f(user_id, item, context) -> score的KV映射，导致其存在大量的sparse参数，同时user_id和item_id等又是离散实体，这导致LRM实际上需要建立一种**离散KV映射**，同时需要用embedding table这种大词表来存放这些巨大的离散KV；而LLM中因为不需要保留这种需要长期记忆的离散KV实体，导致模型基本都是dense特征。

LLM唯一能接受的“稀疏”部分就是MOE，只是MOE的稀疏特性体现在其计算路径上，而非参数索引上。

### 模型

相比LLM几乎全部是batch gemm以及transformer的模型结构，LRM模型存在更多小模块，这也与其sparse特征息息相关：因为其访问稀疏、不连续等特点，需要更多小结构来处理。

## Recap：Transformer & MOE

> Cite: https://www.youtube.com/watch?v=ugWDIIOHtPA&list=PLJV_el3uVTsOK_ZK5L0Iv_EQoL1JefRL4&index=61 

## 参数量-并行策略选型

对于当前的LLM模型，采用dense LLM (即所有参数每个token都参与计算)的模型，其参数量无法增加到特别大的程度：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-01-20-140004.png" alt="2025-12-19-150427" style="zoom:50%;" />

> 截止2025年年末，字节当前LRM模型最大的也就能跑到10B左右，100B探索中

LLM模型在dense架构下，参数量达到100B-200B时就遇到了显著的：

- 训练成本爆炸
- 推理性能下降
- 通信成本增加

所以转而采用MOE架构。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-01-20-135903.png" alt="image-20251222233722535" style="zoom:50%;" />

MOE架构的出现大大增加了LLM系统的并行复杂度，并被LRM广为借鉴。下边我将从最基本的DP/DDP开始，从以下几个方面逐个阐述当前常用的并行策略：

- **参数量选型**：在什么规模的模型参数下，选择何种并行策略较为合适
- **并行策略解释**：从工程原理的角度解释并行策略的优缺点
- **LLM和LRM在该并行策略下的相同与不同**：对于某种特定的并行策略，LLM与LRM在使用上是否因为模型/数据的不同，有比较大的差异
- **核心代码实现**：这里对fsdp/hsdp着重展开，侧重于对zero系列的分析

### 参数量选型

首先我们要明确一个基本问题，就是模型的存储都包含哪些部分：

- **参数**：可以使用BF16存储
- **activation**：可以使用BF16存储
- **梯度**：可以使用BF16存储，但是在做类似梯度累积的操作时，需要使用FP32
- **优化器状态**：对于Adam优化器，需要存储M+V，这些状态必须是FP32的

有了以上背景，我们可以对参数量对并行策略的影响有一个基本的认知，拿最简单的DDP为例：因为参数全部是replicate，我们按照参数占整个80G显存（H100）的(2/(2\*3+2\*4)=1/7)来算，参数最多可以有10G左右，那也就是1B的大小。所以一般参数量小于1B时，使用DDP就是最佳方案：显存放得下，同时不需要额外的通信开销。

在参数量大于1B时，我们有两种策略可以考虑：

- **FSDP**：即zero3，参数全部分片存储
- **HSDP**：机内fully-shard，机间复制

这里涉及到一个基本的知识点，即Zero显存策略优化：

#### Zero系列显存策略优化

Zero有两篇论文值得一读，一个是deepspeed原始的Zero论文，首次提出了zero1,2,3的架构，还有一篇论文是Meta PyTorch团队在FSDP上的工作，偏工程实践一些。这里把两篇论文的解读也放在这里：

##### ZeRO: Memory Optimizations Toward Training Trillion Parameter Models

> https://arxiv.org/pdf/1910.02054 

1. 问题1: 显存占用是如何分配的？

在常见的混精度训练中，一般分为如下几份：bf16的参数+bf16的梯度+fp32的M/V (对于Adam优化器)+fp32的参数拷贝，大概是2+2+12=16的显存占用；

2. 问题2: zero的三个阶段分别是什么？

- Zero1: 分片optimizer状态
- Zero2: 分片梯度
- Zero3: 分片参数

通过上边对显存的占用分析，我们可以知道这几种方案对显存的节省程度如下图：
<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-01-20-141747.png" alt="image-20260120221747577" style="zoom:50%;" />

3. **这几种不同的zero策略，对通信的开销是多大？**

这也是整个zero系列最关键的问题。

要讲明白这个问题，我们要先从DDP开始，DDP的allreduce的通信开销是多大？

> 参考文章：https://zhuanlan.zhihu.com/p/504957661

4. 如何提高通信效率？

做梯度分桶，每个worker负责几个bucket（layers），由于我们可以根据网络梯度计算顺序对梯度bucket做拓扑排序，每个worker上的计算图均按此执行，我们可以在每个worker上的属于shard i的梯度ready之后，参与将梯度reduce到shard i的reduce scatter（当然有的worker reduce scatter的早，有的晚，取决于对应worker分到的bucket位于拓扑图中的哪个位置）。

这样做的好处在于

- 每个worker处理完不属于自己的bucket的梯度之后就可以将其丢弃，节省显存；
- 更好的通信 计算overlap；

###### allreduce通信开销

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-05-18-141918.png" alt="img" style="zoom:50%;" />

当前最先进的allreduce通信通过两阶段完成：reduce-scatter + all-gather；即每个shard将收到所有属于自己这部分的梯度， 在本地加好，再通过一个allgather让其他所有shard也能收到自己加好的这一份，最终每个shard上都保留了完整的做了reduce-sum的整体梯度。这两个操作均可以通过环形通信算法实现（Ring-Allreduce），具体流程可以看上边贴的知乎链接，结论很重要：

对于p个设备，大小为V的矩阵(每个设备上有V/P大小的矩阵)；假设双工通信，出入口带宽均为$\beta$,

1. 所有设备之前传输的**总数据量**为$2*(p-1)v/p*p=2*(p-1)*v$，如果p足够大，近似于$2*pv$.
2. 因为在同一轮内，每个设备在发送数据的同时也可以收到数据，整个过程需要的时间是$(p-1)v/p\beta$，如果p足够大，近似于$v/\beta$，与设备数无关；

> 我们这里的通信时间只考虑传输带宽，而没有考虑每次传输都包含的延迟（latency）。当数据量V比较大时，延迟项可以忽略，上文的分析就是成立的。当 V 特别小，或者设备数 p 特别大时，带宽就变得不重要了，反而是延迟比较关键，这时更好地实现就不是环状算法了，而应该使用树状通信。
>
> 这也是为什么英伟达 NCCL 里既实现了ring all-reduce，也实现了 double-tree all-reduce 算法.

因为我们这里要对比的就是allreduce本身带来的通信开销，假定设备数量为常量，那么allreduce的通信开销和v成正比。

- **Zero1**: 在做参数更新后，由于每个worker只存了属于自己的那份优化器状态，所以需要通过一次allgather收集**更新后的参数**就好，这和原本的allreduce无异；通信开销仍为2pv;

- **Zero2**: 每个worker只存自己的优化器状态+梯度，这要求我们将allreduce拆开为两阶段：在拿到各自worker的梯度后先通过一次reduce scatter，将梯度在对应worker上累加，再做all-gather收集更新后的参数；可以看到，通信开销和先前仍无区别；

  > 虽然这里和DDP一样也是reduce_scatter+all_gather,但是这里all_gather的是参数，而非梯度;梯度已经是通过reduce_scatter之后分片的了

- **Zero3**: 每个worker保存自己的优化器状态+梯度+参数，这要求我们在前向和反向都添加额外的通信操作：在前向计算前，通过一次allgather收集参数：为了计算通信的overlap，我们仍然采用分桶策略，总通信开销为$v/p*p*p$；这样的allgather操作在反向也有一个，同时再加上梯度的reduce scatter，总共需要3pv的通信，也即正常方案的1.5倍；

##### PyTorch FSDP: Experiences on Scaling Fully Sharded Data Parallel

> https://arxiv.org/pdf/2304.11277

待补充。

#### 搜广推场景下的并行策略

千卡训练：16 （DDP）* 8 （FSDP）* 8 （TP）

这里重点看两种组合并行策略：

- HSDP+TP
- HSDP+EP

##### 为什么搜广推模型基本没有PP并行

- 搜广推的模型不像LLM一样，是比较均匀切分的多层transformer堆叠，而是存在很多小结构（比如dense tower），想通过PP将不同layer均匀切分到不同卡上是一件困难的事情。
- 搜广推的模型没有大到必须使用PP才能在显存上放得下的程度，牺牲bubble带来的性能损失不值得。

##### 什么时候适合选用TP

对于超过1B的模型，一般选用HSDP，随着参数量增加，分组内FSDP参数通信的开销增加，在分组内采用TP有可能避免较大的参数通信。但问题在于：

1. TP会引入activation的通信，所以需要评估activation和param的通信量；
2. 由于FSDP的通信可以和计算overlap，所以只有TP的activation的通信量显著小于FSDP需要的param通信量时，才有可能有收益

所以一般只有在10B以上的模型，使用TP才有优势。

##### 搜广推中的TP

不同于LLM中TP的数据在rank间replicate，在我们的场景下，TP rank间的数据仍然是在batch纬度切分的，也就是即使配置了(2, 8)的(dp, tp)并行，输入数据仍然是在world_size上不同的数据。原因是在搜广推场景中，除了transformer，还有大量的参数量少的结构，不适合切分，如果TP rank间数据replicate，在这些结构上就存在大量的冗余计算，带来很多计算浪费。

所以，对于不开启TP的module仍然采用数据并行；对于开启TP的module，在module内根据需要自动做AllGather收集各卡数据。

##### HSDP+TP

- 不太适合开启TP的例子

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-05-20-183340.png" alt="image-20260521023339757" style="zoom: 50%;" />

- 适合开启TP的例子

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-05-20-184418.png" alt="image-20260521024417902" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-05-20-184445.png" alt="image-20260521024444762" style="zoom:50%;" />

对于这个适合开启TP的perTokensAFFN结构，我们来分析一下：

###### TokenMixer


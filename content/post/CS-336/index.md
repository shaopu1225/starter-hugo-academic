---
title:		CS 336
subtitle:	CS 336
summary:	notes of Stanford CS336 Language Modeling from Scratch 
date:		2026-06-25
lastmod:	2026-06-25
author:		shaopu
draft: 		false
type:		book
image:		  
  focal_point: ''
  placement: 2
  preview_only: true

tags:
    - course
    - Machine Learning System

categories:
    - CS course notes
---

## Basics

- 资源量的计算：两个方面：memory & compute

total_flops公式：$6*token\_num*param\_num$ (对每一个输入的token ，前向要跑过所有的参数，每一个参数都要参与矩阵乘，每个元素都需要经过一次加法和乘法，所以前向需要2TP，反向需要对输出和权重各做一次相同的操作，所以一共需要6TP)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-27-045537.png" alt="image-20260627125537371" style="zoom:50%;" />

- Scaling Law的建立：用小模型拟合出scale分布曲线（scaling recipe），要求建立比较细致的recipe，包括BS改变的影响等；这是因为一次大规模资源量消耗太大，不可能不断通过大型实验来发现最佳超参；

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-27-113316.png" alt="image-20260627193316207" style="zoom:50%;" />

## Tokenization

Encode <-> Decode

- 可以通过增大词表(vocab size)的方法，来提高压缩比（string -> indices）；（因为每个token可以表示的信息可能会变得更多），这会导致：

  A. 序列长度更短（对attention友好）

  B. sparsity增大，因为有很多embedding可能都不会被学到，这不是好事

### BPE_tokenizer

**思路**：在原始数据上训练tokenizer，得到一个贴合数据的vocabulary；最终让常见的序列可以用一个token来表示，不常见的序列用很多个token来表示；

**方法**：一开始将每个byte当作一个token，之后不断merge常见的相邻的tokens。

```python
def merge(indices: list[int], pair: tuple[int, int], new_index: int) -> list[int]:  
    """Return `indices`, but with all instances of `pair` replaced with `new_index`."""
    new_indices = []  
    i = 0  
    while i < len(indices):
        if i + 1 < len(indices) and indices[i] == pair[0] and indices[i + 1] == pair[1]:
            new_indices.append(new_index)
            i += 2
        else:
            new_indices.append(indices[i])
            i += 1
    return new_indices

  
@dataclass(frozen=True)
class BPETokenizerParams:
    """All you need to specify a BPETokenizer."""
    vocab: dict[int, bytes]     # index -> bytes
    merges: dict[tuple[int, int], int]  # index1,index2 -> new_index
  
  
def train_bpe(string: str, num_merges: int) -> BPETokenizerParams:  
    Start with the list of bytes of string.
    indices = list(map(int, string.encode("utf-8")))  
    merges: dict[tuple[int, int], int] = {}  # index1, index2 => merged index
    vocab: dict[int, bytes] = {x: bytes([x]) for x in range(256)}  # index -> bytes
    for i in range(num_merges):
        # Count the number of occurrences of each pair of tokens
        counts = count_adjacent_pairs(indices)  
        # Find the most common pair
        pair = max(counts, key=counts.get)  
        # Merge that pair
        new_index = 256 + i  
        merges[pair] = new_index  
        vocab[new_index] = vocab[pair[0]] + vocab[pair[1]]  
        indices = merge(indices, pair, new_index)  
    compression_ratio = get_compression_ratio(string, indices)  
    return BPETokenizerParams(vocab=vocab, merges=merges)

def count_adjacent_pairs(indices: list[int]) -> dict[tuple[int, int], int]:
    """Return a dictionary mapping each adjacent pair of tokens in `indices` to the number of times it occurs."""
    counts = defaultdict(int)
    for index1, index2 in zip(indices, indices[1:]):
        counts[(index1, index2)] += 1
    return counts

  
class BPETokenizer(Tokenizer):
    """BPE tokenizer given a set of merges and a vocabulary."""
    def __init__(self, params: BPETokenizerParams):
        self.params = params
    def encode(self, string: str) -> list[int]:
        indices = list(map(int, string.encode("utf-8")))  
        # Note: this is a very slow implementation
        for pair, new_index in self.params.merges.items():  
            indices = merge(indices, pair, new_index)  
        return indices
    def decode(self, indices: list[int]) -> str:
        bytes_list = list(map(self.params.vocab.get, indices))  
        string = b"".join(bytes_list).decode("utf-8")  
        return string
```

## Resource Accounting

### Einops

- motivation: 因为使用普通的pytorch矩阵行列变换操作很容易出错，einops方便我们对dimension进行操作；

```python
x = torch.ones(3, 4)
y = torch.ones(4, 3)

# --- sum ---
z = x @ y
# 等价于
z = einsum(x, y, "seq1 hidden, hidden seq2 -> seq1 seq2")

z = x @ y.transpose(-2, -1)
# 等价于 (或者手动将...写成'batch')
z = einsum(x, y, "... seq1 hidden, ... seq2 hidden -> ... seq1 seq2")

# --- reduce ---
y = x.sum(dim=-1)
# 等价于
y = reduce(x, "...hidden -> ...", "sum")

# --- rearrange ---
# (3, 8) -> (3, 2, 4)，或者反过来也可以
x = rearrange(x, "...(heads hidden1) -> ... heads hidden1", heads=2)

```

### Flops of matmul operation

x: (B, D), w: (D, K) 每个位置包含一次加法和乘法，故flops为$2*BDK$

- B是点的数量
- (DK)是参数的数量

那么对于一次前向的矩阵乘法，flops就是$2\*tokens\*params$，也是前边整体flops计算公式的由来。

### MFU

MFU: Model FlOPS Utilization = actual flop_per_second / promised flop_per_second

> promised flop per second: 可以在设备指标中找到

一般大于等于0.5的MFU就算是很好了.

#### arithmetic intensity

两个组件：计算单元& memory，所以计算耗时取决于两个因素：

1. Accelerator speed (FLOP/s)
2. memory bandwith (bytes/s)

衡量程序是compute boundh还是memory bound有两种方法：

首先可以比较communication time和compute time：

- communication time: bytes / h100_bytes_per_second
- compute time: flops / h100_flop_per_second

我们假设通信和计算可以overlap，那么：

- memory bound: communication time > compute time
- compute bound: compute time > communication time

另一种等价的衡量方式：

- Accelarator intensity: h100_flop_per_second / h100_bytes_per_second
- Arithmetic intensity: flops / bytes

那么：

- memory bound: Accelerator intensity > arithmetic intensity
- compute bound: Arithmetic intensity > accelerator intensity

##### example: matmul

bytes: 2nn+2nn+2nn

Flops: nn(2n-1)

只要matrix足够大，就是一个compute bound的操作。

> 为什么推理过程是memory bound的？因为推理大多做的是matrix-vector multiplication.
>
> ```python
>     n = 1024
>     x = torch.ones(n, dtype=torch.bfloat16, device=cuda_if_available())
>     w = torch.ones(n, n, dtype=torch.bfloat16, device=cuda_if_available())
>     y = x @ w
>     bytes = (2 * n) + (2 * n * n) + (2 * n)  # Read x, read w, write y
>     flops = n * (2 * n - 1)  # n dot-products
>     arithmetic_intensity = flops / bytes  # ~1 
> ```
> 
>H100_accelerator_intensity >> arithmetic intensity

#### roofline plots

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-142651.png" alt="image-20260630171554556" style="zoom:50%;" />

## Architecture

### Layernorm

#### Pre-vs-post norm

将$x_i$->$x_{i+1}$的通路称为residual stream，目前主流的方法是右侧的pre-norm，即在mha和FFN之前做layer norm. 因为不在residual stream上，所以也被称为non-residual norm.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-144243.png" alt="image-20260630224243612" style="zoom: 50%;" />

优势：

- 即使不经过warmup，也可以有相比post-norm更好的稳定性，更少的gradient spike现象。

现在还有一种方法是在计算之后也加上layernorm：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-144922.png" alt="image-20260630224921419" style="zoom:50%;" />

也被叫做double norm.

#### Layernorm vs. RMSNorm

观察这两者的公式区别：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-151120.png" alt="image-20260630231120523" style="zoom:50%;" />

RMSNorm相比layernorm，有着更少的操作（不需要计算mean）和更少的参数（没后bias term），事实上这两个diff对性能的提升至关重要：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-151325.png" alt="image-20260630231325814" style="zoom:50%;" />

可以看到虽然normalization和element-wise的bias term计算的flop很少，但是占用的时间却很长。

> 在现代的LLM transformer架构中，bias term也经常是没有的.

### Activations

一个architecture设计的经验之谈：gating往往很有帮助；其实就是一个矩阵乘法。

比如在activation的设计中，从relu到**GLU的演化就是多了一个gate function:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-155855.png" alt="image-20260630235855229" style="zoom:50%;" />

各种GLU的不同就在于新增加的参数矩阵V的选择的不同，对于目前最通用的SwiGLU，选择的是一个$sigmoid(x)$:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-155954.png" alt="image-20260630235954155" style="zoom:50%;" />

这里的一个小细节是为了让总参数量和之前一样（因为增加了一个新的权重矩阵v），对dim需要进行缩减。

### Serial vs. Parallel layers

GPTJ ,PaLM, GPT-NeoX等模型提出了将原本序列化运算的transformer结构改造成parallel的：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-06-30-160636.png" alt="image-20260701000635971" style="zoom:50%;" />

但目前不常用。

### 位置编码

参考资料：

> - https://kazemnejad.com/blog/transformer_architecture_positional_encoding/
> - https://zhuanlan.zhihu.com/p/721032991
> - https://spaces.ac.cn/archives/8231
> - https://zhuanlan.zhihu.com/p/642884818
> - https://mp.weixin.qq.com/s/-1xVXjoM0imXMC7DKqo-Gw
> - https://kexue.fm/archives/8265/comment-page-2
> - https://mp.weixin.qq.com/s?__biz=MzA3MTgwODE1Ng==&mid=2247484826&idx=1&sn=8935f0bcb2e09f438cbf3ae63825d671&chksm=9f26a069a851297f568ba7cd111082e603108716928b8444a253457233f24d09d3a18447d6b9&cur_album_id=3199751010206973953&scene=189#wechat_redirect

- 为什么要有位置编码？

因为attention结构本身无法捕捉token顺序。

位置编码有以下几个要求：

1. 能够表示一个token在序列中的绝对位置；
2. 能够用绝对位置表示token间的相对位置；
3. 具有外推性，即可以表示模型在训练过程中没有见过的长度；

#### Sinusoidal位置编码

正余弦位置编码的思路来自于位置本身的二进制表示，提供了一种有界又连续的编码方法：
<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-07-03-145206.png" alt="Sinusoidal position encoding" style="zoom:50%;" />

- 为什么$\omega_k = \frac{1}{10000^{2k / d}}$?

为了满足设想：相关距离越远的embedding，相关性应该越小；也即远程衰减性：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-07-03-145552.png" alt="image-20260703225551965" style="zoom:50%;" />

- 绝对位置编码如何表达相对位置信息？

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-07-03-145704.png" alt="image-20260703225703942" style="zoom:50%;" />

#### ROPE

对于二维向量：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-07-03-150032.png" alt="image-20260703230032401" style="zoom:50%;" />

即在向量上乘上一个旋转矩阵，同样地对于多偶数维向量，可以将其两两分组(注意这里的$\theta$对每个d的值是不同的)，我们接下来会证明为什么这个形式是可以表达相对位置信息的；

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-07-03-150123.jpg" alt="Image" style="zoom:50%;" />

上式中的旋转矩阵十分稀疏，为了节省算力，可以以下面的方式等效实现：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2026-07-03-150253.jpg" alt="Image" style="zoom:50%;" />

上述公式里$\theta$的取值可以复用先前正余弦位置编码的方法，同样带来一定的远程衰减性。

- ROPE是如何用绝对位置编码表示相对位置信息的？

我们考察二维下的ROPE，注意到其相当于在embedding上乘了一个旋转矩阵，设旋转矩阵为$R$，那我们尝试证明：

$$<R_aX,R_bY>=<X,R_{b-a}Y>$$

注意到旋转矩阵的性质：

1. $$R_a^T=R_{-a}$$
2. $$R_aR_b=R_{a+b}$$

则：

$$<R_aX,R_bY>=(R_aX)^TR_bY=X^TR_a^TR_bY=X^TR_{b-a}Y=<X,R_{b-a}Y>$$

那么对于高维向量，由于内积具有线性性质，即$<a,b>=a_0b_0+a_1b_1+a_2b_2+a_3b_3+...=<a^0,b^0>+<a^1b^1>+...$，其中$a^0=[a_0,a_1]$，以此类推；所以将高维向量做两两分组并分别应用旋转矩阵后，上述在二维空间推导出的性质仍然成立。

- ROPE为何具有外推性？

本质上是因为旋转矩阵的存在，让位置编码具备了**周期性**和**远程衰减性**，这两个性质允许我们做类似线性插值（将推理时没有见过的旋转角度恢复到训练时见过的角度范围内），以及后续的优化高频信息的NTK插值等方法，通过缩小旋转弧度$m\theta_i$达到长度扩展的目的，具体参见参考文章的最后一篇内容。




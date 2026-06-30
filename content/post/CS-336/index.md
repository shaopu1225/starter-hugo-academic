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

<img src="/Users/bytedance/Desktop/tool-code-lib/blog_repo/starter-hugo-academic/content/post/CS-336/image-20260630171554556.png" alt="image-20260630171554556" style="zoom:50%;" />

## Architecture






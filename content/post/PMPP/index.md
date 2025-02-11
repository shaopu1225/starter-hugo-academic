---
layout:		post
title:		PMPP Reading notes
date:		2025-02-11
author:		shaopu
header-img:	img/code.png
catalog:	true

tags:
   - C++
   - Parallel Computing
   - CUDA
---

本篇文章更新自己在阅读PMPP book的过程中记录的有趣的点。

## Chapter 3: Scalable parallel execution

1. `__syncthreads()`会barrier一整个block中的所有threads，需要注意的是，如果在if-else branch中的每个分支都出现了该barrier，那么我们需要保证线程之间不会互相等待。
2. 线程在被分配资源时，按照block-by-block的顺序分配。资源被组织在SM（Streaming Multiprocessor）上。每个设备会从以下几个方面限制资源的使用率：

- 每个SM允许分配的最大block数量
- 每个CUDA device上处于活跃状态的block数量
- 每个SM允许分配的最大线程数量

3. 




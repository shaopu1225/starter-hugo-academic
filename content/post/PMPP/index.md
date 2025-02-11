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
2. 线程在被分配资源时，按照**block-by-block**的顺序分配。资源被组织在SM（Streaming Multiprocessor）上。每个设备会从以下几个方面限制资源的使用率：

- 每个SM允许分配的最大block数量
- 每个CUDA device上处于活跃状态的block数量
- 每个SM允许分配的最大线程数量

3. 在线程被block-by-block的分配到SM上以后，会按照warp的方式调度。同一个warp中的线程有相同的执行时间。每一个SM可以同时执行一小部分warp，这里有一个问题是：为什么SM没有能力在同一时刻执行所有的warp，但是我们仍然需要很多warp？这是为了通过调度来掩盖long-latency ops，比如全局内存访问。

这种调度方式带来的另一个好处是我们不需要像CPU那样准备很多很大的cache，从而可以把chip上更多的区域出让给浮点数计算单元等。






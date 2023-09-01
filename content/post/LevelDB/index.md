---
title:		LevelDB Analysis
subtitle:	LevelDB Analysis
summary:	LevelDB Analysis
date:		2021-03-13
lastmod:	2023-08-31
author:		shaopu
draft: 		false
image:		  
  focal_point: ''
  placement: 2
  preview_only: true

tags:
    - Database
    - LevelDB

categories:
    - Open Source Frameworks
---

很久没有更新博客了，自从搞完`61C`就懒得更新了，这段时间发生了很多很多事情，但总体上还是在一个萝卜一个坑的在朝着成为一名合格的计算机科学从业者的方向努力着。

本文写在学习完`CMU 15445`和`MIT 6.S081`两门课程之后，由于觉得数据库这个东西实在太抽象，很多地方不落到代码上忘的很快，而且`15445`作为一门基础课程有一些东西没有涵盖到，所以决定把`levelDB`啃下来，不管以后自己是不是做存储方向，哪怕复习一下很久没写的C++也是好的。

首先给出我参考的一些写的很好的相关资料：

- [levelDB源码视频+文档解析](https://hardcore.feishu.cn/mindnotes/bmncnzpUmXNQruVGOwRwisHyxoh)
- [levelDB- handbook](https://leveldb-handbook.readthedocs.io/zh/latest/basic.html)
- [C++多核编程与memory order](https://blog.csdn.net/wxj1992/article/details/103649056?spm=1001.2014.3001.5502)
- [C++ 中的 volatile，atomic 及 memory barrier](https://www.cnblogs.com/maji233/p/16072529.html)

本文并不是一篇完整的`levelDB`源码分析文章，因为我既没有那个能力，也没有那个时间，更多的是对自己在学习过程中的疑问的解答（如果我找到了答案的话）。

## skiplist


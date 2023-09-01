---
title:		Python Slice
subtitle:	Python Slice
summary:	Python Slice
date:		2020-05-23
lastmod:	2023-08-31
author:		shaopu
draft: 		false
image:		  
  focal_point: ''
  placement: 2
  preview_only: true

tags:
    - Python

categories:
    - Programming Language
---



# 对数组切片问题的讨论

这个模块的起源是这行代码：
```python
for scale in np.linspace(0.2, 1.0, 20)[::-1]:
```
1. range(a,b,c)和a : b : c的含义是一样的，其中c都是步长。
2. reverse和reversed的区别是有没有返回值，类似于sort和sorted的区别
3. 对于string类型来说，没有reverse()方法，只有reversed()，reversed()
的返回值是iterater object，说白了就是一个迭代类型，因此str(reversed(‘abc’))
仍然是reversed object，想要拿反转后的结果得用’’.join(reversed(‘abc’))，其实list也一样需要转化成list
4. arr[a:b] [c:d]相当于先按行[a,b)再按行[c,d).
5. a[:,0]是取二维数组中第一维的所有数据，相对应地，a[:,1]是取二维数组中第二维的所有数据，a[:,m:n]就是从m到n-1维的
更高维度数组同理
6. a[i:j:1]相当于a[i:j]。**当s<0时**，i缺省时，默认为-1. j缺省时，默认为-len(a)-1，从这个性质我们可以看出，翻转数组可以写做：
a[::-1]，或者a[-1:-1]也可以，如果是翻转局部数组，可以是a[x:y:-1]，必须要满足x>y(因为顺序本身设定的是负数)
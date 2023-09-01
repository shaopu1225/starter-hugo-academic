---
title:		Python zip function
subtitle:	Python zip function
summary:	Python zip function
date:		2020-05-22
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

# zip的用法

这个专题讨论zip函数在Python3中的使用方法，因为Python2与Python3对这个函数的用法产生了很大变化，需要Python2的可以自行上网查阅。

这专题开设的原因来自于下边这行代码：

```python
(cnts, boundingBoxes) = zip(*sorted(zip(cnts, boundingBoxes), key=lambda b: b[1][i], reverse=reverse))
```
zip在英文中有拉链的意思，我们经常用的压缩格式就是zip格式，所以这个函数从字面上就比较好理解功能，
就是起到了一个压缩的功能。zip()是Python的一个内置函数，他接受一系列可迭代的对象作为参数，并把它们
打包进多个tuple，组合起来返回一个可迭代对象，如果传入的对象不等长，那么他会自动按照最小长度的对象进行匹配并保留结果。利用*操作符，
我们可以把压缩包解压。
> 列表可以更改，为动态数组；元组仅可读，为静态数组，缓存与Python运行环境，无需访问内核分配内存

我们可以把这个函数的讨论分成两部分进行：一个列表的组合和N个列表的组合。

对于N个列表组合的情况，我们以2个列表组合为例查看，很容易理解zip函数到底是在做什么：
```python
a = [1,2,3]
b = [4,5,6]

ab = zip(a,b)
print(list(ab))
[(1, 4), (2, 5), (3, 6)]
print(ab)
<zip object at 0x000001FA6C521408>

ab_list = zip(*zip(a,b))
print(ab_list)
<zip object at 0x000001FA6C524088>
print(list(ab_list))
[(1, 2, 3), (4, 5, 6)]

ab_list_ptr = zip(*ab)
print(ab)
<zip object at 0x000001FA6C521408>
print(list(ab))
[]
```
对于1个列表内操作的情况：
```python
a1 = [[1, 2], [3, 4], [5, 6, 7]]
a2 = zip(a1)
print(list(a2))
[([1, 2],), ([3, 4],), ([5, 6, 7],)]
print(list(zip(*a1)))
[(1, 3, 5), (2, 4, 6)]
print(list(zip(*zip(a1))))
[([1, 2], [3, 4], [5, 6, 7])]
```

上边的几行代码虽然简短，但是可以看出很多东西：

1. 想要从zip封装后的结果读取元素，必须转化为list或者tuple等(这一点在Python2中是不需要的)
2. 通过调用zip(*)的方法，可以恢复原来的列表,也就是解压
3. 2中的方法我们可以理解是对指针操作(虽然Python并没有指针......)，所以通过代码最后的输出可以看到，如果我们仍然在原先的结果上操作，会直接导致输出为空，因为数据已经被改变了, 在实际使用时要注意这个问题
4. 对于一个列表的情况而言，我们可以理解成与之匹配的第二个列表是空列表，所以压缩后会出现([1,2],)这种tuple格式；同理
解压缩返回原格式；**但是如果我们直接对这个列表解压缩**，我们可以理解成当前的列表格式是进行了一次压缩之后的结果，那么自然地就应该返回[(1, 3, 5), (2, 4, 6)]
5. 这种使用方式本质上是一种高维矩阵行列变换，或者矩阵转置，我们也可以用如下方法来进行变换：（list的trick）
```python
a1 = [[row[col] for row in a] for col in range(len(a[0]))]
print(a1)
[[1, 4, 7], [2, 5, 8], [3, 6, 9]]
```
> 这种列表变换方法的末尾还可以加上if判断

但zip的方法要比上述这种list的操作方法更快，在Python3中，zip生成的是可迭代的对象，按需获取数据，
不需全部载入内存，提高速度，减少内存的使用率。

zip函数还可以很方便的生成字典：
```python
keys = [1,2,3]
values = ['a','b','c']
d = dict(zip(keys,values))
print(d)
{1: 'a', 2: 'b', 3: 'c'}
```